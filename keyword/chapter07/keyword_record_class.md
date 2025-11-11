# DTO public static VS record 비교하기

DTO를 만들 때 `class`로 작성할지, 아니면 `record`로 작성할지 고민되는 경우가 많다.

그동안은 별 고민 없이 DTO를 `class`로 만들어왔지만, Java에서 `record`라는 새로운 타입을 접하고 나니 이제는 DTO 작성 시 어떤 방식을 선택하는 게 더 적절한지 생각해볼 필요가 생겼다.

특히 Spring 환경에서는 요청/응답 객체가 자주 등장하고,
DTO가 순수히 데이터 전달만을 목적으로 하는 경우가 대부분이기 때문에
 `record`와 `class`를 비교해 보며 각 상황에 어떤 타입이 더 자연스러운지 판단해보면 좋을 것 같다.


## Record가 뭐지..?

`record`는 Java에서 **불변 값 객체**를 만들기 위한 문법이다.

데이터 전달이나 값 자체를 표현할 때 쓰며, 자동으로 아래를 생성해준다.

- `private final` 필드
- 생성자
- getter 역할을 하는 accessor
- `equals()`, `hashCode()`, `toString()`

### Class 랑 비슷한거 같은데 왜 등장했지..?

기존의 클래스는 생성자, getter, equals, hashCode, toString 등 함께 만들어야 할 메서드가 너무 많았다. 

record를 사용하면 이러한 메서드들을 자동으로 생성해주지만, 사실상 필요한 요소들이 많다는 근본적인 특성 자체는 변하지 않는다. Lombok을 활용하면 어노테이션으로 간단하게 처리할 수 있지 않나? 라고 생각할 수 있지만, 결국 동일한 구조를 갖춰야 한다는 점에서는 큰 차이가 없다.

이런 복잡한 구조를 단순한 데이터 객체에까지 적용하는 건 과한 접근이었기 때문에, 더 가벼운 객체를 간편하게 정의하기 위한 방법으로 record가 등장하게 되었다.


## public static class DTO

```java
public static class MemberResponse {
    private Long id;
    private String name;

    public void setName(String name) { this.name = name; }
    public String getName() { return name; }
}
```

→ 객체 상태 변경 가능

→ 실수로 값이 바뀌어도 발견 어려울 수 있음

→ 코드량이 많다.

→ builder 가능

## record DTO

```java
public record MemberResponse(Long id, String name) {}
```

→ 값 고정 (Immutable) 된다는 점!

→ 데이터 이동에만 집중

→ 코드량이 적다

→ builder 불가능


## DTO에서 record vs class, 언제 무엇을 쓸까?

DTO를 설계할 때 가장 중요한 기준은 

**해당 데이터가 변경될 가능성이 있는지**, 그리고 **객체가 어떤 역할을 할 것인지**이다.

`record`는 생성 이후 값이 바뀌지 않는 **불변 데이터 객체**이기 때문에 입력값이나 응답 데이터처럼 단순히 값을 전달하기만 하는 경우에 적합하다. 

특히 요청 DTO는 외부에서 들어온 값이 비즈니스 로직 중간에 변경되면예기치 못한 오류나 보안 문제가 생길 수 있기 때문에 record로 선언하면 그런 위험을 원천적으로 차단할 수 있다. 

또한 record는 구조가 단순한 데이터에 매우 잘 맞고, 필드 수가 많지 않다면 간결하고 명확한 코드를 유지할 수 있다.

반면 DTO를 `class`로 작성하는 경우는 **값이 변경될 수 있는 상황**이나,**생성 시점이 복잡하거나 빌더 패턴이 필요할 때**가 해당된다.

예를 들어 필드가 매우 많고 선택적 입력이 많다면 record보다 class + builder 조합이 더 직관적일 수 있다.

또한 어떤 DTO는 단순히 값만 담는 것이 아니라 내부에서 보조 로직이나 변환 메서드를 제공하기도 하는데, 이런 경우 record보다는 유연한 class가 더 적절하다. 즉, DTO가 정말로 **값 전달만** 하는 역할이라면 record가 훨씬 깔끔하고 안전하며, **상태나 생성 방식이 유연해야 한다면 class**가 더 낫다.

요약하자면 “**변하지 않는 DTO → record**”, “**복잡하거나 동적인 DTO → class**”라고 보면 된다.