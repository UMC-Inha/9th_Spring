# 최적화 해보기 2

## 실제 운영 API의 성능 병목 분석과 개선

이번 글은 실제로 운영되는 API 하나(`/api/rental/all`)를 기준으로 성능 병목을 찾고 개선한 과정을 기록한 것이다. 단계는 단순하다.

1. SQL 로그를 본다.
2. 비효율을 찾는다.
3. QueryDSL로 필요한 데이터만 한 번에 뽑는다.
4. 트랜잭션을 읽기 전용으로 바꾼다.

그런데 흥미로운 점은, 대부분 "쿼리를 최적화하면 빨라지겠지"라고 생각하지만 실제로 가장 큰 체감 차이를 만든 것은 `@Transactional(readOnly = true)` 한 줄이었다. 왜 그런지까지 정리해보려고 한다.


## SQL 로그를 찍어보니 생각보다 단순하지 않았다

Spring JPA 설정에서 `spring.jpa.show-sql=true`와 `logging.level.org.hibernate.SQL=DEBUG`를 활성화하고 실제로 `/api/rental/all`을 호출했다.

로그를 보면 단순히 한 번의 조회로 끝날 것 같던 요청이 생각보다 복잡하게 흘러가고 있었다.

```
select u1_0.* from users u1_0 where u1_0.id=?;
select u1_0.* from users u1_0 where u1_0.id=?;

select p1_0.*
from place p1_0
join users o1_0 on o1_0.id=p1_0.owner_id
where o1_0.id=?;

select
    r1_0.id,
    u1_0.id,
    u1_0.name,
    r1_0.start_time,
    r1_0.end_time,
    r1_0.status
from rental r1_0
join users u1_0 on u1_0.id=r1_0.user_id
join place p1_0 on p1_0.id=r1_0.place_id
where p1_0.id=?
order by r1_0.start_time desc, r1_0.id desc;

```

문제는 세 가지였다.

> **첫째**, 같은 사용자를 같은 요청 내에서 여러 번 조회하고 있었다.

`userRepository.findById(userId)`가 인증 단계와 서비스 단계 양쪽에서 중복 호출되며 `users where id=?` 쿼리가 반복됐다.

> **둘째**, place를 찾는 쿼리에서 불필요하게 `users` 테이블을 조인하고 있었다.

사실 `place.owner_id`만으로도 충분한데, 실제 SQL은 `join users ... where owner.id = ?` 형태였다.

> **셋째**, rental(대여 이력) 조회 쿼리도 불필요하게 place까지 조인하고 있었다.

이미 placeId를 알고 있는데 굳이 `join place`를 추가하고, `user` 전체를 조인해 가져오면서 과도한 데이터를 끌어오고 있었다.

즉, 한마디로 정리하면 **같은 데이터 중복 조회**, **필요 없는 조인**, **전체 엔티티 오버페치**의 삼중 콤보였다. 이런 구조는 데이터가 조금만 늘어나도 바로 병목이 된다.


## QueryDSL로 rental 조회를 재구성

핵심 원인은 rental 쿼리였다. 그래서 이 부분을 QueryDSL로 직접 재작성했다.

핵심은 **필요한 데이터만 한 번에 DTO로 뽑아오는 것**이었다.

```sql
public List<PlaceRentalHistoryResponse> findAllHistoryByPlace(Long placeId) {
        QRental r = QRental.rental;
        QUser u = QUser.user;

        return queryFactory
                .select(Projections.constructor(
                        PlaceRentalHistoryResponse.class,
                        r.id,         // rentalId          Long
                        u.id,         // userId            Long
                        u.name,       // userName          String
                        r.startTime,  // startTime         LocalDateTime
                        r.endTime,    // endTime           LocalDateTime
                        r.status      // status            RentalStatus (enum)
                ))
                .from(r)
                .join(r.user, u)
                .where(r.place.id.eq(placeId))
                .orderBy(r.startTime.desc(), r.id.desc())
                .fetch();
    }
```

QueryDSL을 도입하자 쿼리가 훨씬 단순해졌다. `rental`을 기준으로 `user`만 조인하고, `place`는 `where` 조건으로만 필터링하면서 쿼리 플랜이 단순해지고 실행 속도도 눈에 띄게 빨라졌다. 또한 `Projections.constructor()`를 사용해 필요한 필드만 직접 DTO에 매핑함으로써, 엔티티 전체를 메모리에 올리지 않고도 응답에 필요한 데이터만 효율적으로 조회할 수 있었다. 그 결과 Lazy 로딩이 완전히 제거되어 컨트롤러에서 직렬화할 때 추가 쿼리가 발생하지 않았고, 성능이 안정적으로 유지되었다. 특히 QueryDSL은 컴파일 타임에 DTO 생성자와 타입을 검증하기 때문에 JPQL의 `"select new ..."` 방식처럼 런타임 오류가 발생할 위험도 없다. 이러한 구조 덕분에 코드의 유지보수성과 안정성이 크게 향상되었다.


## Place 조회 쿼리에서 조인 제거

다음으로 `place` 조회 쿼리를 확인했다. 단순히 `findByOwnerId(Long ownerId)` 메서드를 사용했을 뿐인데, Hibernate가 내부적으로 `join users`를 추가하고 있었다. 원인은 `@ManyToOne(fetch = LAZY)` 매핑이었다. Spring Data JPA는 메서드 이름을 해석하는 과정에서 `owner.id`를 따라가며 자동으로 `join p.owner o`를 생성하기 때문이다. 이를 방지하기 위해 `@Query`를 명시적으로 작성해 불필요한 조인이 수행되지 않도록 수정했다.

```java
@Query("select p from Place p where p.owner.id = :ownerId")
Optional<Place> findByOwnerId(@Param("ownerId") Long ownerId);
```

이렇게 바꾸자 실제 SQL이 깔끔하게 변했다.

```
select p1_0.*
from place p1_0
where p1_0.owner_id=?
```

조인이 완전히 제거되었고, place 테이블만 읽게 되었다.

이는 인덱스 효율에도 직접적인 이점을 주었다.


## 의외로 가장 큰 차이를 만든 한 줄

쿼리 최적화를 통해 쿼리 수는 줄었지만, 체감 속도는 여전히 다소 느렸다. 

그런데 `@Transactional(readOnly = true)`를 적용하자 응답 속도가 급격히 개선되었다.

이 설정은 Hibernate에 “이 트랜잭션에서는 데이터를 수정하지 않는다”는 힌트를 주는 역할을 하며, 그 한 줄로 인해 내부적으로 여러 비용이 큰 동작들이 생략된다.

우선 **Dirty Checking**이 비활성화되어, 엔티티의 변경 여부를 판단하기 위해 스냅샷을 따로 만드는 과정이 사라진다. 또한 **Flush 검증**이 생략되어 커밋 시점에 변경된 엔티티가 있는지 검사하지 않으며, **영속성 컨텍스트** 역시 읽기 전용 모드로 작동해 관리 오버헤드가 크게 줄어든다.

이 API는 “대여 이력 전체 조회”처럼 데이터를 대량으로 읽기만 하는 전형적인 요청이었기 때문에 `readOnly` 모드가 이상적으로 적합했다. 실제로 QueryDSL로 쿼리를 단순화한 것보다 `@Transactional(readOnly = true)` 한 줄의 효과가 훨씬 컸다. 적용 후 응답 시간이 약 30~40ms**대에서 20ms 수준**으로 감소했다!


## 최종 로그 결과

최종적으로 `/api/rental/all` 실행 시 SQL 로그는 단 2개만 찍힌다.

```
select * from place where owner_id=?;
select r1_0.id, u1_0.id, u1_0.name, r1_0.start_time, r1_0.end_time, r1_0.status
from rental r1_0
join users u1_0 on u1_0.id=r1_0.user_id
where r1_0.place_id=?
order by r1_0.start_time desc, r1_0.id desc;

```

- user 중복 조회 제거
- place 조인 제거
- rental 조회 시 필요한 user만 join
- Lazy 로딩 완전 제거

모든 조회가 “2쿼리-1트랜잭션”으로 수렴했다.


## 진짜 병목은 DB가 아니라 JPA였다

이번 개선의 핵심은 의외로 단순했다. 복잡한 캐시나 인덱싱을 적용하는 대신, JPA의 기본 동작을 정확히 이해하고 불필요한 기능을 꺼주는 것만으로도 충분한 효과를 얻을 수 있었다. 불필요한 조인을 제거하자 SQL이 단순해지고 실행 계획이 깔끔해졌으며, DTO Projection을 활용해 N+1 문제를 방지하면서 필요한 데이터만 효율적으로 조회할 수 있었다. 여기에 `@Transactional(readOnly = true)`를 적용하니 Hibernate 내부의 불필요한 검사와 관리 로직이 생략되어 전반적인 성능이 획기적으로 개선되었다. 이처럼 단순한 설정 변경만으로도 쿼리 튜닝만으로는 얻기 힘든 체감 속도 향상이 가능했다. 결국 조회 전용 API의 성능을 높이는 비결은 복잡한 코드를 추가하는 것이 아니라, 시스템이 불필요하게 수행하던 일을 덜어내는 데 있었다.