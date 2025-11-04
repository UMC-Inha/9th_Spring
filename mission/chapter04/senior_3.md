# **동시성 문제가 발생할 수 있는 시나리오**를 고민하고 해결책 적용

지금 구조에서는 동시성 문제가 명확히 드러나지 않지만, 예를 들어 미션의 개수가 한정되어 있는 상황을 생각해보면 문제가 분명하게 보인다. 사용자는 가게에서 등록한 미션을 위치 기반으로 조회한 뒤 수락할 수 있는데, 이때 가게 측에서 수락 가능한 인원 수에 제한을 걸었다고 하자. 그런데 마지막 한 자리를 두고 두 명의 사용자가 거의 동시에 미션을 수락 요청을 보내면, 동시성 제어가 제대로 이루어지지 않는다면 두 요청이 모두 성공해버려서 실제로 허용된 최대 인원을 초과하는 상황이 발생할 수 있다. 

`acceptMission()` 메서드는 사용자가 미션을 수락할 때 발생할 수 있는 동시성 문제를 해결하기 위해 비관적 락(Pessimistic Lock)을 적용한 함수이다.

```java
@Transactional
public void acceptMission(Long missionId, Long userId) {
    
    Mission mission = missionRepository.findByIdForUpdate(missionId)
        .orElseThrow(() -> new IllegalArgumentException("미션이 존재하지 않습니다."));

    if (mission.getRemainingSlots() <= 0) {
        throw new IllegalStateException("이미 정원이 꽉 찼습니다.");
    }

    missionRepository.decreaseSlot(missionId);
}
```

먼저 `findByIdForUpdate()`를 통해 미션 엔티티를 조회할 때, `@Lock(LockModeType.PESSIMISTIC_WRITE)`를 사용하여 해당 행(row)에 **쓰기 락**을 건다. 이렇게 되면 이 트랜잭션이 종료될 때까지 다른 트랜잭션은 해당 미션 데이터를 수정하거나 삭제할 수 없으며, 대기 상태에 들어가게 된다. 즉, 동시에 여러 사용자가 같은 미션을 수락하려고 해도 한 명만 먼저 락을 점유할 수 있고, 그동안 다른 요청은 순차적으로 처리된다.

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
@Query("SELECT m FROM Mission m WHERE m.id = :missionId")
Optional<Mission> findByIdForUpdate(@Param("missionId") Long missionId);
```

그 다음, 조회된 미션의 남은 인원 수를 확인하여 0 이하일 경우 정원이 꽉 찼다는 예외를 발생시킨다. 만약 남은 자리가 있다면, `decreaseSlot()` 메서드를 호출해 남은 인원 수를 1 감소시킨다. 

이때 `@Modifying(clearAutomatically = true, flushAutomatically = true)`를 사용하여 벌크 연산 전후로 영속성 컨텍스트를 동기화함으로써 DB 상태와 캐시 상태가 일치하도록 한다.

```java
@Modifying(clearAutomatically = true, flushAutomatically = true)
@Query("""
       UPDATE Mission m
          SET m.remainingSlots = m.remainingSlots - 1
        WHERE m.id = :missionId
          AND m.remainingSlots > 0
       """)
int decreaseSlot(@Param("missionId") Long missionId);
```

이 과정을 통해 **조회 시점부터 트랜잭션이 종료될 때까지 락이 유지되므로**, 동시에 여러 사용자가 같은 미션을 수락하더라도 오직 한 트랜잭션만이 성공적으로 슬롯을 차감할 수 있다. 결과적으로, 미션 정원을 초과하는 상황을 방지하고 데이터의 일관성을 보장할 수 있다.