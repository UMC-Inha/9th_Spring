- **지연로딩과 즉시로딩의 차이**
  ### 즉시로딩(Eager)
  데이터를 조회할 때, 연관된 모든 객체의 데이터를 한번에 불러오는 것.
  위의 예시에서처럼 EAGER 방식을 사용하면 member를 조회하는 시점에 바로 member food까지 조회하여 한꺼번에 데이터를 불러온다.
  ### 지연로딩(Lazy)
  필요한 시점에 연관된 객체의 데이터를 불러오는 것.
  지연로딩을 사용하면 member를 조회하는 시점이 아니라 실제 연관된 데이터를 조회할 때 쿼리가 나가도록 하게 된다.
  지연로딩을 사용하면 N+1문제를 줄일 수 있기 때문에 기본적으로는 지연로딩을 사용하는 것이 좋다.
- **JPQL**

  ### JPQL이란?

  SQL을 추상화한 것. JPQL은 SQL로 변환된다. 테이블이 아닌 객체를 대상으로 검색하는 객체지향 쿼리 언어이다.

  ```json
  String jpql = "select m From Member m where m.name like ‘%hello%'";
  List<Member> result = em.createQuery(jpql, Member.class).getResultList();

  for (Member member : result) {
      System.out.println("member = " + member);
  }
  ```

  이런식으로 쓸 수 있다. 실제 쿼리문을 작성하는 것과 비슷하다.

  ### 문제점

  - 기본 문자열로 작성되기 때문에 컴파일 시에 에러를 발생하지 않는다. 따라서 문제가 있어도 정상적으로 작동해서 배포할 때 문제가 생길 수 있다.
  - 동적으로 쿼리 언어를 작성하는 데 효율적이지 못하다. 특정 조건의 참일 경우에는 A쿼리, 거짓일 경우에는 B쿼리를 실행하는 등 이런저런 문제가 있다.

- **Fetch Join**
  페치 조인은 SQL에서 이야기하는 조인의 종류는 아니다.
  JPQL에서 성능 최적화를 위해 제공하는 조인의 종류이다.

  ```java
  String query = "select b from Book b join fetch b.library";

  em.flush();
  em.clear();

  List<Book> books = em.createQuery(query, Book.class).getResultList();

  for (Book book : books) {
      System.out.println(book.getLibrary());
  }
  ```

  JOIN FETCH 명령어를 사용하여 Book의 Library를 함께 조회한다.
  실제 쿼리문에서도 JOIN 쿼리가 나가게 되고 데이터가 함께 조회되기에 추가적인 쿼리가 나가지 않는다.
  JPQL의 일반적인 조인과 다르게 `b.library`의 별칭이 존재하지 않는데 페치 조인은 별칭을 사용할 수 없다.
  페치 조인을 사용하였기에 Book과 Library가 객체 그래프를 유지하면서 데이터를 조회할 수 있다. Library는 프록시가 아닌 실제 엔티티이므로 Book 엔티티가 영속성 컨텍스트에서 분리되어 준영속 상태가 되어도 연관된 Library를 조회할 수 있다.

- **@EntityGraph**
  **`@EntityGraph`는 JPQL 또는 Spring Data JPA 쿼리에서 연관된 엔티티를 함께 즉시 로딩(EAGER)하도록 지정하는 기능이다.**
  즉, **N+1 문제를 해결**하고 **쿼리 수를 줄이는** 방법이다.
  @EntityGraph 를 사용하면 join fetch 없어도 연관 엔티티를 한번에 가져오게 할 수 있다.

  ```java
  @Entity
  class Member {
      @Id @GeneratedValue
      private Long id;
      private String name;

      @ManyToOne(fetch = FetchType.LAZY)
      private Team team;
  }

  @Entity
  class Team {
      @Id @GeneratedValue
      private Long id;
      private String name;
  }

  ```

  여기서

  ```java
  List<Member> members = memberRepository.findAll();
  ```

  이 쿼리를 실행하게 되면

  1. `select * from member`
  2. 그다음 각 member의 team을 접근할 때마다

     `select * from team where id = ?` (팀 수만큼 실행됨)
     즉, Member가 10명이면
     **1(멤버 전체 조회) + 10(팀 개별 조회) = 11번 쿼리 발생**
     근데
     @EntityGraph를 사용하면 **join fetch 없이도 연관 엔티티를 한 번에 가져오게 할 수 있다.**

  ```java
  @EntityGraph(attributePaths = {"team"})
  @Query("select m from Member m")
  List<Member> findAllWithTeam();
  ```

  이런식으로 사용할 수 있다.

  ```sql
  select m.*, t.*
  from member m
  left join team t on m.team_id = t.id;
  ```

  이게 실제 실행되는 SQL인데 한번의 조인 쿼리로 멤버랑 팀을 한번에 가져온다.
  → 이러면 N+1문제를 해결할 수 있음.

- **commit과 flush 차이점**
  | 용어 | 핵심 의미 |
  | ---------- | ---------------------------------------------------------------- |
  | **flush** | 영속성 컨텍스트의 변경 내용을 **DB에 동기화(=SQL 실행)** 하는 것 |
  | **commit** | 트랜잭션을 **종료**하고, **DB에 변경 내용을 확정(저장)** 하는 것 |
  JPA는 SQL을 바로 DB에 보내지 않고, **트랜잭션 커밋 직전에 한 번에 처리**하려고 쓰기 지연(SQL 저장소)에 쌓아둔다. 이걸 **flush** 하면 SQL이 실제 DB로 전송된다.
  | 구분 | flush | commit |
  | ------------------------ | ------------------------------------------- | ---------------------------- |
  | **역할** | 영속성 컨텍스트 → DB 동기화 (SQL 실행) | 트랜잭션 종료, DB 반영 확정 |
  | **DB 반영 여부** | SQL만 보냄, 아직 확정 아님 | 실제 DB에 **확정(commit)** |
  | **영속성 컨텍스트 영향** | 비워지지 않음 | 비워지거나 새 트랜잭션 시작 |
  | **rollback 가능 여부** | 가능 | commit 후에는 불가능 |
  | **호출 시점** | 명시적 `em.flush()` 또는 JPA 내부 자동 호출 | 트랜잭션이 끝날 때 자동 호출 |
- **QueryDSL, OpenFeign의 QueryDSL**
  QueryDSL은 자바 코드로 JPQL을 대체할 수 있는, 타입 안전한 쿼리 빌더임.

  ```sql
  // 일반 JPQL
  String jpql = "select m from Member m where m.age > 20";

  // QueryDSL
  QMember m = QMember.member;
  List<Member> result = queryFactory
          .selectFrom(m)
          .where(m.age.gt(20))
          .fetch();
  ```

  이렇다고 함.

  ### OpenFeign

  **OpenFeign은 Netflix에서 처음 만들어진 HTTP Client 도구로 REST API 호출을 더 간편하고 직관적으로 만들어 주는 라이브러리**
  **어노테이션을 주로 사용하는 선언적 요청 방식으로 메서드의 방식이 Spring Data Jpa, Spring MVC와 유사함.**
  → API 호출을 인터페이스처럼 만드는 HTTP 클라이언트.
  **장점:**
  **1. 인터페이스 및 어노테이션 방식으로 작성하여 코드가 간결해진다.**
  **2. 다른 Spring Cloud 기술들와 쉽게 통합 가능하다.**
  **3. 코드 가독성이 높아 유지보수가 간소화된다. Spring Data Jpa, Spring MVC를 자주 사용하는 사람들은 쉽게 이해할 수 있는 정도이다.**
  **단점:**
  **1. Spring Cloud 의존성을 추가해야 한다. Spring Cloud는 사용하지 않고 Open Feign만 사용하는데도 의존성을 추가하여 관리에 복잡함을 더할 수 있다.**
  **2. 공식적으로 비동기를 위한 Reactive모델을 지원하지 않는다. (비공식적으로 'feign-reactive'라는 것이 존재한다.)**
  **3. HttpClient가 Http2를 지원하지 않는다.(추가 설정이 필요하다.)**
  **4. 독자적인 테스트 도구를 제공하지 않는다 (@SpringBootTest로 띄어 API를 직접 호출하여 테스트하는 것은 가능하다.)**

- **N+1 문제 해결할 수 있는 여러 방안들**
  | 해결 방식 | 설명 | 특징 |
  | ---------------------------------- | ------------------------------------------------ | ----------------------- |
  | ① **Fetch Join** | JPQL/QueryDSL로 조인하여 한 번에 가져오기 | 가장 직관적, 한 방 쿼리 |
  | ② **@EntityGraph** | 메서드 호출 시 fetch join 효과 | 선언적이고 간단 |
  | ③ **Batch Fetching** | Lazy 로딩을 유지하되 여러 건을 묶어 한 번에 로딩 | 페이징 시 유용 |
  | ④ **FetchMode.SUBSELECT** | 동일 부모의 자식들을 한 번의 서브쿼리로 로딩 | 컬렉션 로딩 시 효과적 |
  | ⑤ **DTO Projection** | 필요한 데이터만 한 방 쿼리로 가져옴 | 읽기 전용 조회에 최적 |
  | ⑥ **쿼리 최적화 / 분리 로딩 전략** | 두 단계 조회 (부모 → 자식 IN 쿼리) | 대용량 페이징에 안정적 |
  ### (1) Fetch Join
  > 한 방에 쿼리로 연관 객체를 함께 조회함
  > **JPQL 예시**
  ```java
  @Query("select m from Member m join fetch m.team")
  List<Member> findAllFetchJoin();
  ```
  **결과**
  ```sql
  select m.*, t.*
  from member m
  join team t on m.team_id = t.id;
  ```
  **장점**
  - N+1 완전 해결
  - 쿼리 1회로 모든 연관 데이터 로드
    **단점**
  - 컬렉션(`@OneToMany`) + 페이징 시 비효율/오류
  - 조인 개수가 많으면 중복 데이터 증가
  ***
  ### (2) @EntityGraph
  > 리포지토리 단에서 fetch join 선언적 적용
  ```java
  @EntityGraph(attributePaths = {"team"})
  List<Member> findAll(); // 내부적으로 fetch join 수행
  ```
  **장점**
  - 코드 간결, 설정만으로 해결
  - fetch join과 같은 효과
    **단점**
  - 복잡한 조건 쿼리엔 한계 있음 (단순한 경우에 적합)
  ***
  ### (3) Batch Fetching (배치 로딩)
  > Lazy 로딩을 유지하면서 여러 건을 묶어 IN 쿼리로 가져오기
  > **설정**
  ```yaml
  spring:
    jpa:
      properties:
        hibernate.default_batch_fetch_size: 100
  ```
  **결과**
  ```sql
  select * from team where id in (?, ?, ?, ...);
  ```
  **장점**
  - 페이징 시에도 안정적
  - 쿼리 수가 N에서 2~3으로 줄어듦
    **단점**
  - 완전한 한 방 쿼리는 아님 (지연로딩은 유지)
  ***
  ### (4) FetchMode.SUBSELECT
  > 한 번에 여러 부모의 자식들을 서브쿼리로 조회
  ```java
  @OneToMany(mappedBy = "member")
  @Fetch(FetchMode.SUBSELECT)
  private List<Order> orders;
  ```
  **결과**
  ```sql
  select * from order
  where member_id in (select id from member ...);
  ```
  **장점**
  - 비슷한 시점에 여러 부모 엔티티의 자식 접근 시 효율적
    **단점**
  - 서브쿼리 실행 타이밍 제어가 어려움
  ***
  ### (5) DTO Projection
  > 필요한 데이터만 쿼리로 직접 가져와 DTO에 매핑
  > **JPQL 예시**
  ```java
  @Query("""
  select new com.example.MemberDto(m.name, t.name)
  from Member m join m.team t
  """)
  List<MemberDto> findAllDto();
  ```
  **장점**
  - 원하는 데이터만 → 빠름
  - N+1 구조 자체를 제거 (엔티티 X)
    **단점**
  - 엔티티 변경감지 불가(조회 전용)
  - 쿼리 복잡도 약간 증가
  ***
  ### (6) 쿼리 분리 전략 (두 단계 조회)
  > ① 부모 ID만 조회 → ② 그 ID들로 자식 조회
  ```java
  List<Member> members = findMemberIds(pageable);
  List<Team> teams = teamRepository.findByMemberIds(members);
  ```
  **장점**
  - 대용량 페이징에서도 안전
  - 쿼리 단위 명확
    **단점**
  - 코드가 약간 길어짐
- **영속 상태의 종류**
  | 상태 | 설명 | 특징 | 주요 메서드 |
  | ---------------------- | ------------------------------------------- | ----------------------------------------------- | -------------------------------- |
  | **비영속 (Transient)** | JPA와 전혀 관련 없는 **순수한 새 객체** | DB와 연관 ❌, 영속성 컨텍스트에 등록 ❌ | `new` |
  | **영속 (Persistent)** | **EntityManager가 관리하는 상태** | 1차 캐시에 저장됨, 트랜잭션 commit 시 DB에 반영 | `persist()` |
  | **준영속 (Detached)** | 한때 영속이었지만 **더 이상 관리되지 않음** | 더 이상 변경 감지❌, flush 시 DB 반영❌ | `detach()`, `clear()`, `close()` |
  | **삭제 (Removed)** | 삭제로 등록된 상태 | 트랜잭션 commit 시 `DELETE` 쿼리 실행 | `remove()` |
  JPA에서 엔티티는 **비영속 → 영속 → 준영속/삭제** 상태로 이동함.
  영속 상태일 때만 JPA의 **변경 감지, 1차 캐시, 쓰기 지연, flush** 기능이 작동함.
