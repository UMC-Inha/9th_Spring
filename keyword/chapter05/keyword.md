### 즉시 로딩 (Eager) VS 지연 로딩 (Lazy)

즉시 로딩은 엔티티를 조회할 때 연관된 엔티티도 즉시 함께 조회하지만, 지연 로딩은 연관된 엔티티는 실제 사용할 때 조회한다.

User와 UserMission이 1대 다의 관계를 맺고있다고 해보자.

지연 로딩이라면 DB에서 User에 관한 조회를 할 때 아래 쿼리만 실행될 것이다.

```java
 SELECT * FROM user WHERE id = 1; 
```

아래처럼 UserMission를 사용하려는 순간 그제서야 user_mission에 관한 쿼리가 나간다.

```java
  List<UserMission> missions = user.getMissions(); //<- 사용하려는 순간에야
  //SELECT * FROM user_mission WHERE user_Id =1' <-- 이 쿼리가 나갈 것이다. 
```

필요한 데이터만 조회하기에 성능을 높일 수 있다.

주의할 점은 트랜잭션 범위 밖에서 접근하면 오류가 발생하는 점, DB가 아닌 프록시에서 데이터를 가져온다는 점이다.

즉시 로딩이라면 user에 접근한 순간 아래처럼 연관된 정보를 한번에 가져오는 쿼리가 나갈 것이다.

```java
SELECT u.*, m.* FROM user u LEFT JOIN user_mission m ON u.id = m.user_id WHERE u.id = 1;
```

EAGER 방법은 객체를 바로 사용 가능해 편리하다는 점이 있지만, 잘못 사용하면 N+1 문제를 겪을 수 있기에 필요한 경우에서만 사용해야한다. (fetch join 사용하면 N+1문제 해결가능)

### JPQL(Java Persistence Query Language)

JPA에서 제공하는 객체지향 쿼리 언어로, SQL과 문법이 비슷하지만 DB 테이블이 아닌 엔티티 객체를 대상으로 쿼리를 작성한다.

즉, **SQL → 테이블 중심, JPQL → 객체(Entity) 중심**이라고 보면 된다.

JPQL 사용 방법엔 두가지가 있다.

1. **EntityManager 인터페이스 활용**

    ```java
    EntityManager em = entityManagerFactory.createEntityManager();
    em.getTransaction().begin();
    
    List<Member> members = em.createQuery(
        "SELECT m FROM Member m WHERE m.age > 20", Member.class)
        .getResultList();
    
    em.getTransaction().commit();
    ```

    - createQuery() 안에 JPQL 작성
    - 엔티티를 대상으로 조회 → 결과는 영속성 컨텍스트와 연계되어 1차 캐시에 저장됨
2. **Repository 인터페이스 활용 (Spring Data JPA)**

    ```java
     public interface MemberRepository extends JpaRepository<Member, Long> {
        @Query("SELECT m FROM Member m WHERE m.age > :age")
        List<Member> findOlderThan(@Param("age") int age);
    }
    ```

    - @Query 어노테이션으로 JPQL 작성 가능
    - EntityManager 직접 사용하지 않아도 트랜잭션 관리, 변경 감지 등 JPA 기능 그대로 사용 가능

JPQL을 정리하자면,

- 대상: 엔티티 객체
- 장점: DB 독립적, 객체 지향적 쿼리 가능, 연관관계 탐색 가능
- 사용 방법: EntityManager, Repository 인터페이스
- 주의점: INSERT는 JPQL로 불가, Lazy 컬렉션 접근 시 트랜잭션 유지 필요

**INSERT는 JPQL로 불가?**

→ JPQL은 객체를 조회, 수정, 삭제하는데 최적화된 언어이다. JPA는 새 엔티티를 저장할 때 영속성 컨텍스트에 먼저 등록하고, flush/commit 시점에 DB에 INSERT SQL을 생성한다. → JPQL은 객체 상태를 질의하거나 변경하는 목적이라 객체 생성과 등록 과정을 직접 다루지 않는다.

→ JPQL은 조회와 기존 객체 변경용, INSERT는 영속성 컨텍스트를 통해 처리해야 함.

**Lazy 컬렉션 접근 시 트랜잭션 유지 필요?**

```java
em.getTransaction().begin();

User user = em.find(User.class, 1L); // Lazy 로딩인 missions는 아직 안 불러옴
em.getTransaction().commit(); // 트랜잭션 종료

List<UserMission> missions = user.getMissions(); 
// LazyInitializationException 발생!
// 트랜잭션이 끝난 상태라 DB 연결이 끊겨 SELECT 불가`
```

### Fetch Join

JPQL에서 연관된 엔티티를 한 번의 쿼리로 함께 조회하는 방법이다.

- 일반 Join은 연관 Entity에 Join을 걸어도 실제 쿼리에서 SELECT하는 Entity. 즉 오직 JPQL에서 조회하는 주체가 되는 Entity만 조회하여 영속화한다.
- Fetch Join은 조회의 주체가 되는 Entity 이외에 Fetch Join이 걸린 연관 Entity도 함께 Select 하여 모두 영속화한다.
    - Fetch Join이 걸린 Entity 모두 영속화하기 때문에 FetchType이 Lazy인 Entity를 참조하더라도 이미 영속성 컨텍스트에 들어있기 때문에 따로 쿼리가 실행되지 않은 채로 N+1 문제가 해결된다.

**일반 Join**

```java
SELECT u FROM User u JOIN u.missions m //영속성 컨텍스트엔 User만 있어서. 
```

- SQL 변환: JOIN은 수행하지만 결과는 User 엔티티만 반환
- missions 컬렉션은 여전히 Lazy → 접근 시마다 DB SELECT 발생

**Fetch Join**

```java
SELECT u FROM User u JOIN FETCH u.missions// 영속성 컨텍스트에 Mission, User. -> 더이상의 쿼리 X 
```

- SQL 변환:

```java
SELECT u.*, m.*
FROM user u
LEFT JOIN user_mission m ON u.id = m.user_id
```

- JPQL 결과: User + missions 컬렉션 모두 조회
- Lazy 컬렉션을 미리 DB에서 채움 → N+1 문제 예방


### @EntityGraph

연관관계가 있는 엔티티를 조회할 경우 Lazy loading으로 설정되어 있으면 연관관계에서 종속된 엔티티는 쿼리 실행 시 select 되지 않고 proxy 객체를 만들어 엔티티가 적용시킨다. 그 후 해당 proxy 객체를 호출할 때마다 그때그때 select 쿼리가 실행된다.

위 같은 연관관계가 지연 로딩으로 되어있을 경우 fetch join을 사용하여 여러 번의 쿼리를 한번에 해결할 수 있다.

@EntityGraph는 Data JPA에서 fetch join을 어노테이션으로 사용할 수 있도록 만들어준다.

```java
@EntityGraph(attributePaths = {"missions", "roles"})
List<User> findAll();
```

- mission, roles 컬렉션이 Lazy 로딩이어도 한 번의 쿼리로 미리 가져오기
- 내부적으로 JPQL의 JOIN FETCH와 동일한 효과

**이해 정리**

- Lazy 컬렉션: 사람을 만나기 전까지는 비어 있는 택배 상자
- Fetch Join: 한 번에 택배 상자를 열어서 모든 내용물 확인
- @EntityGraph: 택배 상자를 열 필요 없게, 요청할 때 자동으로 다 꺼내주는 친구

### commit VS flush

**flush()란?**

flush()는 영속성 컨텍스트의 변경사항을 즉시 데이터베이스에 반영하는 역할을 한다. 즉, 데이터베이스와 영속성 컨텍스트 사이의 스냅샷을 일치시키는 작업이다.

→ 다만 트랜잭션을 커밋하지는 않는다. flush()가 호출되면 변경된 엔티티가 SQL로 변환되어 실행되지만, 트랜잭션이 끝난 것은 아니다.

언제 사용? → 변경 사항을 즉시 데이터베이스에 반영해야 하지만, 트랜잭션은 유지해야 할 때 사용(대량 데이터 처리)

**commit()이란?**

commit()은 현재 트랜잭션을 완료하고 모든 변경 사항을 확정하는 역할을 한다. 내부적으로 flush()를 수행한 후, 실제로 트랜잭션을 커밋한다. commit()이 실행되면 변경 사항이 영구적으로 저장되며 ROLLBACK할 수 없다.

언제 사용? → 데이터베이스 변경 사항을 최종적으로 확정할 때 사용

             → 하나의 작업 단위를 끝내고 안정적인 저장이 필요할 때 사용

### QueryDSL

자바 코드 기반의 타입 안전 쿼리 빌더 라이브러리

- 쿼리를 자바 코드로 작성 → SQL/JPQL 문자열 작성 필요 없음
- 컴파일 시점에서 오류 확인 가능 → 안전함
- 동적 쿼리 작성 편리 → 조건을 필요할 때만 추가 가능
- 메서드 체인으로 복잡한 쿼리도 가독성 좋게 작성

**예시**

```java
QMember member = QMember.member;

List<Member> result = queryFactory
    .selectFrom(member)
    .where(member.age.gt(20).and(member.status.eq("ACTIVE")))
    .orderBy(member.name.asc())
    .fetch();
```

- QMember는 QueryDSL이 자동 생성하는 클래스
- .where(), .orderBy()를 메서드 체인으로 연결해 조건/정렬 작성

### OpenFeign의 QueryDSL

OpenFeign: Spring Cloud에서 제공하는 REST API 호출용 클라이언트

QueryDSL과 연관성: OpenFeign 자체는 쿼리 빌더가 아니지만, QueryDSL로 만든 동적 조건을 REST 호출 파라미터로 변환해서 보낼 수 있음

**사용 예시**

```java
// QueryDSL로 조건 생성
BooleanExpression condition = QMember.member.age.gt(20)
                              .and(QMember.member.status.eq("ACTIVE"));

// Map으로 변환
Map<String, Object> queryParams = QuerydslUtil.toMap(condition);

// OpenFeign 인터페이스
@FeignClient(name = "memberClient", url = "http://api.example.com")
public interface MemberClient {
    @GetMapping("/members")
    List<MemberResponse> getMembers(@RequestParam Map<String, Object> params);
}

// 호출
List<MemberResponse> members = memberClient.getMembers(queryParams);
```

- 장점: 서버나 다른 서비스로 **동적 조회 조건을 안전하게 전달 가능**
- 요약: **QueryDSL은 조건 생성 → OpenFeign은 REST 요청 전송**

### N+1 문제 해결할 수 있는 여러 방안들

예를 들어, Member와 Order가 1:N 관계라고 해보자.

첫 번째 쿼리가 select * from member일 때 각 멤버마다 select * from orders where member_id = ? 이런 쿼리가 발생할 것이다.

→ 즉, 한 번 조회했는데 추가로 N번의 쿼리가 발생한것이다.

1. **Fetch Join**

   쿼리 한 번으로 연관된 엔티티를 전부 가져온다.

    ```java
    @Query("select m from Member m join fetch m.orders")
    List<Member> findAllWithOrders();
    ```

    - 장점: 성능 좋고 간단함
    - 단점: 1:N 관계에서는 페이징 불가능, 중복 가능성 있음
2. @EntityGraph

   fetch join을 어노테이션 기반으로 표현하는 방식(Spring Data JPA 문법)

    ```java
    @EntityGraph(attributePaths = {"orders"})
    @Query("select m from Member m")
    List<Member> findAllWithOrders();
    ```

    - 장점: 코드가 깔끔하고 재사용 쉬움
    - 단점: 복잡한 조인 쿼리에는 한계 있음
3. @BatchSize

   Hibernate 전용 설정으로, 지연 로딩 시 여러 엔티티를 한 번에 로드

    ```java
    @Entity
    @BatchSize(size = 100)
    public class Member {
        @OneToMany(mappedBy = "member")
        private List<Order> orders;
    }
    ```

    - 장점: 설정만 추가하면 N+1이 완화됨
    - 단점: 완전한 한 번의 쿼리는 아님 (배치 단위로 묶어서 여러 번 실행됨)

### 영속 상태의 종류

- **비영속**: 영속성 컨텍스트와 전혀 관계없는 새로운 객체 → new로 생성만 한 상태, DB와 전혀 연동 X
- **영속**: EntityManager가 관리중인 상태 - 영속성 컨텍스트에 등록됨 → em.persist(entity) 호출 시 진입, 1차 캐시에 저장됨
- **준영속**: 한때 영속 상태였지만 지금은 관리되지 않는 상태 → em.detach(entity), em.clear(), em.close() 등으로 빠짐
- **삭제**: 삭제 명령이 예약된 상태 → em.remove(entity)호출 시 DB에서 삭제될 예정

**예시**

```java
//비영속 
User user = new User("해원"); // 단순 객체 생성

//영속 
em.persist(user); // 영속성 컨텍스트에 등록됨 (DB엔 아직 X)

//준영속 
em.detach(user); // 이제 JPA가 더 이상 user를 추적하지 않음

//삭제 
em.remove(user); // delete 예약됨 (flush 시 DB에서 삭제)

```