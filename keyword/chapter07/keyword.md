# 🌐 RestContollerAdvice

---

### 🧩 RestControllerAdvice = ControllerAdvice + ResponseBody

- **ControllerAdvice**  
  👉 전역 컨트롤러 보조자 (모든 컨트롤러 대상)
- **ResponseBody**  
  👉 리턴 값을 JSON 등으로 반환

📌 **@RestControllerAdvice**는 애플리케이션 전반에서 발생하는 예외를 전역적으로 처리함
```java
throw new UserNotFoundException("해당 사용자를 찾을 수 없습니다.");
//이런 예외가 발생했을 때

@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ApiResponse<?>> handleUserNotFound(UserNotFoundException e) {
        return ResponseEntity
                .status(HttpStatus.NOT_FOUND)
                .body(ApiResponse.onFailure("U404", e.getMessage(), null));
    }
}
//이런식으로 @RestControllerAdvice를 확인한다.
```
---

## ⚙️ @ExceptionHandler란

> **Spring에서 예외(Exception)를 잡아서 처리하는 메서드에 붙이는 어노테이션**이에요.

---

## 🔄 클라이언트의 요청이 들어오고, 요청 처리 중 발생하는 에러 핸들링 흐름

---

### 1️⃣ **클라이언트 요청 처리**
- (GET 요청에 대한 비즈니스 로직을 처리할 경우 **TempQueryService**)
```java
@Service
@RequiredArgsConstructor
public class TempQueryServiceImpl implements TempQueryService {
    @Override
    public void CheckFlag(Integer flag) {
        if(flag == 1){
            throw new TempHandler(ErrorStatus.TEMP_EXCEPTION); // 조건에 따라 예외 발생
        }
    }
}
```

---

### 2️⃣ **사용자 정의 예외 클래스 : TempHandler와 GeneralException**
- **GeneralException**은 에러 코드를 기반으로  
  에러 메시지와 HTTP 상태 코드를 포함하는 **ErrorReasonDTO** 객체를 반환합니다.
```java
public class TempHandler extends GeneralException { //TestException = TempHandler
    public TempHandler(BaseErrorCode errorCode) {
        super(errorCode);
    }
}

@Getter
@AllArgsConstructor
public class GeneralException extends RuntimeException {
    private BaseErrorCode code;

    public ErrorReasonDTO getErrorReason(){
        return this.code.getReason();
    }

    public ErrorReasonDTO getErrorReasonHttpStatus(){
        return this.code.getReasonHttpStatus();
    }
}
```
---

### 3️⃣ **ExceptionAdvice: 예외를 감지하고 처리하는 클래스**
```java
@Slf4j
@RestControllerAdvice(annotations = {RestController.class})
public class ExceptionAdvice extends ResponseEntityExceptionHandler {

    @ExceptionHandler(value = GeneralException.class)
    public ResponseEntity<Object> onThrowException(GeneralException generalException, HttpServletRequest request) {
        ErrorReasonDTO errorReasonHttpStatus = generalException.getErrorReasonHttpStatus();
        return handleExceptionInternal(generalException, errorReasonHttpStatus, null, request);
    }

    private ResponseEntity<Object> handleExceptionInternal(Exception e, ErrorReasonDTO reason,
                                                           HttpHeaders headers, HttpServletRequest request) {
        ApiResponse<Object> body = ApiResponse.onFailure(reason.getCode(), reason.getMessage(), null);
        WebRequest webRequest = new ServletWebRequest(request);
        return super.handleExceptionInternal(
                e,
                body,
                headers,
                reason.getHttpStatus(),
                webRequest
        );
    }
}
```
- **onThrowException** 메서드는  
  **GeneralException**이 발생할 경우  
  에러 메시지와 상태 코드를 포함하는 **ApiResponse** 객체를 생성해 반환합니다.

- **handleExceptionInternal** 메서드는  
  **ApiResponse** 형식의 응답을 생성하며,  
  표준화된 에러 응답을 클라이언트에 전달합니다.

---

### 4️⃣ **ApiResponse: 표준화된 에러 응답 생성**
```java
public class ApiResponse<T> {
    private final Boolean isSuccess;
    private final String code;
    private final String message;
    @JsonInclude(JsonInclude.Include.NON_NULL)
    private T result;

    public static <T> ApiResponse<T> onFailure(String code, String message, T data) {
        return new ApiResponse<>(false, code, message, data);
    }
}
```
- **ExceptionAdvice** 클래스에서 이 객체를 사용하여  
  클라이언트에게 표준화된 에러 응답을 전송합니다.

---

## 🧭 예외 발생부터 클라이언트 응답까지의 전체 흐름 요약

1. **클라이언트 요청**  
   → flag 값을 포함한 요청이 들어오면,  
   TempQueryServiceImpl의 CheckFlag 메서드가 실행됩니다.

2. **예외 발생**  
   → flag 값이 1일 경우, TempHandler 예외가 발생합니다.

3. **ExceptionAdvice에서 예외 처리**
    - TempHandler 예외가 GeneralException을 상속하므로,  
      ExceptionAdvice의 onThrowException 메서드에서 감지됩니다.
    - onThrowException 메서드는 GeneralException의 에러 코드를 사용해  
      **ErrorReasonDTO**를 가져와 표준화된 에러 응답을 생성합니다.

4. **ApiResponse 응답 생성**  
   → ApiResponse.onFailure 메서드를 통해  
   표준화된 실패 응답을 생성하고, 클라이언트에 전송합니다.

5. **클라이언트 응답**  
   → 클라이언트는 통일된 형식으로  
   에러 메시지, 에러 코드, 상태 코드를 포함한 JSON 응답을 받게 됩니다.

---

📦 **정리**
> 예외 발생 시 `@RestControllerAdvice`와 `@ExceptionHandler`를 통해  
> 전역적으로 예외를 감지하고,  
> `ApiResponse`를 통해 표준화된 형식으로 클라이언트에 전달합니다.

---

# 🍃 Lombok(롬복)

---

## 📘 Lombok이란?

**Lombok(롬복)**은 Java 개발에서 반복적으로 작성해야 하는  
**보일러플레이트 코드(boilerplate code)**를 자동으로 만들어주는 **라이브러리**예요.

---

## 🧱 보일러플레이트 코드(boilerplate code)란?

> 소프트웨어 개발에서 자주 **"반복적으로 사용하는 기본적인 코드 블록"**을 말합니다.

---

## ⚙️ Lombok의 주요 기능

Lombok은 주로 클래스의 **어노테이션(annotations)**을 이용하여  
`getter`, `setter`, `toString`, `equals`, `hashCode` 메서드를 **자동으로 생성**하거나,  
**builder 패턴을 지원**하는 기능을 제공합니다.

이 라이브러리를 사용하면 코드의 **간결함**과 **가독성**을 높일 수 있습니다. ✨

---

## 🏷️ Lombok 주요 어노테이션 정리

| 어노테이션 | 역할 | 예시 |
| :---: | :--- | :--- |
| `@Getter` | 모든 필드의 getter 메서드 자동 생성 | `user.getName()` 가능 |
| `@Setter` | 모든 필드의 setter 메서드 자동 생성 | `user.setAge(20)` 가능 |
| `@ToString` | `toString()` 메서드 자동 생성 | `User(name=홍길동, age=20)` |
| `@NoArgsConstructor` | 기본 생성자 자동 생성 | `new User()` 가능 |
| `@AllArgsConstructor` | 모든 필드를 인자로 받는 생성자 생성 | `new User("홍길동", 20)` |
| `@RequiredArgsConstructor` | `final` 또는 `@NonNull` 필드만 인자로 받는 생성자 생성 | 의존성 주입할 때 자주 사용 |
| `@EqualsAndHashCode` | `equals()`와 `hashCode()` 자동 생성 | 객체 비교에 사용 |
| `@Builder` | 빌더 패턴 자동 생성 | `User.builder().name("홍길동").age(20).build()` |

---

## ⚠️ Lombok 사용 시 주의사항

정말 **간단**하고 **강력한 어노테이션**이지만 **주의❗️**해야 할 점이 있습니다.

### 🔒 1. 모든 필드에 대해 getter와 setter가 생성됨
- **민감한 데이터**나 **불변 객체**에는 적합하지 않음.

### ⚖️ 2. equals와 hashCode의 자동 생성
- 성능 문제나 비교 기준에 주의가 필요함.

### 🧩 3. toString 메서드의 자동 생성
- **민감한 정보**가 포함될 수 있어 주의가 필요함.

### 🧱 4. 불변 객체에 적합하지 않음
- 불변 객체는 **@Value**를 사용해야 함.

### 🧬 5. 상속과 인터페이스
- 상속 구조에서 `equals`, `hashCode`, `toString`의 영향을 고려해야 함.

### 📦 6. 직렬화 문제
- 객체(Object)를 JSON, XML, 또는 바이트 형태로 변환하는 과정에서  
  필드 처리를 신경 써야 함.

---

✅ **정리하자면,**
Lombok은 코드의 양을 줄이고 유지보수를 쉽게 만들어주는 매우 유용한 도구지만,  
**데이터 보안**과 **객체의 특성(가변/불변)**을 반드시 고려하여 사용해야 합니다.

---

# 📘 dto 형식 public static VS record 비교하기

---

## ✳️ 먼저 dto 형식을 public static으로 하는 이유
```java
public class UserDto {
    public class Request {
        private String name;
    }
}
->
UserDto dto = new UserDto();
UserDto.Request request = dto.new Request();  // ❗ 외부 객체 필요
```

static을 사용하면
```java
public class UserDto {
    public static class Request {
        private String name;
    }
}
->
UserDto.Request request = new UserDto.Request();  // ✅ 외부 객체 필요 없음
```
---

## 🧩 **reacord란?**

Java 16부터 추가된 특별한 유형의 클래스로, **불변 데이터 클래스**이다.  
다음과 같이 인자에 필드를 넣어주는 것만으로,  
클래스 내부의 선언 없이 `private final` 필드를 선언할 수 있다.
```java
public record UserDto(Long userId, String nickname, String email, String profileImg) {}  
```

DTO에서는 `Getter`, `equals()`, `hashCode()`, `toString()` 메서드를 직접 작성해야 하지만  
`record`를 사용하면 Java가 이 모든 것들을 자동으로 생성해준다.

물론 DTO도 Lombok 같은 도구를 사용하면 보일러플레이트 코드를 줄일 수 있지만,  
Record만큼 간단하지 않다.

---

## 🧮 DTO와 Record는 언제 사용할까?

### ✅ DTO를 사용해야 할 때
1. 데이터 수정이 필요한 경우
2. 추가적인 동작이나 검증 로직이 필요한 경우
3. DTO 필드가 많을 경우

### 🧱 Record를 사용해야 할 때
1. 간결하고 불변성을 가진 데이터 전달 객체가 필요한 경우
2. 읽기 전용 데이터 전송이 필요한 경우
3. 코드를 간결화하고 싶은 경우  

---

# ⚙️ RestControllerAdvice의 장점

1. 모든 Controller의 예외를 한 곳에서 처리
2. 예외 타입에 따라 다른 HTTP 상태 코드 및 메시지 반환
3. 각 Controller마다 try-catch를 반복할 필요 없음

---

## 🚫 없을 경우
```java
@GetMapping("/test")
public ResponseEntity<?> test() {
    try {
        // ...
    } catch (TestException e) {
        return ResponseEntity.badRequest()
                .body(ApiResponse.onFailure(e.getCode(), e.getMessage(), null));
    } catch (Exception e) {
        return ResponseEntity.internalServerError()
                .body(ApiResponse.onFailure("INTERNAL_ERROR", e.getMessage(), null));
    }
}
```
이런식으로 전부 try-catch를 반복 해야됨

