- **SOLID**

  SOLID 원칙은 객체지향 프로그램에서 지키면 좋은 5가지 원칙을 의미한다

  해당 원칙을 지킨다면 새로운 요구사항이 나오더라도, 큰 문제 없이 이를 해결할 수 있다.

    1. S(SRP, 단일 책임 원칙)
        - 하나의 클래스가 하나의 책임만 가져야 한다
    2. O(OCP, 개방 폐쇄 원칙)
        - 확장에는 열려있고 수정에는 닫혀 있다
        - 이로 인해 기존의 코드를 고치지 않고도 새로운 기능을 추가하거나 변경사항을 반영할 수 있다
    3. L(LSP, 리스코프 치환 원칙)
        - 자식 타입은 언제나 부모 타입으로 교체할 수 있다
        - 다형성을 이용할 때 사용하는 개념이다
    4. I(ISP, 인터페이스 분리 원칙)
        - 인터페이스 역시 하나의 책임을 가지고 있어야 한다
        - 인터페이스는 클라이언트의 목적과 용도에 적합한 인터페이스만을 제공해야 한다
    5. D(DIP, 의존 역전 원칙)
        - 구체척인 것에 의존하는 것이 아니라 추상적인것(인터페이스)에 의존하자


- **DI**

  Dependency Injection 즉, 의존 관계 주입을 의미한다

  필요한 의존 관계를 직접 생성하는것이 아니라, 외부에서 주입을 받는것을 의미한다

  효과

    - 유지보수성 향상
        - 코드를 바꾸지 않고 주입하는 객체만 바꾸어서 다른 동작을 유도할 수 있다
    - 결합도를 낮출 수 있다
        - 실행 시점에 어떤 구현체를 사용할건지 선택이 된다

  의존관계

    - 정적 의존 관계
        - 코드에서 나타나는 클래스간 의존 관계
        - 컴파일 당시에 정해진다
        - 정확히 어떤 구현체를 사용하는지는 알 수 없다
    - 동적 의존 관계
        - 실행 시점에 나타나는 의존관계
        - 런타임에 정해진다

  → DI 를 하게 되면 정적인 의존 관계는 바꾸지 않으면서 동적인 의존관계는 외부에서 결정할 수 있도록 할 수 있다



- **IoC**

  스프링에서는 개발자가 스프링 프레임워크 안에서 코드를 작성하고 이를 실행하면 해당 코드의 흐름을 개발자가 아닌 스프링이 가져가게 된다

  → 이처럼 프로그램의 흐름을 개발자가 직접 하는것이 아닌 외부에서 나타나는 것을 제어의 역전(IOC) 라고 한다

  스프링에서 객체를 생성하고 DI 컨테이너를 관리하는 일을 개발자가 직접하기 보다 스프링이 알아서 한다

  → IOC의 예시

  ### IOC가 없는 경우

    ```java
    class Chef{
        private Tomato tomato;
    
        public Chef() {
            this.tomato = new Tomato();
        }
    
        public void cook(){
            tomato.prepare();
            System.out.println("토마토 요리 완성");
        }
    }
    
    class Tomato{
        public void prepare(){
            System.out.println("토마토를 자릅니다");
        }
    }
    
    public class IocExample {
        public static void main(String[] args) {
            Chef chef = new Chef();
            chef.cook();
        }
    }
    ```


### 스프링 IOC가 있는 경우

```java
import java.beans.BeanProperty;

interface Ingredient{
    void prepare();
}

class Chef{
    private Ingredient ingredient;

    public Chef(Ingredient ingredient) {
        this.ingredient = ingredient;
    }

    public void cook(){
        ingredient.prepare();
        System.out.println("요리 완성");
    }
}

class Tomato implements Ingredient{
    public void prepare(){
        System.out.println("토마토를 자릅니다");
    }
}

class Potato implements Ingredient{
    public void prepare(){
        System.out.println("감자를 자릅니다");
    }
}

@Configuration
class AppConfig{
    @Bean
    public Ingredient ingredient(){
        return new Potato();
    }

    @Bean
    public Chef chef(){
        return new Chef(ingredient());
    }
}

public class IocExample {
    public static void main(String[] args) {

        AnnotationConfigApplicationContext context =
                new AnnotationConfigApplicationContext(AppConfig.class);

        Chef chef = context.getBean(Chef.class);
        chef.cook();
    }
}

```

- **생성자 주입 vs 수정자, 필드 주입 차이**
    - 생성자 주입
        - 주로 롬복의 @RequiredArgsConstructor로 쓰인다
        - 객체가 생성되는 순간 딱 1번 이 생성자가 호출되기 때문에 불변이고 필수적인 의존관계에 사용이 된다
    - 수정자 주입
        - setter이라는 수정자 메소드를 활용한다
        - 생성자 주입과 다르게 변경이 될수 있는 의존성을 주입할 때 사용한다
    - 필드 주입
        - 가장 간단한 방식
        - 테스트 하기가 어렵다
            - 테스트에는 단위 테스트와 통합 테스트가 있다
                - 단위 테스트는 각 기능 별로 테스트를 하는것을 의미한다. 대부분 순수 자바 로직을 사용한다
                - 통합 테스트는 스프링, DB 등등이 같이 돌아가는 테스트를 의미한다
                - 필드 주입을 선택하게 되면 Mock 객체를 주입하기 어려워 단위 테스트에서 사용하기에 어렵다

  → 다양한 방식으로 의존성을 주입할 수 있다

  의존성을 주입할 때 대상 빈이 2개 이상일 경우 어떻게 해결할 수 있을까?

    1. @Autowired 필드명 매칭

       Autowired는 우선 타입 매칭을 시도한다. 이때 여러 빈이 조회가 된다면 필드 이름, 파라미터 이름으로 추가적인 매칭을 진행한다

    2. @Primary 사용

       Autowired를 할 때 @Primary 라고 적혀 있는 빈이 우선권을 가지는 방식이다

    3. @Qualifier 사용

       추가적으로 구분할 수 있는 조건을 붙여주는 방식이다


- **AOP**

  프로그램의 핵심 로직과 공통 관심사를 분리해서 관리하는 기법

  → 비즈니스 로직은 핵심 기능을 구현하고, 공통된 기능은 따로 분리하자

  Spring AOP

  스프링에서 관점 지향 프로그래밍을 지원하는기술이다

  관점지향 프로그래밍

    - 메소드나 객체의 기능을 핵심 관심사와 공통 관심사로 나누어 프로그래밍한다
    - 여러 클래스에서 반복해서 사용하는 코드가 있다면 해당 코드를 모듈화해 공통 관심사로 분리한다

  핵심개념

    - Aspect
        - 공통 기능을 모은 모듈
    - Join Point
        - 공통 기능을 끼워넣을 수 있는 지점
    - Advice
        - 실제 실행된 공통 기능 코드
        - @Before, @After, @Around
    - Pointcut
        - 어떤 메서드에 적용할지 지정하는 표현식
    - Weaving
        - Aspect를 실제 코드에 적용하는 과정
    - Target
        - Aspect가 적용될 대상

  Advice 종류

    - @Before
        - 메서드 실행 전에 실행
    - @After
        - 메서드 실행 후에 실행
    - @AfterReturning
        - 메서드가 정상적으로 리턴한 이후에 실행
    - @AfterThrowing
        - 예외가 발생했을 때 실행
    - @Around
        - 메서드 실행 전/후를 모두 제어한다

  사용 예시

    ```java
    package com.example.demo;
    
    import org.springframework.context.annotation.AnnotationConfigApplicationContext;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    
    interface Ingredient{
        void prepare();
    }
    
    class Chef{
        private Ingredient ingredient;
    
        public Chef(Ingredient ingredient) {
            this.ingredient = ingredient;
        }
    
        public void cook(){
            ingredient.prepare();
            System.out.println("요리 완성");
        }
    }
    
    class Tomato implements Ingredient{
        public void prepare(){
            System.out.println("토마토를 자릅니다");
        }
    }
    
    class Potato implements Ingredient{
        public void prepare(){
            System.out.println("감자를 자릅니다");
        }
    }
    
    ```

    ```java
    package com.example.demo;
    
    import org.aspectj.lang.ProceedingJoinPoint;
    import org.aspectj.lang.annotation.Around;
    import org.aspectj.lang.annotation.Aspect;
    import org.springframework.stereotype.Component;
    
    @Aspect
    @Component
    public class CookingAspect {
    
        @Around("execution(* com.example.demo.Chef.cook(..))")
        public Object logCookingProcess(ProceedingJoinPoint joinPoint) throws Throwable{
    
            System.out.println("요리 시작");
    
            long start = System.currentTimeMillis();
    
            Object result = joinPoint.proceed(); //메서드 실행
    
            long end = System.currentTimeMillis();
            System.out.println("요리 끝남");
            System.out.println("걸린 시간 : " + (end - start) + "ms");
    
            return result;
        }
    
    }
    
    ```

    ```java
    package com.example.demo;
    
    import org.springframework.boot.CommandLineRunner;
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    
    @SpringBootApplication
    public class DemoApplication implements CommandLineRunner {
    
        private final Chef chef;
    
        public DemoApplication(Chef chef) {
            this.chef = chef;
        }
    
        @Override
        public void run(String... args) {
            chef.cook();
        }
    
        public static void main(String[] args) {
    		SpringApplication.run(DemoApplication.class, args);
    	}
    
    }
    
    ```

  로그

  요리 시작

  감자를 자름니다

  요리완성

  요리 끝남

  걸린 시간 : 7ms

  효과

    - 중복 코드 제거/유지보수성 향상
        - 공통기능을 함께 관리한다
    - 관심사 분리
        - 핵심 기능과 공통기능을 나누어서 관리한다

- **서블릿**

  클라이언트의 요청을 처리하는 서버측 프로그램

  서블릿 동작 과정

    1. 클라이언트가 특정 url로 요청을 보낸다
    2. 톰캣과 같은 서블릿 컨테이너가 요청을 받는다
    3. 해당 url에 매핑이 된 서블릿 객체를 실행한다
    4. 서블릿이 HttpServletRequest 객체로 요청 정보를 받는다
    5. HttpServletResponse 객체에 결과를 담아 클라이언트에게 준다
    6. 브라우저가 응답을 화면에 표시한다

  서블릿 생명 주기

    1. 서블릿 로드 & 인스턴스 생성
        - 클라이언트 요청이 처음 들어왔을때 톰캣이 서블릿 클래스를 메모리에 로딩
    2. init() 호출 (초기화, 1회 실행)
        - 생성된 서블릿 인스턴스에서 최초 1회만 실행한다
        - 애플리케이션이 시작될때 필요한 작업을 수행한다
        - 해당 요청이 다시 와도 init()은 불려지지 않는다
    3. service()
        - 클라이언트에서 요청이 들어올때마다 실행된다
        - 스레드 단위로 요청을 처리한다
        - HTTP 메서드에 따라 다른 메소드가 실행된다
            - GET → doGet()
            - POST → doPost()
            - PUT → doPut()
            - DELETE → doDelete()
            - 지원하지 않는 메소드 일 경우 → 405
    4. destroy()
        - 컨테이너가 서버가 종료될때 호출한다
        - 주로 톰캣/애플리케이션이 종료될 때 불린다
        - 리소스 정리, 연결 종료 등 마무리 작업 수행

  HttpServletRequest

    - 서블릿은 HTTP 요청 메세지를 쉽게 사용할 수 있도록 이를 파싱해주고 그  결과를 HttpServletRequest를 통해 사용할 수 있다
    - HTTP 메시지에 나온 메서드, url, uri, 프로토콜, 헤더 정보 등등을 쉽게 가져올 수 있다

  클라이언트로 부터 데이터 받기

    - 쿼리 파라미터
        - url의 쿼리 파라미터를 활용하는 방식
        - `request.getParameter("username")` 과 같이 getParameter() 메소드로 조회할 수 있다
        - 만일 이름이 같은 파라미터가 여러개라면 getParameterValues()를 활용해 조회할 수 있다
    - HTML FORM
        - form을 사용하는 경우 메시지 바디에 쿼리 파라미터와 동일한 형식으로 데이터가 전달된다
        - 서버는 위에서 설명한 쿼리 파라미터와 동일한 형식으로 데이터를 사용하면된다
    - 메시지 바디
        - json
            - json 형식을 파싱하기 위해서 개발자가 직접 객체를 생성할 수 있다
            - 또는 Jackson 라이브러리를 활용할 수 있다

  HttpServletResponse

    - HTTP 응답 메시지를 생성할 때 사용한다
    - 이를 통해 content, 쿠키, 리다이렉팅 등등 여러 정보를 메시지에 쉽게 생성할 수 있다

스프링 mvc의 DispatcherServlet

- 생명주기
    1. init() 으로 스프링 컨텍스트 초기화, 빈 로딩
    2. service() 모든 요청을 받는다
    3. HTTP 메서드 분기
    4. 핸들러 매핑으로 사용할 컨트롤러 메소드 결정
    5. 핸들러 어댑터가 컨트롤러 호출
    6. 컨트롤러 메소드 실행
    7. 뷰 리졸버가 리턴값 해석
    8. 응답을 렌더링 한 후 반환
    9. destroy() 컨테이너 종료시 사용 , 빈 소멸

- 자바의 Optional 클래스

  이는 자바8부터 나온 기능으로 null을 안전하게 다루기 위해(NPE를 줄이기 위해) 등장했다

  Optional은 값이 존재할수도 하지 않을수도 있다는것을 표현한다

  기본 사용법

    ```java
    //값 생성하기
    Optional<String> op1 = Optional.of("hello");
    Optional<String> op2 = Optional.ofNullable("123") // null이 될 가능성이 있으면 사용
    
    //값 꺼내기 
    System.out.println(op1.get()); //값이 없으면 NoSuchElementException
    System.out.println(op1.orElse("default")); // 값이 없으면 "default"
    System.out.println(op1.orElseThrow(() -> new RuntimeException("없음!")));
    
    //자주 사용하는 메소드
    op1.isPresent() //값이 있으면 true
    op1.isEmpty()  // 값이 없으면 true
    ```

  옵셔널 orElse(), orElseGet(), orElseThrow()의 차이

    - orElse
        - 파라미터로 **값**을 받는다
        - 해당 인자는 항상 실행(평가)이 된다

      → Optional에 값이 있어도 orElse 표현식은 늘 실행이 된다

    - orElseGet
        - 파라미터로 **함수형 인터페이스(함수)**를 받는다
        - 지연 평가가 된다

      → Optional에 값이 없을 때만 실행이 된다

    - orElseThrow
        - 예외를 생성한다
        - 값이 없을 때만 실행된다
        - 값이 없는 상황을 예외 상황으로 처리해야 할때 사용한다

  사용 예시

    ```java
    // 잘못된 사용
    userRepository.findByUserEmail(userEmail)
        .orElse(createUserWithEmail(userEmail)); 
    
    // 이 경우 유저를 찾든 못찾든 createUserWithEmail(userEmail)은 무조건 실행이 된다
    -> 이로 인해 만약 unique 제약 조건이 있는 경우 Unique constraint violation 에러가 나타날수 있다
    
    //해결책
    userRepository.findByUserEmail(userEmail)
        .orElseGet(() -> createUserWithEmail(userEmail));
    
    // 해당 코드는 유저를 찾지 못하는 경우에만 createUserWithEmail(userEmail)이 실행되기 때문에 안전하다
    ```

  옵셔널 함수들

    - map
        - 옵셔널 안에 값이 존재하면 그 값을 다른 형식으로 바꿔서 돌려준다, 만약 값이 없다면 빈 옵셔널 반환

        ```java
        Optional<String> name = Optional.of("blue");
        
        Optional<Integer> len = name.map(s-> s.length());
        System.out.println(len.get()); //6
        ```

    - flatMap
        - map과 비슷하다 다만 map은 일반 값을 반환하고 flatMap은 Optional을 반환한다
            - map에서 함수가 일반 값을 반환할때 사용한다 이후에 그 값을 다시 옵셔널로 감싸준다
            - flatMap은 함수가 반환할때 이미 옵셔널을 반환할 때 사용한다
        - 옵셔널이 중첩되지 않게 평탄화를 해준다
          - 

            ```java
            Optional<String> color = Optional.of("black");
            
            //map
            ```

    - filter
        - 값이 있다면 해당 값이 조건을 만족하는지 검사한다
        - 값이 없다면 빈 옵셔널을 반환한다

- Stream API

  컬렉션이나 배열 같은 데이터를 선언적 방식으로 다룰 수 있게 해준다

    - 이를 통해 데이터를 순차적으로 처리할 수 있고 원본데이터는 유지한채 새로운 결과를 만들어낼 수 있다

  자주 사용하는 연산

    - filter() : 데이터를 필터링한다
    - map() : 데이터의 형식을 변환
    - sorted() : 정렬
    - distinct() : 중복 제거
    - limit() : 앞에서 n개만 가져옴
    - forEach() : 요소들을 반복처리한다
    - collect() : 특정 자료형으로 모은다
    - count() : 개수 세기

    ```jsx
    public class StreamExample {
        public static void main(String[] args) {
            List<String> colors = Arrays.asList("red", "blue", "white", "blac");
    
            
            List<String> result = colors.stream()
                                       .map(String::toUpperCase)
                                       .collect(Collectors.toList());
                                       
    
            System.out.println(result); // [RED, BLUE, WHITE, BLACK]
        
        
    		    List<Integer> nums = List.of(1, 2, 3, 4, 5);
    
            nums.stream()
                    .filter(n -> n % 2 == 0)
                    .forEach(System.out::println); 
            // 출력: 2
            //      4
    
            nums.stream()
                    .sorted()
                    .forEach(System.out::println); 
            // 출력: 1 2 3 4 5 (오름차순)
    
            nums.stream()
                    .sorted((a, b) -> b.compareTo(a))
                    .forEach(System.out::println); 
            // 출력: 5 4 3 2 1 (내림차순)
        
        }
    }
    
    ```

  지연 연산

  스트림 연산은 실제로 사용할때 까지 실행되지 않는다

  스트림 연산의 종류

    1. 중간 연산
        1. map,filter, distinct 과 같이 연산
        2. 결과를 바로 만들지 않고 계획만 세운다
    2. 최종 연산
        1. forEach, collect, count 과 같은 연산
        2. 이때 실제 연산이 실행된다

  → 즉, 중간 연산을 모으고 최종 연산이 실행될때 순차적으로 처리가 된다

  지연 연산 예제

    ```java
    List<Integer> num = List.of(1,2,3);
    
    num.stream()
    			.filter(n -> {
    								System.out.println(n);
    								return n %2 == 0;
    								});
    								
    // 출력 결과 존재하지 않음
    
    num.stream()
    			.filter(n -> {
    								System.out.println(n);
    								return n %2 == 0;
    								})
    			.forEach(System.out::println);
    			
    // 출력 결과 : 1
    							2
    							2
    							3
    ```

  지연 연산의 장점

    - 불필요한 연산을 줄인다
        - 예를 들어 distinct로 걸러지면 map과 같은 다음 연산이 실행되지 않는다
    - 성능 최적화
    - 무한 스트림 처리 가능