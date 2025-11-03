## 즉시 로딩(Immediate Loading)이란?

<aside>
💡

엔티티를 조회할 때 해당 엔티티와 연관된 모든 엔티티를 동시에 조회하는 방식

</aside>

- 객체 간의 관계를 활용하기 편리하다.
- 조인을 사용하여 복잡한 연관 관계를 해결할 필요 없이, 즉시 로딩으로 한번에 데이터를 가지고 올 수 있다.
- 불필요한 연관된 엔티티도 불러와야 하기 때문에 N+1 문제가 계속해서 발생한다.
- `@XXToOne`연관 관계의 디폴트 Fetch Type이다.
- `@XXToMany`연관 관계일 경우 `fetch = FetchType.EAGER` 설정을 넣어주면 즉시 로딩 방법을 사용할 수 있다.

## 지연 로딩(Lazy Loading)이란?

<aside>
💡

연관된 엔티티를 처음에 조회하지 않고, 실제로 해당 엔티티가 필요한 시점에 조회하는 방식

</aside>

- 불필요한 쿼리 발생을 방지할 수 있다.
- 연관된 엔티티가 지연 로딩으로 호출될 경우, 프록시 객체를 불러온다.
- 실제로 엔티티가 필요한 시점에 실제 객체로 바뀌어서 호출된다.
- `@XXToMany`연관 관계의 디폴트 Fetch Type이다.
- `@XXToOne`연관 관계일 경우 `fetch = FetchType.LAZY` 설정을 넣어주면 지연 로딩 방법을 사용할 수 있다.

### 즉시 로딩 vs 지연 로딩

- 즉시 로딩과 지연 로딩 두 fetch Type 모두 N + 1 문제를 피하기 힘들다.
    
    → 보통 실무에서는 지연 로딩을 필수적으로 사용하고, N + 1 문제가 발생할 가능성이 있는 경우에는 fetch join 등 다양한 방법으로 미리 예방한다.
---

# JPQL(Java Persistence Query Language)란?

<aside>
💡

데이터 기반이 아닌 엔티티 객체를 기반으로 조회하는 객체 지향 쿼리

</aside>

- SQL을 추상화하여 정의되어 있기 때문에, 데이터 베이스에 의존하지 않는다.
- 데이터 베이스 ‘테이블과 컬럼’이 아닌 ‘자바 클래스와 변수(객체)’에 작업을 수행한다.
- JPQL은 결국 SQL로 변환된다!!

## JPQL의 쿼리 반환 타입

### TypedQuery

- 반환되는 타입이 명확할 때 사용하는 클래스이다.
- 변환 타입을 미리 지정하기 때문에 컴파일 시점에 오류를 잡을 수 있어 안정적이다.

```java
EntityManager em;

TypedQuery(Member) typedQuery = em.createQuery("select m from Member as m where m.id = ?", Member.class);
```

### Query

- 반환되는 타입이 명확하지 않을 때 사용하는 클래스이다.
- 다양한 타입의 결과로 반환 받을 수 있지만, 반환 타입을 런타임 시점에 확인할 수 있어 안정성이 떨어진다.

```java
EntityManager em;

Query query = em.createQuery("select m from Member as m where m.id = ?");
```

## JPQL 파라미터 방식

- JPQL 쿼리를 처리할 때 파라미터에 따라 동적으로 처리하는 방식

### 이름 기준 파라미터 바인딩

- 쿼리 내에서 “:변수명” 형태로 선언한 후에 setParameter메서드를 사용하여 값을 설정하는 방식이다.

```java
EntityManager em;

TypedQuery(Member) typedQuery = em
				.createQuery("select m from Member as m where m.id = :id and m.name = :name", Member.class);
				.setParameter("id", 1)
				.setParameter("name", seoyeon);
```

### 위치 기준 파라미터 바인딩

- 쿼리 내에 파라미터를 “?번호”형태로 선언한 후에 setParameter메서드를 사용하여 값을 설정하는 방식이다.
- 이때, 위치 번호는 1부터 시작한다.

```java
EntityManager em;

TypedQuery(Member) typedQuery = em
				.createQuery("select m from Member as m where m.id = ?1 and m.name = ?2", Member.class);
				.setParameter(1, 1)
				.setParameter(2, seoyeon);
```

## JPQL 쿼리 결과 조회 방식

- 쿼리를 수행한 후 결과를 조회하는 방식

### getSingleResult()

- 이 함수는 쿼리의 결과로 ‘단일 엔티티 객체’를 반환한다.
- 쿼리의 결과가 없을 때 → `NoResultException`
- 쿼리의 결과가 2개 이상일 때 → `NonUniqueResultException`

```java
Member member = em
				.createQuery("select m from Member as m where m.id = ?1 and m.name = ?2", Member.class);
				.setParameter(1, 1)
				.setParameter(2, seoyeon)
				.getSingleResult();
```

### getResultList()

- 이 함수는 쿼리의 결과로 ‘List 형태의 객체’로 반환한다.
- 쿼리 결과가 없는 경우 빈 리스트를 반환한다.

```java
List<Member> members = em
				.createQuery("select m from Member as m where m.id = ?1 and m.name = ?2", Member.class);
				.setParameter(1, 1)
				.setParameter(2, seoyeon)
				.getResultList();
```

**‼️ getSingleResult()의 결과가 없을 경우 예외를 피하기 위해 getResultList()를 사용할 수 있다.**

- List로 반환한 다음, 리스트의 첫 번째 값을 가져온다.

```java
List<Member> members = em
				.createQuery("select m from Member as m where m.id = ?1 and m.name = ?2", Member.class);
				.setParameter(1, 1)
				.setParameter(2, seoyeon)
				.getResultList();
				
Member member = members.isEmpty() ? null : member.get(0);
```

## JPQL의 한계

### 1. 동적 쿼리의 사용이 어렵다.

- JPQL의 경우 컴파일 시점에 쿼리가 결정되기 때문에 사용자의 입력 값에 따라 동적으로 쿼리를 변경하기 어렵다.
- 동적 쿼리가 필요한 경우에는 Criteria API나 QueryDSL과 같은 도구를 사용하는 것이 더 효율적이다.

### 2. 문자열 오류 확인이 어렵다.

- 쿼리를 문자열로 작성하기 때문에, 문자열 내부에 있는 쿼리의 문법 오류나 오타를 컴파일 시점에 체크할 수 없다.
- 따라서, 잘못된 쿼리를 입력하면 실행 시점에 예상하지 못한 오류가 발생할 수 있다.

### 3. SQL의 모든 기능을 사용할 수 없다.

- SQL을 추상화하여 데이터 베이스에 의존하지 않는 대신, 특정 데이터 베이스에 특화된 SQL문법을 사용할 수 없다.
- 특화된 문법을 사용하고 싶은 경우에는 native query를 사용해야 한다.

### 4. 복잡한 Join을 이용한 쿼리를 작성하는 데에 제약이 있다.

- JPQL에서 서브 쿼리는 Where절 내에서만 부분적으로 허용하기 때문에, 복잡한 Join 조건을 사용할 수 없다.
- 필요할 경우에는 native query를 사용해야 한다.

## Fetch Join이란?

- JPQL에서 성능 최적화를 위해 제공하는 Join의 종류이다.
- 연관된 엔티티나 컬렉션을 SQL 한 번에 함께 조회하는 기능이다.

<aside>
💡

[ LEFT [ OUTER ] | INNER ] JOIN FETCH 

</aside>

```java
- JPQL
	Select m From Member m Join Fetch m.team
	
- SQL 
	Select m.*, t.* From Member m Inner Join Team t On m.team_id = t.id
```

## Fetch Join을 왜 사용할까?

- 보통 엔티티 간의 연관 관계는 fetch = FetchType.Lazy로 설정해 지연 로딩 방법을 사용한다.
- 지연 로딩 방법을 사용하면서 연관된 엔티티를 조회하려고 할 때, 일반 Join을 사용하면 추가적인 쿼리가 많이 발생하게 돼서 성능이 저하된다.
- 일반 Join을 사용하면 엔티티를 조회할 때, 연관된 엔티티는 프록시 객체로 가져오고, 연관된 엔티티를 조회하기 위해서는 쿼리가 한 번 더 필요하다.

<aside>
💡

Fetch Join은 연관 관계의 엔티티나 컬렉션을 프록시가 아닌 진짜 데이터를 한번에 가져와 같이 조회하는 기능이다. 

</aside>

# @EntityGraph란?

- 엔티티의 연관 관계를 지연 로딩으로  설정하면, 종속된 엔티티는 프록시 객체로 가져오게 된다.
- 이후에 해당 프록시 객체를 호출할 때마다 객체 마다 select 쿼리가 실행된다.
- 이런 문제를 JPQL의 fetch join으로 해결할 수 있다.

<aside>
💡

@EntityGraph 어노테이션은 fetch join을 자동으로 적용해 편리하게 사용할 수 있는 기능을 제공한다.

</aside>

## @EntityGraph 적용 방법

### 함수 오버라이딩하여 적용

```java
@Override
@EntityGraph(attributePaths = {"team"})
List<Member> findAll();
```

- findAll() 함수는 Repository에서 제공하기 때문에  오버라이딩해서 Fetch Join을 적용한다.

### JPQL과 함께 사용

```java
@Query("select m from Member m")
@EntityGraph(attributePaths = {"team"})
List<Member> findAllEntityGraph();
```

- 사용자 정의 JPQL을 함께 사용하여 Fetch Join을 적용한다.

### 메소드이름 쿼리와 함께 사용

```java
@EntityGraph(attributePaths = {"team"})
List<Member> findEntityGraphByUsername(@Param("username") String username);
```

- 메서드 이름으로 작성한 쿼리와 함께 사용하여 Fetch Join을 적용한다.

# flush()❓

- 영속성 컨텍스트의 변경 내용을 데이터베이스에 반영하는 작업을 한다.
- SQL 쿼리는 DB로 전송되지만, 해당 transaction이 commit 되지 않으면 최종 반영은 되지 않는다.
    - 즉, 변경된 사항은 DB에 영구적으로 저장되지 않은 상태임.

### 왜 flush()가 필요할까?

- JPA는 변경사항에 대해 실시간을 즉시 DB에 반영하지 않고 최적화된 SQL을 한번에 전송하기 위해서 쓰기 지연 매커니즘을 사용한다.
    - update를 시도하면 바로 DB에 SQL문을 전송하는 것이 아니라, 영속성 컨텍스트에만 반영된다.
    - 영속성 컨텍스트의 1차 캐시와 데이터 베이스 상태를 비교해 쿼리를 SQL 저장소에 저장하는 변경 감지(Dirty Checking)가 수행된다.
- flush() 가 실행되면, 영속성 컨텍스트에 저장되어 있는 SQL문을 DB로 전송한다.

# commit()❓

- 현재 transaction을 완료하고 모든 변경 사항을 확정하는 역할을 한다.
- 내부적으로 flush()를 수행한 후, transaction을 커밋한다. → 이때, 데이터 베이스에 영구적으로 저장!
    
    → commit() 은 flush() 의 역할과 트랜잭션을 확정하는 역할을 한다고 생각하면 된다.

# QueryDSL이란?

- 다양한 데이터베이스 플랫폼에 접근하여 SQL과 유사한 문법으로 쿼리를 작성하여 데이터 처리를 수행하는데 도움을 주는 프레임워크이다.
- 데이터의 타입을 체크하여 오류를 예방할 수 있는 타입-세이프를 보장한다.
- SQL 형태가 아닌 자바 코드로 작성하며 매서드 체이닝을 이용해 복잡한 쿼리 작성을 쉽게 만들어준다.

## QueryDSL 특징

### 타입 안전성 보장

- 쿼리 작성 시 컴파일 시점에 쿼리 오류를 검출할 수 있다.
- 컴파일러를 통해 쿼리 문법 오류, 필드 이름 오타, 매서드 이름 오류 등을 컴파일 단계에서 발견하여 런타임에서 발생할 오류를 방지할 수 있다.
- JPQL의 경우 문자열 타입으로 쿼리를 정의하기 때문에, 컴파일 단계에서 쿼리의 문제를 확인할 수 없기 때문에 QueryDSL이 좋은 대안이 될 수 있다.

### 동적 쿼리 지원

- 프로그램 실행 중에 조건을 변경할 수 있는 동적 쿼리를 지원한다.

### 자동완성 기능 제공

- 쿼리를 작성하는 동안에 IDE의 자동완성 기능을 통해서 쿼리를 쉽게 작성할 수 있다.

## QueryDSL의 보안 취약점

- 2024년 QueryDSL 라이브러리에서 보안 취약점이 발견되었다.
- JPAQuery의 orderBy 매서드에서 사용자 입력을 적절히 검증하지 않아 사용자가 악의적으로 SQL 코드를 포함한 문자열을 삽입하면, QueryDSL은 해당 문자열을 코드 생성에 사용하게 된다.
- 이런 코드가 런타임에서 QueryDSL 동적 쿼리 생성 과정에 포함될 경우 데이터 베이스의 민감한 정보에 접근할 수도 있고, 복잡한 쿼리를 주입하여 시스템의 리소스를 과도하게 사용하게 만들 수도 있다.
- 따라서 현재 QueryDSL 프로젝트는 공식적으로 개발 중단을 선언하지는 않았으나, 2024년 10월 발표한 버전을 마지막으로 현재까지 새로운 릴리스가 이루어지지 않고 있다.

## OpenFeign팀의 QueryDSL 버전 업데이트 지원

- 대부분의 기업 프로젝트에서는 이미 구축된 QueryDSL 기반 코드베이스를 유지하려고 했기 때문에, Netflix의 OpenFeign팀은 QueryDSL 프로젝트를 포크하여 기존 QueryDSL과의 하위 호완성을 유지하면서도 여러 개선 작업을 시작했다.
- 이전에 문제가 되었던 보안 문제도 OpenFeign팀이 수정함으로써 기존 사용자들은 코드베이스를 크게 변경하지 않고도 보안 문제를 해결할 수 있었다!!
- 결과적으로 OpenFeign팀의 QueryDSL 프로젝트가 기존 QueryDSL사용자들에게 최신 환경과 보안 요구사항을 충족하는 실질적인 대안으로 자리잡게 되었다.

# N + 1 문제 해결 방법

## 🧩 outer join fetching

1. fetch join : JPQL문법인 fetch join을 사용하자!!

```sql
-- INNER JOIN
select t from Team t join fetch t.members

-- OUTER JOIN
select t from Team t left join fetch t.members
```

```java
public interface TeamRepository extends JpaRepository<Team, Long> {
	
	@Query("select t from Team t join fetch t.members")
	List<Team> findAllWithInnerFetchJoin();
	
	@Query("select t from Team t left join fetch t.members")
	List<Team> findAllWithOuterFetchJoin();
	
}
```

- Spring Data JPA를 사용하면, `@Query`를 이용해서 JPQL을 직접 사용할 수 있습니다.
    
    → 팀을 불러올 때, 대응하는 멤버 엔티티도 영속성 컨텍스트 1차 캐시에 함께 저장하기 때문에 다시 지연 로딩으로 연결된 객체(멤버)를 불러올 때 조회 쿼리가 수행되지 않는다.
    
- `@EntityGraph` 어노테이션을 이용해 Fetch Join을 적용하는 방법도 있다.

## 🧩 batch & subselect fetching

- 두 방법 모두 N+1 문제에서 발생하는 추가 조회 쿼리를 없애는 방향이 아니라, 추가 조회 쿼리를 1개의 쿼리로 줄이는 방향으로 문제를 해결한다..

### batch fetching

- 추가 쿼리로 조회 되는 Entity위에 @BatchSize를 추가하면 Entity가 조회될 때 IN절을 통해 한번에 조회할 수 있다.

```java
@Entity
public class Team {
		...
		
		@OneToMany(mappedBy = "team", fetch = FetchType.LAZY)
		@BatchSize(size = 3)
		private List<Member> members = new ArrayList<>();
		
		...
		
}
```

- size를 3으로 설정했기 때문에, Member들을 조회할 때 where절 안에 in (team1, team2, team3)이 들어가 세 개의 팀의 멤버들을 한번에 조회할 수 있다.
- BatchSize를 전역적으로 구성파일에서도 지정 가능하다.

### subselect fetching

- `@Fetch(value = FetchMode.SUBSELECT)` 을 사용하면, IN절과 서브쿼리를 사용하여 한번에 조회할 수 있다.

```java
@Entity
public class Team {
		...
		
		@OneToMany(mappedBy = "team", fetch = FetchType.LAZY)
		@Fetch(value = FetchMode.SUBSELECT)
		private List<Member> members = new ArrayList<>();
		
		...
		
}
```

## 엔티티의 생명주기(LifeCycle)

### <1> 비영속 상태

- JPA와 전혀 관계가 없이 단순히 객체를 생성한 상태를 의미한다.

```java
Member member = new Member();
member.setId("member1");
mamber.setName("seoyeon");
```

### <2> 영속 상태

- 객체가 영속성 컨텍스트에 저장된 상태를 의미한다
- `persist()` 를 통해 엔티티를 영속 상태로 만들 수 있다.
- `em.find()` 또는 JPQL로 조회한 엔티티도 영속 상태가 된다.

### <3> 준영속 상태

- 영속성 컨텍스트에 저장되었다가 분리된 상태를 의미한다.
- `em.detach()` 를 통해 엔티티를 영속성 컨텍스트에서 분리한다.
- `em.clear()` 를 통해 영속성 컨텍스트를 완전히 초기화해 객체를 준영속 상태로 만든다.
- `em.close()` 를 통해 영속성 컨텍스트를 종료 시킬 수도 있다.

### <4> 삭제 상태

- 영속성 컨텍스트와 데이터 베이스에서 삭제된 상태를 의미한다.
- `em.remove()`를 통해 객체를 데이터 베이스와 영속성 컨텍스트에서 삭제한다.