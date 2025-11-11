### QueryDSL에서 FetchJoin 하는 법

**Fetch Join이란?**

기본적으로 JPA에서는 연관된 엔티티가 LAZY로 설정되어 있으면, 실제로 접근할 때마다 추가 쿼리(N+1) 가 발생한다.

```java
//예시! Review -> Store (다대일 관계)
Review review = reviewRepository.findById(1L).get();
String storeName = review.getStore().getName();
```

이걸 한 번에 가져오도록 하는게 Fetch Join이다.

**JPQL 기준으로 Fetch Join**

```java
select r from Review r
join fetch r.store
where r.id = :id
```

이렇게 하면 Review와 Store가 한 번의 쿼리로 조인되어 즉시 로딩된다.

**QueryDSL에서 Fetch Join문법**

기본 구조

```java
JPAQueryFactory queryFactory = new JPAQueryFactory(em);

QReview review = QReview.review;
QStore store = Qstore.store;

List<Review> results = queryFactory
        .selectFrom(review)
        .join(review.store, store).fetchJoin()
        .fetch();
```

- .join(review.store, store) → Review와 Stroe를 조인
- .fetchJoin() → 조인 시점에 함께 select해서 로딩
- .fetch() → 리스트 결과 반환

**Left Fetch Join**

관계가 optional일 경우에는 leftJoin()을 자주 사용한다.

```java
List<Review> results = queryFactory
        .selectFrom(review)
        .leftJoin(review.store, store).fetchJoin()
        .leftJoin(store.region, region).fetchJoin()
        .fetch();
```

→ 이렇게 하면 Review 조회 시 Store, Region까지 한 번에 모두 불러온다. (store가 null이어도 Review 포함됨!)

### DTO 매핑 방식 (+DTO안에 DTO)

**DTO(Data Transfer Object)**

→ 계층 간 데이터 전달용 객체

→ 엔티티를 직접 노출하지 않기 위해 사용

→ Controller ↔ Service ↔ Repository 간 데이터를 안전하게 전달

**DTO 매핑 방식의 종류**

1. 수동 매핑 → 직접 필드 하나하나 할당 → 가장 명시적, 디버깅 쉬움, 하지만 코드 길어짐
2. ModelMapper / MapStruct 자동 매핑 → 라이브러리 사용 → 반복 코드 줄어듦, 유지보수 편함
3. QueryDSL / JPQL Projection → DB 조회 시 바로 DTO로 매핑 → 성능 좋고 효율적, read 전용에 적합

**수동 매핑 방식**

단일 DTO 예시 →

```java
//Entity 
public class User {
	private Long id;
	private String name;
	private int age;
}

//DTO
public class UserResponseDTO{
	private Long id;
	private String name;
	
	public UserResponseDTO(User user) {
		this.id = user.getId();
		this.name = user.getName();
	}
}
```

장점: 명시적이라 디버깅 쉬움

단점: DTO 많아질수록 코드량 증가

DTO 안에 DTO가 포함된 경우 (중첩 DTO) 예시 →

```java
//Entity 구조
public class Review{
	private Long id;
	private String content;
	private User user;
}

public class User{
	private Long id;
	private String name;
}
```

**(1) DTO 안에 DTO 넣기**

```java
//UserDTO
public class UserDTO {
	private Long id;
	private String name;
	
	public UserDTO(User user){
		this.id = user.getId();
		this.name = user.getName();
	}
}

//ReviewDTO
public class ReviewDTO {
	private Long id;
	private String content;
	private UserDTO user;
	
	public ReviewDTO(Review review) {
		this.id = review.getId();
		this.content = review.getContent();
		this.user = new UserDTO(review.getUser());
	}
}
```

→ 이렇게 하면 JSON 응답은 다음과 같다.

```java
{
	"id": 1,
	"content": "리뷰 내용"
	"user": {
		"id": 3,
		"name": "해원"
	}
}
```

장점: 명확한 구조, API 응답 시 직관적

단점: 매번 매핑 코드 작성 필요 ( → MapStruct로 해결 가능)

**(2) MapStruct 자동 매핑 예시**

```java
@Mapper(componentModel = "spring")
public interface ReviewMapper{
	ReviewDTO toDTO(Review review);
	UserDTO toDTO(User user):
}
```

→ Review안에 User가 포함되어 있어도 UserDTO 매퍼를 알아서 호출해서 ReviewDTO 안에 매핑해줌.

```java
Review review = reviewRepository.findById(id).orElseThrow();
ReviewDTO dto = reviewMapper.toDTO(review);
```

장점: 중첩 DTO 매핑까지 자동으로 처리

**(3) QueryDSL Projection으로 바로 DTO 매핑**

DTO를 쿼리 결과로 직접 매핑한다.

```java
// ReviewDTO
public class ReviewDTO {
    private Long id;
    private String content;
    private String userName;

    public ReviewDTO(Long id, String content, String userName) {
        this.id = id;
        this.content = content;
        this.userName = userName;
    }
}

// Repository
public List<ReviewDTO> findAllReviewDTOs() {
    QReview review = QReview.review;
    QUser user = QUser.user;

    return queryFactory
        .select(new QReviewDTO(
            review.id,
            review.content,
            user.name
        ))
        .from(review)
        .join(review.user, user)
        .fetch();
}
```

장점: 성능 좋음(엔티티 → DTO 변환 과정 생략)

단점: 읽기 전용, 복잡한 로직엔 부적합

### 커스텀 페이지네이션

**페이지네이션이란?**

Pagination은 데이터를 여러 페이지로 나누어 클라이언트에 전달하는 기법이다.

DB 쿼리에선 이런 식으로 사용한다 👇

```java
SELECT * FROM review ORDER BY id DESC LIMIT 10 OFFSET 20
```

**Spring Data JPA의 기본 페이지네이션**

Spring은 기본적으로 Pageable과 Page를 제공한다.

```java
public Page<Review> findAll(Pageable pageable);
```

**커스텀 페이지네이션이 필요한 이유**

기본 Pageable로는 단순 Entity 기준 조회만 가능하고, DTO 변환이나 복잡한 조건(검색, 필터, 정렬 등)을 추가하기 어렵다.

→ 즉, 조회결과를 예쁘게 가공하거나, Join된 DTO로 반환하려면 직접 페이지네이션을 만들어야 한다.

커스텀 페이지네이션은 주로 이렇게 구성된다. 👇

```java
Controller → Service → Repository (QueryDSL or JPQL)
                          ↓
                     PageResponse<T> ← totalCount, currentPage, dataList
```

**예시로 흐름 이해하기**

**(1) Controller**

요청이 들어오면 page, size를 받음.

```java
@GetMapping("/reviews")
public PageResponse<ReviewDTO> getReviews(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size) {
    return reviewService.getReviewPage(page, size);
}
```

→ 여기서 PageResponse는 직접 만든 응답용 페이지 DTO이다.(content, totalCount, totalPage 이런 메타데이터를 담는 용도)

**(2)Service**

컨트롤러에서 받은 page,size를 그대로 repository에 넘겨줌.

```java
@Service
@RequiredArgsConstructor
public class ReviewService {
    private final ReviewCustomRepository reviewRepository;

    public PageResponse<ReviewDTO> getReviewPage(int page, int size) {
        return reviewRepository.findReviewsWithPaging(page, size);
    }
}
```

**(3) Repository** ⭐

커스텀 페이지네이션이 실제로 구현되는 부분!

```java
@Repository
@RequiredArgsConstructor
public class ReviewCustomRepository {

    private final JPAQueryFactory queryFactory;

    public PageResponse<ReviewDTO> findReviewsWithPaging(int page, int size) {
        QReview review = QReview.review;
        QUser user = QUser.user;

        long offset = (long) page * size; // 시작점 계산

        // 데이터 가져오기
        List<ReviewDTO> reviews = queryFactory
                .select(Projections.constructor(ReviewDTO.class,
                        review.id,
                        review.content,
                        user.name))
                .from(review)
                .join(review.user, user)
                .orderBy(review.id.desc())
                .offset(offset)     // 몇 번째부터 가져올지
                .limit(size)        // 몇 개 가져올지
                .fetch();

        // 전체 개수 조회
        long totalCount = queryFactory
                .select(review.count())
                .from(review)
                .fetchOne();

        int totalPages = (int) Math.ceil((double) totalCount / size);

        return new PageResponse<>(reviews, page, totalPages, totalCount);
    }
}
```

여기서 중요한 부분

→ offset/limit계산 → 몇 번째부터 몇 개 가져올지

→ count 쿼리 분리 → 전체 데이터 수 계산

→ DTO로 select → Entity가 아니라 DTO 형태로 직접 조회

**(4) PageResponse(응답 포맷 정의)**

```java
@Getter
@AllArgsConstructor
public class PageResponse<T> {
    private List<T> content;     // 실제 데이터
    private int currentPage;     // 현재 페이지 번호
    private int totalPages;      // 전체 페이지 수
    private long totalElements;  // 전체 데이터 개수
}
```

**커스텀 페이지네이션의 장점**

1. 유연성 → 쿼리/DTO/정렬/필터 완전 제어 가능
2. 성능 최적화 → 불필요한 join, fetch 최소화 가능
3. API 일관성 → PageResponse형태로 통일 가능
4. 비즈니스 로직 결합 가능 → 조건부 페이징, 필터 추가 등 가능

**주의할점**

1. count 쿼리 → 별도로 실행하므로 비용이 듬(데이터 많을수록 느려짐_
2. offset 성능 저하 → 페이지가 깊어질수록 느려짐

### transform - groupBy

1. **QueryDSL 에서의 transform() + groupBy()**

   transform()이나 groupBy()는 QueryDSL에서 복잡한 조회 결과를 DTO나 Map 형태로 묶어낼 때 사용된다.

   단순히 .fetch()를 하면 List<Tuple> 형태로 평면적인 결과가 나온다. 그런데 특정 컬럼 기준으로 그룹화해서 하나의 DTO 안에 여러개의 하위 리스트를 넣고 싶을 때가 있다.

   예를 들어 하나의 User에 여러 Review를 같이 담고 싶다고 하자.

    ```java
    List<UserWithReviewsDTO> results = queryFactory
        .from(user)
        .leftJoin(review).on(review.user.eq(user))
        .transform(
            groupBy(user.id).list(
                Projections.constructor(UserWithReviewsDTO.class,
                    user.id,
                    user.name,
                    list(Projections.constructor(ReviewDTO.class,
                        review.id,
                        review.content,
                        review.score))
                )
            )
        );
    ```

    1. groupBy([user.id](http://user.id)) → 결과를 user.id 기준으로 그룹화함. 같은 유저의 리뷰들을 묶어줌
    2. list() → 그룹별로 결과를 리스트 형태로 만듦
    3. transform() → 위의 그룹화 규칙에 따라 실제 객체(DTO)로 변환함
    4. Projections.constructor() → DTO로 변환하기 위해 필드 매핑 수행

   **결과 형태**

   | User ID | User Name | Reviews(List) |
       | --- | --- | --- |
   | 1 | “박콩” | [ReviewDTO(1,“맛있어요”,5), ReviewDTO(2,“별로예요”,2)] |
   | 2 | “지혀니” | [ReviewDTO(3,“괜찮아요”,4)] |

   **transform + groupby를 사용하는 이유**

    1. 복잡한 조인 결과를 객체 구조로 깔끔하게 매핑
    2. 성능 최적화 → 불필요한 중복 ROW 제거(DB에서 중복된 데이터 안 묶고 애플리케이션 단에서 그룹화)
    3. DTO 안에 DTO 형태 지원

2. **Java Stream에서의 groupBy()**

   QueryDSL을 쓰지 않고, DB 결과를 List로 받은 뒤 가공할 때도 groupingBy()를 사용한다.

    ```java
    Map<Long, List<Review>> reviewMap = reviews.stream()
        .collect(Collectors.groupingBy(review -> review.getUser().getId()));
    ```

   이건 DB에서 평면 데이터(List 형태)를 받아서 자바 코드 단에서 유저별 리뷰 리스트로 묶는 방식이다.

   이 경우엔 transform은 없고, 직접 DTO로 매핑해야한다. 👇

    ```java
    List<UserWithReviewsDTO> result = reviewMap.entrySet().stream()
        .map(entry -> new UserWithReviewsDTO(entry.getKey(), entry.getValue()))
        .collect(Collectors.toList());
    ```


결론은 transform()은 QueryDSL에서 조회 결과를 특정 구조(DTO, Map)으로 변환하는 기능이고, groupBy()는 그 안에서 특정 필드를 기준으로 결과를 그룹화하여 DTO안에 DTO 형태를 가능하게 해준다.

### order by null

**order by null이란?**

정렬을 하지 말라는 뜻이다. 즉, DB에게 정렬 연산을 아예 생략하라고 명령하는 것이다.

**왜 쓰는가?**

보통 GroupBy, Distinct, 또는 Union같은 쿼리 뒤에는 DB가 자동으로 정렬 연산(SORT)를 수행하는 경우가 있다. 그럴 때 order by null을 명시하면, 정렬 과정을 생략해서 성능을 높일 수 있다.

**예시**

**일반 GROUP BY)**

```sql
 SELECT user_id, COUNT(*)
FROM review
GROUP BY user_id;
```

이 쿼리는 내부적으로 MYSQL이 user_id로 정렬하면서 그룹을 나눈다. → 데이터 많을수록 비용 증가

**ORDER BY NULL을 추가한 GROUP BY**

```sql
SELECT user_id, COUNT(*)
FROM review
GROUP BY user_id
ORDER BY NULL;
```

이건 DB에게 그룹은 하되, 정렬하지마!!! 하고 명시한것이다. 그래서 DB는 정렬을 건너뛰고 바로 그룹 결과를 낼 수 있다. → 성능이 더 빠르다.

**왜 NULL일까?**

SQL 문법상 ORDER BY 뒤에는 컬럼 이름이나 식이 와야하는데, NULL은 어떤 컬럼도 나타내지 않는다. → 정렬 기준이 없다는 뜻이 된다.

한줄 요약하자면 !! order by null은 정렬 스킵해서 쿼리 더 빠르게 한다!!는것