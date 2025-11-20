## Java Exception

- 애플리케이션 실행 중에 발생 수 있는 개발자가 처리할 수 있는 문제를 의미한다.
- Exception을 상속받는 Runtime Exception은 **`Unchecked Exception`**이다.
    - Unchecked Exception: 런타임 시점에 발생하는 예외로 컴파일러가 예외 처리를 강제하지 않는다.
- Runtime Exception을 제외한 예외는 **`Checked Exception`**이다.
    - Check Exception: 컴파일 시점에 반드시 처리해야 하는 예외. 개발자가 반드시 try-catch문으로 처리해야한다.

### Checked Exception

- **Exception**
    - 모든 Checked Exception의 부모 클래스
- **IOException**
    - 읽기/쓰기 하는 도중 오류가 발생했을 때 발생하는 예외 클래스
    - 네트워크를 통해서 다른 컴퓨터와 데이터 교환 중 오류가 발생했을 때 발생하는 예외 클래스
- **FileNotFoundException**
    - 파일을 찾을 수 없을 때 발생하는 예외 클래스
- **SQLException**
    - 데이터베이스 액세스 작업 중 오류가 발생했을 때 발생하는 예외 클래스

### Unchecked Exception

- **RuntimeException**
    - 모든 Unchecked Exception의 부모 클래스
- **NullPointerException**
    - 참조 변수의 값이 NULL인 상태에서 필드나 메소드를 사용할 때 발생하는 예외 클래스
- **ClassCaseException**
    - 클래스의 형변환이 가능하지 않을 때 발생하는 예외 클래스
- **ArithmeticException**
    - 나눗셈에서 어떤 값을 0으로 나눌 때 발생하는 예외 클래스
- **IndexOutOfBoundsException**
    - 배열, 리스트, 문자열에서 인덱스 범위를 벗어난 위치를 조회했을 때 발생하는 예외 클래스
- **NumberFormatException**
    - Integer.parseInt(s), Double.parseDouble(s) 등을 실행할 때 발생하는 예외 클래스
    - IllegalArgumentException을 상속 받는다.
- **IllegalArgumentException**
    - 매개 변수에 전달된 인자가 유효하지 않을 때 발생하는 예외 클래스

## @Valid

- Rest Controller를 이용하여 @RequestBody객체를 사용자로부터 들어오는 값을 검증할 수 있는 어노테이션이다.
- 이 검증의 세부 사항은 객체 내부에 정의할 수 있다.

```java
// MemberReq DTO
public class MemberReq {
		@NotNull // 세부 사항 객체 내부 정의
		private String name;
		
		@Positive
		private int age;
}
```

```java
// Member Controller
@RestController
public class MemberController {
		@PostMapping("/api/members")
		public Member save (
				@Valid @RequestBody MemberReq memberReq // @Valid 설정 
		) {
				...
		}
}
```

- `message = “”`  속성으로 해당 필드에 유효하지 않은 값이 오는 경우 알릴 메시지를 지정할 수 있다.
- 유효성 검증에 실패할 경우 `MethodArgumentNotValidException`이 발생한다.

### @Valid 적용 방법

- 검증할 대상의 앞에 @RequestBody와 함께 @Valid 어노테이션을 붙여주면 해당 input에 대해서 검증을 진행한다.
- 단일 파라미터를 검증 하고 싶은 경우 @Valid가 아닌 유효성 검사 어노테이션을 바로 붙여서 검증할 수 있다.
- 검증할 객체 안에 중첩된 dto가 존재한다면 @Valid를 붙여 중첩으로 검증할 수 있다.

### 문자열 검증

- @NotBlank: null이 아닌 값
    - 공백이 아닌 문자를 하나 이상 포함해야 한다.
    - 반드시 값이 존재하고 공백 문자를 제외한 길이가 0보다 커야 한다.
- @NotEmpty
    - null 이거나 empty가 아니어야 한다.
    - 반드시 값이 있어야 하며 길이가 0보다 커야 한다.
- @NotNull
    - null이 아닌 값은 어떤 타입이든 상관 없다.
    - 반드시 값이 있어야 한다.
- @Null
    - 타입은 상관 없으며 null 값을 허용한다.

### 최대 / 최소 검증

- @DecimalMax
    - 지정된 최댓값보다 작거나 같아야 한다.
    - `@DecimalMax(value = “1000”)`
- @DecimalMin
    - 지정된 최솟값보다 크거나 같아야 한다.
    - `@DecimalMin(value = “0”)`
- @Max
    - 지정된 최댓값보다 작거나 같아야 한다.
    - `@Max(value = 1000)`
- @Min
    - 지정된 최솟값보다 크거나 같아야 한다.
    - `@Min(value = 0)`

### 범위 검증

- @Positive
    - 양수인 값이어야 한다.
- @PositiveOrZero
    - 0이거나 양수인 값이어야 한다.
- @Negative
    - 음수인 값이어야 한다.
- @NegativeOrZero
    - 0이거나 음수인 값이어야 한다.

### 시간 값 검증

- @Future
    - Now 보다 미래의 날짜, 시간이어야 한다.
- @Past
    - Now 보다 과거의 날짜, 시간이어야 한다.

### Boolean 검증

- @AssertTrue
    - 항상 True 이어야 한다.
- @AssertFalse
    - 항상 False 이어야 한다.

### 크기 검증

- @Size
    - 값의 크기가 max 보다 작거나 같아야 한다.
    - 값의 크기가 min 보다 크거나 같아야 한다.
    - `@Size(max=10, min=1)`