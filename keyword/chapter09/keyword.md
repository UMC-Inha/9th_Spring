## 객체 그래프 탐색이란

- **객체가 서로 연관 관계로 연결되어 있을 때**, 한 객체를 시작 점으로 **다른 객체까지 참조를 따라가며 원하는 데이터를 조회하는 과정**을 말한다.

**즉, 객체 = 노드(node), 연관 관계 = 간선(edge) ⇒ Graph**

### 예시

```java
class Review {
    @ManyToOne
    private Member member;
}

class Member {
		private String name;
}
```

- `review.getMember().getName();` → 객체 그래프 탐색
- `memberRepository.findById().getName()` → 레파지토리를 이용한 탐색, Member의 id를 직접 알고 있어야한다.

### 객체 그래프 탐색의 장단점

- 원하는 데이터를 별도의 쿼리를 사용하지 않고 메서드 체이닝으로 간편하게 접근할 수 있다.
- 지연 로딩 사용 시에 N + 1 문제가 발생할 수 있다!
    - review.getMember() → 여기서는 JPA가 Member의 프록시 객체로 반환한다. (실제 DB 조회는 일어나지 않음)
    - 이후에 .getName() 이 호출되면, `select * from member where member_id = ?` 쿼리가 추가로 날아가서 해당 객체의 이름을 불러온다.
    - 만약 여기서 10개의 리뷰를 조회한다면? → 추가 쿼리도 10번 실행됨! → **N + 1** 문제
    
    ```java
    List<Reviews> reviews = reviewRepository.findAll(); -> 1번 쿼리
    
    for(Review r : reviews) {
    		r.getMember().getName();  -> 추가 쿼리 N 번 발생
    } 
    ```
    
    → 따라서 객체 그래프 탐색을 이용할 때 발생할 수 있는 N + 1 문제에 대해 미리 예측하고 대비해야한다!