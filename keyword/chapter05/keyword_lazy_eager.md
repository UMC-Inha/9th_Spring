### 1. 지연 로딩 (Lazy Loading)

불필요한 데이터 조회를 줄여 성능을 높이기 위해서 연관된 엔티티(다른 테이블의 데이터)를 **즉시 가져오지 않고**, **실제로 그 데이터가 필요할 때 가져오는 방식**이다.

연관 객체를 프록시(proxy) 형태로 두었다가, 해당 객체의 필드나 메서드에 접근하는 순간에 DB 쿼리를 실행한다.


**예시**

```java
@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;

    private String name;

    @ManyToOne(fetch = FetchType.LAZY)  // 지연 로딩
    private Team team;
}

```

```java
Member member = em.find(Member.class, 1L);
System.out.println(member.getName());      // Team 조회 안 함
System.out.println(member.getTeam().getName()); // 이 시점에 Team 조회 쿼리 실행
```

1. `select * from member where id=1`
2. `member.getTeam()` 호출 시 → `select * from team where id=?` 실행

---

### 2. 즉시 로딩 (Eager Loading)

- 연관된 엔티티를 **주 엔티티를 조회할 때 함께 가져오는 방식**이다.
- JPA가 내부적으로 **JOIN 쿼리**를 사용해 한 번에 데이터를 불러온다.
- **목적:** 한 번의 쿼리로 모든 연관 데이터를 미리 조회.

**예시**

```java
@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;

    private String name;

    @ManyToOne(fetch = FetchType.EAGER)  // 즉시 로딩
    private Team team;
}
```

```java
Member member = em.find(Member.class, 1L);
System.out.println(member.getName());
System.out.println(member.getTeam().getName()); // 이미 함께 로딩됨
```

`select m.*, t.* from member m join team t on m.team_id = t.id`

즉시로딩은 필요한 데이터와 필요 없는 데이터를 구분하지 않고 **모두 가져오기 때문에**, 불필요한 조회가 생기면 성능이 떨어질 수 있다.