# Page와 Slice 차이 알아보기

## 1. Page와 Slice가 각각 어떻게 출력값이 나오는 지 알아보기

모든 리뷰를 조회하는 API를 **동일한 조건**으로 `Page`와 `Slice` 두 가지 방식으로 각각 구현해보았다.

- `Page`는 이미 JpaRepository 기본 메서드로 제공된다.
- `Slice`는 따로 `findAllBy(Pageable pageable)` 만 추가해주면 된다.

두 방식 모두 `Slice.map()`, `Page.map()` 을 사용해 DTO 형태로 content를 변환했다.

아래는 **동일한 리뷰 데이터를 Page/Slice 각각으로 조회했을 때의 실제 JSON**이다.

> 코드 예시는 포함했지만, 목적에 맞춰 **계층을 세분화하지 않고 하나의 컨트롤러 안에서 모든 로직을 단순하게 처리하도록 구성했다.** 이렇게 함으로써 구조적 복잡도를 줄이고, Page와 Slice의 차이를 확인하는 데에만 집중할 수 있도록 했다.
> 

### (1) Page 응답

**간단 코드**

```java
@GetMapping("reviews/all/page")
    public Page<ReviewResDTO.ReviewPreViewDTO> getReviewsByPage(Pageable pageable) {
        Page<Review> page = reviewRepository.findAll(pageable);

        return page.map(review ->
                ReviewResDTO.ReviewPreViewDTO.builder()
                        .ownerNickname(review.getMember().getName())
                        .score(review.getStar())
                        .body(review.getContent())
                        .createdAt(review.getCreatedAt().toLocalDate())
                        .build()
        );
    }
```

**JSON 결과**

```java
{
    "content": [
        {
            "ownerNickname": "홍길동",
            "score": 4.5,
            "body": "매장이 깨끗하고 음식이 정말 맛있었어요!",
            "createdAt": "2025-11-16"
        },
        {
            "ownerNickname": "홍길동",
            "score": 4.5,
            "body": "매장이 깨끗하고 음식이 정말 맛있었어요!",
            "createdAt": "2025-11-16"
        },
        {
            "ownerNickname": "홍길동",
            "score": 4.5,
            "body": "매장이 깨끗하고 음식이 정말 맛있었어요!",
            "createdAt": "2025-11-16"
        }
    ],
    "pageable": {
        "pageNumber": 0,
        "pageSize": 20,
        "sort": {
            "empty": true,
            "unsorted": true,
            "sorted": false
        },
        "offset": 0,
        "unpaged": false,
        "paged": true
    },
    "last": true,
    "totalPages": 1,
    "totalElements": 3,
    "size": 20,
    "number": 0,
    "sort": {
        "empty": true,
        "unsorted": true,
        "sorted": false
    },
    "numberOfElements": 3,
    "first": true,
    "empty": false
}
```

`Page`에서는 `totalPages`와 `totalElements` 같은 값들이 함께 제공되기 때문에 전체 데이터가 얼마나 있는지, 그리고 그 데이터를 기준으로 총 몇 페이지로 구성되는지를 한눈에 파악할 수 있다.

 즉, **전체 데이터 규모를 기준으로 하는 정보들이 모두 포함되어 있어**, “총 몇 페이지인지”, “전체 데이터가 몇 개인지”와 같은 전체적인 맥락이 필요한 화면에서 특히 유용하다. 게시판처럼 페이지 번호를 이동하거나 전체 페이지 수를 표시해야 하는 UI에서는 이러한 정보가 필수적이기 때문에 `Page` 방식이 더 적합하다.

### (2) Slice

**간단 코드**

```java
@GetMapping("/all/slice")
    public Slice<ReviewResDTO.ReviewPreViewDTO> getReviewsBySlice(
            @PageableDefault(size = 10, sort = "createdAt")
            Pageable pageable
    ) {
        Slice<Review> slice = reviewRepository.findAllBy(pageable);

        return slice.map(review ->
                ReviewResDTO.ReviewPreViewDTO.builder()
                        .ownerNickname(review.getMember().getName())
                        .score(review.getStar())
                        .body(review.getContent())
                        .createdAt(review.getCreatedAt().toLocalDate())
                        .build()
        );
    }
```

**JSON 결과**

```java
{
    "content": [
        {
            "ownerNickname": "홍길동",
            "score": 4.5,
            "body": "매장이 깨끗하고 음식이 정말 맛있었어요!",
            "createdAt": "2025-11-16"
        },
        {
            "ownerNickname": "홍길동",
            "score": 4.5,
            "body": "매장이 깨끗하고 음식이 정말 맛있었어요!",
            "createdAt": "2025-11-16"
        },
        {
            "ownerNickname": "홍길동",
            "score": 4.5,
            "body": "매장이 깨끗하고 음식이 정말 맛있었어요!",
            "createdAt": "2025-11-16"
        }
    ],
    "pageable": {
        "pageNumber": 0,
        "pageSize": 10,
        "sort": {
            "empty": false,
            "sorted": true,
            "unsorted": false
        },
        "offset": 0,
        "paged": true,
        "unpaged": false
    },
    "size": 10,
    "number": 0,
    "sort": {
        "empty": false,
        "sorted": true,
        "unsorted": false
    },
    "first": true,
    "last": true,
    "numberOfElements": 3,
    "empty": false
}
```

`Slice`는 `totalPages`나 `totalElements` 같은 전체 데이터 규모를 알려주는 값이 제공되지 않는다. 

대신 **다음 페이지가 존재하는지 여부**를 판단할 수 있는 `hasNext()`가 핵심 정보로 제공된다. 

이 때문에 전체 데이터 개수를 알 필요 없이, 사용자가 스크롤을 내릴 때마다 이어서 데이터를 불러오기만 하면 되는 **무한 스크롤이나 ‘더보기’ 형태의 UI**에 특히 적합하다. 또한 전체 개수를 세기 위한 count 쿼리를 수행하지 않기 때문에 `Page` 방식보다 **쿼리 비용이 적고 성능이 더 효율적**이라는 장점이 있다.

`Slice`와 `Page` 모두 기본 응답 구조에 포함되는 값이 너무 많기 때문에, 실제로 필요한 정보만 선택해서 사용하기 위해서는 DTO로 변환하거나 공통 응답 형태로 감싸는 방식이 중요하다. 이렇게 하면 불필요한 필드 노출을 막고, 클라이언트가 사용하는 데이터 구조도 한눈에 이해하기 쉬운 형태로 정리할 수 있다.


## 2. Page, Slice 각각 적용 시 장단점 파악하기

### Page의 장단점

`Page`는 전체 데이터에 대한 정보를 함께 제공한다는 점이 가장 큰 장점이다. `totalElements`와 `totalPages`를 통해 전체 데이터 개수와 전체 페이지 수를 바로 확인할 수 있기 때문에, "1 / 5페이지"처럼 페이지 번호를 표시하거나 마지막 페이지로 이동하는 기능처럼 **전체 기준이 필요한 UI**를 구현할 때 매우 유용하다. 특히 관리자 화면이나 통계·리포트 화면처럼 전체 데이터 규모를 기준으로 동작해야 하는 경우에는 사실상 필수적인 방식이다.

하지만 단점도 분명하다. `Page`는 반드시 한 번의 `count(*)` 쿼리를 추가로 실행해 전체 개수를 계산하기 때문에, 데이터가 많이 쌓인 테이블에서는 이 추가 쿼리가 성능에 상당한 부담이 될 수 있다. 또한 단순히 “더보기” 또는 무한 스크롤 방식의 리스트처럼 전체 개수를 굳이 알 필요가 없는 시나리오에서도 불필요한 count 쿼리가 수행되어 효율적이지 않다는 점이 단점이다.

---

### Slice의 장단점

`Slice`는 전체 개수보다는 **“다음 페이지가 존재하는지”** 여부에 초점을 맞춘 가벼운 페이징 방식이다. 내부적으로는 limit+1 방식으로 다음 데이터가 더 있는지만 판단하기 때문에 전체 개수를 세는 연산이 없어 `Page`에 비해 훨씬 **성능상 유리**하다. 이러한 특성 덕분에 사용자 입장에서 전체 개수보다는 실제로 더 데이터를 볼 수 있느냐가 중요해지는 **무한 스크롤이나 ‘더보기’ 버튼 기반 UI**에서 특히 잘 맞는다. 트래픽이 많고 데이터가 지속적으로 증가하는 서비스—예를 들어 피드, 타임라인, 커뮤니티 리스트—같은 곳에서도 Slice 방식은 매우 효율적이다.

다만 Slice는 전체 페이지 수나 전체 데이터 개수를 알려주지 않기 때문에, “총 몇 페이지인지 알려주세요” 또는 “마지막 페이지로 바로 보내주세요”와 같은 **전체 기준 정보가 필요한 기능**에는 사용할 수 없다. 또한 페이지 번호 기반(page=0,1,2…) UI와도 완전히 맞지 않아서, 그보다는 “계속 더 불러오기” 형태의 자연스러운 흐름에 더 적합한 방식이다.


## 3. 언제 적용하면 좋을 지 파악하기

### Page를 쓰기 좋은 상황

`Page`는 전체 데이터 기준 정보가 중요할 때 가장 효과적이다. 예를 들어 게시판이나 관리 도구처럼 **정확한 페이지 번호를 기반으로 이동해야 하는 UI**에서는 “1페이지 / 총 10페이지”처럼 전체 페이지 수를 명확히 표시해야 하며, 특정 페이지로 바로 뛰어가는 랜덤 접근도 자연스럽다. 또한 관리자 화면, 통계 화면처럼 **전체 데이터의 규모(전체 사용자 수, 전체 주문 수 등)**가 중요하게 사용되는 도메인에서는 totalElements와 totalPages 정보가 필수적이다. `< 1 2 3 4 5 >` 와 같은 페이징 네비게이션이 필요한 경우에도 Page가 가장 적합하다.

결국 Page는 **“전체 데이터 중 현재 위치가 어디인지”가 중요한 환경**에서 가장 강점을 발휘한다.

---

### Slice를 쓰기 좋은 상황

반면 `Slice`는 전체 데이터 개수보다 **지금 이후에 더 데이터가 있는지 여부**가 중요할 때 사용하기 적합하다. SNS 피드나 리뷰 목록처럼 사용자가 화면을 아래로 스크롤하며 자연스럽게 더 많은 데이터를 불러오는 UI에서는 전체 개수를 알 필요가 없기 때문에 Slice가 훨씬 효율적이다. 모바일 환경이나 대용량 테이블에서는 count 쿼리 하나만 줄어도 성능 차이가 크게 나는데, Slice는 totalElements 계산을 하지 않기 때문에 성능상 이점이 크다. “더보기” 버튼이나 무한 스크롤처럼 **hasNext()만 판단해도 충분한 흐름**에서는 Slice가 Page보다 훨씬 가볍고 실용적이다.

즉 Slice는 **“전체 개수보다 다음 페이지가 있는지 여부가 더 중요한 흐름”**에서 가장 적합한 선택이다.