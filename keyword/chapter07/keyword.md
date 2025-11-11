### RestControllerAdvice

**1-1. @Controller**

- 스프링 MVC의 기본 컨트롤러
- 주로 뷰(View, HTML)를 반환
- 반환값이 문자열이면 View 이름으로 해석

**1-2. @RestController**

- @Controller + @ResponseBody 합친것
- 반환값을 HTTP 응답 본문(Response Body)으로 바로 사용
- JSON 등 REST API 응답에 최적화

—> JSON 반환

**2-1. @ControllerAdvice**

- 여러 컨트롤러에서 발생하는 예외, 바인딩, 모델 전역 설정을 한 곳에서 처리
- View 기반 프로젝트에서 주로 사용

**2-2. @RestControllerAdvice**

@RestControllerAdvice는 스프링에서 전역 예외 처리 및 응답 처리를 담당하는 어노테이션이다.

- @ControllerAdvice + @ResponseBody 역할을 동시에 함
- 주로 REST API에서 예외를 JSON 형태로 반환할 때 사용함

즉, 컨트롤러에서 발생하는 예외를 한 곳에서 잡아서, 일관된 형태의 응답을 만들어주는 역할을 한다.

```java
@Controller          → View 반환
@RestController      → ResponseBody(JSON) 반환

@ControllerAdvice    → 여러 @Controller 예외 전역 처리, View 반환
@RestControllerAdvice → 여러 @RestController 예외 전역 처리, ResponseBody(JSON) 반환
```

**장점**

- **전역 예외 처리 가능**
    - 모든 컨트롤러에서 발생하는 예외를 한 곳에서 처리
    - 예외 처리 코드 중복 제거 → 유지보수 용이
- **응답 형식 통일**
    - 성공/실패, 에러 코드, 메시지 구조를 API 전반에서 일관되게 유지 가능
- **가독성 및 코드 간결**
    - 각 컨트롤러에서 try-catch를 반복 작성하지 않아도 됨
    - 공통 로직을 하나로 관리 가능
- **유지보수 편리**
    - 에러 로직, 로그 출력, 알림 처리 등을 한 곳에서 관리
    - 새로운 예외 추가 시 `RestControllerAdvice`만 수정하면 됨

**없을 경우 불편한 점!**

1. **컨트롤러마다 중복 코드 발생**
    - 각 API에서 try-catch를 반복 작성해야 함
    - 예외 처리 로직이 여러 곳에 분산 → 수정 시 실수 발생 가능
2. **응답 형식 불일치**
    - 컨트롤러마다 예외 반환 메시지가 다를 수 있음
    - API 사용자 입장에서 예측 어려움
3. **로그, 모니터링 관리 어려움**
    - 예외 발생 시 로깅/모니터링 코드도 각 컨트롤러에 흩어짐
    - 유지보수 및 운영 시 효율성 감소
### Lombok
Lombok은 자바에서 반복되는 보일러플레이트 코드(getter, setter, 생성자, toString, equals, hashCode 등)를 애노테이션으로 자동 생성해주는 라이브러리이다.

**주요 기능**

1. **Getter/Setter**

```java
@Getter
@Setter
public class User{
	private String name;
	private int age;
}
```

- 자동으로 getName(), setName() 등 생성.

→ 원래는 getName(){ retrun name} 이런 함수 다 작성해야하는데 이 애노테이션만 쓰면 안해도 된다!!!

1. **생성자**
- @NoArgsConstructor → 기본 생성자
- @AllArgsConstructor → 모든 필드를 파라미터로 받는 생성자
- @RequiredArgsConstructor → final이나 @NotNull이 붙은 필드만 파라미터로 받는 생성자

```java

@NoArgsConstructor
@AllArgsConstructor
public class User {
    private String name;
    private int age;
}
```

→ 이 애노테이션이 없었다면

```java
public User(String name, int age, String email) {
    this.name = name;
    this.age = age;
    this.email = email;
}
```

→ 이런 생성자를 직접 작성해줬어야함. → 필드가 많아질수록 코드가 길어짐

1. **@Data**

→ 이 애노테이션 하나만 써도 아래 기능들을 자동 생성함.

- getter/setter
- toString
- equals() / hashCode()
- RequiredArgsConstructor

```java
@Data
public class User {
    private String name;
    private int age;
}
```

1. **@Builder**
- 빌더 패턴 자동 생성

```java
import lombok.Builder;

@Builder
public class User {
    private String name;
    private int age;
}
-> @Builder를 썼기에 아래처럼 코드 작성이 가능한것임!
public class Main {
    public static void main(String[] args) {
        User user = User.builder()    // ← Lombok이 자동으로 만들어주는 builder() 호출
                        .name("Alice")
                        .age(25)
                        .build();  // ← User 객체 생성
    }
}
```

→ 사용안하면 이 코드를 내가 작성해야함

```java
public class User {
    private String name;
    private int age;

    // 빌더 객체를 만듦
    public static UserBuilder builder() {
        return new UserBuilder();
    }

    public static class UserBuilder {
        private String name;
        private int age;

        public UserBuilder name(String name) {
            this.name = name;
            return this;
        }

        public UserBuilder age(int age) {
            this.age = age;
            return this;
        }

        public User build() { //최종객체 반환
            User user = new User();
            user.name = this.name;
            user.age = this.age;
            return user;
        }
    }
}
```

1. **기타 유용한 애노테이션**
- @ToString → toString() 자동 생성
- @EqualsAndHashCode → equals(), hashCode() 자동 생성
- @Slf4j → 로거 필드 자동 생성

```java
@Slf4j
public class MyClass {
    public void run() {
        log.info("로그 출력");
    }
}

```

**Rombok 사용 시 장점**

1. 코드 간결화: 반복되는 getter/setter, 생성자, 빌더 코드 제거
2. 가독성 향상: 핵심 비즈니스 로직에 집중 가능

### dto 형식 public static VS record 비교하기

DTO는 애플리케이션의 다양한 부분 간에 데이터를 이동시키는데 사용되는 Java 객체이다. 애플리케이션의 계층 간에 데이터를 운반하는 컨테이너라고 생각하면 된다. 일반적으로 DTO에는 비즈니스 로직이나 복잡한 동작이 없다. 그저 데이터를 담고있는 역할만 한다.

일반적인 DTO는 다음과 같은 요소를 포함한다.

- 데이터를 담는 private 필드
- 데이터에 접근하고 수정하기 위한 Getter와 Setter 메서드
- 객체를 생성하는 생성자
- 객체를 비교하거나 출력할 때 유용한 toString(), hashCode(), equals() 메서드를 재정의(Override)

```java
public class Outer {
 public static class UserDto {
        private String name;
        private int age;

        public UserDto() {}  // 기본 생성자

        public UserDto(String name, int age) {  // 전체 필드 생성자
            this.name = name;
            this.age = age;
        }

        public String getName() { return name; }
        public void setName(String name) { this.name = name; }

        public int getAge() { return age; }
        public void setAge(int age) { this.age = age; }

        @Override
        public String toString() {
            return "UserDto{name=" + name + ", age=" + age + "}";
        }

        @Override
        public boolean equals(Object o) {
            // equals 구현
            return this == o || (o instanceof UserDto other && 
                    name.equals(other.name) && age == other.age);
        }

        @Override
        public int hashCode() {
            return Objects.hash(name, age);
        }
    }
}
```

- 보일러플레이트 코드가 많다. → getter, setter, 생성자, equals, hashCode, toString 모두 직접 작성해야한다. → Lombok 사용하면 되긴 하나 Record는 기본적으로 불변성을 가지며 반복적인 코드를 제거하는 또다른 방식을 제공한다.

**Record**

Record는 Java16에 정식 출시된 특별한 유형의 클래스이다. 불변 데이터를 간결하고 읽기 쉽게 담는 데 초점을 맞추고 있으며, 기존 클래스(DTO 등)에서 필요했던 반복적인 코드를 많이 줄여준다.

- 불변성 → 일반적으로 변경할 수 있는 DTO와는 다르게 한 번 Record를 만들면 그 데이터를 변경할 수 없다.
- 간결한 문법 → 필드만 선언하면 Java가 자동으로 생성자, Getter, equals(), hashCode(), toString() 메서드를 만들어준다.
- Setter 없음: Record는 불변 객체이기 때문에 Setter 메서드를 제공하지 않는다.

→ 방금 그 길었던 코드가 이렇게 변함

```java
public record UserDto(String name, int age) {}
```

**DTO VS Record**

- 불변성 → Record는 기본적으로 불변성을 가지며, 한 번 생성된 Record 인스턴스의 데이터는 변경할 수 없다. 반면, DTO는 일반적으로 가변성을 가지며, 객체가 생성된 후에도 필드를 변경할 수 있다. DTO를 불변으로 만들려면 Setter 메서드를 사용하지 않거나, final 필드를 사용하여 신중하게 설계해야 한다.

    ```java
    UserDTO userDTO = new UserDTO("haewon",23);
    userDTO.setAge(24); // DTO는 기본적으로 값 변경 허용
    
    UserRecord userRecord = new UserRecord("haewon",23);
    userRecord.name = "dd"; //컴파일 에러 
    ```

- 보일러플레이트 코드 → Record는 반복적인 코드를 크게 줄일 수 있다. DTO를 사용할 때는 Getter, Setter 등의 메서드를 직접 작성해야한다. 물론 Lombok을 사용하면 줄일 수 있지만 Record만큼 간단하지 않다.
- 데이터 표현 방식 → Record는 데이터를 간결하고 직관적으로 표현하는 방법을 제공한다. Record 선언에는 필드만 포함되므로 코드가 더 깔끔하고 읽기 쉽다.

```java
//Record: 간결하고 단순
public record UserRecord(String name, int age){}
//DTO:더 많은 코드 필요
public class UserDTO{
	private String name;
	private int age;
	
	//생성자, setter,~~등
}
```

- 커스터마이징 → DTO의 장점 중 하나는 커스터마이징이 용이하다는 점이다. DTO에서는 데이터 유효성 검사, 데이터 변환 메서드 또는 비즈니스 로직을 추가할 수 있다.(권장은 아님)

  하지만 Record는 커스터마이징이 제한적이다. Record는 가볍고 불변성을 유지하도록 설계되었기에 내부 상태를 수정하거나 복잡한 로직을 쉽게 추가할 수 없다. 만약 데이터 객체에 커스터마이징된 동작이나 로직이 필요하다면, DTO가 더 유연한 선택이다.


**DTO와 Record는 언제 사용해야할까?**

DTO

- 데이터 수정이 필요할 떄
- 추가적인 동작이나 검증 로직이 필요할 떄
- 이전 버전의 Java와 호환이 필요할 때

Record

- 간결하고 불변성을 가진 데이터 전달 객체가 필요할 때
- 읽기 전용 데이터 전송이 필요할 때
- Java 16 이상에서 사용할 때