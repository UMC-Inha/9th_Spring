# QueryDSL에서 FetchJoin 하는 법

# 기본 사용법 (OneToMany, ManyToOne 상황)

```java
//QueryDsl을 통해서 Q클래스를 생성하고 해당 클래스로 객체 생성
QMember member = QMember.member; 
QTeam team = QTeam.team; 

//.join(연관관계 필드, 별칭 Qtype).fetchJoin()
List<Member> result = queryFactory
    .selectFrom(member)
    .join(member.team, team).fetchJoin()
    .fetch();
```

위 코드에서 각 멤버는 하나의 팀에만 속하기 떄문에 ManyToOne 관계이다.

```java
List<Team> teams = queryFactory
    .selectFrom(team)
    .leftJoin(team.members, member).fetchJoin() //leftJoin
    .selectDistinct() //중복제거 
    .fetch();
```

이번 코드에서는 OneToMany 관계이다. 따라서 위와 달라지는 점이 있다.

우리가 위의 코드를 통해 팀들을 조회하며 그에 해당하는 멤버도 조회하는 상황이라면 당연하게 모든 팀들을 조회하고 각 팀에 해당하는 멤버들이 조회되는것을 기대할 것이다.

따라서 leftJoin()을 통해 멤버가 없는 팀도 제외하지 않고 가져올 수 있도록 한다. 또한 중복이 발생할 수 있기 때문에 selectDistinct()를 통해 중복을 제거해준다.



# DTO 매핑 방식 (+DTO안에 DTO)

---

# DTO 매핑 사용이유

```java
List<Member> members = queryFactory
    .selectFrom(member)
    .fetch();
```

간단한 멤버를 조회하는 코드이다. 이 방식으로 멤버들을 조회하면 member 객체가 반환되고 이로 인한 단점이 있다.

- 불필요한 필드까지 노출될 가능성

  → 멤버의 닉네임과 이메일 주소만 필요한 상황을 가정할 때  비밀번호나 개인정보가 포함된 필드까지 전달하게 됨

- 엔티티를 수정할 경우 API 응답 구조까지 영향 (결합도 상승)
- 연관관계가 얽히면 fetch join 등으로 쿼리 성능 최적화가 어려워짐

---

# DTO 매핑 4가지 주요 방식

## 1. Projections.bean()

특징 : setter 기반

장점 : 단순하지만 setter 필요

```java
List<MemberDto> result = queryFactory
    .select(Projections.bean(MemberDto.class,
            member.username,
            member.age))
    .from(member)
    .fetch();
```

[조건]

DTO에 기본 생성자 + setter가 있어야 함.

필드 이름이 DTO와 일치해야 함.

[동작과정]

DTO를 기본 생성자 이용해 생성 후 setter를 이용해 해당 필드 set

## 2. Projections.fields()

특징 : 필드 이름 기반

장점 : 필드명만 일치하면 private 필드도 직접접근 가능

```java
List<MemberDto> result = queryFactory
    .select(Projections.fields(MemberDto.class,
            member.username,
            member.age))
    .from(member)
    .fetch();
```

[조건]

setter 없어도 된다.

이름이 일치해야 하고 다를 경우 .as(”alias”)로 맞춰줘야 한다.

[동작과정]

private 필드라도 리플렉션으로 값이 들어간다

### 리플렉션이란?

프로그램 실행 중에 자기 자신의 구조를 탐색하고 조작할 수 있는 기능

```java
MemberDto dto = new MemberDto();

// 1️⃣ 클래스 정보(Class 객체) 얻기
Class<?> clazz = dto.getClass();

// 2️⃣ 필드 정보 가져오기
Field field = clazz.getDeclaredField("username");

// 3️⃣ private 접근 허용 설정
field.setAccessible(true);

// 4️⃣ 필드에 값 주입
field.set(dto, "영준");

System.out.println(dto.getUsername()); // "영준"
```

위 코드 3번을 통해 private에 직접 접근을 허용 할 수 있다.

ORM, Spring DI, QueryDSL, Hibernate 같은 프레임워크들이 개발자가 명시하지 않은 필드나 메서드를 자동으로 조작할 때 반드시 사용하는 기술

## 3. Projections.constructor()

특징 : 생성자 기반

장점 : 불변 DTO에 유리 → 모든 필드가 final이고 setter 없기때문

```java
List<MemberDto> result = queryFactory
    .select(Projections.constructor(MemberDto.class,
            member.username,
            member.age))
    .from(member)
    .fetch();
```

[조건]

DTO의 생성자 파라미터 순서와 타입이 정확히 일치해야 함.

[주의]

런타임 시점에 타입이 안맞으면 예외 발생 가능 → 컴파일 시점 검증 불가

## 4. @QueryProjection (QDto 생성)

특징 : Q타입 생성

장점 : 타입 안전성이 좋음 (컴파일 타임 검증)

```java
public class MemberDto {
    private final String username;
    private final int age;

    @QueryProjection
    public MemberDto(String username, int age) {
        this.username = username;
        this.age = age;
    }
}
```

MemberDto를 생성하고 @QueryProjection Annotation을 추가.

컴파일 시 QueryDSL용 QMemberDto가 생성.

```java
List<MemberDto> result = queryFactory
    .select(new QMemberDto(member.username, member.age))
    .from(member)
    .fetch();
```

QueryDSL에서는 다음과 사용가능 .select()에 QDto 객체를 통해서!

[장점]

- QMemberDto는 실제 MemberDto의 생성자 시그니처를 그대로 따른다. 따라서 컴파일 시점에 파라미터 타입/순서 오류를 잡을 수 있다.
- new MemberDto를 직접호출 (=리플렉션 사용하지 않음) 따라서 성능이 빠름.

[단점]

- QueryDSL 의존성
- QDto 재빌드 필요 (DTO 변경시 재빌드)
- QueryDSL 없는 환경에선 쓰기 어려움

---

# DTO안에 DTO

```java
public class MemberDto {
    private String username;
    private int age;
    private TeamDto team; 
}

public class TeamDto {
    private String teamName;
}
```

DTO안에 DTO는 위 코드와 같은 상황을 의미한다.

MemberDto 안에 TeamDto를 포함하는 상황이다.

## 1. flat DTO

```java
List<MemberFlatDto> result = queryFactory
    .select(Projections.constructor(MemberFlatDto.class,
            member.username,
            member.age,
            team.name))
    .from(member)
    .join(member.team, team)
    .fetch();
```

위와같이 FlatDto를 이용해 평면 구조로 받고 (team에 대해서 dto를 받지 않고 dto에 필요한 필드를 받음)

```java
List<MemberDto> response = result.stream()
    .map(flat -> new MemberDto(
            flat.getUsername(),
            flat.getAge(),
            new TeamDto(flat.getTeamName())))
    .toList();
```

이후 service 계층에서 다음과 같이 처리

[장점]

- QueryDSL 쿼리는 기존과 크게 다르지 않고 단순
- 타입 안전 유지

[단점]

- 변환 로직 필요

## 2. 중첩 DTO를 Projection으로 직접 생성

```java
List<MemberDto> result = queryFactory
    .select(Projections.constructor(MemberDto.class,
            member.username,
            member.age,
            Projections.constructor(TeamDto.class, team.name))) // ✅ 내부 DTO 직접 생성
    .from(member)
    .join(member.team, team)
    .fetch();
```

TeamDto가 들어가야 하는 부분에 직접 Projections.constructor를 이용해 생성

## 3. @QueryProjection 이용

```java
public class TeamDto {
    private final String name;

    @QueryProjection
    public TeamDto(String name) {
        this.name = name;
    }
}

public class MemberDto {
    private final String username;
    private final int age;
    private final TeamDto team;

    @QueryProjection
    public MemberDto(String username, int age, TeamDto team) {
        this.username = username;
        this.age = age;
        this.team = team;
    }
}
```

TeamDto와 MemberDto를 이용해 QTeamDto,QMemberDto 모두 생성하면

```java
List<MemberDto> result = queryFactory
    .select(new QMemberDto(
        member.username,
        member.age,
        new QTeamDto(team.name)))
    .from(member)
    .join(member.team, team)
    .fetch();
```

다음과 같이 간편하게 구현가능


# 커스텀 페이지네이션

커스텀 페이지네이션은 Spring Data JPA의 기본 Pageable 기능을 쓰지 않고 직접 페이지네이션 로직을 구현하는 방식!

# 간단한 코드로 사용법 이해

정의, 설명보다 기본 코드가 이해하기 더 쉬웠기 때문에 코드로 간단한 설명.

```java
public interface MemberRepositoryCustom {
    Page<MemberDto> findMembers(int page, int size);
}
```

page, size 값을 전달받아 Page<MemberDto>에 받아오는 코드이다.

먼저 인터페이스를 선언.

```java
import org.springframework.data.support.PageableExecutionUtils;

@Override
public Page<MemberDto> findMembers(int page, int size) {
    Pageable pageable = PageRequest.of(page, size);

    // 1️⃣ 데이터 조회 쿼리
    List<MemberDto> content = queryFactory
            .select(Projections.constructor(MemberDto.class,
                    member.username,
                    member.age))
            .from(member)
            .orderBy(member.username.asc())
            .offset(pageable.getOffset())
            .limit(pageable.getPageSize())
            .fetch();

    // 2️⃣ count 쿼리 정의 (아직 실행 X)
    JPAQuery<Long> countQuery = queryFactory
            .select(member.count())
            .from(member);

    // 3️⃣ getPage()를 통해 지연 실행
    return PageableExecutionUtils.getPage(content, pageable, countQuery::fetchOne);
}
```

구현 코드를 보면 크게 3단계로 나누어진다.

1. 데이터를 조회한다.
2. 전체 데이터 개수를 조회하는 과정을 따로 수행한다.
3. new PageImpl<>(content, pagable, total)로 직접 Page를 만든다.

```java
@RequiredArgsConstructor
public class MemberRepositoryImpl implements MemberRepositoryCustom {

    private final JPAQueryFactory queryFactory;

    
    public Slice<MemberDto> findMembers(Long lastId, int size) {

        // 1️⃣ 커서 조건 설정 (처음 페이지면 조건 없이 시작)
        BooleanExpression cursorCondition = (lastId != null)
                ? member.id.lt(lastId) // lastId보다 작은 ID부터 조회
                : null;                // 첫 페이지면 전체에서 시작

        // 2️⃣ 데이터 조회 쿼리
        List<MemberDto> content = queryFactory
                .select(Projections.constructor(MemberDto.class,
                        member.username,
                        member.age))
                .from(member)
                .where(cursorCondition)
                .orderBy(member.id.desc()) // 최신순 정렬 (ID 기준)
                .limit(size + 1)           // 다음 페이지 존재 여부 확인용 +1
                .fetch();

        // 3️⃣ 다음 페이지 존재 여부 판단
        boolean hasNext = false;
        if (content.size() > size) {
            hasNext = true;
            content.remove(size); // 다음 페이지 확인용 데이터 제거
        }

        // 4️⃣ Slice 객체로 반환 (Page보다 가벼움)
        return new SliceImpl<>(content, PageRequest.ofSize(size), hasNext);
    }
}
```

아래와 같이 cursor 기반으로도 구현할 수 있다.

여기까지 봤을 때 내가 직접 생각해본 커스텀의 장점은  데이터 조회 쿼리를 직접 짠다는 것이다. Pageable을 사용하는 경우 이런 부분을 건드릴 수 없었던 것 같다.

### 💡 Slice, Page

여기서 사용된 두 구조체는 Spring Data JPA가 제공하는 표준 페이징 구조체로

두 구조체의 차이점은 Count의 포함 여부이다. 따라서 Page는 일반적인 페이지네이션에서 주로 사용하고 Slice는 무한 스크롤, 커서 기반 상황에서 주로 사용된다.

# 왜 JPA의 Pageable 대신?

### → 개발자가 SQL 전부를 컨트롤 할 수 있다!

예를들어 Pageable만을 이용하면 dto 반환이 불가하기 때문에 멤버를 dto로 받아오며 페이징하는 기술은 pageable만으로는 구현이 불가능하다.

```java
Page<Member> findAll(Pageable pageable);
```

위 코드와 같이 Member를 반환하는 페이징은 쉽게 가능하지만 Member를 MemberDto로 바꿀 수 없다.

하지만 커스텀 페이지네이션을 사용하면 개발자가 직접 데이터를 조회하는 쿼리를 짤 수 있기 떄문에 멤버를 dto로 받아오며 페이징할 수 있다.

일부 상황에서 count 를 지연로딩하여 생략할 수 있다고 한다. 하지만 이는 특별한 상황이나 일부 케이스이기 때문에 성능 향상은 부과효과 정도로 볼 수 있다.


# transform - groupBy

> groupBy() : SQL의 GROUP BY 와 다름, 결과를 메모리에서 그룹화
>

> transform() : 조회 결과를 특정 구조 (Map, DTO, List 등) 으로 변환
>

# 사용하는 상황과 사용법

```java
(1)
teamA | member1  
teamA | member2  
teamB | member3  

(2)
teamA -> [member1, member2]  
teamB -> [member3]
```

한명의 멤버가 하나의 팀을 가지는 연관관계에서 (1)처럼 튜플 형태로 데이터를 조회했다고 가정하자.

(1)의 데이터를 DB로 부터 받아 (2)의 데이터 처럼 각 팀별로 멤버를 묶으려고 한다.

```java
Map<String, List<String>> result = queryFactory
    .from(member)
    .join(member.team, team)
    .transform(groupBy(team.name).as(list(member.username)));
```

groupBy()에 들어가는 값이 함수 이름 그대로 무엇으로 group화 할지 즉 이 상황에서는 team의 이름으로 그룹화를 한다. 따라서 이 경우 team 이름이 Map에서 key의 역할을 하게된다.

.as(list())이렇게 list를 넣으면 값(member.username)을 리스트 형태로 map의 value로 설정

⇒ Map<String, List<String>>

.as(set())이렇게 set을 넣으면 값(member.username)을 set 형태로 map의 value로 설정

⇒ Map<String, Set<String>>

# 주의사항

- SQL의 GROUP BY가 아닌 DB로부터 데이터를 가져와서 자바에서 그룹화
- 메모리상에서 그룹화하기 때문에 데이터의 양이 많으면 메모리 사용량 증가
- 1 : N 관계 결과를 깔끔하게 묶어야 할 때 유용 (위의 멤버 팀 관계같은 상황)
# order by null

# order by 에 대해 기존에 알고 있던 것!

order by 숫자 ASC → 숫자 오름차순 1,2,3,4,5…

order by 숫자 DESC → 숫자 내림차순 10,9,8,7,6,5…

order by 문자 ASC → 알파벳순 (사전순)

order by 문자 DESC → 알파벳 역순

order by 날짜 ASC → 오래된 날짜부터 최신날짜로

order by 날짜 DESC → 최신날짜에서 오래된 날짜로

여러개의 값으로 정렬할 때는 여러개의 column을 명시하면 먼저 명시한 column 기준으로 먼저 정렬 후 첫번째 column이 동일한 데이터에 대해 두번째 명시한 column으로 정렬

NULL 값이 포함된 컬럼을 정렬하면 다음과 같이 정렬됨 (이는 Mysql 기준으로 DBMS마다 다름)

ASC → NULL값이 위로

DESC → NULL값이 아래로

⭐이번에 order by null 관련 검색으로 알게 되었는데 queryDSL 기본문법에서 null이 앞이나 뒤에 오도록 (ASC, DESC 관계없이 명시해서) 할 수 있다고 한다.

```java
List<Member> members = queryFactory
				.selectFrom(member)
				.where(member.age.eq(10))
				.orderBy(member.age.desc(), member.username.asc().nullsLast()) //반대는 nullsFirst()
				.fetch();
```

# order by null이란?

정렬X 를 명시적으로 표현한 것!

왜? 정렬이 필요없는 상황에서 정렬을 하는것은 성능에만 악영향

### 1. GROUP BY 이후에 사용

DB마다 다르지만 MySQL의 경우 GROUP BY 실행 후 내부적으로 자동 정렬 수행

```sql
SELECT name, team_id FROM member GROUP BY team_id
```

이 경우 team_id로 묶이고 자동으로 team_id에 의해서 정렬이 수행됨

만약에 정렬이 필요없는 상황이라면? → 불필요한 정렬로 인해 성능저하

```sql
SELECT team_id, COUNT(*) FROM member GROUP BY team_id ORDER BY NULL;
```

따라서 이렇게 정렬이 필요없는 상황에서는 ORDER BY NULL을 명시함으로써 정렬을 수행하지 않고 이를 통해 성능을 개선시킬 수 있다.

### 2. UNION (DISTINCT)

UNION은 기본적으로 중복 제거(DISTINCT)를 수행 따라서 내부적으로 자동으로 정렬 수행

중복 제거가 필요 없는 상황이라면 ORDER BY NULL 명시

이와 비슷하게 UNION ALL을 사용하는 경우 이는 중복을 애초에 허용하는 버전이기 때문에 정렬이 의미없다 따라서 ORDER BY NULL 명시가능


# 구현법 의문점

실습 미션 구현하면서 생긴 의문

ReviewRepository extends JpaRepository → 이거는 간단한 상황에서만 사용하고

QueryDsl, QueryDslImpl → 이거는 복잡한 상황에서만 사용하면

필요에 따라 서비스 계층이 리포지토리 2개(Dsl,Jpa)를 주입받아서 쓰는게 더 잘 나누어지는거 같고 간단하지 않나? 도대체 왜 extends JpaRepository<>, ReviewQueryDsl을 통해 복잡한 상황을 만들까? → 복잡해서 이해가 안됐음

내가 생각한 방식의 장점

- 역할 명확히 분리
- 유지보수하기 쉬움 (DSL과 JPA가 분리되어 있기 때문에)

내가 생각한 방식의 단점

- Service에서 2개의 Repository 주입

---

자료에서 설명된 방식

기존에 존재하던 ReviewRepository (JPA만 쓰던) 에

```java
public interface ReviewRepository extends JpaRepository<Review, Long>, ReviewQueryDsl
```

위와같이 QueryDsl Interface를 추가.

ReviewQueryDsl 이라고 명시했을 때 스프링에서 자동으로 ReviewQueryDsl + Impl이라는 이름의 클래스를 찾아서 연결을 해줌

따라서 결국 스프링이 자동으로 연결을 해주기 때문에 1번 방식과 2번 방식 모두 코드를 작성하는 측면에서는 똑같고 Service 계층에서 Repository를 한번만 주입하면 되는 편리함이 생긴다.

---

결론

→DSL 사용법을 제대로 이해를 못해서 불편한 것 같다고 생각했지만 연결하는 동작을 스프링이 알아서 진행해주기 때문에 특별히 단점이 아니었다..