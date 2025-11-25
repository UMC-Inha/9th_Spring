# @Valid


`@Valid`는 **Bean Validation(자바 표준 JSR-303/380)**에서 제공하는 애너테이션으로,

객체에 선언된 검증 규칙을 **실제로 실행하도록 지시하는 트리거 역할**을 하는 애너테이션이다.

즉, 개발자가 DTO 또는 객체 필드에 부착한 `@NotNull`, `@Size`, `@Email` 등과 같은 제약 조건을

**Spring이 검증 단계에서 적용하도록 만드는 스위치**라고 볼 수 있다.


## 사용 위치

`@Valid`는 다음 위치에서 사용할 수 있다.

1. **Controller의 요청 DTO 파라미터**
    
    요청 JSON → DTO 변환 시 검증이 적용된다.
    
2. **DTO 내부 필드(중첩 객체)**
    
    필드에 `@Valid`가 부착되어 있어야 내부 객체까지 검증이 연쇄(cascade) 방식으로 적용된다.
    
3. **Service 메서드 파라미터 및 반환값**
    
    단, 이 경우에는 클래스 또는 메서드에 `@Validated`가 함께 선언되어 있어야
    
    Spring이 해당 빈에 AOP 기반 메서드 검증을 적용한다.
    


## Spring MVC 내부 동작 과정

컨트롤러에서 `@Valid`가 동작하는 과정은 다음 단계로 구성된다.

### (1) DispatcherServlet이 요청을 수신한다

요청 URL과 HTTP 메서드를 기반으로 어느 컨트롤러 메서드를 호출할지 결정한다.

### (2) 메서드 파라미터 생성 단계가 진행된다

Spring MVC는 **HandlerMethodArgumentResolver**를 이용하여 메서드 파라미터 한 개씩을 생성한다.

이 단계에서 **JSON → DTO 객체로 변환이 완료된다.**

### (3) 파라미터에 @Valid 또는 @Validated가 존재하는지 검사한다

Resolver는 해당 파라미터에 검증이 필요하다는 사실을 인지한다.

### (4) WebDataBinder가 생성되고 Validator가 호출된다

Spring은 DTO 객체를 대상으로 **WebDataBinder**를 만들고, 여기에 전역 Validator(보통 Hibernate Validator)를 붙여 `validate()`를 호출한다.

### (5) 검증 애너테이션 적용

Hibernate Validator는 DTO 클래스의 메타데이터를 읽고 각 필드에 선언된 제약 조건을 순서대로 검사한다.

- @NotBlank → NotBlankValidator
- @Email → EmailValidator
- @Min → MinValidator

각 애너테이션마다 대응되는 Validator가 존재하며, 이를 이용해 조건을 검증한다.

### (6) 검증 실패 시 처리 방식 결정

검증 실패 시 처리 방식은 컨트롤러 메서드의 시그니처에 따라 달라진다.

1. **BindingResult 존재 시**
    
    예외가 발생하지 않고, BindingResult 객체에 에러 정보가 저장된다.
    
2. **BindingResult 부재 시**
    
    Spring은 즉시 `MethodArgumentNotValidException` 예외를 발생시키고, 
    
    이후 전역 예외 처리기에서 처리하게 된다.
    

이 구조는 *유효성 검사 실패 시 예외 흐름을 제어할 수 있게 해주는 중요한 차이점*이다.

