### java의 Exception 종류들
![image](./image/java-exception-handling-class-hierarchy-diagram.jpg)
위는 자바의 예외 계층 구조를 그린 다이어그램이다.

자바에서 모든 예외와 오류의 최상위 부모는 Throwable 클래스이다(예외 처리 시스템 안에서의 최상위 부모). 이 Throwable은 크게 두가지로 나뉜다.

1. **Error(오류)**

   이것은 시스템 레벨의 심각한 문제를 의미하며, 개발자가 코드(try-catch)로 복구할 수 없는 상황을 말한다.

    - **원인**: 주로 JVM(자바 가상 머신)자체에 문제가 생겼을 때 발생
    - 예시:
        - OutOfMemoryError: 메모리가 부족할 때 발생

            ```java
            while (true) {
                        list.add(new Object[1_000_000]); // 100만 칸짜리 배열을 계속 추가
                    }
            ```

        - StackOverflowError: 메서드 호출이 너무 깊어져 스택이 가득 찰 때 발생(무한재귀)

            ```java
            public class StackOverflowExample {
                public static void callMyself() {
                    // 종료 조건(base case) 없이 자신을 계속 호출 (무한 재귀)
                    callMyself();
                }
            main에서 callMyself() 호출 시 무한재귀
            ```

2. **Exception(예외)**

   이것은 개발자가 직접 처리(복구)할 수 있는 문제들이다. Exception은 두 가지 핵심 그룹으로 나뉜다

    1. **CheckedException(검사받는 예외)**

       이름 그대로 컴파일러가 “이 코드는 위험하니 예외 처리를 했는지 검사하겠다!”라고 강제하는 예외이다.

        - **특징**: 이 예외를 발생시킬 가능성이 있는 코드를 사용하면, 반드시 try-catch로 감싸거나, throws 키워드로 이 예외 처리를 다른 메서드에게 떠넘겨야 한다.

          → 그렇게 하지 않으면 컴파일 오류 발생

        - **발생 상황**: 주로 네트워크, 파일, 데이터베이스 등 외부 자원과 통신할 때처럼 개발자가 제어할 수 없는 외부 요인으로 발생
        - **예시:**
            - IOException: 파일 읽기/쓰기 실패 시 발생

                ```java
                FileReader reader = new FileReader("a_file_that_does_not_exist.txt")
                //이때 존재하지 않는 파일을 읽으려고 시도하면 IOEception
                ```

            - SQLException: 데이터베이스 접근시 오류 발생

              → 존재하지 않는 DB에 접속 시도시 SQL 예외

            - ClassNotFoundException: 해당 클래스를 찾지 못할 때 발생

              → 존재하지 않는 클래스를 찾으려고 할 때 이 예외

    2. **Unchecked Exception(검사받지 않는 예외)**

       RuntimeException 클래스와 그 자식들을 말한다. Checked Exception과 달리 컴파일러가 예외 처리를 강제하지 않는다.

        - **특징:**
            - 개발자가 try-catch나 throws를 쓰지 않아도 컴파일 오류가 나지 않는다.
            - 주로 개발자의 논리적인 실수(버그)로 인해 발생한다
        - **발생 상황:** 잘못된 코드 작성이나 데이터 처리로 인해 프로그램 실행 중에(런타임) 발생한다.
        - **예시:**
            - NullPointerException(NPE): null인 객체의 메서드나 변수를 사용하려고 할 때 발생

                ```java
                String text = null;
                System.out.println(text.length()); <-이때 NPE 발생
                ```

            - ArrayIndexOutOfBoundsException: 배열의 범위를 벗어난 인덱스를 접근할 때 발생

                ```java
                int[] numbers = new int[3];
                int number = numbers[5] <- 이때 예외 발생
                ```

            - IllegalArgumentException: 메서드에 잘못된 인자(값)을 전달했을 때 발생

                ```java
                public static void setAge(int age) {
                        }
                다른 곳에서 setAge(-10) 이런식으로 호출 시 잘못된 인자 전달로 예외 발생
                ```

            - ClassCastException: 잘못된 타입으로 형 변환을 시도할 때 발생

                ```java
                // 1. Object(모든 것) 타입 상자에 글자(String)를 담음
                        //   (부모 타입에 자식 객체를 담는 것은 OK)
                        Object anyObject = "사과";
                // 2. 이 상자(anyObject)에 사과(String)가 들어있는데
                //   "이건 숫자(Integer)야!"라고 강제로 형 변환을 시도
                    
                Integer myNumber = (Integer) anyObject; <-사과는 숫자가 될 수 없기 때문에 여기서 예외가 발생
                
                           
                ```
              
### @Valid

**@Valid의 역할**

@Valid를 한마디로 정의하면 유효성 검사 실행 트리거이다.

비유를 해보자면 @Valid는 공항의 보안 검색대 직원이다.

- DTO: 반입 금지 물품 목록(규칙서)
- @Valid(검색대 직원): 승객(Request)이 DTO(규칙서)를 들고 올 때, “잠깐 멈춰! 이 규칙서대로 검사 시작하겠습니다!!!”라고 검사를 실행하는 역할을 한다.

→ 만약 이 직원(Valid)이 없다면, 규칙서(@NotBlank 등)가 DTO에 아무리 많이 붙어있어도 아무도 검사하지 않고 그냥 통과시켜 버린다.

**@Valid의 작동 원리**

@Valid는 혼자 일하지 않는다. DTO, 컨트롤러, 예외 핸들러와 유기적으로 연동되며 다음과 같은 순서로 동작한다.

**1단계: 요청 접수 (Controller)**

클라이언트가 회원가입을 위해 JSON 데이터를 POST로 보낸다

```java
@PostMapping("/sign-up")
public ApiResponse<JoinDTO> signUp(@RequestBody @Valid UserReqDTO.JoinDto joinDto) {
    // ...
}
```

**2단계: 검사 시작 (Controller)**

객체 변환 직후, Spring은 @Valid 어노테이션을 발견한다. 이때 검색대 직원이 검사 시작!!을 외친다.

**3단계: 규칙 대조 (DTO)**

Spring은 UserReqDTO.JoinDTO 클래스 내부를 샅샅이 뒤지며 유효성 검사 어노테이션(규칙서)를 찾는다.

```java
// UserReqDTO.java
public record JoinDto(
    @NotBlank // (규칙 1) "name은 비어있으면 안 됨!"
    String name,
    
    // ... (다른 규칙들)
    
    @ExistFoods // (규칙 N) "preferCategory는 DB에 존재하는 Food ID여야 함!"
    List<Long> preferCategory
){}
```

Spring은 이 모든 규칙(@NotBlank, @ExistFoods 등)을 하나씩 전부 실행한다.

**4단계: 결과 판정**

여기서 길이 두 갈래로 나뉜다.

A. 검증 성공(Success)

- Spring은 아무 일 없다는 듯이 signUp 메서드의 본문 코드를 실행한다.

B. 검증 실패(Failure)

- 검색대 직원(Valid)이 위반!!!을 외친다.
- Spring은 즉시 signUp 메서드 실행을 중단하고 MethodArgumentNotValidException이라는 이름의 예외를 발생시킨다.

**@Valid와 @RestControllerAdvice**

@Valid가 검증에 실패해서 MethodArgumentNotValidException을 던지면, Spring은 이 예외를 처리할 수 있는 예외 처리 전문가를 찾는다. 그 전문가가 바로 @RestControllerAdvice이다.

→ @Valid는 컨트롤러의 1차 방어선 역할을 한다. 이 방어선이 뚫리면(검증 실패) MethodArgumentNotValidException이라는 경고 사이렌을 울리고, 이 사이렌을 @RestControllerAdvice가 듣고 출동하여 깔끔한 보고서(ApiResponse)를 작성해 클라이언트에 응답하는 것이다.

**@Valid를 쓰는 이유 → 관심사의 분리**

@Valid가 없다면 Service 안에서 아래처럼 유효성 검사 코드를 싹 다 써야한다. → 코드가 엄청 길어짐

```java
if (joinDto.name() == null || joinDto.name().isBlank()) {
        throw new IllegalArgumentException("이름은 필수입니다.");
    }
```

하지만 Valid를 사용하면 모든 지저분한 검증 로직이 컨트롤러 단계에서 깔끔하게 처리된다. Service Layer에서는 이 데이터는 이미 검증되었겠지..라고 안심하고 핵심 비즈니스 로직에만 집중할 수 있다.