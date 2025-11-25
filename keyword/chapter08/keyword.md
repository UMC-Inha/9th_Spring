- java의 Exception 종류들


    ### 에러와 예외의 차이
    
    에러는 프로그램 코드에 의해 수습될 수 없는 심각한 오류
    
    - OutOfMemoryError과 같이 복구할 수 없는 오류
    
    예외는 프로그램 코드에 의해 수습될 수 있는 미약한 오류 
    
    자바에서는 예외를 클래스로 나누었다
    
    JVM은 프로그램을 실행하는 도중에 예외가 발생하면 예외 클래스를 객체로 생성하고 예외 처리 코드에서 예외 객체를 이용할 수 있게 해준다 
    
    ### Throwable 클래스
    
    예외와 에러 모두 Throwable을 상속 받는다
    
    이 클래스는 오류나 예외에 대한 메시지를 담는 역할을 한다 
    
    - getMessage(), printStackTrace()
    
    ### Exception 클래스
    
    이들은 크게 런타임 에러와 컴파일 에러로 나뉜다 
    
    ### 런타임 예외 클래스 종류
    
    ArithmeticException
    
    - 어떤 수를 0으로 나누는 것과 같이 비정상 계산 중 발생
    
    NullPointerException
    
    - NULL 객체 참조시 발생
    
    IllegalArgumentException
    
    - 메소드의 전달 인자값이 잘못될 경우 발생
    
    ArrayIndexOutOfBoundException
    
    - 배열의 범위를 넘어선 인덱스를 참조할때 발생
    
    NumberFormatException
    
    - 정수가 아닌 문자열을 정수로 변환할 때 발생
    
    ClassCastException
    
    - 타입 변환은 상속이나 구현관계인 경우에만 가능하다
    - 이 규칙을 어기고 억지로 타입을 변환할 때 발생하는 에러이다
    
    InputMismatchException
    
    - 의도치 않는 입력 오류시 발생하는 예외
    
    ### 컴파일 예외 클래스 종류
    
    IOException
    
    - 컴퓨터와 상호작용하는 I/O 에 관한 예외
    
    FileNotFoundException
    
    - 파일에 접근하려고 하는데 파일을 찾지 못했을때 발생하는 에러
    
     
    
    ### 예외는 Checked Exception과 Unchecked Exception으로 나뉠수 있다
    
    Checked Exception은 컴파일 예외 클래스이고
    
    Unchecked Exception은 런타임 예외 클래스 이다 
    
    Checked Exception
    
    - 명시적으로 예외를 처리해야 한다
    - 컴파일 시점에 확인
    
    UnCheckedException
    
    - 명시적인 예외를 하지 않아도 된다
    - 런타임 시점에 확인

- @Valid


    DTO 유효성 검사를 해주는 어노테이션이다 
    
    주로 컨트롤러의 요청 DTO 파라미터에 사용한다
    
    ```jsx
     public ApiResponse<?> createReview(
                @NotBlank @PathVariable("memberId") Long memberId,
                @NotBlank @PathVariable("storeId") Long storeId,
                @RequestBody @Valid CreateReviewDto createReviewDto
                ){
    ```
    
                
    유효성 어노테이션 예시
    
    ```jsx
    public class TestDto {
    
        @Notnull
        Long id;
    
        @NotBlank
        String name;
    
        @Min(value = 1)
        @Max(value = 5)
        Integer rate;
    }
    ```
    
    해당 Validation을 지키지 않을 경우 
    
    MethodArgumentValidException 을 발생 시킨다 
    
    @RestControllerAdvice에서 다음과 같이 작성 할 수 있다 
    
    ```jsx
    @ExceptionHandler(MethodArgumentNotValidException.class)
        public ResponseEntity<ApiResponse<Void>> handleValidDtoException(
                GeneralException ex
        ) {
        
            return ResponseEntity.status(ex.getCode().getStatus())
                    .body(ApiResponse.onFailure(
                            ex.getCode(),
                            null
                    ));
                    
        }
    ```
    
    @Valid와 @Validated 의 차이
    
    @Valid
    
    - DTO 내부의 필드 검증
    - @NotNull, @Size, @Min 등등 사용
    - MethodArgumentValidException 발생
    
    @Validated
    
    - 스프링이 제공하는 확장된 검증 기능
    - @Valid와 달리 그룹 검증 가능
    - Dto가 아닌 메소드 인자에도 검증 가능
    - 컨트롤러가 아닌 계층 에서도 검증 가능
    
    ```jsx
    // Controller 에서 단일 파라미터 검증
    
    public ApiResponse<?> get ((@Validated @Min(1) @RequstParam  Long id) {}
    
    -> ConstraintViolationException 발생 
    ```
    
    서비스 레이어에서 사용
    
    ```jsx
    @Service
    @Validated
    public class TestService {
        
        public void create(@Max(10)Long Id){
            ...
        }
    }
    
    해당 메소드를 호출하면 유효성 검사 가 일어나고 
    위반하는 경우 예외를 터트린다  
    ```
    
    검증 그룹 사용
    
    검증 그룹
    
    - 같은 DTO이지만 상황에 따라서 다른 검사를 하고 싶을 때 사용하는 기능이다
    
    사용 예시
    
    ```jsx
    public interface Update {
    }
    
    public interface Create {
    }
    
    -----------------------------------------------------------------------
    
    public class ReviewDto {
    
        @NotBlank(groups = Create.class)
        private String name;
    
        @NotNull(groups = Update.class)
        private Long id;
    
        @Min(value = 1, groups = {Create.class, Update.class})
        private int rate;
        
        
    
        //이 경우 name은 Create 에서만 검사
        // id는 Update 시에만 검사
        //  rate는 둘다 검사
    }
    
    -------------------------------------------------------------------
    
    @RestController
    @RequiredArgsConstructor
    @RequestMapping("/temp")
    public class controller {
    
        @PostMapping("/review")
        public void createReview(
                @Validated(Create.class) @RequestBody UserDto dto
        ){
    
            ...
        }
    
        @PatchMapping("/review")
        public void createReview(
                @Validated(Update.class) @RequestBody UserDto dto
        ){
    
            ...
        }
    
    }
    
    ```
    
    이를 통해서 DTO를 재사용 할 수 있다