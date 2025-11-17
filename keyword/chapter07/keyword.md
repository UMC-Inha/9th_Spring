- RestContollerAdvice


    전역 예외 처리를 위해 사용하는 애노테이션이다
    
    전역적으로 예외를 잡아서 JSON 형태로 응답할 수 있게 해주는 기능이다
    
    @ControllerAdvice와 @ResponseBody를 합한 역할을 수행한다
    
    @ControllerAdvice
    
    - 모든 @Controller에서 발생한 예외를 한 곳에서 처리할 수 있게 한다
    - AOP 방식으로 동작한다
    
    @ResponseBody
    
    - 반환값을 View가 아니라 Json 형태로 바꿔서 클라이언트에게 전달
    
    @RestControllerAdvice
    
    - REST API 용 예외 처리기
    
    코드 예시
    
    ```jsx
    @RestControllerAdvice
    public class GeneralExceptionAdvice {
    
        @ExceptionHandler(IllegalArgumentException.class)
        public ResponseEntity<ApiResponse<Void>> handleException(
                GeneralException ex
        ) {
            return ResponseEntity.status(ex.getCode().getStatus())
                    .body(ApiResponse.onFailure(
                            ex.getCode(),
                            null
                    ));
        }
        
        
        @ExceptionHandler(Exception.class)
        public ResponseEntity<ApiResponse<Void>> handleException(
                GeneralException ex
        ) {
            return ResponseEntity.status(ex.getCode().getStatus())
                    .body(ApiResponse.onFailure(
                            ex.getCode(),
                            null
                    ));
        }
    
    }
    
    ```
    
    자주사용하는 예외 핸들링 클래스
    
    - MethodArgumentNotValidException
        - @Valid 유효성 검증
        - DTO 검증
    - ConstraintViolationException
        - @Validated 유효성 검증
        - PathVariable 검증
    - EntityNotFoundException
        - DB 조회 실패
    - AccessDeniedException
        - 스프링 시큐리티 사용시 발생하는 권한 문제

- lombok


    자바에서 반복적으로 작성해야 하는 코드를 자동으로 생성해주는 컴파일 타임 라이브러리이다
    
    동작 원리
    
    - 컴파일 시점에 Annotation Processor이 동작해서 자동으로 특정 메소드 주입
    - 컴파일 결과물에는 해당 메소드가 존재하게 된다
    
    자주 사용하는 롬복 어노테이션
    
    - @Getter, @Setter
        - 모든 필드에 세터와 게터 생성
        - 필드의 상태를 함부로 바꾸면 안되는 상황에서는 @Setter을 남발하지 말자…
    - @ToString
        - toString() 자동 생성
    - @EqualsAndHashCode
        - equals(), hashCode() 자동 생성
    - @NoArgsConstructor
        - 기본 생성자 생성
    - @AllArgsConstructor
        - 모든 필드를 포함한 생성자 생성
    - @RequiredArgsConstructor
        - final 또는 @NonNull 필드만 포함해서 생성자 생성
    - @Builder
        - 빌더 패턴 생성
    - @Data
        - @Getter, @Setter, @ToString, @EqualsAndHashCode, @RequiredArgsConstructor 을 전부 포함한다
    - @Slf4j
        - 로그 객체를 자동으로 생성한다
    
    예시 코드
    
    ```jsx
    @Service
    @RequiredArgsConstructor
    public class Testing {
        
        private final TestRepository testRepository;
        
        
    }
    
    ---------------------------------------------------------
    
    public class Testing {
    
        private final TestRepository testRepository;
    
        public Testing(TestRepository testRepository) {
            this.testRepository = testRepository;
        }
    }
    ```
    
    ```jsx
    @AllArgsConstructor
    @NoArgsConstructor
    @Getter
    @Setter
    @ToString
    public class Testing {
    
        private String name;
        
        private Long id;
        
        private Long age;
        
        
    }
    
    -------------------------------------------------------
    
    public class Testing {
    
        private String name;
    
        private Long id;
    
        private Long age;
    
        public Testing(String name, Long id, Long age) {
            this.name = name;
            this.id = id;
            this.age = age;
        }
    
        public Testing() {
        }
    
        public String getName() {
            return name;
        }
    
        public void setName(String name) {
            this.name = name;
        }
    
        public Long getId() {
            return id;
        }
    
        public void setId(Long id) {
            this.id = id;
        }
    
        public Long getAge() {
            return age;
        }
    
        public void setAge(Long age) {
            this.age = age;
        }
    
        @Override
        public String toString() {
            return "Testing{" +
                    "name='" + name + '\'' +
                    ", id=" + id +
                    ", age=" + age +
                    '}';
        }
    }
    
    ```

- dto 형식 public static VS record 비교하기


    public static class DTO
    
    - 내부 클래스를 DTO로 정의한다
    - DTO 마다 getter, setter, 생성자 등이 존재 해야 한다
        - 주로 롬복을 사용
    - 수정이 가능하다
    - 주로 요청용 DTO 에 사용한다
    - 장점
        - 필요시 setter을 추가 할 수 있다
        - 스프링 검증에 최적화
            - @Valid, @NotNUll
    - 단점
        - 불변성이 보장되지 않는다
        - 보일러 플레이트 코드 존재
            - 프로그램의 핵심 로직과는 상관 없이 반복적으로 작성해야 하는 코드
    
    예시 코드
    
    ```jsx
    @AllArgsConstructor
    @NoArgsConstructor
    @Getter
    
    public static class Testing {
    
        private Long id;
        
        private String name;
    }
    
    --------------------------------------
    
    public  class Testing {
    
        private Long id;
    
        private String name;
    
        public Testing() {
        }
    
        public Testing(Long id, String name) {
            this.id = id;
            this.name = name;
        }
    
        public Long getId() {
            return id;
        }
    
        public String getName() {
            return name;
        }
    }
    ```
    
    record DTO 
    
    - 불변 데이터 전용 타입
    - 모든 필드가 final (불변), getter, toString 등이 자동 생성
    - 불변 데이터 컨테이너로 수정이 불가능하다
    - 주로 응답용 DTO에 사용한다
    - 장점
        - 코드가 보다 간결함
        - 다양한 함수들이 자동으로 구현되어 있다
        - 불변 객체로 불변성이 보장된다
    - 단점
        - 모든 필드에 final이 붙기에 값을 변경하지 못한다
        - 기본 생성자가 없다
        - @Valid 검증이 어렵다
        
    
    예시 코드
    
    ```jsx
    public record Testing(Long id, String name){
    	
    }
    
    //컴파일러가 자동으로 아래 코드를 만든다 
    ---------------------------------------------------------
    
    final  class Testing extends Record{
    
        private final Long id;
    
        private final String name;
    
        
        public Testing(Long id, String name) {
            this.id = id;
            this.name = name;
        }
    
        public Long getId() {
            return id;
        }
    
        public String getName() {
            return name;
        }
    
        @Override
        public boolean equals(Object o) {
            if (o == null || getClass() != o.getClass()) return false;
            Testing testing = (Testing) o;
            return Objects.equals(id, testing.id) && Objects.equals(name, testing.name);
        }
    
        @Override
        public int hashCode() {
            return Objects.hash(id, name);
        }
    
        @Override
        public String toString() {
            return "Testing{" +
                    "id=" + id +
                    ", name='" + name + '\'' +
                    '}';
        }
    }
    ```


