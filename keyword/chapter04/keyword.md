# 계층형 구조 vs 도메인형 구조

## 1. 계층형 구조

### [개념]

기능이 아니라 역할을 기준으로 코드를 나누는 구조

### [예시]

com.example.project

┣ controller/

┣ service/

┣ repository/

┗ domain/ (또는 entity/)

MemberRepository, MissionRepository등 XXXXXRepository에 대해 Repository 패키지에 넣음
Controller, Service, Domain 모두 동일

### [장점]

- 명확한 역할 분리 : Controller, Service, Repository는 각각 역할을 지니고 이 역할에 따라 분류했기 때문에 역할을 명확히 분리 할 수 있음
- 재사용성 : 같은 Service가 여러 Controller에서 재사용되기 쉬움
- Controller -> Service -> Repository 의 전체적인 흐름 파악 용이
- 구조가 명확하고 역할 분리가 쉬워 초기에 이해하기 쉬움, 작은 프로젝트에 적합

### [단점]

- 규모가 커지면 한 계층에 많은 클래스가 모여 복잡, 도메인 파악이 어려움
- 기능별 코드가 흩어짐 (회원에 관련된 코드들이 controller, service, repository에 분산)
- 특정 기능 (회원가입, 회원조회 등)을 파악하기 위해서 여러 레이어를 확인해야함

## 2. 도메인형 구조

### [개념]

기능(도메인) 단위로 코드를 묶는 구조

### [예시]

com.example.project

┣ domain/

┃ ┣ member/

┃ ┃ ┣ controller/

┃ ┃ ┣ service/

┃ ┃ ┣ repository/

┃ ┃ ┗ entity/

┃ ┗ mission/

┃ ….┣ controller/

┃ ….┣ service/

┃ ….┗ repository/

┗ global/

….┗ common config, error, util …

member, mission, term등 도메인을 기준으로 각 도메인과 관련된 controller, service, repo, entity등은 domain의 하위 패키지에 위치

### [장점]

- 기능 단위로 묶기 때문에 예를들어 회원 관련 코드가 하나의 폴더에 모여 있어서 한눈에 파악 가능
- 유지보수 용이 (도메인별로 독립적이기 때문에 다른 도메인에 영향 없이 특정 도메인 수정 가능)
- 확장성 높음 (새 기능 추가 시 독립된 패키지를 추가)
- 팀별로 도메인 단위를 담당해 팀을 분리하기에 용이

### [단점]

- 초보자 입장에서는 직관적이지 않음
- 작은 프로젝트에서는 계층형이 더 간결할 수 있음
- 도메인별 공통 로직이 중복될 수 있음

# JPA

:Java Persistence API

### [개념]

JPA는 자바 객체와 데이터베이스 테이블을 자동으로 매핑해주는 ORM(Object Relational Mapping) 표준 인터페이스

### [필요한 이유]

JPA 사용 이전의 JDBC 코드는 SQL문을 직접 작성하고 객체와 테이블을 수동으로 매핑 -> 코드가 복잡하고 유지보수 어려움

### [JPA는 Interface]

JPA는 인터페이스이므로 우리는 Interface에 대한 구현체 중 Hibernate를 사용

### [JPA 주요 Annotation]

- @Entity
- @Table(name=““)
- @Id, @GeneratedValue
- @Column(name=““)
- @OneToMany, @ManyToOne, @OneToOne, @ManyToMany

### [장점]

- 생산성 향상 (INSERT, SELECT등의 SQL을 직접 작성할 필요 없음)
- 유지보수 용이 (DB Column이 변경되어도 개발자는 Java Class의 필드만 수정, 관련된 모든 SQL Query를 수정하지 않아도 됨 )
- 패러다임 불일치 해결

### [단점]

- 여러가지 개념이 많아 초기 학습이 필요
- SQL이 자동으로 생성되기 때문에 예측이 어려움
- 캐시, 지연로딩 전략을 잘못 설계하면 성능 저하 가능성

### [동작과정]

1. 개발자 : 자바 객체를 JPA에 저장, 조회, 수정 등의 명령을 내림

    ```java
    Member member = new Member("영준");
    memberRepository.save(member);
    ```

2. JPA : 전달받은 객체의 정보를 분석 (@Entity, @Table, @Column 등), 적절한 SQL Query 자동으로 생성 (INSERT, UPDATE, SELECT, DELETE)
3. JPA : 생성된 SQL을 JDBC API를 사용해 DB에 전달하고 통신

    ```java
    Connection conn = DriverManager.getConnection(...);
    PreparedStatement ps = conn.prepareStatement("INSERT INTO member ...");
    ```

4. DB : 쿼리를 실행하고 결과반환
5. JPA : DB에서 받은 결과를 다시 자바 객체로 변환하여 애플리케이션에 전달

# N+1 문제

### [개념]

- 한 번의 조회로 가능 할 것 같던 작업이 연관된 데이터를 조회하면서 추가로 N번의 쿼리가 실행되는 문제
- 한 번의 쿼리로 부모 엔티티를 가져온 후, 각 부모에 연관된 자식 엔티티를 가져오기 위해 추가 쿼리 N개가 발생하는 현상

### [예시]

```java
@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;

    private String name;

    @OneToMany(mappedBy = "member", fetch = FetchType.LAZY)
    private List<Order> orders = new ArrayList<>();
}

@Entity
public class Order {
    @Id @GeneratedValue
    private Long id;

    private String productName;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "member_id")
    private Member member;
}
```

위와같이 Member, Order Entity가 있다고 가정하고

```java
List<Member> members = em.createQuery("select m from Member m", Member.class)
                         .getResultList();

for (Member member : members) {
    System.out.println(member.getOrders().size());
}
```

위의 예제 코드를 실행하게 된다면

```sql
select * from member;                 -- (1) 회원 전체 조회
select * from orders where member_id=1; -- (2)
select * from orders where member_id=2; -- (3)
select * from orders where member_id=3; -- (4)
...                                     -- (N+1)
```

이 경우 반복문 내부에서 member.getOrders()가 호출될 때마다 개별 쿼리가 날아간다.

따라서 N+1번의 쿼리가 발생한다.

위 예시에서 확인할 수 있는데 JPA는 필요할 때마다 불러와야 효율적일 것이라고 생각해 반복문이 한번씩 돌아갈 때 마다 지연 로딩을 하게된다.

Eager Loading 사용하면 members를 불러오는 순간 즉시 N+1번 호출된다.

### [발생이유]

JpaRepository에 정의한 인터페이스 메서드 실행

→ JPA는 메서드 이름을 분석해서 JPQL 생성 및 실행

(JPQL은 SQL을 추상화한 객체지향 쿼리 언어로서 특정 SQL에 종속되지 않고 엔티티 객체와 필드 이름을 가리고 쿼리를 한다)

→ JPQL은 findAll() 메서드 수행 시 select * from member 만 실행

→ JPQL 입장에서는 연관관계 데이터를 무시하고 해당 엔티티 기준으로 쿼리를 조회

→ 따라서 Eager, Lazy Loading에 상관없이 연관된 엔티티 데이터가 필요한 시점에 조회를 별도로 호출

### [해결방법]

1. Fetch Join 사용

    ```java
    @Query("select m from Member m join fetch m.orders")
    List<Member> findAllWithOrders();
    ```

   JOIN해서 한번에 가져와라! 를 직접 명시해서 한 번의 쿼리로 해결

2. @EntityGraph사용

    ```java
    @EntityGraph(attributePaths = {"orders"})
    @Query("select m from Member m")
    List<Member> findAllWithEntityGraph();
    ```

   JPA가 내부적으로 fetch join처럼 동작하게 만듦.

3. Batch Size 설정

    ```sql
    select * from orders where member_id in (1,2,3,4,...,100);
    ```

   Batch Size를 명시해서 IN Query로 묶어서 조회

   이 경우에는 Lazy를 유지 하면서 페이징을 병행 시 유용하다고 함


# 기본 키 생성 전략

JPA를 통해 id 값을 어떻게 자동으로 생성할지 결정하는 설정

## 1. GenerationType.IDENTITY

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

### [특징]

- DB의 기능을 사용해 PK생성
- INSERT 시점에 DB가 id를 생성
- JPA는 save() 시점에 바로 INSERT 쿼리를 실행해야 id를 알 수 있음 (⇒ 쓰기 지연 동작하지 않음 )

### [장점]

- 설정이 간단함
- DB가 PK를 생성하기 떄문에 충돌 위험 낮음

### [단점]

- save()할 때마다 즉시 DB 접근 필요 → 성능 저하
- DB 의존적 (DB의 기능을 사용하기 때문, AUTO_INCREMENT)

## 2. GenerationType.SEQUENCE

```java
@Id
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "member_seq_gen")
@SequenceGenerator(
    name = "member_seq_gen",
    sequenceName = "member_seq", // DB 시퀀스 이름
    allocationSize = 1
)
private Long id;
```

### [특징]

- DB의 Sequence 객체를 사용
- JPA가 insert ‘전에’ id를 미리 확보 → Entity에 할당

### [장점]

- id를 생성 후 받아오지 않고 미리 확보 가능 →  DB 접근 최소화 가능 (쓰기 지연 가능)
- 대량 INSERT시 효율적

### [단점]

- DB 의존적 (DB의 기능을 사용하기 때문, SEQUENCE객체)

## 3.GenerationType.Table

```java
@Id
@GeneratedValue(strategy = GenerationType.TABLE, generator = "member_table_gen")
@TableGenerator(
    name = "member_table_gen",
    table = "id_table",
    pkColumnName = "entity_name",
    valueColumnName = "next_id",
    allocationSize = 1
)
private Long id;

```

### [특징]

- 별도의 키 관리용 테이블을 만들어 id를 관리

| entity_name | next_id |
| --- | --- |
| member | 4 |
| review | 10 |
- 어떤 DB에서도 동일하게 작동

### [장점]

- DB에 의존하지 않음

### [단점]

- 키 전용 테이블을 매번 읽고 업데이트해야 해서 성능이 가장 느림

## 4.GenerationType.AUTO

```java
@Id
@GeneratedValue(strategy = GenerationType.AUTO)
private Long id;
```

### [특징]

- 사용하지 DB Dialect에 따라 자동 선택

### [장점]

- 설정 간단
- 이식성이 좋음 (DB가 변경되었을 때 자동으로 바꿔서 동작)
- (예) Oracle → MySQL ⇒ SEQUENCE → IDENTITY

### [단점]

- 예측 불가능성 : 실제 어떤 SQL이 나가고 어떤 전략이 적용될지 개발자가 명확히 알기 어려움

## 5. 자동 생성 없이 직접 PK 지정하기

### [특징]

- 개발자가 지정한 PK가 설정됨

### [장점]

- DB 의존하지 않음
- 이메일,주민번호 등 의미 있는 값을 PK로 쓸 수 있음
- 외부 시스템 연동에 유리 (외부 API등으로 부터 ID를 미리 받은 경우 그대로 사용 가능)

### [단점]

- PK 중복위험 - 중복 관리를 개발자가 책임져야 함
- 실수 위험