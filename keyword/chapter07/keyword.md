- RestContollerAdvice
## @ControllerAdvice란?

- `@ExceptionHandler`, `@ModelAttribute`, `@InitBinder` 가 적용된 메소드들에 AOP를 적용하여 Controller에 적용하기 위해 사용되는 어노테이션이다. (보통은 ExceptionHandler목적으로 많이 사용한다고 한다.)
- `@ControllerAdvice`는 클래스에 선언되며, 모든 컨트롤러에 걸쳐 공통으로 부가 기능을 설정 할 수 있게 해준다.
- `@RestControllerAdvice` 는 이런 기능에 `@ResponseBody` 가 추가된 어노테이션으로 응답을 Json형태로 하고자 할 때 사용한다.

### @ExceptionHandler

- 컨트롤러에서 전역적으로 발생하는 예외를 한번에 처리 가능하게 해준다.

```java
@RestControllerAdvice
public class GeneralExceptionAdvice {
	
	@ExceptionHandler(IllegalArgumentException.class) // IllegalArugumentException이 발생하면 이 핸들러가 실행되는거임.
	public ResponseEntity<String> handleException(
		IllegalArgumentException ex
	) {
		return ResponseEntity.status(HttpStatus.BAD_REQUEST)
													.body(new ErrorResponse(ex.getMessage()));
	}
```

### @ModelAttribute

- 모든 컨트롤러의 Model에 공통으로 담아줄 데이터를 설정할 수 있다.

```java
@ControllerAdvice
public class GeneralModelAdvice {
	
	@ModelAttribute("username") // 모든 컨트롤러 모델에 username이 자동으로 들어가는거
	public String addUserName() {
			return userService.getLoginUser();
	}
}
```

- json형태를 반환하는 REST방식에서는 컨트롤러가 View를 랜더링 하지 않기 때문에 @RestControllerAdvice 에선 이 행위가 의미가 없다.

### @InitBinder

- 요청 데이터를 객체로 바인딩할 때 사용하는 변환 규칙을 커스터마이징할 수 있다.

```java
@RestControllerAdvice
public class GeneralInitAdvice {
	
	@InitBinder
	public void initBinder(WebDataBinder binder) {
			binder.registerCustomEditor(LocalDate.class, new PropertyEditorSupport() {
				@Override
				public void setAsText(String text) {
						setValue(LocalDate.parse(text));
				}
			});
}
```

- LocalDate 타입의 파라미터를 받을 경우에, 이 바인더가 호출돼서 사용자가 정의한대로 문자열을 바인딩 해준다.

- lombok
## @Lombok 이란?

- 어노테이션을 기반으로 코드를 자동 완성 해주는 라이브러리이다.
- 불필요한 코드 작성을 대신 해주기 때문에 생산성이 향상된다.
- 반복되는 코드를 숨길 수 있기 때문에 가독성 및 유지 보수성이 향상된다.

## Lombok의 다양한 주요 기능들

- **@Getter, @Setter**
    - 롬복에서 가장 많이 사용되는 어노테이션이다.
    - 클래스 이름 위에 적용시키면 모든 변수들에 적용되며, 변수 위에 어노테이션을 적용하면 해당 변수에만 적용된다.
    - 해당되는 변수들에 대해서 Getter와 Setter 메서드를 lower camelCase로 생성한다.
    
    ```java
    @Getter @Setter
    public class Member {
    	private String name;
    	private int age;
    }
    ```
    
- **@AllArgsConstructor**
    - 해당 클래스 내에 선언된 모든 변수를 사용하는 생성자를 자동완성 시켜주는 어노테이션이다.
    
    ```java
    @AllArgsConstructor
    public class Member {
    	private String name;
    	private int age; // name과 age를 매개변수로 갖는 생성자를 롬복이 만들어줌.
    }
    ```
    
- **@NoArgsConstructor**
    - 어떤 변수도 매개변수로 갖지 않는 기본 생성자를 자동완성 시켜주는 어노테이션이다.
    
    ```java
    @NoArgsConstructor
    public class Member {
    	private String name;
    	private int age; 
    	
    // public Member() {} 기본생성자를 롬복이 자동 생성해줌.
    }
    ```
    
- **@RequiredArgsConstructor**
    - 특정 변수만 매개변수로 갖는 생성자를 자동완성 시켜주는 어노테이션이다.
    - final로 선언한 변수과 `@NonNull`어노테이션을 붙인 변수를 생성자의 인자로 추가한다.
    
    ```java
    @RequiredArgsConstructor
    public class Member {
    	private final String name;
    	private int age; 
    	
    // public Member(String name) {
    //    this.name = name;
    // } -> final 변수만 생성자 매개변수로 가짐.
    }
    ```
    
- **@EqualsAndHashCode**
    - 클래스 위에 선언하면 equals 함수와 hashCode함수를 자동으로 생성해준다.
    - `hashCode()`: 두 객체의 내부의 값이 같은지 숫자로 변환해서 확인.
    - `equals()`: 같은 객체인지 확인.
    
    ```java
    @EqualsAndHashCode(of = {"name", "birth"}) // name과 birth가 같으면 같은 객체로 인식해라. 만약 안적으면 모든 필드가 같아야 같은 객체
    public class Member {
    	private final String name;
    	private int age; 
    	private String birth;
    }
    ```
    
- **@ToString**
    - 클래스의 변수들을 기반으로 ToString메소드를 자동으로 완성시켜 준다.
    - `@ToString.Exclude`를 변수위에 설정하면 ToString에 포함되지 않게할 수 있다.
    
    ```java
    @ToString
    public class Member {
    	private final String name;
    	private int age; 
    	private String birth;
    } 
    
    // Member(name=서연, age=22, birth=040103)
    ```
    
- **@Data**
    - 앞에서 설명한 @ToString, @EqualsAndHashCode, @Getter, @Setter, @RequiredArgsConstructor 를 모두 적용해 자동완성 해주는 어노테이션이다.
    - 모두 사용하면 무겁고 객체의 안정성을 지키기 힘들기 때문에 거의 사용하지 않는다.
- **@Builder**
    - 해당 클래스의 객체 생성에 Builder패턴을 적용시켜주는 어노테이션이다.
    - 클래스 위에 선언하면 모든 필드에 대해 적용가능하고, 특정 필드만 적용하고 싶으면 생성자를 정의한 후 `@Builder`를 붙어주면 된다.
    
    ```java
    @Builder
    public class Member {
    	private final String name;
    	private int age; 
    	private String birth;
    } 
    
    Member.builder()
    	.name("서연")
    	.age(22)
    	.birth("040103")
    	.build(); 이렇게 사용할 수 있음!!
    ```
    
- **@Singular**
    - @Builder와 보통 같이 쓰이며 컬렉션 필드를 추가할 때 builder 패턴을 적용하여 유연하게 추가할 수 있게 해주는 어노테이션이다.
    
    ```java
    private List<String> items;
    
    Member.builder()
    	.items("노트북")
    	.items("키보드")
    	.build(); 리스트를 통째로 추가하지 않고 하나씩 유연하게 다룰 수 있다.
    ```
    
- **@Delegate**
    - 다른 클래스의 매서드를 자신의 클래스에서 직접 사용할 수 있게 해주는 어노테이션이다.
    
    ```java
    public class Member {
    	private final String name;
    	private int age; 
    	private String birth;
    	
    	public String myName() {
    		return name;
    	}
    } 
    
    public class B {
    	@Delegate
    	private Member member = new Member("서연");
    }
    
    -> 이렇게 하면 B에서도 Member의 myName함수를 호출할 수 있다.!
    public String myName() {
    	return this.member.myName();
    } 함수가 컴파일 후에 생성됨.
    ```
    
- **@UtilityClass**
    - 해당 클래스를 final로 설정하고 모든 매서드를 정적으로 만들어주며 인스턴스화 방지를 위한 private생성자를 생성해주는 어노테이션이다.
    
    ```java
    @UilityClass
    public (final) class StringUtils { // fianl 클래스로 바꿈
    	public (static) <List<String> splitString(String input) { // static 매서드로 바꿈
    			if(input == null || input.isEmpty()) {
    					return Collections.emptyList();
    			}
    			return Arrays.asList(input.split(","));
    	}
    	
    	// private StingUilts() {} // private 기본 생성자 생성
    } 
    ```
    
- **@Synchronized**
    - 해당 메서드에 락을 걸어 하나의 스레드만 접근 가능하게 해주는 어노테이션이다.
    
    ```java
    public class Counter {
    		
    		private int count = 0;
    		
    		@Synchronized
    		public void increment() { // 이 함수에 lock을 걸어주는 거임
    			count++;
    		}
    }
    ```
    
- **@Log4j2**
    - 해당 클래스의 로그 클래스를 자동 완성 시켜주는 어노테이션이다.
    ```java
    @RestController
    @Log4j2
    public class MemberController {

            @GetMapping("/log")
            public ResponseEntity log() {
                    log.error("Error"); // log 사용 가능!
                    return ResponseEntity.ok().build();
            }
    }
    ```
- DTO public static VS record 비교하기

## public static DTO

- 하나의 클래스 안에 여러개의 static class를 정의해 여러개의 DTO를 체계적으로 관리할 수 있는 방식이다.
- static class를 선언하면 더 적은 메모리를 사용하며, 바깥 클래스에 대한 참조가 필요 없기 때문에 가비지 컬렉션의 대상이 된다.

```java
public class MemberDTO {
		
		public static class MemberSimpleDTO {
				public Long id,
				public String name
		}
		
		public static class MemberDetailDTO {
				public Long id,
				public String name,
				public int age,
				public String birth
		}
}
```

## record DTO

- Record 클래스는 값의 집합으로 이루어진 간단한 객체를 생성할 때 사용한다.
- final 클래스 이므로 다른 클래스를 상속하거나 상속시킬 수 없다.
- 생성자, getter, equals, toString 등 매서드를 자동생성 해준다.

```java
public record Member(Long id, String name, int age) {
		// getter 자동생성
}
```
