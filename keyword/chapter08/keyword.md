# java의 Exception 종류들
  - 예외 처리 : 프로그램의 실행 도중에 발생할 수 있는 오류나 예기치 않은 상황에 대한 대비를 위해 코드를 작성하는 것 → 실행 중인 프로그램의 갑작스러운 비정상 종료를 방지하고 정상적인 실행 상태를 유지할 수 있도록하는 목적

  ![image.png](image%20(3).png)

  - 에러와 예외의 차이
    - 에러는 프로그램 코드에 의해 수습될 수 없는 심각한 오류
    - 예외는 프로그램 코드에 의해 수습될 수 있는 다소 미약한 오류

  - 예외 처리 종류

  ![image.png](image%20(4).png)

  - 확인된 예외 (Checked Exception)
    - 컴파일러가 강제로 처리하도록 하는 예외
    - try-catch 혹은 thorws 키워드 사용하여 명시적으로 처리
      - try - catch 사용할 때 모든 에러 Exception 사용?
        - 모든 예외를 잡으려면 특정 유형의 오류를 식별하고 처리하기가 어려워져 코드에서 잠재적인 문제와 예기치 않은 동작 발생 가능
      - throws : 메서드가 예외를 던질 수 있다는 것을 선언하는데 사용. 이에 처리하는 책임은 코드나 호출 스택의 상위 메소드에 위임
    - 주로 입출력, 네트워크 연결 등 외부 리소스와 관련된 예외
    - 컴파일 단계에서 발생하지 않는 예외처리 어떻게?
      - 런타임 예외처리 : 실행 중에 발생할 수 있는 예외를 처리하는 코드
      - 오류 코드 반환
      - 로그기록

    ```
                public void readFile() throws IOException {
                    FileReader fr = new FileReader("test.txt");
                }
    ```

  | 예외 클래스 | 설명 |
                          | --- | --- |
              | `IOException` | 입출력 작업 시 발생 |
              | `SQLException` | 데이터베이스 작업 오류 |
              | `ClassNotFoundException` | 클래스 로딩 실패 |
              | `FileNotFoundException` | 파일을 찾을 수 없는 경우 |
              | `InterruptedException` | 스레드가 중단될 때 |
            
- 확인되지 않은 예외 (Unchecked Exception)
  - 개발자의 실수나 예상치 못한 상황 등으로 발생하는 예외
  - 컴파일러가 강제로 처리하지 않고 예외가 발생하면 프로그램이 중단된다
  - 주로 배열 인덱스 오류, null pointer 오류 등의 런타임 에러

  | **오류 유형** | **설명** |
                            | --- | --- |
                | **NullPointerException** | 프로그램이 null인 객체 참조를 접근하거나 조작하려고 할 때 발생합니다. |
                | **ArrayIndexOutOfBoundsException** | 배열 인덱스가 범위를 벗어날 때 발생합니다. (음수이거나 배열 크기보다 큰 경우) |
                | **ClassCastException** | 호환되지 않는 타입으로 캐스트하려고 할 때 발생합니다. |
                | **IllegalArgumentException** | 메서드나 생성자에 잘못된 인수가 전달될 때 발생합니다. |
                | **IllegalStateException** | 객체의 상태가 요청된 작업에 적합하지 않을 때 발생합니다. |
                | **NoSuchMethodError** | 클래스나 슈퍼클래스에서 참조된 메서드를 찾을 수 없을 때 발생합니다. |
                | **OutOfMemoryError** | Java 가상 머신 (JVM)이 메모리를 다 사용했을 때 발생합니다. |
                | **StackOverflowError** | 재귀 호출로 인해 호출 스택이 제한을 초과할 때 발생합니다. |

# @Valid

  @Valid : Java Bean Validation 스펙을 구현한 환경에서 객체의 유효성 검사를 수행하도록 지정하는데 사용 , 컨트롤러에서만 사용 가능

  → 객체를 처리하기 전에 해당 객체 내부의 필드들에 설정된 유효성 제약 조건이 제대로 충족되는지 자동으로 확인하고 검증할 수 있다

  [동작원리]

  1. 모든 요청 프런트 컨트롤러인 디스패처 서블릿을 통해 컨트롤러로 전달. 전달과정에서 컨트롤러 메서드의 객체를 만들어주는 ArgumentResolver가 동작하고 @Valid 역시 이에 의해 처리
  2. @RequestBody는 Json 메세지를 객체로 변환해주는 작업이 ArgumentResolver의 구현체에 의해 처리된다. 그리고 이 내부에서 @Value로 시작하는 어노테이션이 있는 경우 유효성 검사를 진행한다
  3. 검증에 오류가 있다면 MethodArgumentNotValidExceptoin 예외가 발생하고 되고 디스패처 서블릿에 기본으로 등록된 Exception Resolver인 DefaultHandlerExceptionResolver에 의해 400 BadRequest 에러 발생

  ⇒ 따라서 기본적으로 컨트롤러에서만 동작하고 다른 계층에서는 검증되지 않는다


  [유효성 검사] : 각 계층에서 들어오는 데이터에 대해 의도한 형태대로 값이 들어오는지 체크하는 과정
    
  ![image.png](image%20(5).png)
    
  [Bean Validation] : Java에서 Bean Validation이라는 프레임워크를 제공하고 있고 스프링 부트에서는 Bean Validation의 구현체인 Hibernate Validator를 유효성 검사의 표준으로 채택
    
  [사용 예시]
    
  ```
    public class UserDTO {
        @NotNull
        private String name;
    
        @Email
        private String email;
    
        @Min(18)
        private int age;
    
        // getters, setters
    }
    
   ```
    
  ```
    @PostMapping("/sign-up")
        public ApiResponse<MemberResDTO.JoinDTO> signUp(
                @RequestBody @Valid MemberReqDTO.JoinDTO dto
        ){
        
        // 자동으로 유효성 검사
  ```
    
@Validated : 스프링에서 제공하는 @Valid 기능을 확장한 어노테이션이고 그룹핑 목적으로 사용
    
→ 특정 필드만 유효성 검사를 하고 싶을 경우에는 필드를 그룹핑하여 일부만 유효성 검사 진행
    
⇒ 스프링 빈이라면 유효성 검증 가능
    
```
    @Service
    @Validated
    public class UserService {
    
    	public void addUser(@Valid AddUserRequest addUserRequest) {
    		...
    	}
    }
    
    // 클래스에 @Validated 붙이고, 유효성 검증할 파라미터에 @Valid를 붙여주면 된다
```