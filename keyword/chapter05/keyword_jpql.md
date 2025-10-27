## JPQL이란?

**JPQL (Java Persistence Query Language)** 은 JPA(Java Persistence API)에서 **엔티티 객체(Entity)** 를 대상으로 데이터를 조회하거나 조작하기 위해 사용하는 **객체 지향 쿼리 언어**이다.

SQL처럼 보이지만, **테이블이 아닌 엔티티와 그 필드 이름**을 기준으로 작성한다는 점이 가장 큰 특징이다.

JPA 구현체(예: Hibernate)가 이 JPQL을 실제 데이터베이스의 SQL로 변환하여 실행한다.

---

### `@Query` 어노테이션

`@Query`는 Spring Data JPA에서 **JPQL(또는 네이티브 SQL)을 직접 작성할 수 있게 해주는 어노테이션**이다. 리포지토리 인터페이스에 선언하여 복잡한 쿼리를 간단하게 지정할 수 있다.

예를 들어, 다음과 같이 사용할 수 있다

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    // JPQL을 이용한 쿼리
    @Query("SELECT m FROM Member m WHERE m.age >= :age")
    List<Member> findMembersOlderThan(@Param("age") int age);
}
```

위 예시에서 `"SELECT m FROM Member m WHERE m.age >= :age"` 부분이 **JPQL**이다.

여기서 `Member`는 DB의 테이블이 아니라 **엔티티 클래스 이름**, `m.age`는 **엔티티의 필드 이름**을 의미한다.

---

### 네이티브 쿼리와의 차이

`@Query`는 **JPQL**뿐만 아니라 **SQL(네이티브 쿼리)**도 사용할 수 있다.

단, 네이티브 쿼리를 사용할 때는 `nativeQuery = true` 옵션을 명시해야 한다.

```java
@Query(value = "SELECT * FROM member WHERE age >= :age", nativeQuery = true)
List<Member> findByNativeQuery(@Param("age") int age);
```

| 구분 | JPQL | 네이티브 SQL |
| --- | --- | --- |
| 대상 | 엔티티와 필드 | 테이블과 컬럼 |
| 의존성 | JPA 표준 | 특정 DB에 종속될 수 있음 |
| 변환 | JPA가 SQL로 변환 | 그대로 실행 |
| 장점 | 이식성과 유지보수성 높음 | 복잡한 쿼리 표현 가능 |