- QueryDSL에서 FetchJoin 하는 법
    
    QueryDSL에서 `fetchJoin`을 사용하는 방법은 JPA의 페치 조인(fetch join)을 타입 세이프(type-safe)하게 실행하는 것이다.  **N+1 문제를 해결**하기 위해 연관된 엔티티를 처음 쿼리할 때 함께 가져오는 것/
    
    일반 `join` 또는 `leftJoin` 뒤에 `.fetchJoin()` 메서드를 붙여주면 된다. 
    
    ---
    
    - `Member` (N) : `Team` (1) - 여러 멤버가 하나의 팀에 속함
    - Q 클래스: `QMember member = QMember.member;`, `QTeam team = QTeam.team;`
    
    ### 1. ToOne (N:1, 1:1) 관계 페치 조인
    
    가장 일반적인 경우. 멤버를 조회하면서 해당 멤버의 팀 정보도 함께 가져온다.
    
    **❌ Fetch Join 미사용 시 (N+1 발생)**
    
    ```java
    // 1. 멤버 조회 쿼리 (1)
    List<Member> members = queryFactory
        .selectFrom(member)
        .fetch();
    
    // 2. 루프를 돌면서 팀 정보 접근 시 N개의 추가 쿼리 발생
    for (Member m : members) {
        System.out.println("Member: " + m.getUsername() + ", Team: " + m.getTeam().getName());
        // m.getTeam().getName()을 호출하는 시점에 지연 로딩(Lazy Loading)으로 인해 
        // 멤버 수(N)만큼 팀을 조회하는 추가 쿼리가 발생
    }
    ```
    
    **✅ Fetch Join 사용 시 (N+1 해결)**
    
    ```java
    List<Member> members = queryFactory
        .selectFrom(member)
        .join(member.team, team).fetchJoin() // <- 여기서 fetch join 사용
        .fetch();
    
    // 조회된 member 객체들은 이미 team 정보를 채운 상태로 반환
    for (Member m : members) {
        // 추가 쿼리 없이 바로 팀 이름에 접근 가능
        System.out.println("Member: " + m.getUsername() + ", Team: " + m.getTeam().getName()); Warning: 
    }
    ```
    
    - `join(member.team, team).fetchJoin()`: `member`와 `team`을 내부 조인(inner join)하고, 조회 결과에 `team` 엔티티 정보를 즉시 로딩함.
    - `leftJoin(member.team, team).fetchJoin()`: `team`이 없는 `member`까지 포함하여 조회할 경우 `leftJoin`을 사용함.
    
    ---
    
    ### 2. ToMany (1:N) 관계 페치 조인
    
    팀을 조회하면서 해당 팀에 속한 모든 멤버를 함께 가져오는 경우
    
    **✅ 컬렉션(Collection) 페치 조인**
    
    Java
    
    ```java
    // QMember member = QMember.member;
    // QTeam team = QTeam.team;
    
    List<Team> teams = queryFactory
        .selectFrom(team)
        .leftJoin(team.members, member).fetchJoin() // 1:N 관계는 보통 leftJoin을 사용
        .where(team.name.eq("Team A"))
        .distinct() // <- 1:N 조인 시 중복 제거를 위해 'distinct'가 필수임
        .fetch();
    
    // "Team A"에 속한 모든 멤버 정보가 이미 로드되어 있음
    for (Team t : teams) {
        System.out.println("Team: " + t.getName());
        for (Member m : t.getMembers()) { // 추가 쿼리 발생 안 함
            System.out.println("  - Member: " + m.getUsername());
        }
    }
    ```
    
    - **`distinct()` 사용 이유**: 1:N 조인을 하면 데이터베이스 결과셋은
    
     `(Team A, Member 1)`, `(Team A, Member 2)`처럼 팀이 중복되어 나타난다. JPA는 이 중복된 결과에서 `Team` 객체를 생성하므로, 애플리케이션단에서 `List<Team>`에 `Team A`가 여러 번 포함될 수 있다.
    
    - QueryDSL의 `distinct()`는 SQL에 `DISTINCT`를 추가하고, 애플리케이션 레벨에서도 중복된 엔티티(같은 ID)를 제거해 준다.
    
    ---
    
    ### ⚠️ 중요 주의사항 및 한계점
    
    **컬렉션(1:N) 조인 시** 주의해야함.
    
    ### 1. 컬렉션 페치 조인과 페이징(Paging)
    
    > 컬렉션(OneToMany)을 페치 조인하는 경우, 페이징(offset, limit)을 사용할 수 없다.
    > 
    
    QueryDSL의 `offset()`이나 `limit()`을 사용하면 **메모리에서 페이징 처리**를 시도하며, 이건 매우 위험함.
    
    - **이유**: 데이터베이스는 `Team` 기준으로 10건을 가져오는 것이 아니라, `Team`과 `Member`가 조인된 *전체 결과*를 기준으로 10건을 가져옴. 이는 우리가 원하는 결과(팀 10개)와 다르다.
    - **경고**: Hibernate는 이 경우 경고 로그(`HHH000104: firstResult/maxResults specified with collection fetch; applying in memory!`)를 출력한다.
    - **해결책 (일반적인 전략):**
        1. 먼저 페이징을 적용하여 **대상 엔티티(예: Team)의 ID**만 조회.(`select(team.id).from(team)... offset().limit()`)
        2. 조회된 ID 리스트를 사용하여 `fetchJoin` 쿼리를 다시 실행
        
                (`where(team.id.in(teamIds))`)
        
    
    ### 2. 둘 이상의 컬렉션 페치 조인
    
    하나의 쿼리에서 둘 이상의 컬렉션(예: `team.members`와 `team.projects`)을 동시에 `fetchJoin`하면 `MultipleBagFetchException`이 발생할 수 있습니다. 이는 조인 결과가 예측 불가능한 카테시안 곱(Cartesian Product)이 되어 데이터 정합성이 깨질 수 있기 때문이다. 
    
    - **해결책**: 쿼리를 분리하거나, `@BatchSize` 어노테이션을 사용하여 지연 로딩 성능을 최적화하는 방법을 사용해야 한다.
    
    ---
    
    ### 요약
    
    - **ToOne (N:1)** 관계: `join(member.team, team).fetchJoin()`을 자유롭게 사용해도 좋음. 페이징에도 문제가 없다.
    - **ToMany (1:N)** 관계: `leftJoin(team.members, member).fetchJoin()`을 사용하되, **`distinct()`*를 꼭 추가해야 한다
    - **ToMany + Paging**: **절대 함께 사용하면 안된다!**  ID 조회 후 별도 쿼리로 `fetchJoin`을 적용하는 방식을 사용해야함.
    
- DTO 매핑 방식 (+DTO안에 DTO)
    
    DTO 매핑은 **엔티티(Entity) 객체를 DTO(Data Transfer Object) 객체로 변환**하는 과정을 말한다.
    
    엔티티는 데이터베이스 테이블과 1:1로 매핑되는 핵심 객체인 반면, DTO는 API 응답이나 요청 등 특정 계층(Layer) 간 데이터 전송을 위해 사용하는  데이터 전달용 상자다.
    
    이 변환(매핑)을 하는 이유 →
    
    - **API 스펙 맞춤:** API가 요구하는 정확한 JSON 형식(필드명, 데이터 구조)을 만들기 위해.
    - **데이터 은닉:** 엔티티의 모든 필드(예: `password`, 민감한 내부 ID)를 노출하지 않고, 필요한 데이터만 골라서 외부에 전달하기 위해.
    - **관심사 분리:** 엔티티 객체가 뷰(View)나 API 스펙에 종속되지 않도록 분리함.
    
    ---
    
    ### 1. 🧑‍🔧 수동 매핑 (생성자 또는 빌더)
    
    가장 기본적이고 간단한 방법. 엔티티 객체를 받아서 DTO 객체를 생성하는 로직을 직접 작성한다.
    
    - **장점:** 직관적이며, 별도 라이브러리 의존성이 없다.
    - **단점:** 필드가 많아지면 코드가 길어지고, 필드 변경 시 수정이 번거롭다.
    
    **예시 코드 (Service 레이어에서 변환)**
    
    ```java
    // Entity
    public class Member {
        private Long id;
        private String username;
        private String email;
        private String password; // DTO에 포함시키지 않을 필드
        // ... getters
    }
    
    // DTO
    public class MemberResponseDto {
        private String username;
        private String email;
    
        // 방법 A: 생성자를 이용
        public MemberResponseDto(Member member) {
            this.username = member.getUsername();
            this.email = member.getEmail();
        }
        
        // 방법 B: 정적 팩토리 메서드 (권장)
        public static MemberResponseDto from(Member member) {
            MemberResponseDto dto = new MemberResponseDto();
            dto.username = member.getUsername();
            dto.email = member.getEmail();
            return dto;
        }
    }
    
    // 서비스 로직
    public MemberResponseDto findMember(Long memberId) {
        Member member = memberRepository.findById(memberId)
                            .orElseThrow(() -> new EntityNotFoundException());
        
        // 엔티티를 DTO로 직접 변환
        return MemberResponseDto.from(member); 
    }
    ```
    
    ---
    
    ### 2. ⚙️ 라이브러리 사용 (ModelMapper, MapStruct)
    
    반복적인 매핑 코드를 줄이기 위해 자동화 라이브러리를 사용한다.
    
    - **장점:** 매핑 코드가 획기적으로 줄어듦
    - **단점:** 라이브러리 설정 및 학습 비용이 발생함
    
    ### A. ModelMapper (리플렉션 기반)
    
    설정이 간단하고 사용하기 편하고,  필드 이름이 같으면 자동으로 매핑해 준다
    
    ```java
    // 의존성 추가 필요 (예: build.gradle)
    // implementation 'org.modelmapper:modelmapper:3.1.0'
    
    // 설정 (Config 파일에 Bean으로 등록)
    @Configuration
    public class AppConfig {
        @Bean
        public ModelMapper modelMapper() {
            return new ModelMapper();
        }
    }
    
    // 서비스 로직
    @Autowired
    private ModelMapper modelMapper;
    
    public MemberResponseDto findMember(Long memberId) {
        Member member = memberRepository.findById(memberId).get();
        
        // 한 줄로 매핑 끝
        MemberResponseDto dto = modelMapper.map(member, MemberResponseDto.class);
        return dto;
    }
    ```
    
    ### B. MapStruct (코드 생성 기반)
    
    컴파일 시점에 매핑 코드를 자동으로 생성한다.  리플렉션이 없어 **성능이 매우 뛰어나다.** (최근 가장 선호되는 방식)
    
    ```java
    // 의존성 추가 및 설정이 ModelMapper보다 복잡함
    // (build.gradle에 annotationProcessor 'org.mapstruct:mapstruct-processor:...')
    
    // 매퍼 인터페이스 정의
    @Mapper(componentModel = "spring") // Spring Bean으로 등록
    public interface MemberMapper {
        MemberMapper INSTANCE = Mappers.getMapper(MemberMapper.class);
    
        // Member -> MemberResponseDto 매핑
        MemberResponseDto toDto(Member member);
    }
    
    // 서비스 로직
    @Autowired
    private MemberMapper memberMapper; // DI 받아서 사용
    
    public MemberResponseDto findMember(Long memberId) {
        Member member = memberRepository.findById(memberId).get();
        
        // 컴파일 시 생성된 구현체로 매핑
        return memberMapper.toDto(member);
    }
    ```
    
    ---
    
    ### 3. 💡 QueryDSL `Projections` (조회 시점 매핑)
    
    이 방법은 앞의 두 가지와 접근 방식이 다르다.
    
     QueryDSL은 **엔티티를 조회한 후 DTO로 변환**하는 것이 아니라, **DB에서 조회할 때부터 필요한 컬럼만 DTO 객체로 직접** 받아올 수 있다.
    
    - **장점:** **성능 최적화에 가장 유리함.** 엔티티로 변환하는 과정이 생략되고(영속성 컨텍스트에 안 올림), SQL `SELECT` 절에서 필요한 데이터만 가져옴.
    - **단점:** DTO가 QueryDSL 쿼리에 의존하게 됨.
    
    **예시 코드 (Repository 또는 QueryDslSupport 클래스)**
    
    ```java
    // DTO에 @QueryProjection 어노테이션 사용 (권장)
    // (build.gradle에 querydsl-apt 의존성 필요)
    public class MemberResponseDto {
        private String username;
        private String email;
    
        @QueryProjection // 이 생성자를 QueryDSL이 인식하게 함
        public MemberResponseDto(String username, String email) {
            this.username = username;
            this.email = email;
        }
    }
    
    // ----------------------------------------------------
    // 컴파일(build/generated/...) 후 QMemberResponseDto 생성됨
    // ----------------------------------------------------
    
    // QueryDSL 쿼리 로직
    // QMember member = QMember.member;
    
    public MemberResponseDto findDtoById(Long memberId) {
        return queryFactory
            .select(new QMemberResponseDto( // Q-Type DTO 사용
                member.username,
                member.email
            ))
            .from(member)
            .where(member.id.eq(memberId))
            .fetchOne();
    }
    ```
    
    - `Projections.constructor`, `Projections.bean` 등을 사용할 수도 있지만, `@QueryProjection` 방식이 타입 세이프(type-safe)하고 가장 깔끔.
    
    ---
    
    ### 요약 및 추천
    
    | **방법** | **장점** | **단점** |
    | --- | --- | --- |
    | **1. 수동 매핑** | 간단함, 의존성 없음 | 반복 작업(보일러플레이트) |
    | **2. 라이브러리** | 코드 간결, 자동화 | 라이브러리 학습/설정 필요 |
    | **3. QueryDSL `Projections`** | **최고의 성능 (조회 최적화)** | DTO가 쿼리에 의존적 |
    
    **추천:**
    
    - **조회 성능이 중요하다면:** `QueryDSL Projections` (@QueryProjection)
    - **엔티티가 이미 있고, 변환 로직이 복잡하다면:** `MapStruct`
    - **프로젝트가 아주 작고 간단하다면:** `수동 매핑`
    
- 커스텀 페이지네이션
    
    커스텀 페이지네이션은 Spring Data JPA가 기본으로 제공하는 `Page` 객체와 `Pageable` 인터페이스를 직접 사용하지 않고, **페이지네이션 로직을 직접 구현**하는 것을 의미한다.
    
    가장 일반적인 구현 방식은 **두 개의 쿼리를 분리해서 실행**하는 것
    
    1. **Content 쿼리:** 실제 DTO 목록을 가져오는 쿼리 ( `offset` + `limit` 적용)
    2. **Count 쿼리:** 전체 데이터 개수를 세는 쿼리 ( `totalPages` 계산용)
    
    ---
    
    ### 🧐 커스텀 페이지네이션이 필요한 이유
    
    단순한 조회는 Spring Data JPA의 `Pageable`이 매우 편리하지만, 복잡한 쿼리에서는 한계가 있음.
    
    1. **1:N `fetchJoin` 과 페이징의 충돌 (가장 중요)**
        - 1:N `fetchJoin` (컬렉션 페치 조인)을 사용하면 페이징(`offset`, `limit`)이 불가능함.
        - JPA는 경고 로그를 띄우고 *메모리에서 페이징*을 시도하는데, 이는 데이터가 많으면 100% 장애로 이어짐.
        - 이 문제를 해결하려면 `fetchJoin`을 쓰지 않거나, 커스텀 페이지네이션을 도입해야 함!
    2. **복잡한 동적 쿼리**
        - QueryDSL로 매우 복잡한 `where` 절이나 `join` 로직을 구현할 때, `Pageable`을 결합하기 까다롭거나 성능 저하가 발생할 수 있음.
    3. **Count 쿼리 최적화**
        - JPA가 자동으로 생성하는 `count` 쿼리는 `content` 쿼리만큼 복잡할 때가 많아 불필요하게 느림.
        - 커스텀 페이지네이션을 사용하면 `count` 쿼리를 단순하게 최적화할 수 있음.
    
    ---
    
    ### 🚀 구현 핵심 2단계: 쿼리 분리
    
    `Member`를 조회하여 `MemberResponseDto`로 반환한다고 가정하자. 
    
    ### 1단계: Content 쿼리 (데이터 목록 조회)
    
    `offset`과 `limit`을 사용하여 실제 페이지에 표시될 데이터를 조회한다. 
    
    ```java
    // QMember member = QMember.member;
    
    // 1. 입력 파라미터 (Request)
    int page = 0;     // 현재 페이지 (0부터 시작)
    int size = 10;    // 페이지 당 아이템 수
    long offset = (long) page * size;
    
    // 2. Content 쿼리 실행
    List<MemberResponseDto> content = queryFactory
        .select(new QMemberResponseDto( // DTO로 바로 조회
            member.username,
            member.email
        ))
        .from(member)
        // .where(...) // 필요한 경우 동적 조건
        .orderBy(member.createdAt.desc()) // 정렬은 필수!
        .offset(offset) // (page * size) 만큼 건너뛰기
        .limit(size)    // size 만큼 가져오기
        .fetch();
    ```
    
    > ⚠️ (중요) 1:N fetchJoin과 페이징 문제
    > 
    > 
    > 위 `content` 쿼리에서는 `fetchJoin`을 사용하지 않았음.
    > 
    > 만약 1:N 관계(예: `member.getOrders()`)를 DTO에 포함해야 한다면, 이 쿼리에서는 `fetchJoin`을 쓰면 안된다. 대신, 조회된 `content` (DTO 리스트)를 기반으로 `@BatchSize`를 사용하거나 별도 쿼리로 N+1 문제를 해결해야 함.
    > 
    
    ### 2단계: Count 쿼리 (전체 개수 조회)
    
    페이지 계산을 위해 `offset`과 `limit`을 적용하기 *전*의 전체 데이터 개수를 조회한다. 
    
    ```java
    // 3. Count 쿼리 실행 (QueryDSL 5.0 이상 권장 방식)
    Long totalCount = queryFactory
        .select(member.count()) // count() 함수 사용
        .from(member)
        // .where(...) // content 쿼리와 동일한 조건 적용!
        .fetchOne(); // 단 건 조회
    ```
    
    - **주의:** `content` 쿼리에 `where` 조건이 있다면, `count` 쿼리에도 **반드시 동일한 `where` 조건**이 들어가야 함. (단, `orderBy`, `offset`, `limit`은 제외)
    
    ---
    
    ### 📦 DTO로 조합하기
    
    이제 두 결과를 합쳐 클라이언트에 보낼 표준 응답 DTO를 만듦.
    
    ### 1. 요청 DTO (Pageable 대용)
    
    Spring의 `Pageable` 대신 간단한 요청 객체를 만듦.
    
    ```java
    public class CustomPageRequest {
        private int page = 0;  // 기본값 0
        private int size = 10; // 기본값 10
        // ... getter/setter, 생성자
        
        public long getOffset() {
            return (long) page * size;
        }
    }
    ```
    
    ### 2. 응답 DTO (Page<T> 대용)
    
    Spring의 `Page` 인터페이스와 유사한 응답 객체를 만듦.
    
    ```java
    public class PageResponse<T> {
        private List<T> content;      // 데이터 목록
        private int page;             // 현재 페이지
        private int size;             // 페이지 크기
        private long totalElements;   // 전체 데이터 수
        private int totalPages;       // 전체 페이지 수
        private boolean first;        // 첫 페이지 여부
        private boolean last;         // 마지막 페이지 여부
    
        // 생성자
        public PageResponse(List<T> content, CustomPageRequest pageRequest, long totalElements) {
            this.content = content;
            this.page = pageRequest.getPage();
            this.size = pageRequest.getSize();
            this.totalElements = totalElements;
            
            // 페이지 계산
            this.totalPages = (int) Math.ceil((double) totalElements / this.size);
            this.first = (page == 0);
            this.last = (page == totalPages - 1) || (totalPages == 0);
        }
    }
    ```
    
    ### 3. 서비스 로직에서 사용
    
    ```java
    public PageResponse<MemberResponseDto> searchMembers(CustomPageRequest pageRequest) {
        
        // 1. Content 쿼리
        List<MemberResponseDto> content = queryFactory
            .select(...)
            .from(member)
            .where(...) // 검색 조건
            .orderBy(member.createdAt.desc())
            .offset(pageRequest.getOffset())
            .limit(pageRequest.getSize())
            .fetch();
    
        // 2. Count 쿼리
        Long totalCount = queryFactory
            .select(member.count())
            .from(member)
            .where(...) // 동일한 검색 조건
            .fetchOne();
    
        // 3. PageResponse DTO로 조합하여 반환
        return new PageResponse<>(content, pageRequest, totalCount);
    }
    ```
    
    ---
    
    ### ✨ 성능 최적화: `Slice` (무한 스크롤) 방식
    
    `totalPages`나 `totalElements`가 필요 없는 '더보기' (무한 스크롤) 방식이라면, `count` 쿼리를 아예 실행하지 않을 수 있다.
    
    `limit(size)` 대신 `limit(size + 1)`로 1개 더 조회함.
    
    - 조회된 개수가 `size + 1`개 (예: 11개)
    -> **다음 페이지가 있다 (hasNext = true)**. 응답 시에는 10개만 보낸다.
    - 조회된 개수가 `size`개 이하 (예: 10개 또는 그 미만)
    -> **다음 페이지가 없다 (hasNext = false)**.
    
    이 방식은 `count` 쿼리가 아예 실행되지 않아 성능이 매우 좋음!
    
- transform - groupBy
    
    QueryDSL의 `transform` 메서드는, 특히 `groupBy`와 함께 사용될 때, **1:N 관계의 조인 결과를 DTO로 한 번에 매핑**하는 강력한 방법을 제공함.
    
    `fetchJoin`이 페이징과 함께 사용될 수 없는 문제를 해결하기 위한 **가장 효과적인 대안 중 하나다.**
    
    > 💡 SQL로 1:N 조인 → transform(groupBy(...)) → 애플리케이션 메모리에서 그룹핑 → DTO 컬렉션으로 변환
    > 
    
    ---
    
    ### `transform(groupBy(...))`가 필요한 이유
    
    `Team` (1) : `Member` (N) 관계에서, `TeamDto` 안에 `List<MemberDto>`를 포함시키고 싶다고 가정하자
    
    1. **`fetchJoin` 사용 시:**
        - `leftJoin(team.members, member).fetchJoin()`
        - `List<Team>`을 반환하고 `team.getMembers()`를 N+1 없이 사용할 수 있다.
        - **문제점:** 하지만 여기에 `offset().limit()` (페이징)을 적용하면, DB가 아닌 **메모리에서 페이징**을 시도하여 OOM(Out of Memory) 위험이 있다.
    2. **`transform(groupBy(...))` 사용 시:**
        - `fetchJoin()`을 사용하지 *않고* 일반 `leftJoin`을 사용한다.
        - DB에서 `offset().limit()`을 적용하여 딱 필요한 만큼의 "평평한(flat)" 조인 결과만 가져온다.
        - 가져온 결과를 애플리케이션 메모리에서 `transform`을 통해 DTO 계층 구조로 재조립한다.
        - **결과:** **페이징과 1:N DTO 조회를 동시에** 안전하게 수행할 수 있다.
    
    ---
    
    ### 사용 방법 및 예제
    
    `Team` (1) - `Member` (N) 관계를 DTO로 조회하는 예제
    
    ### 1. DTO 준비
    
    `TeamDto`는 `List<MemberDto>`를 필드로 가져야 합니다.
    
    ```java
    // MemberDto
    public class MemberDto {
        private String username;
        // ... 생성자, getter, setter
    }
    
    // TeamDto
    public class TeamDto {
        private Long id;
        private String name;
        private List<MemberDto> members; // 1:N 관계를 표현할 리스트
    
        // ... 생성자, getter, setter
    }
    ```
    
    ### 2. QueryDSL 쿼리 작성
    
    `transform`과 `groupBy`를 사용하는 쿼리
    
    ```java
    import static com.querydsl.core.group.GroupBy.groupBy;
    import static com.querydsl.core.group.GroupBy.list;
    
    // Q-Type 정의
    QTeam team = QTeam.team;
    QMember member = QMember.member;
    
    // 페이징 요청 객체 (커스텀 페이지네이션에서 만든 것)
    CustomPageRequest pageRequest = new CustomPageRequest(0, 10);
    
    // 쿼리 실행
    Map<Long, TeamDto> transformMap = queryFactory
        .from(team) // select() 절은 transform에서 처리하므로 'from'으로 시작
        .leftJoin(team.members, member) // 일반 조인 (fetchJoin 아님)
        
        // --- 페이징 및 조건 적용 (DB 레벨에서 실행) ---
        .where(team.name.startsWith("A")) // 예: 'A'로 시작하는 팀
        .offset(pageRequest.getOffset())
        .limit(pageRequest.getSize())
        .orderBy(team.id.asc())
        // ----------------------------------------
        
        .transform(
            groupBy(team.id).as( // 1. "One" 쪽의 ID (Team ID)로 그룹핑
                Projections.bean(TeamDto.class, // 2. "One" 쪽 DTO (TeamDto)
                    team.id,
                    team.name,
                    
                    // 3. "Many" 쪽 DTO 리스트 (List<MemberDto>)
                    list( // list() 함수로 집계
                        Projections.bean(MemberDto.class,
                            member.username
                        )
                    ).as("members") // TeamDto의 'members' 필드에 매핑
                )
            )
        );
    
    // 4. 결과 변환 (Map -> List)
    // transform(groupBy(...))의 결과는 Map<Key, Dto> 입니다.
    List<TeamDto> content = new ArrayList<>(transformMap.values());
    ```
    
    ---
    
    ### 작동 원리 (단계별)
    
    위 코드는 다음과 같이 동작함.
    
    1. **SQL 실행 (DB):**
        - `FROM team LEFT JOIN member ... WHERE ... ORDER BY ... LIMIT 10 OFFSET 0`
        - 페이징이 적용된 "평평한" 결과셋을 DB에서 가져옴. (`team` 데이터가 `member` 수만큼 중복된 상태)
        
        | **team.id** | **team.name** | **member.username** |
        | --- | --- | --- |
        | 1 | Team A | member1 |
        | 1 | Team A | member2 |
        | 2 | Team B | member3 |
    2. **`transform` (애플리케이션 메모리):**
        - `groupBy(team.id)`: 가져온 결과를 `team.id` (1, 2) 기준으로 메모리에서 그룹핑한다.
            - **Group 1 (Key: 1):** `[ (1, Team A, member1), (1, Team A, member2) ]`
            - **Group 2 (Key: 2):** `[ (2, Team B, member3) ]`
        - `.as(Projections.bean(...))`: 각 그룹을 `TeamDto`로 변환.
            - **Group 1 처리:**
                - `team.id` (1), `team.name` ("Team A")을 `TeamDto`에 매핑.
                - `list(Projections.bean(MemberDto.class, ...))`가 `member.username` ("member1", "member2")을 모아 `List<MemberDto>`를 생성.
                - 이 리스트를 `.as("members")`에 따라 `TeamDto.members` 필드에 주입.
                - **결과:** `TeamDto(id=1, name="Team A", members=[MemberDto("member1"), MemberDto("member2")])`
            - **Group 2 처리:**
                - **결과:** `TeamDto(id=2, name="Team B", members=[MemberDto("member3")])`
    3. **최종 반환:**
        - 결과는 `Map<Long, TeamDto>` (Key는 `team.id`)
        - `transformMap.values()`를 통해 `List<TeamDto>`를 얻을 수 있다.
    
    ---
    
    ### 주의사항
    
    - **메모리 사용:** `transform`은 DB에서 조회한 결과를 **일단 메모리에 모두 로드**한 후 그룹핑을 시작한다. `limit`으로 페이징을 했기 때문에 OOM 위험은 거의 없지만, `limit` 없이 매우 많은 데이터를 조회하면 메모리 문제를 일으킬 수 있다.
    - **1:N:M 관계:** 1:N:M (예: `Team` -> `Member` -> `Orders`)처럼 다중 컬렉션 조인은 `transform`으로도 깔끔하게 처리하기 어렵습니다. 이때는 쿼리를 분리하는 것이 좋다.
    - **중복 제거:** `list()` 대신 `set()`을 사용하면 중복된 `MemberDto`를 제거할 수 있다.
    
    요약하자면, `transform(groupBy(...))`는 **"페이징이 필요한 1:N DTO 조회"** 시나리오를 해결하는 N+1 방지 기술임
    
- order by null
    
    `ORDER BY NULL`은 주로 SQL에서 `GROUP BY`를 사용할 때 발생하는 **불필요한 기본 정렬을 비활성화**하여 쿼리 성능을 최적화하기 위해 사용되는 기법이다.
    
    "정렬을 하지 말라"고 명시적으로 데이터베이스에 알리는 역할을 한다.
    
    ---
    
    ### 🚀 핵심 용도: `GROUP BY`의 암시적 정렬(Implicit Sort) 방지
    
    1. **문제 상황:** MySQL은 `GROUP BY` 절을 실행할 때, 기본적으로 `GROUP BY`의 기준이 된 컬럼으로 암시적인 정렬(implicit sort)을 시도한다.
        - `GROUP BY team.id` 쿼리를 실행하면, DB는 결과를 `team.id` 순서로 정렬하려고 추가 작업을 함.
        - 이 작업은 종종 불필요하며, 데이터가 많을 경우 `Using filesort`라는 비싼 작업을 유발해 성능을 저하시킨다
    2. **해결책:** 쿼리 마지막에 `ORDER BY NULL`을 붙여준다.
        - 이는 "데이터베이스야, `GROUP BY` 했다고 굳이 정렬까지 할 필요 없어"라고 명시적으로 지시하는 것임.
        - 데이터베이스는 이 지시를 받고 불필요한 정렬 작업을 생략하여 쿼리 속도가 향상될 수 있다.
    
    ---
    
    ### 📝 QueryDSL 예제
    
    ### QueryDSL 예제
    
    팀별 회원 수를 세는 쿼리일 때
    
    ```java
    // QMember member = QMember.member;
    
    // 팀별 회원 수를 셀 때, 불필요한 정렬을 방지
    List<Tuple> result = queryFactory
        .select(member.team.id, member.count())
        .from(member)
        .groupBy(member.team.id)
        .orderBy(null) // <-- 불필요한 정렬을 비활성화
        .fetch();
    ```
    
    ---
    
    ### ⚠️ 주의사항 및 한계
    
    1. **`transform(groupBy(...))`와는 다르다.**
        - 이전에 질문하신 `transform(groupBy(...))`는 **SQL 쿼리 실행 후 애플리케이션 메모리**에서 데이터를 그룹핑하는 기능임.
        - `ORDER BY NULL`은 **데이터베이스 SQL 레벨**에서 `GROUP BY`를 사용할 때의 최적화 기법이다.
        - `transform`을 사용할 때는 `ORDER BY NULL`이 아무런 의미가 없음. 오히려 `transform`이 올바르게 그룹핑하려면 `groupBy`의 기준 키(예: `team.id`)로 **먼저 정렬(`orderBy`)**되어 있어야 함.
    2. **DBMS 의존적**
        - 이 최적화는 주로 **MySQL**에서 유효한 방식이다.
        - PostgreSQL, SQL Server 등 다른 DBMS는 `GROUP BY` 시 암시적 정렬을 기본으로 하지 않으므로 `ORDER BY NULL`이 필요 없거나 문법 오류를 일으킬 수 있음.