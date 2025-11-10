# QueryDSL에서 Fetch Join 하는 법

## 🔸 Fetch Join 미적용
```java
QMember member = QMember.member;
QTeam team = QTeam.team;

Member result = queryFactory
    .selectFrom(member)
    .join(member.team, team)
    .where(member.id.eq(1L))
    .fetchOne();
```

## 🔸 Fetch Join 적용
```java
QMember member = QMember.member;
QTeam team = QTeam.team;

Member result = queryFactory
    .selectFrom(member)
    .join(member.team, team).fetchJoin()   // ✅ fetch join 적용
    .where(member.id.eq(1L))
    .fetchOne();
```

---

## ✅ Fetch Join이 잘 적용됐는지 확인하는 방법
```java
boolean loaded = emf.getPersistenceUnitUtil().isLoaded(result.getTeam());
assertThat(loaded).as("패치 조인 적용 여부").isTrue();
```

**Member 엔티티의 `team` 필드(연관 객체)** 가
- **지연 로딩 상태인지 (아직 로드 안 됐는지)**
- **이미 로딩되어 있는지 (즉시 로딩됐는지)**
- isLoaded(result.getTeam()) →
  JPA가 Member 엔티티의 team 필드를 이미 로딩했는지(=DB에서 가져왔는지) 검사
```java
QMember member = QMember.member;
QOrder order = QOrder.order;

List<Member> members = queryFactory
    .selectFrom(member)
    .leftJoin(member.orders, order).fetchJoin()   // 컬렉션 fetch join
    .distinct()                                     // 중복 제거
    .where(member.status.eq(Status.ACTIVE))
    .fetch();
```
---
**컬렉션(OneToMany) 연관관계 FetchJoin 예시 컬렉션을 fetch join할 때는 중복 결과가 생기기 쉬우니 distinct()를 함께 쓰는 것이 일반적**
```java

QMember member = QMember.member;
QOrder order = QOrder.order;

List<Member> members = queryFactory
    .selectFrom(member)
    .leftJoin(member.orders, order).fetchJoin()   // 컬렉션 fetch join
    .distinct()                                     // 중복 제거
    .where(member.status.eq(Status.ACTIVE))
    .fetch();
```
JPA가 중복 제거를 자동으로 해주긴 하지만 List<Member> 형태로 가져오면, SQL 결과 자체는 중복가능

---

# DTO 매핑 방식 (+DTO안에 DTO)

## DTO란?

Entity 객체와 달리 각 계층끼리 주고받는 우편물이나 상자의 개념입니다. 순수하게 데이터를 담고 있다는 점에서 Entity 객체와 유사하지만, 목적 자체가 전달이므로 읽고, 쓰는 것이 모두 가능하고, 일회성으로 사용되는 성격이 강합니다.

---

## 엔티티(Entity)와 DTO(Data Transfer Object) 사이의 매핑

### 1. 자바 코드 직접 매핑
```java
TeamDto teamDto = new TeamDto(member.getTeam().getName());
MemberDto memberDto = new MemberDto(member.getUsername(), member.getAge(), teamDto);
```

#### 장점
- 객체 변환을 위한 별도의 과정을 거치지 않고 메서드 호출만 하기 때문에 성능에 대한 영향이 없다.
- 이름이 다른 필드 간의 매핑도 그저 Getter 등을 작성하여 올바르게 조합하기만 하면 된다.
- 매핑하는 필드 타입이 다른 경우에 컴파일 타임에 이를 식별할 수 있다.

#### 단점
- 객체의 필드 명 변경이나 추가 시 매핑하는 코드 부분도 같이 수정하여야 한다. (변경 지점이 늘어날 수 있다.)
- 필드가 너무 많거나 조합하는 형태의 데이터가 많다면 흔히 말하는 휴먼 에러가 발생할 수 있다. (다른 필드와의 매핑이나 데이터 누락 등)

---

### 2. MapStruct

```java
@Mapper
public interface MemberMapper {
    MemberMapper INSTANCE = Mappers.getMapper(MemberMapper.class);

    @Mapping(source = "team.name", target = "team.name")
    MemberDto toDto(Member member);
}

MemberDto dto = MemberMapper.INSTANCE.toDto(member);
```
#### 장점
- 간결한 코드 작성이 가능하다.
- 객체 필드의 변경 사항이 다른 로직에 영향을 주지 않는다.
- 컴파일 시점에 코드를 생성하면서 타입이나 매핑이 불가능한 상태 등의 문제가 발생한 경우 컴파일 에러를 발생시킨다. *이는 상대적으로 런타임에서 안전성을 보장한다.*
- 앞서 보았던 자바 코드 매핑 방식과 같은 수준의 성능을 가진다.

#### 단점
- 전혀 다른 형태의 필드 매핑을 시도하는 경우 제공되는 기능으로 해결 가능한 경우가 많으나, Mapping 로직이 매우 복잡해진다.
- 변경 불가능한 필드에 대한 매핑을 제공하지 못한다. (final 필드 - Constructor 주입)
- Lombok Library와 충돌이 발생할 수 있다. (실제로는 Lombok annotation processor가 getter나 builder 등을 만들기 전에 mapstruct annotation processor가 동작하여 매핑할 수 있는 방법을 찾지 못해 발생하는 문제이다. )

---

### 3. ModelMapper

```java
ModelMapper modelMapper = new ModelMapper();

// TeamDto 내부 매핑 설정
modelMapper.typeMap(Member.class, MemberDto.class)
           .addMappings(mapper -> mapper.map(src -> src.getTeam().getName(),
                                               (dest, v) -> dest.getTeam().setName((String)v)));

// 변환
MemberDto dto = modelMapper.map(member, MemberDto.class);
```
#### 장점
- 간결한 코드 작성이 가능하다.
- 일반적으로 필드 변경 사항에 대해서 고려하지 않아도 된다.
- Lombok 라이브러리와 충돌없이 같이 사용할 수 있다. (런타임에서 객체를 분석하고 매핑하기 때문에)

#### 단점
- 런타임에 매핑하므로 성능이 느림
- 내부 동작이 블랙박스라 디버깅 어려움
- 필드 이름이 다르면 제대로 안 됨

---

## QueryDSL은 **쿼리 실행 시 바로 DTO에 매핑** 할 수 있다.

엔티티를 먼저 가져오지 않아도 되기 때문에 성능이 좋다

```java
List<MemberDto> result = queryFactory
    .select(Projections.constructor(MemberDto.class,
        member.username,
        member.age,
        member.team.name))
    .from(member)
    .fetch();
```

---

## DTO 안에 DTO를 QueryDSL로 매핑하는 방법
```java
List<MemberDto> result = queryFactory
    .select(Projections.constructor(MemberDto.class,
        member.username,
        member.age,
        Projections.constructor(TeamDto.class, member.team.name)  // 내부 DTO 매핑
    ))
    .from(member)
    .fetch();
//constructor 말고 fields,bean으로도 가능
```

| 구분 | 설명 | 비고 |
| --- | --- | --- |
| `Projections.constructor()` | 생성자 기반 매핑 | DTO 안 DTO 매핑에 가장 안정적 |
| `Projections.fields()` | 필드명 기반 매핑 | 필드 이름이 동일해야 함 |
| `Projections.bean()` | setter 기반 매핑 | setter 있어야 작동 |

---

문제는

**QueryDSL의 `Projections.constructor()`, `fields()`, `bean()`은 ‘단일(Flat) 구조의 DTO’만 매핑 가능하다.**

❌ 컬렉션(List<>,Set<>)이나 중첩된 DTO 필드를 한 번에 매핑하는 건 불가능하다.
```java
class MemberDto {
    private Long id;
    private String name;
    private List<MovieDto> movies;
}
```
---

## 해결법:
- 먼저 상위 DTO(MemberDto) 조회
- 하위 DTO(MovieDto) 조회
- Map으로 그룹핑
- MemberDto에 주입

---

# 커스텀 페이지네이션

---

### **📌 1. 개념**

Spring Data JPA에서 제공하는 기본 `Pageable`은 단순하고 편리하지만,

복잡한 쿼리(`fetch join`, `DTO 매핑`, `동적 검색`)를 수행할 때는 한계가 있다.

이러한 상황에서 개발자가 직접 **content 쿼리(목록 조회)** 와 **count 쿼리(전체 개수 조회)** 를 나누어 작성하여 페이징 로직을 제어하는 방식을 **커스텀 페이지네이션(Custom Pagination)** 이라고 한다.

---

### 📌 2. 기본 Pageable의 한계

기본적으로 `Pageable`은 **엔티티(Entity)** 기반으로 동작하며, 내부적으로 두 개의 쿼리를 자동 실행한다.

| 쿼리 종류 | 설명 |
| --- | --- |
| content 쿼리 | 실제로 페이지에 보여줄 데이터 조회 (`limit`, `offset` 포함) |
| count 쿼리 | 전체 데이터 개수를 세는 쿼리 (`count()` 사용, `limit` 없음) |

그러나 다음과 같은 경우에는 Pageable이 정상 작동하지 않는다.

1. **`fetch join`** 사용 시: Hibernate가 메모리 페이징으로 처리 → 성능 저하 및 중복 발생
2. **DTO 매핑 시**: `Pageable`이 자동 생성하는 count 쿼리를 만들 수 없음
3. **복잡한 조건(QueryDSL, group by 등)** 이 포함된 경우 자동 count 생성 실패

이로 인해 엔티티 기반 Pageable만으로는 실무 요구사항을 만족시키기 어렵다.

---

### 📌 3. 커스텀 페이지네이션의 원리

커스텀 페이지네이션은 다음 두 가지 쿼리를 **명시적으로 분리**해서 작성한다.

- **content 쿼리:** 현재 페이지에 보여줄 데이터 목록 조회 (`limit`, `offset` 포함)
- **count 쿼리:** 전체 데이터 개수 계산 (성능 최적화 가능)

이 과정을 통해 `fetch join`과 `DTO 매핑`을 안전하게 사용할 수 있고,

count 쿼리를 단순화하여 **성능 향상**도 가능하다.
```java
@Service
@RequiredArgsConstructor
public class ReviewQueryService {

    private final JPAQueryFactory queryFactory;

    public Page<ReviewResponse> searchReviews(Member member, String keyword, Pageable pageable) {
        QReview review = QReview.review;
        QMember m = QMember.member;

        // ✅ 1) content 쿼리 — 실제 데이터 조회 (fetch join + DTO 매핑)
        List<ReviewResponse> content = queryFactory
                .select(Projections.constructor(ReviewResponse.class,
                        review.id,
                        review.title,
                        review.content))
                .from(review)
                .join(review.member, m).fetchJoin()
                .where(m.eq(member)
                        .and(review.content.containsIgnoreCase(keyword)))
                .offset(pageable.getOffset())
                .limit(pageable.getPageSize())
                .fetch();

        // ✅ 2) count 쿼리 — 전체 개수 계산 (fetch join 제거)
        long total = queryFactory
                .select(review.count())
                .from(review)
                .join(review.member, m)
                .where(m.eq(member)
                        .and(review.content.containsIgnoreCase(keyword)))
                .fetchOne();

        // ✅ 3) Page 객체 생성 및 반환
        return new PageImpl<>(content, pageable, total);
    }
}
```

이 방식은 JPA의 자동 Pageable과 달리,

- DTO 매핑 자유롭고
- fetch join 사용 가능하며
- count 쿼리를 단순화하여 성능 최적화까지 가능하다.

---

### TiP

- `fetch join`은 **ManyToOne, OneToOne** 관계까지만 사용하는 것이 안전하다.  
  OneToMany에서는 데이터 중복이 발생할 수 있다.

- 복잡한 count 쿼리는 생략하고 `Slice`로 대체해도 된다.

| 구분 | `Page<T>` | `Slice<T>` |
| --- | --- | --- |
| **count 쿼리** | 실행함 (`select count(*)`) | 실행하지 않음 |
| **전체 페이지 수** | 계산 가능 | 계산 불가 |
| **hasNext()** | 전체 데이터 기반으로 판단 | 다음 페이지 데이터가 하나 더 있는지로 판단 |
| **사용 목적** | 전체 페이지 탐색 UI (e.g. 1/10 페이지) | 무한 스크롤, 다음 페이지 버튼 |

- count 쿼리에는 `fetch join`을 포함시키지 않는 것이 원칙이다.
- `@BatchSize`로 지연 로딩 최적화도 가능하다.
```java
@Entity
public class Review {
    @Id @GeneratedValue
    private Long id;

    private String content;

    @ManyToOne(fetch = FetchType.LAZY)
    @BatchSize(size = 10) // 👈 10개씩 한 번에 로딩
    private Member member;
}
```

리뷰 10개를 조회시 → member 조회를 **1번 추가 쿼리로 묶어서 처리**

```sql
select * from member where member_id in (?, ?, ?, ..., ?)
```
---
| 구분 | 기본 Pageable | 커스텀 페이지네이션 |
| --- | --- | --- |
| 대상 | 엔티티 | DTO / 엔티티 혼용 가능 |
| count 쿼리 | 자동 생성 | 직접 작성 |
| fetch join | ❌ 지원 안됨 | ✅ 가능 (count 분리 시) |
| DTO 매핑 | ❌ 제한적 | ✅ 자유로움 |
| 성능 제어 | 제한적 | ✅ 최적화 가능 |

---

# transform - groupBy

**DTO 안에 컬렉션(List) 형태로 담는 경우**에 매우 유용한 기능

---

### 예시
```java
class MemberDto {
    private Long id;
    private String name;
    private List<MovieDto> movies;
}

class MovieDto {
    private Long id;
    private String title;
}
```

| member_id | member_name | movie_id | movie_title |
| --- | --- | --- | --- |
| 1 | 홍길동 | 10 | 인셉션 |
| 1 | 홍길동 | 11 | 인터스텔라 |
| 2 | 이순신 | 12 | 글래디에이터 |

DTO로 바로 매핑하면 MemberDto가 중복으로 생성될 수도 있다.

---

### `groupBy()`

- SQL의 `GROUP BY`랑 비슷하지만, **QueryDSL에서 메모리 내 그룹핑용**
- 특정 필드를 기준으로 결과를 그룹화 → Map 구조로 변환
```java
GroupBy.groupBy(member.id)
```
---

### `transform()`

- QueryDSL이 만든 **쿼리 결과를 DTO 구조로 변환**
- 실제로는 **Map 또는 List** 형태로 반환 가능
```java
.transform(GroupBy.groupBy(member.id).list(...))
```
```java
List<MemberDto> result = queryFactory
    .from(member)
    .leftJoin(member.movies, movie)
    .transform(
        GroupBy.groupBy(member.id).list(
            Projections.constructor(MemberDto.class,
                member.id,
                member.name,
                GroupBy.list(
                    Projections.constructor(MovieDto.class,
                        movie.id,
                        movie.title
                    )
                )
            )
        )
    );
```
---

### 결과 예시
```java
[
  {
    "id": 1,
    "name": "홍길동",
    "movies": [
      {"id": 10, "title": "인셉션"},
      {"id": 11, "title": "인터스텔라"}
    ]
  },
  {
    "id": 2,
    "name": "이순신",
    "movies": [
      {"id": 12, "title": "글래디에이터"}
    ]
  }
]
```

---

# order by null

SQL에서 `NULL`은 '값이 없음'을 의미하지만, 이는 단순히 비어 있다는 의미가 아니라  
'값이 아직 정해지지 않았거나 알 수 없는 상태'를 뜻한다.  
이처럼 `NULL` 값은 비교나 정렬에서 특별한 처리를 필요로 하며,  
DBMS마다 `NULL` 정렬 처리 방식이 다르다.

---

### 📊 DBMS별 NULL 정렬 규칙

| DBMS | ASC (오름차순) | DESC (내림차순) | 이유 요약 |
| --- | --- | --- | --- |
| **MySQL** | 맨 위 (`NULL`을 가장 작은 값으로 봄) | 맨 아래 (`NULL`을 가장 큰 값으로 봄) | `NULL`을 “-∞”로 취급 |
| **PostgreSQL** | 맨 아래 | 맨 위 | `NULL`을 “+∞”로 취급 |
| **Oracle** | 맨 아래 | 맨 위 | PostgreSQL과 동일 |
| **SQL Server** | 맨 위 (기본 설정) | 맨 아래 (기본 설정) | MySQL처럼 “작은 값”으로 취급 |

---

### ⚙️ QueryDSL로 NULL 정렬 직접 제어하기

| 사용법 | 의미 | 설명 |
| --- | --- | --- |
| `.asc().nullsFirst()` | 오름차순 시 NULL을 맨 앞으로 | SQL → `ASC NULLS FIRST` |
| `.asc().nullsLast()` | 오름차순 시 NULL을 맨 뒤로 | SQL → `ASC NULLS LAST` |
| `.desc().nullsFirst()` | 내림차순 시 NULL을 맨 앞으로 | SQL → `DESC NULLS FIRST` |
| `.desc().nullsLast()` | 내림차순 시 NULL을 맨 뒤로 | SQL → `DESC NULLS LAST` |


