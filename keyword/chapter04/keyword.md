- 계층형 구조 vs 도메인형 구조
    
    ## 1. 계층형 구조 (Layered Architecture)
    
    ### 개념
    
    **“관심사별로 코드를 분리한다”**는 게 핵심
    
    ```
    controller  →  service  →  repository  →  database
    ```
    
    ---
    
    ### 예시 구조
    
    ```
    com.example.user
     ┣ controller
     ┃ ┗ UserController.java
     ┣ service
     ┃ ┗ UserService.java
     ┣ repository
     ┃ ┗ UserRepository.java
     ┗ entity
       ┗ User.java
    ```
    
    ---
    
    ### 동작 흐름
    
    1. **Controller**
        - 요청을 받고 응답을 돌려줌
        - 예: `/api/users` 요청 수신
    2. **Service**
        - 비즈니스 로직 처리
        - 예: 사용자 가입, 비밀번호 변경 등
    3. **Repository**
        - 데이터베이스 접근
        - 예: JPA, MyBatis 등으로 CRUD 수행
    
    ---
    
    ### 장점
    
    - 구조가 명확하고 배우기 쉽다
    - 작은/중간 규모 프로젝트에 적합
    - 역할이 구분되어 협업하기 편함
    
    ### 단점
    
    - **비즈니스 로직이 Service에 몰림**
        
        → 엔티티는 단순 데이터 덩어리(DTO처럼)로만 사용됨
        
    - 도메인이 복잡해질수록 **Service가 비대해짐**
        
        → 유지보수 어려워짐
        
    
    ---
    
    ## 2. 도메인형 구조 (Domain-Oriented / DDD 구조)
    
    ### 개념
    
    코드를 기술(Controller, Repository) 중심이 아니라
    
    **비즈니스 개념(Domain)** 중심으로 구성하자
    
    즉, `UserService`, `UserRepository`가 아니라
    
    → **User 도메인** 안에 이 모든 걸 묶음
    
    ---
    
    ### 예시 구조
    
    ```
    com.example.user
     ┣ User.java              ← 엔티티 (도메인 모델)
     ┣ UserService.java       ← 도메인 로직
     ┣ UserRepository.java    ← 데이터 접근
     ┣ UserController.java    ← 진입점 (웹/REST)
    ```
    
    또는 조금 더 깔끔하게 나누면 
    
    ```
    com.example.user
     ┣ domain
     ┃ ┗ User.java
     ┣ application
     ┃ ┗ UserService.java
     ┣ infrastructure
     ┃ ┗ UserRepositoryImpl.java
     ┣ presentation
     ┃ ┗ UserController.java
    ```
    
    ---
    
    ### 특징
    
    - *도메인(Entity)**가 로직의 중심에 있음
    - **Service는 조율자**, **Entity가 행동의 주체**
    - **패키지별로 도메인 단위**로 묶음
    
    ---
    
    ### 장점
    
    - **도메인 로직이 명확히 분리됨** (User 관련 건 User 패키지에 다 있음)
    - 코드가 **비즈니스 개념에 더 잘 맞음**
    - **확장성**과 **유지보수성**이 뛰어남
        
        (복잡한 시스템일수록 유리)
        
    
    ### 단점
    
    - 설계가 어렵고 초기 구조 잡기가 복잡함
    - 간단한 CRUD 서비스에는 과한 구조일 수 있음
    
    ---
    
    ## 3. 비교 정리
    
    | 구분 | 계층형 구조 | 도메인형 구조 (DDD) |
    | --- | --- | --- |
    | 기준 | 기술(Controller, Service, Repo) 중심 | 비즈니스 도메인 중심 |
    | 패키지 구조 | controller / service / repository | user / order / news 등 도메인별 |
    | 장점 | 단순, 익숙, 빠른 개발 | 유지보수성 높음, 도메인 명확 |
    | 단점 | Service에 로직 몰림, 도메인 약함 | 설계 복잡, 초기 진입장벽 있음 |
    | 추천 상황 | CRUD 위주 소규모 서비스 | 비즈니스 복잡한 대규모 서비스 |
    
    ---
    
    ## 4. 비유로 이해
    
    | 상황 | 설명 |
    | --- | --- |
    | **계층형 구조** | “조리팀 / 서빙팀 / 재료팀”으로 부서를 나눈 식당 구조 |
    | **도메인형 구조** | “일식팀 / 중식팀 / 양식팀”처럼 메뉴(도메인)별로 나눈 구조 |
    
    → 도메인형은 **“한 메뉴를 위한 전 과정을 한곳에서 관리”**
    
    → 계층형은 **“각 단계별로 역할 분리”**
    
- JPA
    
    ## 1. JPA란?
    
    > JPA (Java Persistence API)
    > 
    > 
    > → 자바에서 객체(Object)를 데이터베이스의 테이블과 **매핑(Mapping)** 해주는 기술
    > 
    
    즉,
    
    **DB 테이블을 직접 다루지 않고, 자바 객체로 다루게 해주는 표준 API임**
    
    ---
    
    ## 쉽게 말하면?
    
    ### 과거 방식 (JDBC)
    
    ```java
    String sql = "INSERT INTO user (name, email) VALUES (?, ?)";
    PreparedStatement ps = conn.prepareStatement(sql);
    ps.setString(1, "민재");
    ps.setString(2, "minjae@email.com");
    ps.executeUpdate();
    ```
    
    ➡️ SQL을 직접 써야 했음
    
    ---
    
    ### JPA 방식
    
    ```java
    User user = new User("민재", "minjae@email.com");
    entityManager.persist(user);
    
    ```
    
    ➡️ SQL을 직접 쓰지 않아도
    
    JPA가 **자동으로 SQL을 만들어서 실행**해줌
    
    ---
    
    ## 2. JPA의 핵심 개념
    
    | 용어 | 의미 |
    | --- | --- |
    | **Entity** | DB 테이블과 매핑되는 클래스 |
    | **EntityManager** | 엔티티를 저장, 조회, 삭제 등 관리하는 객체 |
    | **Persistence Context (영속성 컨텍스트)** | 엔티티를 1차 캐시에 저장하고 관리하는 메모리 공간 |
    | **JPQL (Java Persistence Query Language)** | SQL처럼 보이지만, **객체를 기준으로 질의**하는 언어 |
    
    ---
    
    ## 3. JPA의 작동 원리
    
    ```
    (1) 자바 객체 저장 요청 → JPA → SQL 자동 생성 → DB 저장
    (2) DB에서 조회 요청 → JPA → SQL 실행 → 자바 객체로 반환
    ```
    
    예시:
    
    ```java
    User user = new User("민재", "minjae@email.com");
    entityManager.persist(user); // INSERT 쿼리 자동 실행
    
    User found = entityManager.find(User.class, 1L); // SELECT 쿼리 자동 실행
    ```
    
    ---
    
    ## 4. JPA vs Hibernate
    
    - **JPA** : "규약" (인터페이스, 표준)
    - **Hibernate** : JPA를 구현한 "구현체"
    
    즉,
    
    > JPA는 규칙서(인터페이스)
    > 
    > 
    > Hibernate는 **그걸 실제로 구현한 도구**
    > 
    
    스프링 부트에서 JPA를 쓸 때 실제로는 **Hibernate**가 작동하고 있음.
    
    ---
    
    ## 5. JPA의 주요 어노테이션
    
    | 어노테이션 | 설명 |
    | --- | --- |
    | `@Entity` | 이 클래스가 DB 테이블과 매핑됨 |
    | `@Table(name="users")` | 테이블 이름 지정 (생략 가능) |
    | `@Id` | 기본 키 지정 |
    | `@GeneratedValue` | 기본 키 자동 생성 전략 |
    | `@Column(name="email")` | 컬럼 이름 지정 (옵션) |
    | `@OneToMany`, `@ManyToOne` | 엔티티 간 관계 매핑 |
    
    예시 
    
    ```java
    @Entity
    @Table(name = "users")
    public class User {
    
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY) // AUTO_INCREMENT
        private Long id;
    
        private String name;
    
        @Column(nullable = false, unique = true)
        private String email;
    
        protected User() {} // JPA 기본 생성자
    
        public User(String name, String email) {
            this.name = name;
            this.email = email;
        }
    
        // getter/setter
    }
    ```
    
    ---
    
    ## 6. Spring Data JPA
    
    > JPA를 더 쉽게 쓰게 해주는 스프링 모듈
    > 
    
    Spring Data JPA는 `EntityManager` 대신
    
    `Repository` 인터페이스를 자동으로 구현해줌
    
    ```java
    public interface UserRepository extends JpaRepository<User, Long> {
        Optional<User> findByEmail(String email);
    }
    ```
    
    이렇게 사용
    
    ```java
    @Service
    public class UserService {
        private final UserRepository userRepository;
    
        public void register(String name, String email) {
            userRepository.save(new User(name, email)); // INSERT SQL 자동
        }
    
        public User findUser(Long id) {
            return userRepository.findById(id).orElseThrow();
        }
    }
    ```
    
    ➡️ **SQL을 직접 작성하지 않아도**
    
    CRUD가 다 됨.
    
    ---
    
    ## 7. JPA의 장점
    
    | 항목 | 설명 |
    | --- | --- |
    | ✅ 생산성 | SQL 작성 없이 CRUD 가능 |
    | ✅ 유지보수성 | 객체 중심 코드로 깔끔 |
    | ✅ 이식성 | DB 종류에 관계없이 작동 (MySQL, PostgreSQL 등) |
    | ✅ 캐싱 | 1차 캐시로 동일 트랜잭션 내 중복 조회 방지 |
    | ✅ 트랜잭션 관리 | 엔티티 상태를 자동으로 관리 (Dirty Checking) |
    
    ---
    
    ## 8. 단점
    
    | 항목 | 설명 |
    | --- | --- |
    | ❌ 복잡한 쿼리 | JOIN, GROUP BY 같은 고급 쿼리는 어렵다 |
    | ❌ 학습 난이도 | 영속성 컨텍스트, 엔티티 상태 관리 개념이 어렵다 |
    | ❌ 성능 튜닝 | 자동 SQL이라 효율적이지 않을 수 있음 |
    
    → 이런 건 나중에 **QueryDSL**이나 **Native Query**로 보완함.
    
    ---
    
    ## 9. 정리
    
    | 구분 | 설명 |
    | --- | --- |
    | **JPA** | 자바 ORM 표준 API |
    | **ORM(Object-Relational Mapping)** | 객체 ↔ 관계형 DB 매핑 기술 |
    | **Hibernate** | JPA의 구현체 |
    | **Spring Data JPA** | Hibernate를 스프링스럽게 감싼 모듈 |
    | **Entity** | DB 테이블과 매핑되는 클래스 |
- N+1 문제
    
    ## 1. N+1 문제란?
    
    > 한 번의 쿼리를 실행했는데, 그 결과로 인해 추가로 N번의 쿼리가 더 실행되는 문제
    > 
    > 
    > 즉, 의도하지 않게 **쿼리가 총 N+1번 실행되는 상황**을 말한다.
    > 
    
    ## 2. 왜 이런 문제가 생길까?
    
    원인은 **지연 로딩(LAZY Loading)** 때문
    
    - JPA는 `@ManyToOne(fetch = FetchType.LAZY)`가 기본임.
    - 즉, **연관된 엔티티는 실제로 접근할 때까지 SQL을 실행하지 않음**.
    
    👉 `member.getTeam()`을 호출하는 순간
    
    → `Team`을 조회하는 추가 쿼리가 실행됨.
    
    ---
    
    ## 3. N+1 문제의 영향
    
    | 항목 | 설명 |
    | --- | --- |
    | 성능 | 쿼리가 너무 많아짐 (1번 → 수십/수백 번) |
    | 부하 | DB 부하 증가, 네트워크 I/O 증가 |
    | 결과 | 조회 속도 급격히 느려짐 |
    
    > 특히 회원 10,000명에 팀 1,000개면
    > 
    > 
    > 쿼리 10,001번 실행될 수도 있음 ㅜㅜ
    > 
    
    ## 4. 해결 방법
    
    ### (1) Fetch Join 사용
    
    > 연관된 엔티티를 한 번의 쿼리로 함께 조회하도록 만든다.
    > 
    
    ```java
    // JPQL
    List<Member> members = em.createQuery(
        "select m from Member m join fetch m.team", Member.class)
        .getResultList();
    ```
    
    ➡️ 실제 실행 SQL:
    
    ```sql
    select m.*, t.*
    from member m
    join team t on m.team_id = t.id;
    ```
    
    ➡️ Team을 미리 같이 가져오기 때문에
    
    `member.getTeam()` 호출 시 추가 쿼리 ❌
    
    ---
    
    ### (2) EntityGraph 사용
    
    > 스프링 데이터 JPA에서 Fetch Join을 더 선언적으로 처리할 수 있다.
    > 
    
    ```java
    @EntityGraph(attributePaths = {"team"})
    @Query("select m from Member m")
    List<Member> findAllWithTeam();
    
    ```
    
    ➡️ 내부적으로 Fetch Join 쿼리 생성됨
    
    ---
    
    ### (3) BatchSize 설정 (배치 패치)
    
    > N개의 쿼리를 1번에 묶어서 가져오는 방식
    > 
    
    ```java
    @OneToMany(mappedBy = "team")
    @BatchSize(size = 100)
    private List<Member> members;
    ```
    
    ➡️ JPA가 한 번에 100개씩 Team을 in-query로 가져와서
    
    `N+1` → `1 + (N / 100)` 정도로 줄어든다.
    
    ---
    
    ## 5. 핵심 요약
    
    | 구분 | 설명 |
    | --- | --- |
    | **문제** | 1번 쿼리로 조회한 결과가 다시 N번의 추가 쿼리를 유발 |
    | **원인** | 지연 로딩(LAZY) 시, 연관된 엔티티 접근 시마다 SELECT 발생 |
    | **대표 해결법** | `fetch join` 또는 `@EntityGraph` |
    | **비추천 방법** | `EAGER` 로딩 (제어 어렵고, 불필요한 JOIN) |
    | **보조 방법** | `@BatchSize` 로 일괄조회 |
- 기본 키 생성 전략
    
    ## 1. 기본키 생성 전략이란?
    
    > 엔티티를 DB에 저장할 때,
    > 
    > 
    > 그 **`@Id`(기본키)** 를 **어떻게 생성할지 JPA에게 알려주는 설정**
    > 
    
    ---
    
    ## 2. 기본키 생성 전략 종류
    
    JPA는 3가지 방식을 제공함
    
    | 전략 | 설명 | 특징 |
    | --- | --- | --- |
    | `GenerationType.IDENTITY` | DB가 자동 증가(AUTO_INCREMENT) | MySQL에서 주로 사용 |
    | `GenerationType.SEQUENCE` | DB의 시퀀스(Sequence) 객체 사용 | Oracle, PostgreSQL 등 |
    | `GenerationType.TABLE` | 별도의 키 생성용 테이블 사용 | 모든 DB 호환 가능 (비효율적) |
    | `GenerationType.AUTO` | JPA가 DB 종류에 따라 자동 선택 | 기본값 |
    
    ---
    
    ## 3. 예시 코드
    
    ### (1) `IDENTITY` — **MySQL, MariaDB에서 가장 흔함**
    
    ```java
    @Entity
    public class User {
    
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id; // DB의 AUTO_INCREMENT 사용
    
        private String name;
    }
    ```
    
    특징:
    
    - DB가 기본키를 생성함 (`INSERT` 시점에 자동 증가)
    - `INSERT` 쿼리를 먼저 날리고,
        
        DB에서 생성된 ID 값을 다시 가져옴
        
    - 트랜잭션 전에 ID를 알 수 없음 (비영속 상태에선 id=null)
    
    실제 쿼리 예시:
    
    ```sql
    insert into user (name) values ('민재');
    -- DB에서 AUTO_INCREMENT로 id 자동 생성
    ```
    
    ---
    
    ### (2) `SEQUENCE` — **Oracle, PostgreSQL에서 자주 사용**
    
    ```java
    @Entity
    @SequenceGenerator(
        name = "user_seq_generator",
        sequenceName = "user_seq",
        initialValue = 1, allocationSize = 1
    )
    public class User {
    
        @Id
        @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "user_seq_generator")
        private Long id;
    
        private String name;
    }
    ```
    
    특징:
    
    - DB의 시퀀스 객체(`CREATE SEQUENCE user_seq`)를 이용함
    - `INSERT` 전 미리 ID를 가져올 수 있음
    - JPA는 내부적으로 `nextval` 호출로 ID 미리 확보 가능
    
    🧾 실제 쿼리 예시:
    
    ```sql
    select nextval('user_seq');
    insert into user (id, name) values (1, '민재');
    ```
    
    ---
    
    ### (4) `AUTO` — **JPA가 자동 선택**
    
    ```java
    @Entity
    public class User {
    
        @Id
        @GeneratedValue(strategy = GenerationType.AUTO)
        private Long id;
    }
    ```
    
    특징:
    
    - JPA가 사용하는 DB 방언(dialect)에 따라 자동 선택
        - MySQL → `IDENTITY`
        - Oracle → `SEQUENCE`
    - DB 변경 시 유연하지만, 명시적이지 않아 권장되지 않음
    
    ---
    
    ## 4. 비교 요약
    
    | 전략 | ID 생성 주체 | 특징 | 주요 DB |
    | --- | --- | --- | --- |
    | **IDENTITY** | DB | AUTO_INCREMENT, 단순 | MySQL, MariaDB |
    | **SEQUENCE** | DB 시퀀스 | 미리 ID 확보 가능, 빠름 | Oracle, PostgreSQL |
    | **AUTO** | JPA | DB에 따라 자동 선택 | 전체 |
    
    ---
    
    ## 5. `allocationSize` 옵션이 중요한 이유
    
    `@SequenceGenerator`나 `@TableGenerator`에 있는
    
    `allocationSize`는 **ID를 미리 몇 개씩 캐싱할지** 정하는 옵션이다
    
    예시:
    
    ```java
    @SequenceGenerator(name="user_seq_gen", sequenceName="user_seq", allocationSize=50)
    ```
    
    ➡️ JPA는 한 번에 50개의 ID를 미리 가져와서 메모리에 저장해둠.
    
    → `INSERT` 시 마다 DB에 `nextval()` 요청 안 해도 됨.
    
    → **성능 크게 향상!**
    
    단,
    
    여러 애플리케이션 인스턴스가 같은 시퀀스를 쓸 경우 주의 필요.
    
    ---
    
    ## 6. 실제 스프링 부트 환경 예시
    
    ```java
    @Entity
    public class Article {
    
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;
    
        private String title;
    }
    ```
    
    이러면 스프링 부트가 알아서 MySQL의 AUTO_INCREMENT를 이용해
    
    `id`를 자동으로 채워준다
    
    ---
    
    ## 7. 정리 요약
    
    | 전략 | 설명 | 장점 | 단점 |
    | --- | --- | --- | --- |
    | **IDENTITY** | DB가 자동 증가 | 단순, 설정 쉬움 | ID 미리 알 수 없음 |
    | **SEQUENCE** | DB 시퀀스 사용 | 빠름, 미리 할당 가능 | DB가 시퀀스를 지원해야 함 |
    | **AUTO** | DB 방언에 따라 자동 선택 | 유연함 | 예측 어려움 |
    
    ---
    
    ## ✅ 결론
    
    > JPA에서 기본키는 단순히 “id 값”이 아니라
    > 
    > 
    > “엔티티 식별과 영속성 관리의 핵심”이다.
    > 
    > **MySQL → `IDENTITY`**,
    > 
    > **Oracle / PostgreSQL → `SEQUENCE`**
    > 
    > 를 사용하는 것이 일반적이다
    >