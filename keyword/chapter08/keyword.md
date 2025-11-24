- java의 Exception 종류들
    
    ![image.png](attachment:c027a201-1cf8-4dc2-bd14-bf02525706ee:image.png)
    
    ### **1.1. 1. NullPointerException(NPE)**
    
    정말 정말 자주 출몰하는 에러입니다.널포인터익셉션이라고 부르기 길어서 줄여서 NPE라고도 부르기도 합니다.이 예러는 객체 참조가 없는 상태, 즉 null 값을 가지고 있는 참조 변수로 객체 접근 연산자인 도트(.)를 사용했을때 발생합니다.즉, 해당 객체가 null인 상태에서의 접근을 했을때 해당 값이 null에 대한 접근을 하여 발생하는 에러로 객체가 없는 상태에서 객체를 사용하려 하였으나 해당 객체는 없는 상태이기 때문에 발생하는 에러라고 볼 수 있습니다.
    
    ### **1.2. 2. ArrayIndexOutOfBoundsException**
    
    배열을 다룰때 주로 발생하는 예외인데요.
    
    배열에서 할당된 배열의 인덱스 범위를 초과해서 사용할 경우 발생하는 에러입니다.
    
    예를 들어 길이가 3인 int[] arr = new int[3] 배열을 선언했다고 가정하면
    
    arr[0]
    
    arr[1]
    
    arr[2]
    
    해당 배열에서 사용할 수 있는 인덱스의 범위로는 위의 arr[0], arr[1], arr[2]만을 사용할 수 있는데, arr[3]이나 arr[4]등 해당 인덱스의 범위에 벗어나는 경우에는 ArrayIndexOutOfBoundsException가 발생합니다.
    
    ### **1.3. 3. NumberFormatException**
    
    프로그램을 개발하다가 보면 문자열로 되어있는 데이터를 숫자로 변경하는 경우가 매우 자주 발생하곤 합니다.
    
    문자열을 숫자로 변경하는 방법은 여러 가지가 있지만 가장 많이 사용되는 코드는 아래와 같습니다.
    
    | 반환 타입 | 메소드명(매개변수) | 설명 |
    | --- | --- | --- |
    | int | Integer.parseInt(String s) | 주어진 문자열을 정수로 변환해서 리턴 |
    | Doblue | Double.parseDouble(String s) | 주어진 문자열을 실수로 변환해서 리턴 |
    
    이외에도 Float.parseFloat(String s)등과 같이 문자열을 특정 수로 변경할 수 있습니다.
    
    Integer와 Doble은 포장(Wrapper) 클래스라고 하는데
    
    여기서 Wrapper 클래스란 **본 자료타입(primitive type)을 객체로 다루기 위해서 사용하는 클래스들을 래퍼 클래스(wrapper class)라고 합니다.**
    
    만약 parseXXX() 메소드들을 이용하여 문자열을 숫자로 변환할 수 있지만 매개변수로 오는 문자열이 숫자로 변환이 되는 값이면 숫자를 리턴하지만, 숫자로 변환될 수 없는 문자가 온다면 java.lang.NumberFormatException을 발생시키게 됩니다.
    
    ### **1.4. 4. ClassCastException**
    
    타입 변환(Casting)은 상위 클래스와 하위 클래스간에 발생하고 구현 클래스와 인터페이스 간에도 발생한다.
    
    이러한 관계가 아니라면 클래스는 다른 클래스를 타입으로 변환할 수 없다.
    
    억지로 타입변환을 시도할 경우  ClassCastException이 발생한다.
    
- @Valid
    
    # @Valid란?
    
    `@Valid`는 **Jakarta Bean Validation**(JSR-380) 규격을 기반으로 하는 검증 어노테이션을 활성화하는 트리거 역할을 함.
    
    즉,
    
    - DTO 필드에 붙여둔 검증 규칙들
        
        ex) `@NotNull`, `@Size`, `@Email`, `@Min`, `@Pattern` …
        
    - 을 실제로 **검증하라고 시키는 역할**이 바로 `@Valid`.
    
    ---
    
    # 사용 예시
    
    ## 1) DTO에 검증 규칙 설정
    
    ```java
    public class MemberReqDTO {
    
        @NotBlank(message = "이름은 필수입니다.")
        private String name;
    
        @Email(message = "올바른 이메일 형식이 아닙니다.")
        private String email;
    
        @Min(value = 1, message = "나이는 1 이상이여야 합니다.")
        private int age;
    }
    ```
    
    ---
    
    ## 2) Controller에서 @Valid 사용
    
    ```java
    @PostMapping("/members")
    public ApiResponse<?> createMember(
            @Valid @RequestBody MemberReqDTO request
    ) {
        // DTO 검증 통과하면 이곳 실행
        return ApiResponse.onSuccess();
    }
    ```
    
    `@Valid`가 붙으면:
    
    - DTO 검증 실행
    - 실패 시 컨트롤러 메서드에 들어오지도 않음
    - `MethodArgumentNotValidException` 발생 → 전역 예외 처리로 전달
    
    ---
    
    # 검증 실패 흐름
    
    1. 클라이언트가 Request 전송
    2. DTO 필드에 Bean Validation 어노테이션 검증
    3. **실패하면 컨트롤러 안 들어감**
    4. Spring이 `MethodArgumentNotValidException` 던짐
    5. `@RestControllerAdvice`에서 처리
    6. 클라이언트에게 에러 응답 반환
    
    UMC 템플릿에서는 보통 이렇게 처리함:
    
    ```java
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ApiResponse<?>> handleValidationException(
            MethodArgumentNotValidException ex
    ) {
        String message = ex.getBindingResult().getAllErrors()
                .get(0)
                .getDefaultMessage();
    
        return ResponseEntity
                .badRequest()
                .body(ApiResponse.onFailure(GeneralErrorCode.BAD_REQUEST, message));
    }
    ```
    
    ---
    
    @Valid vs @Validated 차이
    
    | 어노테이션 | 특징 |
    | --- | --- |
    | **@Valid** | 표준 Bean Validation, 그룹 기능 없음 |
    | **@Validated** | 스프링 제공, 그룹 검증 가능 (`groups` 기능) |
    
    예: 회원가입과 회원정보 수정에 다른 검증 규칙을 적용하고 싶을 때 → `@Validated` 사용
    
    ---
    
    # 자주 사용하는 Bean Validation 어노테이션
    
    | 어노테이션 | 설명 |
    | --- | --- |
    | `@NotNull` | null 금지 |
    | `@NotBlank` | null, "", "   " 금지 |
    | `@NotEmpty` | null, "" 금지 |
    | `@Size(min, max)` | 문자열/컬렉션 길이 제한 |
    | `@Min`, `@Max` | 숫자 범위 제한 |
    | `@Email` | 이메일 형식 |
    | `@Pattern` | 정규식 패턴 검사 |
    | `@Positive`, `@PositiveOrZero` | 양수/양수 또는 0 |
    
    ---
    
    # body에서만 사용하는 것이 아님!
    
    `@PathVariable`이나 `@RequestParam`에도 사용 가능함.
    
    예시:
    
    ```java
    @GetMapping("/members/{id}")
    public ApiResponse<?> getMember(
            @PathVariable @Min(1) Long id
    ) {
        ...
    }
    ```
    
    ---
    
    # 정리
    
    `@Valid`는…
    
    ✔ DTO 검증 규칙을 실행시키는 트리거
    
    ✔ RequestBody, RequestParam, PathVariable 모두 가능
    
    ✔ 실패 시 `MethodArgumentNotValidException` 발생
    
    ✔ 전역 예외 처리에서 메시지 커스텀 가능
    
    ✔ Bean Validation을 적용하려면 DTO 필드에 @NotBlank 등 붙여야 함