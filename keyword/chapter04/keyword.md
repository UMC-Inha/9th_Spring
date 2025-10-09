### **계층형 구조**

계층형 구조는 애플리케이션의 역할을 기준으로 코드를 구분하는 방식이다.

즉, **요청을 처리하는 Controller**, **비즈니스 로직을 담당하는 Service**, **데이터 접근을 담당하는 Repository** 등 기능별로 분리한다.

```json
com.example.myapp
├── controller
├── service
├── repository
├── entity
├── dto
├── config
└── exception
```

이 방식은 프로젝트의 흐름이 한눈에 들어와서 초기 진입 장벽이 낮고, 구조를 이해하기 쉽다.

- **장점:** 각 계층의 책임이 명확하고, 전체 구조를 빠르게 파악할 수 있다.
- **단점:** 같은 계층에 여러 도메인의 코드가 섞이기 쉬워, 규모가 커질수록 유지보수가 어려워진다.

### 도메인형 구조

도메인형 구조는 **비즈니스 단위**를 중심으로 코드를 구성하는 방식이다. 즉, ‘무엇을 하는 코드인가’(도메인)를 기준으로 관련된 컴포넌트를 묶는다.

예를 들어, 사용자(User) 관련 기능은 user 패키지에, 상품(Product) 관련 기능은  product 패키지 안에 전부 포함한다.

```json
com.projectname
├── member
│   ├── controller
│   │   └── MemberController.java
│   ├── service
│   │   └── MemberService.java
│   ├── repository
│   │   └── MemberRepository.java
│   ├── entity
│   │   └── Member.java
│   ├── dto
│   │   └── MemberResponse.java
│   └── exception
│       └── MemberNotFoundException.java
├── product
│   ├── controller
│   ├── service
│   ├── repository
│   ├── entity
│   └── dto
└── config
    └── AppConfig.java

```

이 구조에서는 각 도메인이 **독립된 작은 모듈처럼 관리**되기 때문에 서비스가 커져도 유지보수가 쉽고, 기능 단위로 분리하기도 편하다.

- **장점:** 비즈니스 로직 단위로 응집도가 높고, 다른 도메인에 영향을 덜 받는다.

  도메인 간 결합이 줄어들어 코드 재사용성과 확장성이 좋아진다.

- **단점:** 프로젝트 규모나 도메인 관계를 잘 모르면 전체 구조를 이해하기 어렵다

**실무에선 이렇게 혼합 구조로 가는 경우가 많다.**

```json
com.example.projectname
└── domain
    ├── user
    ├── product
    └── common
└── global
    ├── config
    ├── exception
    ├── util

```

도메인 중심 구조에 전역 설정이나 공통 모듈만 따로 분리하는 패턴이다.

실무에서는 도메인형 구조를 기반으로 하되, 전역 설정이나 공통 모듈(config, exception, util등)은 별도의 global 패키지로 분리해 관리하는 혼합형 구조를 많이 사용한다.
## JPA

→ 자바 객체(클래스)를 데이터베이스 테이블과 매핑해주는 ORM(Object-Relational Mapping) 표준 명세

즉, SQL문을 직접 쓰지 않아도 자바 객체를 그대로 DB 테이블에 저장하거나 불러올 수 있게 해주는 기술이다.

원래는 DB에 데이터를 넣을 때 이렇게 직접 SQL을 써야했다. 👇

```java
String sql = "INSERT INTO member (name, age) VALUES (?, ?)";
PreparedStatement ps = connection.prepareStatement(sql);
ps.setString(1, member.getName());
ps.setInt(2, member.getAge());
ps.executeUpdate();
```

근데 JPA를 쓰면 이렇게 간단하게 바뀐다. 👇

```java
memberRepository.save(member);
```

그럼 JPA가 내부적으로 위의 SQL을 자동으로 만들어서 실행해준다**.**

**즉, 개발자는 객체만 다루고, JPA가 SQL을 대신 작성해주는 구조다.**

### ORM(Object-Relational Mapping)이란?

ORM이란 객체(Object)와 관계형 데이터베이스(Relational DB)를 자동으로 매핑하는 기술이다.

즉,

- **클래스 ↔ 테이블**
- **필드 ↔ 컬럼**
- **객체 간 관계 ↔ 외래 키(FK)**

이런 대응 관계를 알아서 매핑해준다.

예를들어 👇

```java
@Entity
@Table(name = "member")
public class Member {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private int age;
}
```

이 클래스 하나만 정의해두면 JPA가 자동으로 다음 SQL을 만들어 실행해준다. 👇

```java
 CREATE TABLE member (
  id BIGINT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(255),
  age INT
);
```

**JPA는 표준 “명세”일 뿐 → 실제 동작은 구현체가 한다.**

JPA 자체는 “규칙”만 정의해두고, 실제로 동작하는 건 구현체들이다. Hibernate가 JPA의 대표적인 구현체이다. (다른 구현체들: EclipseLink, OpenJPA)

```java
ORM (개념)
 └── JPA (자바 진영의 ORM 표준)
      └── Hibernate (JPA를 실제로 구현한 대표 라이브러리)
```

**ORM**: 객체 ↔ 테이블 매핑이라는 “아이디어, 개념”

**JPA**: 그 개념을 “자바에서 표준화”한 인터페이스(명세)

**Hibernate**: 그 표준을 “실제로 동작하게 구현”한 도구

### 비유!

“ORM”은 ‘비행기’라는 개념, JPA는 ‘비행기 조종 매뉴얼(표준)이고, “Hibernate”는 그 매뉴얼을 따라 만들어진 실제 ‘비행기’임.

즉, 다른 회사(EclipseLink, OpenJPA)도 자기들만의 비행기를 만들 수 있는데, JPA 규격을 따르면 다 같은 방식으로 조종할 수 있음!

### 예시

- **ORM 개념**

  → 객체와 테이블을 매핑하자!

- **JPA 코드(표준 인터페이스)**

```java
  public interface EntityManager {
    void persist(Object entity);
    <T> T find(Class<T> entityClass, Object primaryKey);
}
```

- **Hibernate 구현체(JPA 동작 담당)**

```java
// 내부적으로 실제 SQL 생성 및 실행 로직 구현
public class HibernateEntityManager implements EntityManager {
    @Override
    public void persist(Object entity) {
        // SQL 생성 후 JDBC로 DB에 저장
    }
}
```

그래서 우리가 memberRepository.save(member)를 쓰면

→ 그 메서드는 결국 EntityManager.persist()를 호출하고

→ 실제로 SQL을 만들어 실행하는건 Hibernate가 처리한다.

## N+1 문제 

JPA를 사용하면 자주 만나게 되는 것이 N+1문제이다.

### N+1 문제란?

연관 관계에서 발생하는 이슈로 연관 관계가 설정된 엔티티를 조회할 경우에 조회된 데이터 개수(n)만큼 연관 관계의 조회 쿼리가 추가로 발생하여 데이터를 읽어오게 된다. 즉, 1번의 쿼리를 날렸을 때 의도하지 않은 N번의 쿼리가 추가적으로 실행되는 것이다. 이를 N+1 문제라고 한다.

즉, “학생 리스트를 한 번에 불러오려고 했는데, 학생마다 속한 반(Classroom)을 따로 조회해버리는 상황”이다.

### 예시로 이해하기

```java
@Entity
public class Student {
    @Id @GeneratedValue
    private Long id;

    private String name;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "classroom_id")
    private Classroom classroom;
}

@Entity
public class Classroom {
    @Id @GeneratedValue
    private Long id;
    private String roomName;
}
```

**문제 코드**

```java
List<Student> students = studentRepository.findAll();
for (Student s : students) {
    System.out.println(s.getClassroom().getRoomName());
}
```

**실행되는 쿼리**

```java
-- 1. 학생 전체 조회 (1번 쿼리)
SELECT * FROM student;

-- 2. 각 학생마다 반(Classroom) 정보 추가 조회 (학생 수만큼 N번)
SELECT * FROM classroom WHERE id = 1;
SELECT * FROM classroom WHERE id = 2;
SELECT * FROM classroom WHERE id = 3;
...
```

그래서 **총 (1 + N)**번의 쿼리가 나간다. → 성능 저하

### 왜 이런 일이 생기냐

→ @ManyToOne(fetch = FetchType.LAZY) 때문이다. LAZY는 “필요할 때만 가져오겠다”라는 뜻이라서 student.getClassroom()에 접근하는 순간 JPA가 추가 쿼리를 날리는 것이다.

즉,

- 처음엔 학생만 불러오고
- 나중에 “반 이름 필요하네?”하면서 DB에 또 접근 → 쿼리 폭발 ..

### 해결 방법

1. **Fetch Join 사용**

   → “학생과 반을 한 번에 JOIN 해서 가져오자!!”

    ```java
    @Query("SELECT s FROM Student s JOIN FETCH s.classroom")
    List<Student> findAllWithClassroom();
    
    ```

   **실행 SQL**

    ```sql
    SELECT s.*, c.*
    FROM student s
    JOIN classroom c ON s.classroom_id = c.id;
    ```

   장점

    - 한 번의 SQL로 Student + Classroom을 전부 가져옴
    - N+1 문제 완전히 해결

   단점

    - 항상 조인하기 때문에 불필요한 데이터까지 가져올 수도 있음
    - 복잡한 구조에서는 JPQL이 길어짐
2. **Batch Fetching(배치 크기 설정)**

   → “한 번에 여러 데이터를 묶어서 가져오자!”

   application.yml 또는 [application.properties](http://application.properties) 설정 👇

    ```yaml
    spring:
      jpa:
        properties:
          hibernate.default_batch_fetch_size: 100
    ```

   이렇게 하면 JPA가 “학생 N명”을 조회할 때, 반을 한꺼번에 100명 단위로 모아서 가져옴.

   **실행 SQL**

    ```sql
    SELECT * FROM classroom WHERE id IN (1, 2, 3, ..., 100);
    ```

   **장점**

    - 코드 수정 없이 설정만으로 최적화 가능
    - LAZY 로딩 유지하면서도 N+1 완화

   **단점**

    - 완벽한 해결은 아니고, 줄여주는 수준(그래도 효과 큼)

## 기본키 생성 전략 
### 기본키 생성 전략이란?

엔티티를 DB에 저장할 때, 기본키(id)값을 누가, 어떻게 생성할지를 결정하는 방법이다.

즉, 👇

```java
@Entity
public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
}
```

이 코드에서 @GenteratedValue(strategy = …) → 바로 기본키 생성 전략을 지정하는 부분이다.

### JPA가 제공하는 4가지 기본키 생성 전략

| **전략** | **설명** | **키를 생성하는 주체** |
| --- | --- | --- |
| IDENTITY | DB의 자동 증가(AUTO_INCREMENT)기능 사용 | 데이터베이스 |
| SEQUENCE | DB 시퀀스 객체를 사용해 키 생성  | 데이터베이스  |
| TABLE | JPA가 별도의 키 생성용 테이블을 만들어 관리 | JPA |
| AUTO | JPA가 DB dialect보고 자동으로 위 셋 중 하나 선택 | JPA(자동 선택) |

### 1. IDENTITY 전략

→ DB가 직접 ID를 자동으로 생성하는 방식

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

**동작 과정**

1. INSERT 시점에 JPA가 ID값을 지정하지 않음
2. DB가 AUTO_INCREMENT로 새 ID를 생성
3. JPA가 DB가 만든 ID를 다시 조회해서 객체에 세팅

```sql
INSERT INTO member (name) VALUES ('해원');
-- DB가 id=1 자동 생성
```

**장점**

- 단순함, DB에서 ID관리하므로 충돌 없음

**단점**

- ID가 DB 저장 후에야 정해지므로 → persist()할 때 즉시 insert가 일어남(지연쓰기 X)( JPA는 내부에 영속성 컨텍스트(persistence context)라는 걸 가지는데 이는 “DB에 바로 안 보내고, 임시 저장소에 모아두는 버퍼”이다. 근데 IDENTITY전략은 DB가 ID를 만들어주기 때문에 이걸 못씀→ 대량의 insert)

### 2. SEQUENCE 전략

→ 시퀀스 객체를 사용해 ID를 미리 가져오는 방식

```java
@Id
@SequenceGenerator(name = "member_seq", sequenceName = "member_seq", allocationSize = 1)
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "member_seq")
private Long id;
```

**동작 과정**

1. JPA가 먼저 시퀀스 객체에서 ID 값을 하나 가져옴

    ```sql
    SELECT nextval('member_seq');
    ```

2. 그 다음 INSERT 실행

    ```sql
    INSERT INTO member (id, name) VALUES (1, '해원');
    ```


**장점**

- ID를 미리 확보해서 버퍼링 가능 → 배치 성능 좋음
- DB에서 중복 없이 순차적 생성

**단점**

- DB마다 시퀀스 문법 다름

### 3. TABLE 전략

→ 별도의 키 생성 전용 테이블을 만들어서 JPA가 ID를 관리하는 방식

```java
@Id
@GeneratedValue(strategy = GenerationType.TABLE, generator = "member_seq_gen")
@TableGenerator(name = "member_seq_gen")
private Long id; 
```

**동작 과정**

- JPA가 자동으로 id_sequence라는 테이블을 하나 만들고 다음과 같이 관리함 👇

| sequence_name | next_val |
| --- | --- |
| member_seq_gen | 1 |
- 새로운 엔티티를 저장할 때마다 이 테이블을 조회(select)하고 next_val을 증가(update)시켜서 그 결과를 ID로 할당

**장점**

- DB에 의존하지 않음(모든 DB에서 동작함)

**단점**

- ID를 가져올 때 **조회 + 수정 + 갱신** 3단계 쿼리가 필요해서 → **성능이 가장 느림**

### 4. AUTO 전략

→ JPA가 사용하는 DB dialect에 맞춰 자동으로 전략 선택

```java
@Id
@GeneratedValue(strategy = GenerationType.AUTO)
private Long id;
```

예를 들어:

- MySQL → `IDENTITY`
- PostgreSQL → `SEQUENCE`
- H2 → `SEQUENCE`
- Oracle → `SEQUENCE`

**장점**

- DB를 바꿔도 코드 수정 필요 없음

**단점**

- 개발자가 의도한 전략이 아닐 수도 있음 (명시적으로 쓰는 걸 추천)