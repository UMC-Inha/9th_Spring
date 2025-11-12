# RestContollerAdvice

  @RestControllerAdvice : Spring에서 전역 예외 처리나 공통 응답 처리를 할 때 사용하는 기능

  → @ControllerAdvice + @ResponseBody

  → @RestController에 대해 전역적으로 적용되는 설정이나 예외 처리를 담당하는 클래스

  [장점]

  1. 예외 처리 코드의 중복 제거
        - 각 컨트롤러마다 try-catch 필요 없이 모든 예외를 한 곳에서 관리 가능
        - 코드가 깔끔해지고 유지보수 쉬워짐

  2. AOP 기반의 전역 처리 가능
        - 스프링의 AOP 기능으로 동작하기 때문에 컨트롤러 내부 코드에 영향 주지 않음

  3. 일관된 에러 응답 형식 유지
        - 모든 에러 응답을 통일할 수 있다.
        - 에러 형식 일정해짐

  [없을 때 불편한점]

  1. 예외 처리할 때 각 컨트롤러마다 try-catch 문을 사용해야한다
  2. 에러 응답 형식이 컨트롤러마다 다를 수 있다
  3. 예외를 하나 추가할 때마다 여러 컨트롤러의 수정이 필요하다
  4. 코드가 복잡하고 중복이 많기 때문에 가독성이 떨어진다

  [같이 사용되는 어노테이션]

  | 어노테이션 | 설명 |
      | --- | --- |
  | `@ExceptionHandler(Exception.class)` | 특정 예외 타입 처리 |
  | `@ResponseStatus(HttpStatus.BAD_REQUEST)` | 응답 상태코드 지정 |
  | `@RestControllerAdvice(basePackages = "com.example.api")` | 특정 패키지에만 적용 |
  | `@Slf4j` | 로그 찍기 (예외 로그 출력용) |

  [@RestControllerAdvice 사용 과정]

  1. 에러 코드 정의

    ```
    package com.example.umc9th.global.apiPayload.code;
    
    import org.springframework.http.HttpStatus;
    
    public interface BaseErrorCode {
        HttpStatus getStatus();
        String getCode();
        String getMessage();
    }
    
    package com.example.umc9th.global.apiPayload.code;
    
    import lombok.AllArgsConstructor;
    import lombok.Getter;
    import org.springframework.http.HttpStatus;
    
    @Getter
    @AllArgsConstructor
    public enum GeneralErrorCode implements BaseErrorCode{
    
        BAD_REQUEST(HttpStatus.BAD_REQUEST,
                "COMMON400_1",
                "잘못된 요청입니다."),
        UNAUTHORIZED(HttpStatus.UNAUTHORIZED,
                "AUTH401_1",
                "인증이 필요합니다."),
        FORBIDDEN(HttpStatus.FORBIDDEN,
                "AUTH403_1",
                "요청이 거부되었습니다."),
        NOT_FOUND(HttpStatus.NOT_FOUND,
                "COMMON404_1",
                "요청한 리소스를 찾을 수 없습니다."),
        INTERNAL_SERVER_ERROR(HttpStatus.INTERNAL_SERVER_ERROR,
                "COMMON500_1",
                        "예기치 않은 서버 에러가 발생했습니다."),
                ;
    
        private final HttpStatus status;
        private final String code;
        private final String message;
    }
    
    ```

  2. 커스텀 예외를 만든다

    ```
    package com.example.umc9th.global.apiPayload.exception;
    
    import com.example.umc9th.global.apiPayload.code.BaseErrorCode;
    import lombok.AllArgsConstructor;
    import lombok.Getter;
    
    @Getter
    @AllArgsConstructor
    public class GeneralException extends RuntimeException {
    
        private final BaseErrorCode code;
    }
    
    ```

  3. @RestControllerAdvice를 선언한 클래스 만든다

    ```
    package com.example.umc9th.global.apiPayload.handler;
    
    import com.example.umc9th.global.apiPayload.ApiResponse;
    import com.example.umc9th.global.apiPayload.code.BaseErrorCode;
    import com.example.umc9th.global.apiPayload.code.GeneralErrorCode;
    import com.example.umc9th.global.apiPayload.exception.GeneralException;
    import org.springframework.http.ResponseEntity;
    import org.springframework.web.bind.annotation.ExceptionHandler;
    import org.springframework.web.bind.annotation.RestControllerAdvice;
    
    @RestControllerAdvice
    public class GeneralExceptionAdvice {
    
        // 애플리케이션에서 발생하는 커스텀 예외를 처리
        @ExceptionHandler(GeneralException.class)
        public ResponseEntity<ApiResponse<Void>> handleException(
                GeneralException ex
        ) {
    
            return ResponseEntity.status(ex.getCode().getStatus())
                    .body(ApiResponse.onFailure(
                                    ex.getCode(),
                                    null
                            )
                    );
        }
    
        // 그 외의 정의되지 않은 모든 예외 처리
        @ExceptionHandler(Exception.class)
        public ResponseEntity<ApiResponse<String>> handleException(
                Exception ex
        ) {
    
            BaseErrorCode code = GeneralErrorCode.INTERNAL_SERVER_ERROR;
            return ResponseEntity.status(code.getStatus())
                    .body(ApiResponse.onFailure(
                                    code,
                                    ex.getMessage()
                            )
                    );
        }
    }
    
    ```

  4. Controller나 Service에서 커스텀 예외를 발생시킨다.

# lombok

  Lombok : 자바 개발을 더욱 편리하게 만들어주는 라이브러리 , 반복적이고 번거로운 코드를 줄여주고 표준적인 메서드와 필드를 자동으로 생성

  → 어노테이션을 클래스나 필드에 추가하면 컴파일 시점에 해당 어노테이션에 해당하는 코드를 자동으로 생성해주는 도구

  [Lombok 어노테이션과 기능]

  1. @Getter , @Setter : 필드에 대한 getter와 setter 메서드를 자동으로 생성
  2. @ToString : 클래스의 toString() 메소드를 자동으로 생성
  3. @EqualsAndHashCode : equals()와 hashCode() 메서드를 자동으로 생성
  4. @NoArgsConstructor : 파라미터가 없는 기본 생성자를 자동으로 생성
  5. @AllArgsConstructor : 모든 필드를 포함하는 생성자를 자동으로 생성
  6. @Data : 1,2,3,4,5 어노테이션을 한 번에 적용
  7. @Builder : 빌더 패턴을 사용하여 객체를 생성하는 빌더 클래스를 자동으로 생성

  [장점]

  | **구분** | **내용** |
      | --- | --- |
  | **코드 간결성** | Getter, Setter, 생성자 등의 **반복적인 코드를 대폭 줄여** 코드를 간결하게 만듭니다. |
  | **가독성 향상** | 클래스의 핵심 필드만 남게 되어, 데이터 모델의 **구조를 한눈에 파악**하기 쉽습니다. |
  | **개발 속도 향상** | 코드 작성량이 줄어들어 개발 시간이 단축됩니다. |

  [단점]

  | **구분** | **내용** |
      | --- | --- |
  | **별도의 설정 필요** | 개발 환경(IDE, Maven/Gradle 등)에 롬복 플러그인이나 의존성을 추가해야 정상적으로 코드가 인식됩니다. |
  | **숨겨진 코드** | 코드가 명시적으로 보이지 않고 컴파일 시점에 생성되므로, 내부 동작을 확인하려면 IDE의 설정(예: 디컴파일된 코드 확인)이 필요하거나 소스 코드를 자세히 읽어야 하는 불편함이 있을 수 있습니다. |
  | **오버헤드 (경미함)** | 코드를 읽는 개발자가 롬복을 모를 경우, 처음에 코드를 이해하는 데 시간이 걸릴 수 있습니다. |

  [예시]

    ```
    public class User {
        private String name;
        private int age;
    
        public User(String name, int age) {
            this.name = name;
            this.age = age;
        }
    
        public String getName() { return name; }
        public void setName(String name) { this.name = name; }
    
        public int getAge() { return age; }
        public void setAge(int age) { this.age = age; }
    
        @Override
        public String toString() {
            return "User{name='" + name + "', age=" + age + "}";
        }
    }
    
    -----------------------------------------------------------------
    
    import lombok.*;
    
    @Getter
    @Setter
    @ToString
    @AllArgsConstructor
    public class User {
        private String name;
        private int age;
    }
    ```

    ```java
    import lombok.*;
    
    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public class User {
        private String name;
        private int age;
    }
    
    /* lombok을 통해 이러한 메서드가 자동 생성된다
    
    getName(), getAge()
    setName(), setAge()
    toString()
    equals(), hashCode()
    User(String name, int age)
    User() (기본 생성자)
    User.builder().name("재준").age(23).build()
    
    ```

# dto 형식 public static VS record 비교하기

자바에서 DTO를 만들 때 대표적으로 쓰이는 두 가지 방식
-> 데이터를 전달하기 위한 용도지만 설계 철학과 사용 목적이 다르다

  1. public static class DTO 방식
        - 전통적인 자바 방식의 DTO
        - 클래스 내부에 public static class 형태로 선언해 관련 DTO들을 한 곳에서 관리할 때 사용
        - getter, setter, builder 등 자유롭게 추가 가능

        ```
        public class UserDto {
        
            // 요청(Request) DTO
            public static class Request {
                private String name;
                private int age;
        
                public Request() {} // 기본 생성자
                public Request(String name, int age) {
                    this.name = name;
                    this.age = age;
                }
        
                public String getName() { return name; }
                public void setName(String name) { this.name = name; }
                public int getAge() { return age; }
                public void setAge(int age) { this.age = age; }
            }
        
            // 응답(Response) DTO
            public static class Response {
                private Long id;
                private String name;
        
                public Response(Long id, String name) {
                    this.id = id;
                    this.name = name;
                }
        
                public Long getId() { return id; }
                public String getName() { return name; }
            }
        }
        ```


  [특징]
  - 한 파일 안에 여러 DTO를 정리 가능
  - 가변 객체 - setter로 수정 가능
  - Lombok 사용 시 코드 간결화
  - 필요시 메서드 추가 및 필드 가공 로직 삽입 가능하기 때문에 유연함
  - 한 클래스 내부에서 여러 DTO 관리 가능
    
  [단점]
  - 코드가 길어짐
  - setter로 인해 불변성이 깨질 수 있다
    
    1. record DTO 방식
        - Java 16 이상에서 지원하는 불변 데이터 클래스
        - 데이터 전달만 담당하기 때문에 DTO 목적에 완벽히 부합한다
        - 필드, 생성자, getter, equals, hashCode, toString 자동 생성
        
        ```
        public record UserResponse(Long id, String name, int age) {}
        
        ---------------------------------------------------------
        
        public final class UserResponse {
            private final Long id;
            private final String name;
            private final int age;
        
            public UserResponse(Long id, String name, int age) {
                this.id = id;
                this.name = name;
                this.age = age;
            }
        
            public Long id() { return id; }
            public String name() { return name; }
            public int age() { return age; }
        
            public boolean equals(Object o) { ... }
            public int hashCode() { ... }
            public String toString() { ... }
        }
        ```
        
    
  [특징]
    
  - 불변객체
  - 간결함
  - 데이터 전달에 특화됨
  - Lombok 불필요
  - JSON 직렬화 완벽 호환
    
  [단점]
    
  - setter, builder 사용불가
  - 복잡한 로직이나 가공 메서드를 담기 어려움
    

  [실제 프로젝트 예시]
    
    ```
    // 📂 dto/UserDto.java
    public class UserDto {
    
        // 요청 DTO , Controller에서 요청을 받을 때
        public static class Request {
            private String name;
            private int age;
        }
    
        // 응답 DTO , Service/Controller에서 클라이언트로 응답을 보낼 때
        public record Response(Long id, String name, int age) {}
    }
    
    ```