# **JPA 기본 키생성 전략**

---

JPA를 사용하여 엔티티를 관리할 때, 모든 엔티티는 고유한 식별자인 기본 키(Primary Key)를 가져야 합니다. JPA는 이러한 기본 키를 생성하는 다양한 방법을 제공하며, 이를 **키 생성 전략**이라고 합니다. 이 전략은 크게 **직접 할당**과 **자동 생성** 두 가지 방식으로 나뉩니다.

# 1. 직접 할당 (Assigned)

**직접 할당**은 개발자가 애플리케이션 코드 내에서 직접 기본 키 값을 설정하는 방식입니다. `@Id` 어노테이션만 사용하며, `@GeneratedValue`는 붙이지 않습니다.

```java
@Entity
public class Member {

    @Id
    private String email;

    // ...
}
```

이 전략은 `persist()`를 호출하여 엔티티를 영속화하기 전에 반드시 ID 값을 할당해야 합니다. 주로 회원의 이메일이나 학번처럼 비즈니스적으로 의미가 있는 **자연 키(Natural Key)**를 기본 키로 사용할 때 유용합니다.

---

# 2. 자동 생성 (GeneratedValue)

**자동 생성**은 JPA가 데이터베이스와 연동하여 키 값을 자동으로 생성하는 방식입니다. `@GeneratedValue` 어노테이션을 사용하여 전략을 지정할 수 있으며, 데이터베이스의 종류와 특성에 따라 네 가지 세부 전략 중 하나를 선택하게 됩니다.

## (1) IDENTITY 전략

**IDENTITY 전략**은 기본 키 생성을 전적으로 데이터베이스에 위임하는 방식입니다. 주로 MySQL의 `AUTO_INCREMENT`나 PostgreSQL의 `SERIAL`과 같이, 데이터베이스가 자체적으로 제공하는 자동 증가 기능을 사용합니다.

엔티티가 데이터베이스에 `INSERT`된 후에야 기본 키 값을 알 수 있습니다. 따라서 JPA는 `persist()` 시점에 `INSERT` 쿼리를 즉시 실행하고, 반환된 키 값을 엔티티에 할당합니다. 이로 인해 쓰기 지연(Transactional Write-Behind)이 동작하지 않습니다.

```java
@Entity
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    // ...
}
```

## (2) SEQUENCE 전략

**SEQUENCE 전략**은 데이터베이스의 **시퀀스 객체(Sequence Object)**를 사용하여 기본 키를 생성합니다. Oracle, PostgreSQL, H2 등 시퀀스를 지원하는 데이터베이스에서 주로 사용되는 고성능 전략입니다.

`persist()` 호출 시, `INSERT` 쿼리를 실행하기 전에 데이터베이스 시퀀스를 먼저 호출하여 PK 값을 조회해옵니다. 가져온 PK 값을 엔티티에 할당한 후, 해당 엔티티를 영속성 컨텍스트에 저장합니다. 따라서 `INSERT` 쿼리는 트랜잭션 커밋 시점에 실행되는 쓰기 지연이 가능합니다.

`allocationSize` 옵션을 통해 여러 PK 값을 미리 확보하여 성능을 최적화할 수 있습니다.

- **시퀀스 객체란..?**
    
    간단히 말해서 **Sequence 객체**는 데이터베이스에서 **숫자를 자동으로 생성해주는 객체**를 말합니다.
    
    말 그대로 **객체**이기 때문에 따로 테이블이 존재하지 않으며, **하나의 Sequence를 여러 테이블에서 함께 사용할 수도 있습니다. MySQL**은 지원하지 않습니다!
    
    ```sql
    CREATE SEQUENCE member_seq
    START WITH 1
    INCREMENT BY 1;
    ```
    
    ```sql
    INSERT INTO member (id, name)
    VALUES (member_seq.NEXTVAL, '홍길동');
    ```
    
    - `NEXTVAL` : 다음 숫자를 생성함 (예: 1 → 2 → 3 …)
    - `CURRVAL` : 현재 숫자를 반환함 (마지막으로 생성된 값)
    
    ```sql
    SELECT member_seq.CURRVAL FROM dual;
    ```
    

```java
@Entity
@SequenceGenerator(
    name = "USER_SEQ_GENERATOR",
    sequenceName = "USER_SEQ", // 매핑할 데이터베이스 시퀀스 이름
    initialValue = 1, 
    allocationSize = 1
)
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "USER_SEQ_GENERATOR")
    private Long id;

    // ...
}
```

### IDENTITY, Sequence 차이점이 뭐지..?

엔티티를 **영속성 컨텍스트에 저장하는 행위**를 `persist()`라고 합니다. 이때 사용하는 기본 키 생성 전략에 따라 동작 방식이 달라집니다.

**Identity 전략**은 **데이터베이스가 기본 키를 직접 생성**하기 때문에, `INSERT` 쿼리가 실행되어야만 기본 키 값을 얻을 수 있습니다. 따라서 `persist()`를 호출하면 **즉시 `INSERT` 쿼리가 실행**되어야 하므로 **쓰기 지연**이 적용되지 않습니다.

반면 **Sequence 전략**은 **DB의 시퀀스 객체를 통해 미리 번호를 생성**할 수 있기 때문에, `INSERT`를 실행하기 전에 기본 키를 이미 알고 있습니다. 따라서 `persist()` 시점에는 엔티티를 영속성 컨텍스트에 등록만 해두고, 실제 `INSERT` 쿼리는 **트랜잭션 커밋 시점까지 지연(쓰기 지연)**할 수 있습니다.

즉, 두 전략 모두 `persist()`를 통해 영속성 컨텍스트에 저장되지만, **기본 키를 언제 확보하느냐**에 따라 쓰기 지연의 동작 여부가 달라지는 것입니다.

## (3) TABLE 전략

**TABLE 전략**은 기본 키 생성을 위한 별도의 테이블을 사용하는 방식입니다. 마치 데이터베이스의 시퀀스 객체를 테이블로 흉내 내는 것과 같습니다.

이 전략은 모든 데이터베이스에서 사용할 수 있어 호환성이 높습니다. 하지만 키를 생성할 때마다 테이블을 조회하고 수정하는 과정에서 락(Lock)이 발생하므로, 다른 전략에 비해 성능 저하가 발생할 수 있습니다. 따라서 운영 환경에서의 사용은 권장되지 않습니다.

```java
@Entity
@TableGenerator(
    name = "ID_GENERATOR",
    table = "ID_SEQUENCES",
    pkColumnName = "SEQUENCE_NAME",
    valueColumnName = "NEXT_VALUE",
    allocationSize = 1
)
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.TABLE, generator = "ID_GENERATOR")
    private Long id;

    // ...
}
```

## (4) AUTO 전략

**AUTO 전략**은 JPA가 사용하는 데이터베이스의 **방언(Dialect)**에 따라 위 세 가지 전략 중 가장 적절한 것을 자동으로 선택하는 기본값입니다.

- **MySQL, MariaDB:** `IDENTITY`
- **Oracle, PostgreSQL:** `SEQUENCE`
- **H2:** 데이터베이스 설정에 따라 다름

개발 초기 단계나 특정 데이터베이스에 종속되지 않는 애플리케이션을 개발할 때 편리하게 사용할 수 있습니다. 하지만 장기적으로는 데이터베이스 종류가 명확하다면 해당 DB에 최적화된 `IDENTITY`나 `SEQUENCE` 전략을 명시적으로 지정하는 것이 좋습니다.