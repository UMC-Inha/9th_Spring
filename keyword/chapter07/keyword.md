# RestContollerAdvice

# @RestControllerAdvice란

스프링 MVC의 컨트롤러 보조 기능인 @ControllerAdvice에 @ResponseBody가 합쳐진 버전 따라서 여기서 정의한 메서드의 반환값은 뷰가 아니라 JSON등 바디로 직렬화되어 나간다.

# 사용하는 상황

1. 전역 예외 처리
2. 바인딩/검증 보조
3. 모든 컨트롤러에 공통 모델 추가

→ 다음 중 1. 전역 예외 처리 용도로 가장 많이 사용한다고 합니다.

워크북 내용도 1 상황이므로 1번 상황에서의 사용법과 특징에 대해 설명하겠습니다.

# 예외처리 흐름 설명

1. 예외 발생
2. DispatcherServlet → HandlerExceptionResolver 체인 가동

   다음의 순서대로 해당 예외를 처리 할 수 있는걸 찾는다.

    - ExceptionHandlerExceptionResolver

      컨트롤러 내부의 @ExceptionHandler

      없으면 전역에서 @RestControllerAdvice/@ControllerAdvice

      찾으면 그 메서드를 호출해서 직렬화하고 체인 종료

      못찾으면 다음으로 넘어감

    - ResponseStatusExceptionResolver
    - DefaultHandlerExceptionResolver
    - 위 세 Resolver가 모두 못찾으면 500으로 떨어지거나 로그만 남고 끝난다.

⇒ @RestControllerAdvice/@ControllerAdvice 이 부분을 우리가 정의해서 해당 단계에서 Exception을 잡아서 처리할 수 있도록 할거다.

```java
//워크북 코드

@RestControllerAdvice
public class GeneralExceptionAdvice {

    @ExceptionHandler(GeneralException.class)
    public ResponseEntity<ApiResponse<Void>> handleException(GeneralException ex) {
        return ResponseEntity
                .status(ex.getCode().getStatus())      
                .body(ApiResponse.onFailure(
                        ex.getCode(),                  
                        null                           
                ));
    }
}
```

위 코드는 워크북 코드로 Exception 발생시 위에서 설명한 단계 중 전역에서 찾는 단계에서 위 클래스도 확인을 할 것이고 만약 예외가 GeneralException.class 이면 해당 코드에서 예외 처리가 진행된다.

# 핵심 목적

예외 처리 로직을 중앙화, 표준화해서 다음과 같은 이점을 얻을 수 있음

- 중복 제거 & 가독성 : 전역 한 곳에서 처리하기 때문에. 예외에 관한 코드가 여기저기 흩어져있지 않음
- 일관된 응답 포멧 : ApiResponse같은 공통 스키마 코드 멤시지 상태코드로 통일
- 관심사 분리

ApiResponse를 이용한 일관된 응답 포멧과 관련해서 워크북에서 배웠기 때문에 이에 관련된 내용을 정리해보면

ApiResponse 클래스를 정의하고 해당 객체를 만들어서 전달함으로써 응답을 통일한다.

응답을 통일했으니까 성공상황과 실패상황 모두 내용(값)이 다른거지 동일한 response 객체 return.

따라서 실패 상황도 해당 ApiResonse 객체가 전달될거기 때문에 해당 규칙에 맞춰서 error control 하는 코드를 짤 수 있고 이 코드를 한곳에 모아 관리하기 위해서 관련된 error끼리 ExceptionHandler를 만들어서 에러 처리해준다.

<br>

---

# lombok

# Lombok 이란?

반복 코드를 컴파일 타임에 자동 생성해주는 자바 Annotation Libarary

(반복 코드 : Getter, Setter, 생성자, Builder, toString, 로그 등)

# 동작 원리

컴파일러가 소스코드를 기반으로 AST를 만들고 AST를 바탕으로 최종 바이트 코드를 생성

→ 이 때 lombok은 AST를 직접 수정

따라서 바이트 코드에는 getter, setter등에 대한 바이트 코드가 소스코드에 있었던 것처럼 추가되어 있음 (=런타임에 특별한 일이 처리되는게 아님)

# 사용법

1. @Getter, @Setter

   각 필드 또는 클래스 전체에 getter, setter 생성

   [주의사항]

    - 핵심 도메인 모델에는 @Setter 사용을 지양
    - 객체의 값이 언제, 어디서 변경되었는지 추적하기가 어려움. 따라서 디버깅이 어려워지고 얘기치 않은 상태 변경을 유발해 버그로 이어질 수 있음
    - user.setName() 보다는 user.updateName() 처럼 비즈니스 로직의 의도를 담은 메서드 사용 권장

1. @NoArgsConstructor, @AllArgsConstructor, @RequiredArgsConstructor

   순서대로 파라미터가 없는 기본 생성자 생성, 클래스의 모든 필드를 파라미터로 받는 생성자 생성, 특정 필드만 파라미터로 받는 생성자 생성 (final 키워드가 붙은 필드, @NonNull 어노테이션이 붙은 필드)

   [주의사항]

    - No → JPA를 사용한다면 Reflection을 이용해 객체를 생성. 이 때 기본 생성자가 없으면 오류. (access = AccessLevel.PROTECTED)를 통해서 JPA에서만 사용하도록 제한
    - All → 필드 순서가 변경되었을 떄 타입이 같으면 컴파일 오류가 발생하지 않아 값이 바뀌어 주입될 수 있음 따라서 @Builder의 사용을 권장

   [특징]

    - Req → Spring 생성자 주입 사용시 @Autowired 생략가능, final 필드에 생성자가 없는 상황이나 새로운 final 필드 추가 시 자동으로 생성자 반영되어 컴파일 오류 발생하기 때문에 버그 방지 측면에서 유용

1. @ToString

   자바 객체의 Object 클래스가 제공하는 toString() 자동생성 → 디버깅이나 로깅 시 현재 상태를 쉽게 확인하기 위해 사용

    ```java
    // exclude 사용
    @ToString(exclude = {"members"})
    @Entity
    public class Team {
        private Long id;
        private String name;
    
        // 양방향 즉 Member 클래스도 team을 갖는다고 가정
        @OneToMany(mappedBy = "team")
        private List<Member> members = new ArrayList<>();
    }
    ```

   [주의사항]

    - 위 코드 상황에서 Team, Member는 양방향 연관관계를 갖는다.
    - 따라서 위 코드에서 toString()을 사용 시 Member가 Team을 참조 Team이 다시 Member 리스트를 참조 이로인해 무한 루프가 발생해 StackOverflowError 발생가능
    - 지연 로딩인 경우 객체를 문자열로 변환할 때  N+1 문제처럼 성능저하 가능성있음.

   → 연관관계 필드를 exclude하여 해결

2. @Builder

   객체 생성할 때 생성자의 파라미터 순서에 얽메이지 않고, 메서드 체이닝 방식으로 원하는 필드 값만 설정할 수 있게 해준다.

    ```java
    // 사용 예시
    Product product = Product.builder()
                            .id(1L)
                            .name("스마트폰")
                            .price(1000000)
                            .build();
    ```

   [특징]

   @Setter를 사용하지 않고 @Builder로 객체를 생성하면 생성 시점에 한 번만 초기화되고 이후에는 변경되지 않아 불변 객체로 만들 수 있다.

3. @EqualsAndHashCode

   객체의 동등성과 해시 코드를 비교하는 메서드를 자동생성 → equals(), hashCode()

   [주의사항]

    - Mutable 객체에서의 사용 금지 → 객체의 필드 값이 변경 가능하면 필드 값이 바뀌면 해시 코드도 바뀐다. 따라서 Set에 이미 저장된 객체를 해시 코드가 바뀌었기 때문에 찾을 수 없음.
    - JPA Entity에서 사용 시 ToString()과 마찬가지로 순환 참조 문제 발생가능 따라서 id값을 이용해 비교할 수 있도록 옵션을 명시하는것이 좋음

4. @Value

   @Getter, @ToString, @EqualsAndHashCode, @AllArgsConstructor와 모든 필드 final로 선언 및 Setter를 생성하지 않음을 한 번에 적용

   [주의사항]

    - 불변 객체, Read-only 객체 생성할 때만 사용
    - JPA Entity에는 부적합

   [특징]

    - 응답 전용 DTO를 만들 때 코드의 양을 줄이고 불변성 보장하여 안정적인 데이터 구조를 만들 수 있다.



--- 


# dto 형식 public static VS record 비교하기

- DTO를 public class로 선언하는것과 record로 선언하는 두 가지 방법에 대해서 특징과 이유 등 알아보기

# Public class 로 DTO 생성

```java
public class MemberDto {

    // 회원 가입 요청 DTO
    public static class JoinRequest {
        private String email;
        private String password;
        private String nickname;

        // 기본 생성자 + getter/setter
        public JoinRequest() {}
        public String getEmail() { return email; }
        public void setEmail(String email) { this.email = email; }
        // ...
    }

    // 회원 정보 응답 DTO
    public static class InfoResponse {
        private Long id;
        private String nickname;

        public InfoResponse(Long id, String nickname) {
            this.id = id;
            this.nickname = nickname;
        }

        public Long getId() { return id; }
        public String getNickname() { return nickname; }
    }

    // 회원 수정 요청 DTO
    public static class UpdateRequest {
        private String nickname;
        private String password;
    }
}
```

위 코드와같이 멤버와 관련된 DTO는 여러 상황에서 각기 다르게 필요할 수 있다.

따라서 MemberDTO 클래스 내부에 여러 상황별 DTO를 static class로 만드는 방법이다.

[특징]

- 선언을 위해 field + annotation 필요 (record 대비 복잡)
- 불변성 확보를 위해서는 @Setter를 제외해야 하고 final 또한 직접 붙여야 함
- Lombok 어노테이션 필요
- Getter는 getId(), getName() 등 Java Bean 규약을 따름

[장점]

- DTO 이름이 많아져도 파일이 하나로 묶여 깔끔함.
- 의미적으로 연관된 DTO들을 한 곳으로 모음
- JoinRequest, UpdateRequest 같은 이름을 여러 도메인에서 재활용 가능.

# Record로 DTO 생성

[특징]

- public class 대비 선언 방식이 매우 간결
- 기본값이 불변이기 때문에 @Setter를 제외하거나 final을 붙일 필요가 없음
- Lombok 어노테이션 별도로 필요 없이 자동으로 제공
- 상속, 확장 불가능
- getter는 필드 이름 그대로 id(), name() 사용

# 비교

위 내용을 바탕으로 비교했을 때 record의 장점을 설명하자면

1. 코드 간결성 → 필드만 나열하면 DTO가 완성됨
2. 안정성 → 불변 객체이기 때문에 보장됨
3. 명확한 의도 → 코드를 보면 데이터를 담는 역할만 한다는 의도를 명확히 파악 가능

# 무조건 Record가 좋을까?

- DTO 생성 후 값을 변경해야 하는 경우 (=가변 객체가 필요할 때) record를 사용할 수 없음
- 상속이나 확장성이 필요할 때 record 불가능
- public class 는 데이터 전달 외에 복잡한 로직 추가에 유연함


---

# return - ApiResponse or ResponseEntity

# 궁금한 점

워크북을 보면서 구현을 했을 때 ApiResponse를 통해 아래처럼 결과만이 아닌 성공여부, 코드, 메시지를 모두 포함해 구현했다.

```java
    @JsonProperty("isSuccess")
    private final boolean isSuccess;

    @JsonProperty("code")
    private final String code;

    @JsonProperty("message")
    private final String message;

    @JsonProperty("result")
    private T result;
```

따라서 controller가 return 해야하는 타입은 ApiResponse라고 생각했다. (위 코드를 구현한 이유라고 생각했음)

그런데 이것저것 찾다보니 ApiResponse를 ResponseEntity로 감싸서 반환하는 코드가 자주 보였다.

# ResponseEntity 사용의 이유

### 1. HTTP 상태코드

response entity를 사용하면 상황에 맞는 정확한 상태코드를 반환할 수 있다.

### 2. 응답 헤더 제어

HTTP 응답 헤더, 캐시를 직접 설정해서 반환할 수 있음

### 3. 일관성과 관심사 분리

ApiResponse는 Api의 응답 데이터가 항상 일관된 JOSN 포맷을 가지도록 보장 (성공 코드, 메시지 등)

ResponseEntity는 데이터의 내용과는 별개로 응답 자체의 프로토콜적 속성(HttpStatus)을 명확하게 분리하여 전달

# HTTP 상태코드는 이미 ApiResponse에도 구현한 것 아닌가?

라는 의문이 있어서 관련 내용을 찾아봤다.

ApiResponse의 코드는 비즈니스 레벨의 코드

ResponseEntity의 코드는 프로토콜 표준 코드

```java
//프로토콜 표준 (201)을 지키면서, 비즈니스 메시지 (S002)를 전달
return ResponseEntity
        .status(GeneralSuccessCode.CREATED.getHttpStatus()) //ResponseEntity의 역할: 201 지정
        .body(ApiResponse.onSuccess(...));                 //ApiResponse의 역할: 본문 구조(S002) 보장
```

# 두가지 방식

- 순수 RESTful의 경우 HTTP 상태 코드만으로 전달.
- 현재 방식의 경우 HTTP 상태 코드 + 상세 에러 코드와 메시지를 통해 디버깅 측면에서 유리, 편의성 고려