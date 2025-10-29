# 지연로딩과 즉시로딩의 차이

- ORM(객체 관계 매핑) 프레임워크에서 연관 관계에 있는 엔티티를 데이터베이스에서 조회하는 방식 결정

### 즉시 로딩(Eager Loading)

  - 특정 엔티티를 조회할 때 연관관계에 있는 다른 엔티티들을 모두 한 번에 같이 로딩
      - @ManyToOne, @OneToOne과 같은 관계에 대해 디폴트로 적용되는 방식
      - fetch = FetchType.EAGER

      ```
      @Entity
      public class Member {
    
          @Id
          private Long id;
    
          // 즉시 로딩 설정 (기본값이 EAGER이므로 생략 가능하나 명시적으로 표현)
          @ManyToOne(fetch = FetchType.EAGER)
          @JoinColumn(name = "team_id")
          private Team team;
    
          // ...
        
          // @ManyToOne(fetch = FetchType.EAGER) 설정
              Member member = entityManager.find(Member.class, 1L);
      }
      ```

      ```
      SELECT
          m.member_id,
          m.name AS member_name,
          t.team_id,
          t.name AS team_name
      FROM
          MEMBER m
      LEFT OUTER JOIN
          TEAM t ON m.team_id = t.team_id
      WHERE
          m.member_id = 1;
    
      -- 쿼리 1회 실행
      ```


    
### 지연 로딩(Lazy Loading)

-   특정 엔티티를 조회할 때 연관 관계에 있는 엔티티는 실제로 사용될 때까지 로딩을 미루는 방식
  - 메인 엔티티만 먼저 뢷ㅇ하고 연관된 엔티티는 프록시 객체로 대체. 연관된 엔티티의 실제 데이터에 접근하는 순간에 비로소 데이터베이스에 쿼리를 보내 로딩한다.
      - fetch = fetchType.LAZY

      ```
      @Entity
      public class Member {
    
          @Id
          private Long id;
    
          // 지연 로딩 설정
          @ManyToOne(fetch = FetchType.LAZY)
          @JoinColumn(name = "team_id")
          private Team team;
    
          // ...
          // @ManyToOne(fetch = FetchType.LAZY) 설정
              Member member = entityManager.find(Member.class, 1L); 
              // Member 객체는 로딩되지만, member.team 필드는 프록시 객체로 채워짐.
      }
      ```

      ```
      SELECT
          member_id,
          name,
          team_id
      FROM
          MEMBER
      WHERE
          member_id = 1;
    
      -- 쿼리 1회 실행
      ```

# JPQL

  #### JPQL : JPA에서 사용하는 객체 지향 쿼리 언어(엔티티 객체 조회)
   - 테이블을 대상으로 쿼리하지 않고 엔티티 객체를 대상으로 쿼리

  [JPQL 특징]
  - 기본적인 연산 지원 : SELECT, UPDATE, DELETE 등의 기본적인 연산 지원하고 함수 연산자 키워드 등 다양한 기능 제공
  - 객체지향 쿼리 언어 : 자바의 특성 최대한 활용할 수 있으며 쿼리 결과를 객체 또는 객체의 컬렉션으로 직접 반환 받을 수 있다.
  - 타입 안정성 제공 : 컴파일 시점에 쿼리의 문법 오류를 검사할 수 있다.

   ```
    SELECT m FROM Member AS m where m.username = ’Hello'
   ```

  1. 엔티티와 속성은 대소문자 구분하지만 JPQL 키워드는 대소문자 구분하지 않는다 
  2. Member AS m에서 m이라는 별칭 필수로 사용해한다
  3. AS는 생략 가능

  [JPQL 처리 방식]

  - 리턴 타입 ‘쿼리 타입’을 지정 : TypedQuery, Query
  - TypedQuery : 반환되는 ‘타입이 명확할 때’ 사용하는 클래스. 반환 타입을 미리 지정하기 때문에 컴파일 시점에 오류를 잡을 수 있어서 안정적

  ```
            @PersistenceContext
            private EntityManager em;
                
            // 쿼리를 수행합니다.
            TypedQuery<UserEntity> typedQuery = em.createQuery("select u from UserEntity u where u.id = :id", UserEntity.class);
            
            // 리스트 형태로 결과값을 반환 받습니다.
            List<UserEntity> resultList = typedQuery.getResultList();
  ```

  - Query : 반환되는 ‘타입이 명확하지 않을 때’ 사용하는 클래스. 다양한 타입의 결과로 반환받을 수 있지만 타입 체크가 런타임 시점에 수행되어 안정성 떨어질 수 있다

  ```
            @PersistenceContext
            private EntityManager em;
                
                
            Query query = em.createQuery("select u from UserEntity u where u.id = :id");
            
            // 단일한 결과를 반환 받습니다 : NoResultException, NonUniqueResultException 오류가 발생 할 수 있음.
            Object result = query.getSingleResult();
            
            // 리스트 형태로 결과를 반환 받습니다.
            List<UserEntity> resultList = query.getResultList();
  ```

 - 쿼리 구성 : createQuery
   - 이름 기준 파라미터 바운딩 : 쿼리 내에서 ‘:변수명’ 형태로 선언한 후에 setParameter 메서드를 사용하여 값드를 사용하여 값을 설정하는 방식

              ```
              @PersistenceContext
              private EntityManager em;
            
              TypedQuery<UserEntity> typedQuery = em
                            .createQuery("select u from UserEntity u where u.id = :id and u.userNm = :userNm", UserEntity.class)
                            .setParameter("id", 1234)
                            .setParameter("userNm", "admin");
              ```

   - 위치 기준 파리미터 바운딩 : 쿼리 내에 파라미터를 ‘?위치번호’ 형태로 선언한 후 setParameter 메서드를 사용하여 값을 설정(권장하지 않음)

              ```
              @PersistenceContext
              private EntityManager em;
                
                
              TypedQuery<UserEntity> typedQuery2 = em
                          .createQuery("select u from UserEntity u where u.id = ?1 and u.userNm = ?2", UserEntity.class)
                          .setParameter(1, 1234)
                          .setParameter(2, "admin");
              ```

 - 쿼리 프로젝션을 설정 : 엔티티, 임베디드 타입, 스칼라 타입 프로젝션

    **쿼리 결과로 ‘특정 필드’만 선택하여 가져오는 것**

     1. 엔티티 프로젝션 : 엔티티를 직접 선택하여 조회하는 방식. 엔티티에 있는 모든 필드가 조회됨

         ```
         @PersistenceContext
         private EntityManager em;
                
                
         // [CASE1] 엔티티(테이블)의 모든 필드(컬럼)을 조회합니다.
         TypedQuery<UserEntity> typedQuery = em
                 .createQuery("select u from UserEntity u where u.id = :id and u.userNm = :userNm", UserEntity.class)
                 .setParameter("id", 1234)
                 .setParameter("userNm", "admin");
         ```

     2. 임베디드 타입 프로젝션 : 엔티티의 특정 부분을 직접 조회하는 방식. 여러 속성을 한번에 묶어서 조회할 수 있음(엔티티내에 필드가 임베디드 타입인 경우에 사용)

         ```
         @Entity
         public class Order {
             @Id
             @GeneratedValue
             private Long id;
            
             @Embedded
             private Address address;
         }
            
         @Embeddable
         public class Address {
             private String street;
             private String city;
             private String state;
             private String zipCode;
         }
            
         @PersistenceContext
         private EntityManager em;
                
                
         List<Address> resultList = em
                 .createQuery("select m.address from Order m", Address.class)
                 .getResultList();
         ```

     3. 스칼라 타입 프로젝션 : 엔티티 내의 특정 필드들을 선택하여 조회하는 방식

         ```
         TypedQuery<UserEntity> typedQuery = em
                     // UserEntity 내에서 userId, userNm의 특정 엔티티 필드만 가져옵니다.
                     .createQuery("select u.userId, u.userNm from UserEntity u where u.id = :id and u.userNm = :userNm", UserEntity.class)
                     .setParameter("id", 1234)
                     .setParameter("userNm", "admin");
         ```

   - 쿼리 파라미터 지정 : 이름, 위치 기준 (쿼리 구성에서 설명)
   - 쿼리 결과 조회 방식을 선택 : getSingleResult(), getResultList()
     - getSingleResult() : 쿼리의 결과로 ‘단일 엔티티 객체’를 반환
     - 쿼리 결과가 없거나 결과가 2개 이상인 경우 예외 발생

       ```
       UserEntity userEntity = em
               .createQuery("select u from UserEntity u where u.id = :id and u.userNm = :userNm", UserEntity.class)
               .setParameter("id", 1234)
               .setParameter("userNm", "admin")
               .getSingleResult();
       ```


        - getResultList() : 쿼리의 결과를 ‘List 형태의 객체’로 반화
            - 쿼리의 결과가 없는 경우 빈 리스트 반환
            
            ```java
            List<UserEntity> userEntityList = em
                    .createQuery("select u from UserEntity u where u.id = :id and u.userNm = :userNm", UserEntity.class)
                    .setParameter("id", 1234)
                    .setParameter("userNm", "admin")
                    .getResultList();
            ```
            
        
    
![image.png](image%20(3).png)
    
   [JPQL 한계]
    
   1. 동적 쿼리 사용의 어려움
   2. 문자열 구성에 대한 오류 발생 : 쿼리 문자열 내부에 오타나 문법 오류가 있을 경우 이를 컴파일 시점에 체크할 수 없다. 이로 인해 실행시점에 오류 발생 가능
   3. SQL의 모든 기능 사용 불가능
   4. 복잡한 JOIN 문제


# Fetch Join

- JPA에서 연관된 엔티티나 컬렉션을 조회할 때 처음부터 함께 로딩하는 방식으로 성능으로 최적화 → A엔티티를 조회할 때 A와 관련된 B엔티티도 SQL 쿼리로 같이 가져와달라고 JPA에게 요청
    - 연관관계의 엔티티나 컬렉션을 프록시가 아닌 진짜 데이터를 한번에 같이 조회하는 기능
    - 지연로딩 전략으로 인해 발생하는 N+1 쿼리 문제를 해결하기 위해 사용

[Fetch Join 사용 시 주의]

- 별칭 사용 불가
- 컬렉션 Fetch Join 시 데이터 중복 : 일대다 같은 컬렉션은 Fetch Join하면, DB에서 데이터 행이 중복되어 조회될 수 있음
- 두 개 이상의 컬렉션 Fetch Join 불가
- 페이징 제한
  [N+1 문제] : N개의 엔티티를 조회하기 위해 1번의 SQL 쿼리 실행 + 조회된 N개의 각 엔티티에 연관된 엔티티 접근(N번의 추가 쿼리 발생) ⇒ 총 1 + N번의 쿼리 발생

[Fetch Join을 통한 N+1 문제해결]

- 연관된 엔티티를 즉시로딩처럼 동작하게 만들면서 N+1 문제가 발생하지 않도록 처음부터 하나의 SQL 쿼리로 모든 데이터를 로드한다.
- Member → Team 으로 @ManyToOne관계일때
    - 일반 조인 : select m from Member m join m.team t → Member와 Team을 조인하지만 실제 영속성 컨텍스트에 로딩되는 것은 Member만이며 Team은 여전히 지연로딩일 수 있다

        ```
         select
                 distinct team0_.id as id1_1_,
                 team0_.name as name2_1_
            from team team0_ 
            inner join
                member members1_
                    on team0_.id=members1_.team_id
        ```

    - Fetch Join : select m from Member m join fetch m.team t → Member를 조회하면서 연관된 Team 엔티티도 프록시가 아닌 실제 엔티티 객체로 한 번의 쿼리로 로드하면서 영속성 컨텍스트에 저장한다

        ```
        Hibernate:
            select
                 distinct team0_.id as id1_1_,
                 members1_.id as id1_0_1_,
                 team0_.name as name2_1_,
                 members1_.age as age2_0_1,
                 members1_.name as name3_0_1_,
                 members1_.team_id as team_id4_0_1_,
                 members1_.team_id as team_id4_0_1_,
                 members1_.id as id1_0_1_
            from team team0_ 
            inner join
                member members1_
                    on team0_.id=members1_.team_id
        ```

# @EntityGraph

  ### EntityGraph

- JPA에서 연관된 엔티티를 함께 조회할 때 어떤 연관 엔티티를 fetch(즉시) 로딩할지를 지정할 수 있는 기능 → JPQL의 Fetch Join을 직접 쓰지 않고도 어떤 연관관계를 함게 가져올지 선언적으로 설정할 수 있는 방법


[필요이유]
        
- LAZY : N + 1 문제 발생 가능
- EAGER : 불필요한 조인이 많아짐
            
⇒ EntityGraph를 사용해서 평소엔 LAZY 유지하고, 특징 조회 시점에만 특정 연관 엔티티를 함께 즉시 로딩하도록 제어가능
            
        
[EntityGraph 정의 방식]
        
1. 즉석 지정 방식(inline)
- Repository 메서드 위에 바로 @EntityGraph(attributePaths = {..}) 작성
            
            ```
            @EntityGraph(attributePaths = {"orders"})
            @Query("SELECT c FROM Customer c")
            List<Customer> findAllWithOrders();
            
            // 조회시 member도 함께 한 번에 가져옴
            ```
            
        
2. 이름 기반 방식(@NamedEntityGraph)
- 엔티티 클래스 안에 미리 그래프를 정의하고 Repository에서 사용함
            
          ```
          @Entity
          @NamedEntityGraph(
              name = "Customer.withOrders",
              attributeNodes = @NamedAttributeNode("orders")
          )
          public class Customer {
              @Id @GeneratedValue
              private Long id;
              private String name;
            
              @OneToMany(mappedBy = "customer")
              private List<Order> orders;
          }
            
          // 이후 레포에서 사용
          @EntityGraph(value = "Customer.withOrders", type = EntityGraph.EntityGraphType.LOAD)
          @Query("SELECT c FROM Customer c")
          List<Customer> findAllWithOrders();
            
          ```
            
        
[@EntityGraph vs Fetch Join]
        
| 구분 | @EntityGraph | JPQL Fetch Join |
| --- | --- | --- |
| 문법 | 애노테이션 기반 | JPQL 직접 작성 |
| 재사용성 | 엔티티 그래프 이름으로 재사용 가능 | JPQL마다 새로 작성 |
| 동적 쿼리 | 불가능 (정적) | 가능 |
| fetch control | 선언적으로 제어 | 쿼리 내에서 제어 |
| 편의성 | ✅ 매우 간단 | ❌ 쿼리문 직접 작성 필요 |


# commit과 flush 차이점

### JPA에서 엔티티 변경 → DB 반영 과정

1. 엔티티 변경(persist, merge, remove 등)
2. 변경 내용이 1차 캐시(영속성 컨텍스트)에 저장됨
3. flush() → SQL이 실제로 DB로 전송(트랙잭션 안 끝남)
4. commit() → 트랜잭션이 종료되고 변경 내용이 DB에 확정됨

### flush() : DB에 쿼리 보내기!

- 영속성 컨텍스트이 변경 내용을 DB에 동기화하는 단계
- SQL을 DB에 보내지만 commit은 하지 않는다
- 트랜잭션은 여전히 활성 상태

→ 지금까지의 변경 내용을 DB에 반영하지만 트랙잭션은 끝내지마

- JPA는 자동으로 flush 실행
- 트랜잭션 커밋 직전
- JPQL/Criteria 쿼리 실행 직전 (DB와 영속성 컨텍스트의 상태 맞추기위해)
- 명시적으로 em.flush() 호출했을 때

        ```
        em.persist(member1);
        em.persist(member2);
        
        // 아직 DB에는 반영되지 않음
        em.flush(); // SQL 전송, 그러나 commit은 안 함
        
        System.out.println("flush 완료");
        
        // 트랙잭션이 rollback되면 데이터는 사라짐
        ```

### Commit() : 보낸 쿼리를 확정시키기!
    
- 트랜잭션을 종료하며 DB에 반영된 변경 내용을 영구히 확정하는 단계
- commit() = flush() + 트랜잭션 확정
- 이후 rollback(트랜잭션 취소할 때 사용) 불가
- DB의 실제 데이터가 바뀌는 시점
        
        ```
        em.persist(member);
        tx.commit(); // 내부적으로 flush() → commit 순서로 실행됨
        
        // 자동적으로 flush() 실행 -> SQL 전송 -> DB transaction commi -> 변경 확정
        ```
        
    
[공통점]
    
1. DB와의 동기화 : 둘 다 영속성 컨텍스트의 변경 내용을 DB에 반영
2. SQL 실행 : 둘 다 INSERT/UPDATE/DELETE SQL을 DB에 실제로 보냄
3. flush 포함 : commit 시 flush 자동 포함
    
[차이점]
    
| 구분 | flush | commit |
| --- | --- | --- |
| **의미** | 영속성 컨텍스트 → DB로 **동기화** | 트랜잭션을 **종료 및 확정** |
| **트랜잭션 상태** | 유지됨 | 종료됨 |
| **롤백 가능 여부** | 가능 (아직 확정 전) | 불가능 (확정 후) |
| **SQL 전송 여부** | 전송함 | 전송 + 확정함 |
| **자동 실행 시점** | JPQL 실행 전, commit 직전 등 | 트랜잭션 끝날 때 |
| **DB 반영의 영속성** | 임시적 (rollback 시 취소) | 영구적 (rollback 불가) |

# QueryDSL, OpenFeign의 QueryDSL

### QueryDSL : 오픈소스 프로젝트로 JPQL을 Java 코드로 작성할 수 있도록 하는 라이브러리

[특징]

- 정적타입 : 엔티티를 기반으로 Q-type 객체 생성하고 이를 이용해 필드와 메스드에 접근한다
    - 컴파일 시점에 오류를 발견할 수 있어 런타임 오류 줄여준다
    - QMission qMissoin = QMission.mission
- 코드 기반 쿼리 : 모든 쿼리가 단순한 문자열이 아닌 Java의 메서드 호출로 구성
    - IDE의 자동완성기능과 리팩토링을 완벽 지원
- 높은 가독성 : 코드가 직관적이기 때문에 복잡한 동적 쿼리 구성을 쉽게 할 수 있다



[사용예시]

| **기능** | **Querydsl 코드** |
| --- | --- |
| **단순 조회** | `queryFactory.selectFrom(qMember).fetch();` |
| **조건 추가** | `queryFactory.selectFrom(qMember).where(qMember.age.gt(20)).fetch();` |
| **정렬** | `queryFactory.selectFrom(qMember).orderBy(qMember.name.desc()).fetch();` |

### OpenFeign QueryDSL

- Query DSL 개발 중단된 후 OpenFeign QueryDSL로 전환
- 기존의 QueryDSL 보안 취약점이 있었고, 새로운 버전 업데이트도 되지 않음
- 대부분의 기업은 QueryDSL 기반 코드베이스를 유지하며 점진적인 전환을 원했고 이를 위해 등장한 것이 OpenFeign 팀이 포크한 QueryDSL 프로젝트이다

[QueryDSL vs OpenFeign QueryDSL]

- 의존성 차이 : QueryDSL은 ‘com.querydsl’ , OpenFeign은 ‘io.github.openfeign.querydsl

# N+1 문제 해결할 수 있는 여러 방안들

<4주차 워크북 내용 사용!>

[N+1 문제를 해결하는 방법]

1. fetch join 사용
- inner join과 outer join으로 테스트

```
select t from Team t join fetch t.members // INNER JOIN

select t from Team t left join fetch t.members // OUTER JOIN
```
```
public interface TeamRepository extends JpaRepository<Team, Long> {

    @Query("select t from Team t join fetch t.members")
    List<Team> findAllWithInnerFetchJoin();
    
    @Query("select t from Team t left join fetch t.members")
    List<Team> findAllWithOuterFetchJoin();
}
```
```
@Test
@DisplayName("페치 조인으로 모든 팀 조회 시 쿼리를 관찰한다.")
void inspect_query_fetch_join_findAll() {
    teamRepository.findAllWithInnerFetchJoin();
    teamRepository.findAllWithOuterFetchJoin();
}
```
→ 두 경우 모두 Team 엔티티를 가져올 때 Member를 join해서 가져오기 때문에 추가 조회 쿼리가 발생하지 않는다. 이렇게 N+1 문제를 해결할 수 있다.

2. EntityGraph 사용
    - 가져올 엔티티를 지정한 EntityGraph를 생성하고, 해당 EntityGraph를 조회하는 메소드에 파라미터로 넘겨서 조회할 때 함께 가져오도록 한다.

```
    EntityGraph<Team> entityGraph = em.createEntityGraph(Team.class);
    entityGraph.addSubgraph("members");
    em.find(Team.class, teamId, Map.of(SpecHints.HINT_SPEC_LOAD_GRAPH, entityGraph));
```

```
    public interface TeamRepository extends JpaRepository<Team, Long> {
    
        @Query("select t from Team t")
        @EntityGraph(attributePaths = "members")
        List<Team> findAllWithEntityGraph();
    }
   ```

```
    @Test
    @DisplayName("EntityGraph로 모든 팀 조회 시 쿼리를 관찰한다.")
    void inspect_query_EntityGraph_findAll() {
        teamRepository.findAllWithEntityGraph();
    }
```

→ 1번 경우와 동일하게 Team Entity를 가져올 때 Member를 join해서 가져오기 때문에 추가 조회 커리 발생하지 않음.

3-1. batch fetching (@BatchSize)

- 설정을 통해 배치 사이즈를 조절하여 추가 조회 쿼리를 줄이는 해결 방법

```
    @Entity
    public class Team {
    
        ...
    
        @OneToMany(mappedBy = "team", fetch = FetchType.EAGER)
        @BatchSize(size = 3)
        private List<Member> members = new ArrayList<>();
    	
        ...
    }
```

→ 조회되는 엔티티 위에 @BatchSize를 추가한 뒤 Member 엔티티가 조회될 때 IN절을 통해 한번에 조회 가능하다.

3-2. subselect fetching (@BatchSize)

- 설정을 통해 추가 조회 쿼리를 서브 쿼리로 줄이는 방법

```
    @Entity
    public class Team {
        ...
    
        @OneToMany(mappedBy = "team", fetch = FetchType.EAGER)
        @Fetch(value = FetchMode.SUBSELECT)
        private List<Member> members = new ArrayList<>();
    
        ...
    }
```

→ Member 엔티티 위에 @Fetch를 사용하면 엔티티가 조회될 때 IN절과 서브 쿼리를 사용하여 한번에 조회 가능.

# 영속 상태의 종류

영속성 컨텍스트 : 엔티티를 영구 저장하는 환경

→ EntityManager로 엔티티를 저장하거나 조회하면 엔티티 매니저는 영속성 컨텍스트에 엔티티를 보관하고 관리할 수 있게 됨.

### 1. 비영속(Transient) : 단순 자바 객체

- JPA와 전혀 관련 없는 단순히 new로 생성한 객체 상태
- 아직 영속성 컨텍스트에 등록되지 않음
- DB에 저장되지 않고 1차 캐시에도 없음
- 변경해도 JPA는 알지 못한다

  ```
  Member member = new Member();
  member.setName("재준");  // 단순한 자바 객체
  ```

### 2. 영속(Persistent) : JPA가 관리

- EntityManger를 통해 엔티티를 영속성 컨텍스트에 저장하거나, DB로부터 조회했을 때  영속성 컨텍스트가 엔티티를 관리한다. 영속성 컨텍스트가 관리하는 엔티티를 영속 상태라고 함
- JPA가 객체를 관리
- 트랜잭션이 commit될 때 DB에 반영
- 같은 트랜잭션 내에서 동일한 객체 보장(1차 캐시로 관리)

    ```
    // em == EntityManager의 참조 변수
    em.persist(member);           // entity를 영속성 컨텍스트에 저장(영속 상태로 만듬)
    em.find(Member.class, id);    // entity를 DB에서 로드(영속 상태로 만듬)
        
    // 영속성 컨텍스트는 엔티티를 식별자 값으로 구분
    // 식별자 값 : @Id 어노테이션이 붙은 필드
    ```


### 3. 준영속(Detached) : 관리 중단
    
- 영속성 컨텍스트가 관리하던 영속 상태의 엔티티를 영속성 컨텍스트가 관리하지 않게 됐을 떄 준영속 상태가 된다. 영속성 컨텍스트에서 분리된 상태
- 변경 감지 일어나지 않음
- DB와 동기화되지 않음
- 다시 관리하려면 merge() 사용
    
  ```
  em.detach(member); // 특정 엔티티만 분리
  em.clear();        // 영속성 컨텍스트 전체 초기화
  em.close();        // EntityManager 닫기 → 전체 준영속
    
  em.merge(~~)
  ```
    
### 4. 삭제(Removed) : 삭제 예정
    
- 엔티티를 영속성 컨텍스트와 데이터베이스(flush() 호출시)에서 삭제한다
- em.remove() → 삭제 예약 상태
- 실제 DB에서 삭제는 트랜잭션 커밋 시점 (flush()) 시점 (DELETE SQL 실행)
    
  ```
  em.remove(member); // 삭제 예약
  ```
    
 | 상태 | 관리됨 | DB 반영 여부 | 주요 메서드 |
 | --- | --- | --- | --- |
 | 비영속 | ❌ | X | new |
 | 영속 | ✅ | O (commit 시) | persist(), find(), merge() |
 | 준영속 | ❌ | X | detach(), clear(), close() |
 | 삭제 | ✅ (삭제 예정) | commit 시 DELETE | remove() |