## 지연로딩이란?

💡 필요할 때까지 연관 객체를 조회하지 않는다.

💡 프록시 객체가 들어있다가 접근 시 진짜 데이터를 가져온다.

## 이해를 위한 예시코드

```java
@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;

    private String name;

    @ManyToOne(fetch = FetchType.LAZY)
    private Team team;
}
```

- FetchType.Lazy를 통해 지연로딩을 하도록 선언했다.
- 이 상황에서 member 조회시 member만 DB에서 가져온다.
- member.getTeam()을 호출하는 시점에 쿼리를 추가로 실행해 team을 조회한다.

## 장점

- 불필요한 쿼리 최소화 (실제로 필요한 시점에서 가져오기 때문)
- 성능 최적화 가능

## 단점

- 영속성 컨텍스트가 닫힌 이후 접근시 LazyInitializationException 발생가능 → fetch join이나 @EntityGraph로 해결가능
- N+1 문제 발생가능

## LazyInitializationException

- 스프링 Controller → Service → Repopsitory → DB 구조에서 @Transactional은 보통 Service 계층에 선언
- 따라서 Controller에서 객체 접근시에는 이미 Transactional 닫힘. 즉 Hibernate 세션이 종료
- 따라서 DB에 접근하지 못함
- 지연로딩인 상태이고 team에 대한 데이터를 DB에서 가져오지 않은 상황에서 Controller에서 member1.getTeam() 호출 시 해당 Exception 발생

## LazyInitializationException에 대한 해결방안

1. 영속성 컨텍스트 범위 내에서 로딩

(= service 계층 안에서 필요한 데이터를 모두 로딩한 뒤 반환)

1. Fetch Join / @EntityGraph 사용

(= 조회 시점에 필요한 연관 객체를 함께 로딩하도록 지정)

1. Controller까지 트랜잭션 확장

(= spring.jpa.open-in-view: true를 통해 Controller까지 트랙잭션 확장가능)

---

## 즉시로딩이란?

💡 엔티티를 조회할 때, 연관된 엔티티까지 즉시 함께 로딩

## 이해를 위한 예시코드

```java
@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;

    private String name;

    @ManyToOne(fetch = FetchType.EAGER)
    private Team team;
}
```

- FetchType.EAGER를 통해 즉시로딩을 하도록 선언했다.
- 이 상황에서 member 조회시 member이외에도 member에 대한 team까지 함께 가져온다.

## 장점

- 코드 간결성 (연관된 객체 사용할 때 별도의 로딩 과정 없이 바로 접근)
- 성능 일관성 및 예측 가능성 (연관된 데이터를 미리 한번에 로드하기 때문에 이후에 따로 가져올 일이 없어 일관되고 예측 가능하다)

## 단점

- 불필요한 데이터 로드 및 메모리 낭비 (실제로 사용하지 않을 수도 있는 연관된 데이터까지 모두 한꺼번에 로드)
- 복잡성과 예상치 못한 쿼리 (연관관계가 복잡할 경우, 어떤 상황에서 어떤 연관된 엔티티까지 로드될지 예측하기 어려워진다)
- N+1 문제 발생가능
- 순환참조

## 순환참조

- 두 엔티티가 서로를 즉시 로딩하도록 설정되었을 때 발생할 수 있는 문제
- Team은 List<Member>를 즉시 로딩
- Member는 자신이 속한 Team을 즉시 로딩
- 이 상황에서 Team 객체 하나 조회하면 Team에 포함된 Member들을 로드하고 Member들은 즉시 로딩으로 연관된 Team을 다시 로드 이 과정이 무한반복

## 순환참조 해결방안

- 한쪽을 지연로딩 하도록 설정 일반적으로 **@ManyToOne 쪽은 EAGER**, **@OneToMany 쪽은 LAZY로 설정**
- 지연 로딩을 기본으로 하고, 필요할 때만 fetch join으로 명시적으로 로딩하도록 설정

---

## 기본설정

| @ManyToOne | **EAGER** |
| --- | --- |
| @OneToOne | **EAGER** |
| @OneToMany | **LAZY** |
| @ManyToMany | **LAZY** |

---

## JPQL이란?

- JPA를 사용할 때 엔티티 객체를 기준으로 작성하는 쿼리 언어.
- SQL처럼 데이터베이스 테이블을 기준으로 조회하지 않고 객체를 기준으로 조회, 수정, 삭제 수행

| **구분** | **SQL** | **JPQL** |
| --- | --- | --- |
| 기준 | 테이블(table) | 엔티티(entity) |
| 조회 대상 | 컬럼(column) | 엔티티의 필드(field) |
| 반환값 | 테이블의 행(row) | 엔티티 객체 |
| 예시 | SELECT * FROM member | SELECT m FROM Member m |

---

## JPQL의 목적

- SQL은 데이터베이스 중심 언어
- JPQL은 객체 중심 언어

```java
// Member 엔티티 기준
SELECT m FROM Member m WHERE m.age > 20
```

```sql
SELECT * FROM member WHERE age > 20;
```

( 위와같이 코드를 객체 중심으로 작성하면 아래처럼 변환해줌 )

- 즉 개발자가 객체 기준으로 쿼리를 작성할 수 있도록 해줌.

## 객체 중심으로 접근이 가능한 이유

- JPA가 엔티티 매핑 정보를 모두 알고 있기 때문
- @Entity, @Table, @Column등의 정보를 모두 입력하여 JPA가 알고 있기 때문에 이 정보를 이용해 객체 중심언어를 DB중심 SQL로 변환가능

---

## 사용법  : Repository

### 1. 메서드 이름으로 생성

| findBy, readBy, getBy | SELECT 쿼리 시작 |
| --- | --- |
| countBy | COUNT 쿼리 |
| existsBy | EXISTS 쿼리 |
| deleteBy, removeBy | DELETE 쿼리 |
| …And… / …Or… | 조건 연결 |
| Between, LessThan, GreaterThan | 범위 조건 |
| Like, Containing, StartingWith, EndingWith | 문자열 검색 |
| OrderBy | 정렬 |

```sql
public interface MemberRepository extends JpaRepository<Member, Long> {
    List<Member> findByName(String name);
    List<Member> findByAgeGreaterThan(int age);
		List<Member> findByTeamNameAndAgeLessThan(String teamName, int age);
		List<Member> findByNameContainingOrderByAgeDesc(String keyword);
}
```

위처럼 정해진 규칙을 가지고 메서드를 만들 수 있고 JPA가 이를 바탕으로 JPQL 생성 및 실행할 때는 SQL로 변환해서 실행

### 2. 메서드 이름으로 생성시 장단점

✅ : 코드 간결 , SQL작성 필요 없음

❗ : 쿼리가 조금만 복잡해져도 메서드 이름이 길고 복잡해지거나 사용하기 어려움

### 3. Query annotation으로 생성

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    @Query(value = "SELECT * FROM member WHERE name = :name", nativeQuery = true)
		List<Member> findNative(@Param("name") String name);
}
```

- 위 코드처럼 JPQL 코드를 직접 작성하면 JPA가 SQL로 변환하여 DB에 전달
- 위 예시에서는 JPQL을 작성했지만 nativeQuery=true로 설정 시 SQL 직접 작성 가능

### 4. Query annotation 장단점

✅  : 복잡한 쿼리 작성 가능

❗ :  JPQL이나 SQL을 직접 작성해야 함, 문자열 기반이기 때문에 컴파일 시점에 오타 검증 불가, 동적 쿼리는 불편

--- 

## Fetch Join이란?

### 💡 JPQL에서 연관된 엔티티를 한번의 쿼리로 함께 조회하는 방법

---

## 예시코드

```java
SELECT m FROM Member m JOIN FETCH m.team
```

멤버를 조회할 때 해당 멤버가 속한 team을 Join을 통해 함께 조회한다.

즉, 별도의 추가 쿼리 없이 한 번의 SQL로 두 엔티티를 모두 가져온다.

---

## Fetch Join 사용이유

### 💡Fetch Join을 이용하면 N+1 문제를 해결할 수 있다.

- N+1문제를 간단히 복습하면 초기쿼리 1번을 통해 N개의 멤버를 가져왔다고 가정할 때 각 N명의 멤버들이 각각 팀에 포함되어 있을 때 팀을 조회하기 위해 N번의 쿼리가 수행된다.

  → 결과적으로 총 N+1번의 쿼리 발생해 성능저하


```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    // Fetch Join 사용 — JPQL을 직접 @Query에 명시
    @Query("SELECT m FROM Member m JOIN FETCH m.team")
    List<Member> findAllWithTeam();

    // 조건 추가도 가능
    @Query("SELECT m FROM Member m JOIN FETCH m.team WHERE m.name = :name")
    Optional<Member> findByNameWithTeam(@Param("name") String name);
}
```

✅ Fetch Join을 사용하면 지연 로딩의 이점을 유지하면서 특정 상황에서만 선택적인 즉시 로드를 가능하게 하여 N+1 문제를 해결

---

## 정리

- 기본은 지연로딩을 사용하여 필요한 시점에만 데이터를 가져온다.
- 모든 연관 데이터를 한 번에 가져와야 하는 경우 fetch join을 통해 명시적으로 조인하여 즉시 로딩 수행한다.

---

## 추가 정보

1. 데이터 중복 발생 가능 (1:N)

    ```java
    SELECT DISTINCT t FROM Team t JOIN FETCH t.members
    ```

   한 팀에 3명의 팀원이 있었다면 이렇게 Fetch Join 수행 시 하나의 팀이 3번 반복되어 나타난다.

   팀1 : 사람A

   팀1 : 사람B

   팀1 : 사람C

   이런 상황에서 DISTINCT를 붙임으로써 같은 팀인 경우 팀을 더 생성하지 않고 기존 팀1에 대해 사람을 추가해서 우리의 예상처럼 동작함

2. Fetch Join은 페이징 불가능 (1:N)

   위에 설명한 데이터 중복 발생으로 인해 조인된 결과에는 row가 늘어나기 때문에 페이징이 불가능하다.

3. Fetch Join은 여러 개를 동시에 사용할 수 있지만 주의 필요

    ```java
    SELECT m FROM Member m
    JOIN FETCH m.team
    JOIN FETCH m.orders
    ```

   이렇게 여러 개를 동시에 fetch join 가능하지만 조인 대상이 많아질수록 결과가 폭발적으로 늘어나고 중복이 심해지기 때문에 주의가 필요.
---
## EntityGraph란?

@EntityGraph는 Fetch Join을 Annotation으로 표현해 직접 JPQL을 명시하지 않고 어떤 연관 Entity를 함께 로드할지 지정할 수 있는 어노테이션이다.

## 사용법

```java
SELECT m FROM Member m JOIN FETCH m.team
```

이 코드는 앞서 FetchJoin에 대해 설명하면서 member와 member가 속한 team을 JOIN을 통해 한번에 불러오는 JPQL 코드이다.

이 때 우리는 member와 join할 entity가 team인것을 JPQL을 직접 작성하여 명시하였다.

이를 더욱 편리하게 만들어 주는것이 EntityGraph이다.

```java
@EntityGraph(attributePaths = {"team"})
@Query("SELECT m FROM Member m")
List<Member> findAllWithTeam();
```

```java
@EntityGraph(attributePaths = {"team"})
    List<Member> findByName(String name);
```

@EntityGraph를 사용해 team을 JOIN해서 로딩하겠다를 다음과 같이 작성할 수 있고 이 경우 @Query는 member를 조회하는 JPQL을 그대로 작성하면된다. 또한 findByName처럼 메서드 이름을 통해 JPQL을 자동 생성하는 코드에서도 동일하게 사용 가능하다.

```java
@EntityGraph(attributePaths = {"team", "orders", "profile"})
List<Member> findAll();
```

attributePaths를 이용해 team이외에도 다른 데이터들을 JOIN FETCH 여러 개 수행해서 가져올 수 있다.

---

## 추가사항

1. JQPL과 @EntityGraph를 혼용시

   JPQL이 주도권을 가진다. 따라서 JPQL이 연관 관계를 명시하지 않았다면 EntityGraph가 FETCH JOIN을 추가하지만 명시 되었다면 EntityGraph는 무시된다.


1. attributePaths = {”~~”} 는 DB 컬럼명이 아니라 엔티티 필드명 기준이다.

---

## Commit과 Flush란?

Commit은 트랙잭션을 종료하고 확정 → DB에 영구 저장O

(DB가 확정 되었기 때문에 insert를 수행했다고 가정 시 되돌리려면 해당 entity를 찾아 delete해야함)

Flush는 SQL실행해서 DB에 반영 → 아직 트랜잭션이 끝나지 않은 상태 → 영구저장X

(Transaction이 끝나면서 Commit이 되기때문에 아직 DB에 확정되지 않음 따라서 Rollback을 해서 insert에 대한 rollback이 가능함)

---

## Flush는 언제 일어날까?

1. 트랜잭션 **commit** 직전

    ```java
    @Transactional
    public void flushExample1() {
        Member m = new Member("A");
        em.persist(m); // flush X
        
        
        //Transaction을 마치며
        //flush
        //commit
    }
    ```

   트랜잭션의 끝에서 자동으로 commit이 발생하는데 이 commit 직전에 flush가 자동 실행됨

2. JPQL / Criteria 쿼리 실행 직전

    ```java
    @Transactional
    public void flushExample2() {
        Member m = new Member("A");
        em.persist(m); // flush X
    
        // ✅ JPQL 실행 직전 자동 flush 발생
        List<Member> result = em.createQuery(
            "SELECT m FROM Member m WHERE m.name = :name", Member.class)
            .setParameter("name", "A")
            .getResultList();
    
        System.out.println("조회된 수: " + result.size());
    }
    ```

   JPQL 실행 직전에 DB에는 Member m에 대한 정보는 없음 영속성 컨테이너에만 있는 상태

   이 상황에서 JPQL을 수행하면 m을 조회할 수 없음 따라서 JPQL의 실행 직전에 flush 자동으로 실행

3. 명시적으로 em.flush() 호출할 때

    ```java
    @Transactional
    public void flushExample3() {
        Member m = new Member("A");
        em.persist(m); // flush X
    
        em.flush(); // ✅ 직접 flush 호출 → 즉시 INSERT SQL 실행됨
    }
    ```

   직접 명시적으로 flush를 호출가능

   flush 이후면서 transaction을 마치는 commit 이전에는 해당 flush에 대한 rollback 가능


---

## Flush를 사용하는 이유는?

매 상황마다 SQL을 날리지 않고 이를 모아두었다가 flush 호출 시점에 한번에 실행함으로써

네트워크 I/O 비용 감소, 쓰기 지연 가능, 효율적 처리등이 가능하다.

---

## 정리

| **항목** | **flush()** | **commit()** |
| --- | --- | --- |
| 수행 위치 | 트랜잭션 내부 | 트랜잭션 마지막 |
| 역할 | SQL 실행(DB 동기화) | DB 트랜잭션 확정 |
| DB 반영 | 즉시 SQL 실행됨 (영구저장X, Rollback가능) | 변경 내용이 영구 저장됨 |
| 롤백 가능 | O | X |
| 자동 실행 시점 | commit 전, JPQL 실행 전, 직접 호출 | 트랜잭션 끝날 때 |

---
## QueryDSL이란?

QueryDSL은 타입 안전성을 보장하는 자바 기반의 쿼리 빌더 라이브러리

→❓이해가 안된다면 다음의 문제점들을 보면 이해가 가능하다.

## QueryDSL이 없는 상황에서의 문제점

1. JPQL은 문자열 기반

    ```java
    @Transactional
    public void flushExample2() {
        Member m = new Member("A");
        em.persist(m); // flush X
    
        // ✅ JPQL 실행 직전 자동 flush 발생
        List<Member> result = em.createQuery(
            "SELECT m FROM Member m WHERE m.name = :name", Member.class)
            .setParameter("name", "A")
            .getResultList();
    
        System.out.println("조회된 수: " + result.size());
    }
    ```

   이 코드는 commit, flush 키워드 정리에 사용된 코드로 JPQL을 실행하는 부분을 보면 단순 문자열 기반으로 쿼리를 생성함. 즉 컴파일 시점에 해당 문자열에 문제가 있다는 점을 확인할 수 없음

2. 조건문 동적 생성이 복잡함

    ```java
    String jpql = "SELECT m FROM Member m WHERE 1=1";
    if (username != null) jpql += " AND m.username = :username";
    if (age != null) jpql += " AND m.age = :age";
    ```

   위와같이 문자열을 상황에 맞게 이어붙이는 작업을 통해 동적 생성처럼 동작하도록 만들어야 하는데 조건이 많아질수록 복잡해짐.


## 여기까지 봤을 때 이해한 QueryDSL이란?

1. SQL에서 JPA,JPQL로 발전시킨 이유는 패더라임 불일치가 불편했기 때문에 객체단위로 처리하기 위함이었음
2. JPA,JPQL 을 통해 객체 단위로 DB와 소통을 할 수 있게 되었는데 JPQL을 통해 객체단위로 소통이 가능해진거지 여전히 Query를 문자열 기반으로 짜야하는 점에서 오는 불편함이 있음.
3. 쿼리를 자바 코드로 짜고 해당 자바 코드를 자동으로 JPQL로 번역을 해주는 도구가 있으면 위의 문제를 해결할 수 있겠다 해서 만들어져 사용되는것이 QueryDSL

## 간단하게 알아보는 QueryDSL 내부동작

1. Member 엔티티가 있으면 빌드 시 자동으로 Q로 시작하는 쿼리 전용 클래스를 생성
2. String으로 쿼리를 작성하던 과정을 이제는 자바 코드를 통해 작성
3. QueryDSL이 정해진 규칙에 따라 java코드를 분석해 적절한 JPQL 문자열 생성
4. JPA가 JPQL을 SQL로 변환
5. DB Query 실행
6. JPA가 결과를 엔티티로 매핑 후 반환

## 쿼리 전용 클래스 (QMember)

```java
@Entity
public class Member {
    private Long id;
    private String username;
    private int age;
}
```

```java
// 자동 생성된 코드
public class QMember extends EntityPathBase<Member> {
    public static final QMember member = new QMember("member");

    public final NumberPath<Long> id = createNumber("id", Long.class);
    public final StringPath username = createString("username");
    public final NumberPath<Integer> age = createNumber("age", Integer.class);
}
```

Member 클래스에서 username은 이름에 대한 string 값을 담는거라면

QMember 클래스가 자동 생성되면서 만들어진 username은 DB의 username 이라는 column까지의 경로를 코드로 표현한 객체

이렇게 DB Column에 대해 경로를 저장해 두어야 자바 코드를 작성했을 때 문자열 기반으로 비교해서 찾는게 아니라 정확히 이게 DB의 어떤 컬럼과 어떤 컬럼에 대해 어떤 동작을 수행하라는건지 이해할 수 있음

- ..

  Q클래스에서는 경로를 DB 테이블의 컬럼까지의 경로를 저장하니까 해당 값이 가리키고자 하는게 명확한거고

  만약에 저걸 String username 해버리면 DB에 username 컬럼이 있다고 해도 이게 같은건 맞지만 사람입장에서 그런거고 JPA가 명확히 하기 어렵다?

  JPQL에서 문자열 치는데 username = :name 이렇게 칠때 usernam 라고 오타가 나도 확인을 못하지만 username column에 대한 경로를 저장해버리면 이런 문제를 고려하지 않아도 된다.


## 기본적인 사용법

```java
List<Member> result = queryFactory
    .selectFrom(m) //SELECT * FROM member 
    .where(m.username.eq("youngjun")) //조건
    .fetch();
    
--------------------------------------------------
    
queryFactory.selectFrom(m)
    .where(
        m.age.gt(20),          // age > 20
        m.username.startsWith("y") // username y로 시작
    )
    .fetch();
    
--------------------------------------------------
BooleanBuilder builder = new BooleanBuilder();
//조건절에 builder를 넣고 if문을 통해 builder에 조건을 추가하며 조립
if (username != null) {
    builder.and(m.username.eq(username)); 
}
if (age != null) {
    builder.and(m.age.gt(age));
}

List<Member> result = queryFactory
    .selectFrom(m)
    .where(builder) 
    .fetch();
```

## OpenFeign QueryDSL

QeuryDSL 원본을 fork한 버전으로 원본에 비해 현재까지 유지보수가 이루어지고 있음

원본은 보안 취약점, 최신 기술 대응 미흡, spring이 fork 버전을 지원하기 시작 등으로 인해 사용

---

## 1. Fetch Join (JPQL)

Fetch Join을 통해 테이블을 JOIN 후 한 번의 쿼리로 조회가능

```java
@Query("SELECT m FROM Member m JOIN FETCH m.team")
List<Member> findAllWithTeam();
```

```sql
SELECT m.*, t.* 
FROM member m 
JOIN team t ON m.team_id = t.id;
```

위의 JPQL을 통해 아래의 SQL이 실행되고 아래 SQL을 살펴보면 member와 member가 속한 team 에 대한 테이블이 JOIN되고 데이터를 한 번에 가져오기 때문에 N+1 문제 해결

⚠️ 단점, 주의사항

- JOIN에서 중복 데이터 발생가능하기 때문에 페이징과 함께 사용 시 주의
- FETCH JOIN 너무 많아지면 데이터가 기하급수적으로 많아짐

## 2. @EntityGraph

JPQL에 FETCH JOIN 을 직접 명시하지 않고 EntityGraph Annotation을 이용해 선언적으로 FETCH JOIN

```java
@EntityGraph(attributePaths = {"team"})
@Query("SELECT m FROM Member m")
List<Member> findAllWithTeam();
```

⚠️ 단점, 주의사항

- JPQL을 직접 제어하는것 보다 유연성이 떨어질 수 있음

## 3. BatchFetching

한 번에 여러 개의 lazy target을 묶어서 가져오기

```java
@BatchSize(size = 100)
private List<Member> members;
```

⚠️ 단점, 주의사항

- 완전히 한 번의 쿼리는 아니다. BatchSize만큼 쿼리가 발생

## 4. DTO Projection

엔티티에서 필요한 컬럼만 담는 DTO로 결과를 바로 projection 하여 가져온다.

⚠️ 단점, 주의사항

- JPA가 변경 감지등의 관리를 하지 않음 → 조회해서 화면에 보여주는 상황 같을때만 사용

---
엔티티는 JPA에서 다음과 같은 생명주기를 가진다.

1. 비영속 (Transient)
2. 영속 (Persistent)
3. 준영속 (Detached)
4. 삭제 (Removed)

---

## 비영속 (Transient)

엔티티 객체가 단순히 new를 통해 생성된 상태

단순 객체 생성은 영속성 컨텍스트에 저장되지 않고 JPA와 전혀 관계없음

```java
Member member = new Member();
member.setName("영준");
```

- member는 영속성 컨테이너와 전혀 상관없는 순수 자바 객체 상태

---

## 영속 (Persistent)

아래 코드 실행 시 부터 JPA가 이 엔티티를 관리하고 영속성 컨텍스트에 저장됨

```java
em.persist(member);
```

- 생성한 객체에 대해 조회를 수행하면 영속성 컨텍스트에서 반환됨
- Dirty Checking 기능을 통해 해당 엔티티가 수정되면 수정되었다는 사실을 저장해두었다가 UPDATE를 개발자가 수행하지 않아도 자동으로 수행해줌
- 커밋 시 INSERT를 수행해줌

---

## 준영속 (Detached)

아래의 함수를 통해 영속성 컨텍스트에서 분리 할 수 있다.

혹은 트랙잭션이 끝나면 영속성 컨텍스트가 닫히게 되며 자동으로 준영속 상태가 된다.

```java
em.detach(member);
```

아래의 코드를 살펴보면 준영속 상태로 변경했기 때문에 수정되었다는 사실을 전혀 감지하지 못하고 아무일도 일어나지 않음 만약 detach 코드가 빠졌다면 setName으로 수정 시 수정사실을 감지하기 때문에 flush시 UPDATE 수행됨

```java
Member member = em.find(Member.class, 1L);
em.detach(member); // 준영속 상태로 변경

member.setName("수정됨");
em.flush(); // 아무 일도 안 일어남 (변경 감지 불가)
```

---

## 삭제 (Removed)

```java
em.remove(member);
```

위 함수를 통해 영속성 컨텍스트에서 해당 엔티티를 삭제할 수 있음.

삭제 이후 COMMIT 수행 시 DELETE SQL이 실행되어 DB에서 해당 엔티티가 삭제됨

---

## 영속상태를 왜 배웠을까?

엔티티가 영속 상태에 있어야 JPA가 관리를 한다.

- 1차 캐시에 저장되어 있기에 조회 시 성능 향상
- 변경을 자동으로 감지해서 UPDATE SQL을 수행해 개발자의 수고를 덜어줌
- 영속성 컨텍스트에서 정보를 관리하다 commit 시점에 한 번에 모아서 DB 반영이 가능해짐 (쓰기 지연으로 인한 성능 향상)