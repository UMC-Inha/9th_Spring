### 1️⃣ @DynamicInsert와 @DynamicUpdate가 어떻게 작동되는 지 파악하기

## 기본 JPA/Hibernate가 쿼리를 만드는 방식

Hibernate는 보통 **엔티티당 INSERT/UPDATE SQL을 한 번 생성해서 재사용**한다.

예를 들어 다음과 같은 엔티티가 있다고 가정한다.

```java
@Entity
@Table(name = "member")
public class Member {

    @Id @GeneratedValue
    private Long id;

    private String name;          // nullable
    private Integer age;          // nullable
    private String address;       // nullable

    @Column(name = "created_at")
    private LocalDateTime createdAt;  // DB default CURRENT_TIMESTAMP 가정
}

```

### 기본 INSERT (dynamicInsert = false, 기본값)

Hibernate는 보통 이런 식의 정적인 SQL을 만든다.

```sql
insert into member (name, age, address, created_at, id)
values (?, ?, ?, ?, ?)
```

모든 컬럼에 대해 **값을 바인딩**한다. 값이 `null`이면 그냥 `null`을 바인딩한다.

이 말은, DB 컬럼에 `DEFAULT CURRENT_TIMESTAMP`가 있어도,

`created_at`에 `null`을 보내버리면 → **DB 기본값이 적용되지 않고 null로 들어갈 수 있다**는 뜻이다.  
  
---
### 기본 UPDATE (dynamicUpdate = false, 기본값)
Hibernate는 보통 UPDATE도 “고정된 SET 목록”을 쓴다.

```sql
update member
   set name = ?,
       age = ?,
       address = ?,
       created_at = ?
 where id = ?
```

- 필드가 바뀌었든 안 바뀌었든, **매번 모든 컬럼을 SET에 포함**한다.
- optimistic locking을 쓰면 `where id = ? and version = ?` 같은 식으로 붙는다.
- 더미로 값을 다시 쓰더라도 DB 입장에서는 “row update”로 처리된다.

Hibernate는 내부적으로 원본 스냅샷을 가지고 **어떤 필드가 더러워졌는지(dirty) 체크**는 하지만,

기본 설정에서는 “더러워졌든 말든 일단 전체 컬럼을 SET에 넣는 쿼리”를 사용한다.

## @DynamicInsert 동작 방식

```java
@Entity
@DynamicInsert
public class Member { ... }
```

이 애너테이션을 붙이면, **INSER**T 시, 값이 **null**인 컬럼은 INSERT 문에서 아예 **제외한다.**

예를 들어, 아래와 같이 엔티티를 저장한다고 하자.

```java
Member m = new Member();
m.setName("홍길동");
m.setAge(null);          // 설정 안 함 (null)
m.setAddress(null);      // 설정 안 함 (null)
m.setCreatedAt(null);    // 설정 안 함 (null)

em.persist(m);
```

### @DynamicInsert 적용 시 INSERT 예시

Hibernate가 이런 SQL을 만든다고 보면 된다.

```sql
insert into member (name)
values (?)
```

`age`, `address`, `created_at` 컬럼은 **INSERT에서 제외**된다.

이 경우, DB의 `DEFAULT`가 설정되어 있으면 그 값이 적용된다.

(예: `created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP`)

즉, **“null을 보내지 않음으로써 DB 기본값/트리거를 살리는 전략”** 이다.  

단점은, 엔티티마다 상황에 따라 컬럼 조합이 달라질 수 있으므로,

Hibernate가 **하나의 정적 SQL을 재사용하지 못하고, 상황에 따라 동적으로 SQL을 조합**하게 된다.

동적으로 조합하기 때문에 DB, Hibernate 양쪽 SQL 캐시 효율이 떨어질 수 있다.

## @DynamicUpdate 동작 방식

```java
@Entity
@DynamicUpdate
public class Member { ... }
```

이 애너테이션을 붙이면,

> UPDATE 시, 실제로 변경된(Dirty) 필드만 SET 절에 포함한다.
> 

예를 들어, DB에 이미 저장된 Member가 있다고 하자.

```java
Member m = em.find(Member.class, 1L);
m.setName("임꺽정");    // name만 변경
// age, address, createdAt은 그대로
```

### **기본(미적용) UPDATE**

```sql
update member
   set name = ?,
       age = ?,
       address = ?,
       created_at = ?
 where id = ?
```

**@DynamicUpdate 적용 시 UPDATE 예시**

```sql
update member
   set name = ?
 where id = ?
```

- 실제로 바뀐 `name`만 포함한다.
- version 필드가 있으면 보통 `where id = ? and version = ?` 형태로 같이 들어간다.

이 역시 **상황에 따라 SET에 포함되는 컬럼 조합이 달라지므로, SQL이 정적이지 않고 동적**이 된다.

즉, 조합마다 다른 SQL이 만들어져 **DB의 statement 캐시 / Hibernate 쿼리 캐시에 “다른 쿼리”로 인식**된다는 점이 있다.

---

