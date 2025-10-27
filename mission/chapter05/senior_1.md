# 최적화 해보기 1
`/api/rental/enter` API에서는 한 장소(`place`)의 정원을 초과해 동시에 입장할 수 없도록, 기존에 `@Lock(PESSIMISTIC_WRITE)` 비관적 락을 걸어 처리하고 있었다. 이 방식은 정원 초과를 방지한다는 점에서는 올바르지만, 락이 너무 이른 시점에 걸리고 너무 오래 유지된다는 문제가 있었다.

즉, 영업 시간 확인·정원 체크·유저 중복 이용 확인 등 **실패할 수도 있는 검증 로직조차 모두 락 안에서 수행**되어, 동시에 여러 요청이 들어오면 뒤 요청이 DB 락에서 대기하게 되는 병목이 발생할 수 있을 것 같았다.

## 검증 쿼리 슬림화 (QueryDSL 도입)


sql을 보다 보니 기존의 검증 로직은 `rental` ↔ `place`, `rental` ↔ `users`를 조인하며 불필요하게 무거운 쿼리를 실행하고 있었다. 이를 `RentalQueryRepository`로 분리하고, `QueryDSL` 기반 단일 테이블 쿼리로 단순화했다.

```sql
select count(r.id)
from rental r
where r.place_id = ? and r.status = ?
```

```sql
select r.id
from rental r
where r.user_id = ? and r.status = ?
limit 1
```

조인이 사라지고 `(place_id, status)` 및 `(user_id, status)` 인덱스로만 처리되며, SQL 로그에서도 실제로 단일 테이블 조회로 최적화된 것이 확인됐다.

이를 통해 **검증 쿼리 비용이 크게 줄고 인덱싱 전략도 단순화**됐다.


## 검증 단계와 삽입 단계 분리

핵심 개선은 락의 범위를 최소화하는 구조적 리팩토링이었다.

### **기존 구조**

모든 검증 로직과 INSERT가 하나의 트랜잭션(`@Lock(PESSIMISTIC_WRITE)`) 안에서 처리됐다.

즉, “락 → 모든 검증 → INSERT” 흐름이었고, 불필요하게 락을 오래 쥐고 있었다.

---
### **개선 구조 (2단계 분리)**

1️⃣ **사전 검증 단계 (`@Transactional(readOnly = true)`)**

- 락 없이 영업 중 여부, 현재 인원 수, 유저 중복 이용 여부 등을 확인.
- 실패할 요청은 여기서 즉시 컷.
- readOnly 트랜잭션이라 Hibernate flush 부담도 없음.

```sql
@Transactional(readOnly = true)
public void precheck(Long userId, String placeCode) {
    // place 그냥 조회 (락 없음)
    // 지금 영업 중인지 확인
    // 현재 인원(usingCount)와 maxCapacity 비교
    // 유저가 이미 USING 중인지 확인
    // 안 되면 여기서 바로 예외 던지고 끝
}
```

2️⃣ **최종 확정 단계 (`@Transactional`)**

- 오직 진입 가능한 요청만 도달.
- 이때만 `findByCodeForUpdate()`로 place row에 락을 걸고, 인원 수를 한 번 더 확인 후 INSERT 실행.
- 락을 잡는 구간이 “정원 재확인 → INSERT”로 한정되어 매우 짧음.

```java
@Transactional
public UserEnterResponse startRental(Long userId, String placeCode) {
    // 여기서만 placeRepository.findByCodeForUpdate(placeCode)
    // 즉 여기서만 비관락
    // 인원 수를 다시 한 번만 재확인 (경쟁 상황 대비)
    // Rental INSERT
}
```

이렇게 **락 보유 시간을 최소화하고, 락을 시도하는 요청 수도 크게 줄였다.**

비록 place 조회가 두 번 발생하지만, 이는 “락을 늦게 잡아 tail latency를 줄인다”는 의도적인 설계 선택이다.


## 측정 결과

락이 걸리는 시간이 너무 짧아 동시성 테스트가 어렵다는 점을 고려해, **일부러 락이 약 1초간 유지되도록 임의로 설정**했다. 

이후 **동일한 `placeCode`로 두 개의 요청을 동시에 보낸 멀티스레드 테스트**를 진행한 결과는 다음과 같다.

```
thread1 duration = 2683ms
thread2 duration = 1629ms
```

이 결과는 **비관적 락이 트랜잭션의 직렬화를 강제하고**, 선점된 락의 **보유 시간이 곧 다른 요청의 대기 시간으로 직결됨**을 명확히 보여준다. 즉, 두 스레드 중 하나가 락을 획득하는 동안 다른 하나는 대기 상태에 머무르며, 이로 인해 전체 응답 시간이 누적되는 형태로 나타났다.

따라서, **락을 가능한 한 짧은 시간 동안만 유지하는 구조로 설계하는 것이 필수적**임을 수치적으로 입증한 실험 결과라 할 수 있다.

이번 최적화는 비관적 락의 범위를 축소하고 트랜잭션 구조를 개선함으로써 **동시성 안전성과 코드의 명확성**을 확보했다. 다만, 실제 운영 환경에서는 **동시에 동일한 place를 대여하려는 요청이 많지 않기 때문에**, 체감 성능 향상이나 실질적인 쿼리 성능 개선 효과는 **크게 두드러지지 않을 것으로 보인다.**

그럼에도 불구하고, 향후 트래픽이 증가하거나 특정 장소에 요청이 집중되는 상황을 대비해 **안정성을 강화할 수 있는 개선 방안을 검토했다는 점에서 의미가 있었다**고 할 수 있다.