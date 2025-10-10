
# 하나의 트랜잭션에서 여러 엔티티를 처리하는 비즈니스 로직 작성하기

`@Modifying`은 **Spring Data JPA에서 데이터 변경 쿼리(UPDATE, DELETE, INSERT 등)를 실행할 때 사용하는 어노테이션**이다.

기본적으로 `@Query`는 **조회용(SELECT)** 으로만 동작하기 때문에, 만약 `update`, `delete`, `insert` 같은 쿼리를 작성하면 **에러가 발생한다.** 이럴 때 `@Modifying`을 함께 붙여줘야 "이 쿼리는 데이터를 변경하는 DML 쿼리다"라고 Spring Data JPA에게 알려주는 것이다.

`@Modifying`중요사항은 **JPA가 쿼리 결과를 엔티티로 매핑하지 않고, 데이터베이스에 직접 반영하도록 만드는 어노테이션**이라는 것이다. 이게 무슨 말이냐면 jpa 는 **영속성 컨텍스트(1차 캐시)** 를 사용하기 때문에 변경 내용이 모았다가 한번에 간다. 하지만 modifying을 사용하면 엔티티 객체가 메모리에 남아 있어도 DB에서 바로 DELETE / UPDATE가 수행되기 때문에 **DB와 영속성 컨텍스트 간 불일치(inconsistency)** 가 생길 수 있다.

이 문제를 해결하기 위해

`clearAutomatically`와 `flushAutomatically` 속성을 함께 사용한다. 

| 속성 | 설명 |
| --- | --- |
| `flushAutomatically = true` | 벌크 연산 전에 영속성 컨텍스트의 변경 내용을 DB에 반영(flush) |
| `clearAutomatically = true` | 벌크 연산 후 영속성 컨텍스트를 초기화(clear)하여 캐시된 엔티티 제거 |

`clearAutomatically = true`는 JPA가 영속성 컨텍스트(1차 캐시)에 엔티티를 저장해두고 이를 기반으로 동작하기 때문에 필요하다. 만약 기존에 조회하던 데이터가 `@Modifying`을 통해 영속성 컨텍스트를 거치지 않고 DB에서 바로 삭제되면, 영속성 컨텍스트 안에 남아 있는 객체는 실제로는 이미 DB에 존재하지 않는, 오래된 데이터가 된다. 이런 상태에서 누군가 해당 객체를 참조하면 데이터 불일치나 예외가 발생할 수 있기 때문에, 벌크 연산 후에는 영속성 컨텍스트를 비워(DB 상태와 동기화시키기 위해) `clearAutomatically = true`를 설정하는 것이다. 

반면 `flushAutomatically = true`는 벌크 연산이 영속성 컨텍스트를 거치지 않고 바로 DB에 반영된다는 특성 때문에 필요하다. 만약 아직 flush되지 않은 변경사항이 영속성 컨텍스트 안에 남아 있다면, 이 변경이 반영되기 전에 벌크 연산이 먼저 실행되어 DB와 캐시 간의 데이터 불일치가 생길 수 있다. 따라서 `flushAutomatically = true`를 설정해 영속성 컨텍스트의 변경 내용을 먼저 DB에 반영한 다음, 그 이후에 `@Modifying` 쿼리를 실행하도록 보장하는 것이다.

---

### `Member`가 탈퇴할 경우 **관련된 모든 데이터를 삭제하는 API** 구현하기

기본적으로 `Member`와 직접적으로 연관되어 있는 엔티티는 다음과 같다. 

- 사용자가 남긴 **리뷰와 리뷰 사진**
- 사용자가 수행한 **사용자 미션**
- 사용자의 **선호 음식**
- 사용자의 **이용 약관 동의 내역**
- 사용자가 남긴 **문의 사항**

이 중 **양방향 연관 관계**를 가지며 `orphanRemoval = true`와 `cascade` 옵션을 적용한 것은 **사용자 미션(MemberMission)** 과 **선호 음식(MemberFood)** 이다. 그래서 `Member`가 삭제되면 JPA가 자동으로 두 엔티티를 함께 삭제해주지만, 사용자 미션의 경우 데이터의 양이 많아질 수 있어 자동 삭제 대신 **@Modifying 쿼리를 통해 벌크 삭제**하도록 처리했다. 반면 선호 음식은 데이터량이 적어 `Member` 삭제 시 `orphanRemoval`을 통해 자동 삭제되도록 그대로 두었다.

그 외의 엔티티(리뷰, 리뷰 사진, 이용 약관, 문의 사항)는 직접 삭제 쿼리를 수행하도록 했으며, Repository 계층에서 다음과 같은 형태로 정의했다.

```java
@Modifying(clearAutomatically = true, flushAutomatically = true)
@Query("delete from MemberPolicy mp where [mp.member.id](http://mp.member.id/) = :memberId")
int deleteAllByMemberId(@Param("memberId") Long memberId);
```

이처럼 각 엔티티별로 `@Modifying`을 적용하여 벌크 삭제 쿼리를 정의하고, 서비스 계층에서 이들을 **순차적으로 호출**하도록 구성했다.

```java
@Transactional
public void deleteMember(Long memberId, boolean hard) {
    if (hard) {
        hardDeleteMember(memberId);
    } else {
        softDeleteMember(memberId);
    }
}
```

`deleteMember()` 메서드에서는 쿼리 파라미터로 `hard` 여부를 받아 하드 삭제 또는 소프트 삭제를 분기하도록 했다. 동시에 삭제 과정 전체를 하나의 트랜잭션으로 묶기 위해 `@Transactional`을 적용하여, 도중에 예외가 발생하더라도 **원자성**을 보장하도록 했다.

```java
public void hardDeleteMember(Long memberId) {

    int deletedPhotos = reviewPhotoRepository.deleteAllByMemberId(memberId);
    int deletedReviews = reviewRepository.deleteAllByMemberId(memberId);
    int deletedMissions = memberMissionRepository.deleteAllByMemberId(memberId);
    int deletedPolicies = memberPolicyRepository.deleteAllByMemberId(memberId);
    int deletedInquiries = inquiryRepository.deleteAllByMemberId(memberId);

    int deletedUsers = memberRepository.hardDeleteById(memberId);

    int total = deletedPhotos + deletedReviews + deletedMissions + deletedPolicies + deletedInquiries + deletedUsers;

    if (total == 0) {
        log.info("Member {} delete is idempotent: nothing to delete.", memberId);
    } else {
        log.info("Member {} hard-deleted: photos={}, reviews={}, missions={}, policies={}, inquiries={}, users={}",
                 memberId, deletedPhotos, deletedReviews, deletedMissions, deletedPolicies, deletedInquiries, deletedUsers);
    }
}

```

하드 삭제 로직은 먼저 `Member`와 연관된 엔티티들을 앞서 정의한 `@Modifying` 쿼리를 통해 한꺼번에 삭제한 뒤, 마지막에 `Member` 자체를 삭제하도록 했다. 이때 `@Transactional`을 따로 메서드에 붙이지 않은 이유는 이미 상위 메서드인 `deleteMember()`에 적용되어 있기 때문이다. 내부 메서드에 중복 적용해도 트랜잭션이 새로 만들어지지 않으므로, 한 번만 걸어주면 된다.

마지막으로 **소프트 삭제(soft delete)** 는 단순히 회원의 상태를 `Withdrawn`로 변경하는 형태로 구현했다.

```java
public void softDeleteMember(Long memberId) {
    Member member = memberRepository.findById(memberId)
            .orElseThrow(() -> new IllegalArgumentException("해당 회원이 존재하지 않습니다."));
    member.softDelete();
}
```

즉, 실제 데이터를 삭제하지 않고 상태 플래그만 변경하여 논리적으로 탈퇴 처리하는 방식이다.

정리하면, 데이터량이 많거나 연관이 깊은 엔티티는 `@Modifying` 으로 벌크 삭제를 수행하고, 가벼운 데이터는 `orphanRemoval`을 이용해 자동 삭제하도록 구분했으며, 전체 삭제 과정은 트랜잭션으로 묶어 안정성을 확보했다.