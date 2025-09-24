### 1. SOLID

SOLID 원칙 : 객체 지향 설계에서 지켜줘야 할 5개의 소프트웨어 개발 원칙

→ 높은 응집도와 낮은 결합도

1. **SRP (Single Responsibility Principle)**: 단일 책임 원칙
    - 객체는 단 하나의 책임만 가져야 한다.
    - 클래스는 단 하나의 책임을 가져야 하며, 클래스를 변경하는 이유는 단 하나의 이유이어야 한다.

2. **OCP (Open-Closed Principle)**: 개방-폐쇄 원칙
    - 기존의 코드를 변경하지 않으면서 기능을 추가할 수 있도록 설계가 되어야 한다.
    - 확장에는 열려 있어야 하고, 변경에는 닫혀 있어야 한다.

3. **LSP (Liskov Substitution Principle)**: 리스코프 치환 원칙
    - 자식 클래스는 최소한 자신의 부모 클래스에서 가능한 행위는 수행할 수 있어야 한다.
    - 상위 타입의 객체를 하위 타입의 객체로 치환해도 상위 타입을 사용하는 프로그램은 정상적으로 동작해야 한다.

4. **ISP (Interface Segregation Principle)**: 인터페이스 분리 원칙
    - 인터페이스는 그 인터페이스를 사용하는 클라이언트를 기준으로 분리해야 한다.

5. **DIP (Dependency Inversion Principle)**: 의존 역전 원칙
    - 의존 관계를 맺을 때 변화하기 쉬운 것 또는 자주 변화하는 것보다는 변화하기 어려운 것, 거의 변화가 없는 것에 의존해야 한다.
    - 고수준 모듈은 저수준 모듈의 구현에 의존해서는 안된다.


: SRP와 ISP는 객체가 단일 책임을 가지게 하고 클라이언트마다 특화된 인터페이스를 구현하게 해서 객체가 커지는 것을 막는다

: LSP와 DIP는 OCP를 돕는다. OCP는 자주 변화되는 부분을 추상화하고 다형성을 이용해서 기능 확장에는 유연하지만 코드 변화에는 보수적이게 만들어준다.

: 변화되는 부분을 추상화할 수 있게 돕는 것이 DIP, 다형성 구현을 돕는 것이 LSP


### 2. DI

DI(Dependency Injection, 의존성 주입) : 객체 지향 프로그래밍에서 객체 간의 의존성을 관리하고 결합도를 낮추기 위한 디자인 패턴, 쉽게 이야기하면 스프링 컨테이너가 객체의 의존관계를 외부에서 주입

→ 결합도를 낮추고 테스트 용이성과 유지보수성을 높임

의존성 : 하나의 객체가 다른 객체에 의존하는 관계

- 이러한 의존관계를 관리하는 객체가 스프링빈(스프링 IOC 컨테이너가 관리하는 객체)
- 스프링빈은 생성하는 방법은 컴포넌트(service, controller 등) 스캔 후 자동등록하는 것과 자바 설정파일에서 수동 등록하는 것이 있다.
- @Autowired : 의존성 주입을 간편하게 해주는 기능. 이를 통해 스프링 프레임워크가 자동으로 필요한 의존성 주입

의존성 주입의 종류

1. 생성자 주입 : 클래스의 생성자를 통해 의존성 주입

```
@Service
public class MyService {
    private final MyRepository myRepository;
    
    @Autowired
    public MyService(MyRepository myRepository) {
        this.myRepository = myRepository;
    }
}
```

2. 필드 주입 : 클래스의 멤버 변수에 직접 주입하는 방법

```
@Service
public class MyService {
    @Autowired
    private MyRepository myRepository;
}
```

3. 세터 주입 : 세터 메서드를 통해  의존성을 주입한는 방법
```
@Service
public class MyService {
    private final MyRepository myRepository;
    
    @Autowired
    public MyService(MyRepository myRepository) {
        this.myRepository = myRepository;
    }
}
```

### 3. IOC

IOC(Inversion of Control) : 프레임워크가 객체의 생성, 관리, 제어 흐름을 담당하도록 변경하는 개념, 제어의 역전. 스프링에서는 이를 지원하기 위해 ApplicationContext 컨테이너 제공한다.

→ 스프링에서 IOC를 구현하는 방법 중 하나가 DI이다


[IOC 컨테이너]

: IOC의 핵심적인 도구로 객체의 생성과 의존성 관리 담당

- 애플리케이션에서 피룡한 객체를 빈으로 관리하고 의존성을 자동으로 주입해주는 역할
- 개발자가 객체 간의 의존성을 직접 관리하지 않고 IOC컨테이너가 이를 자동으로 처리해준다.

[IOC 과정]

1. 객체의 생성 및 관리
    - ApplicationContext를 사용하여 빈을 생성하고 관리한다.
    - 빈은 일반적으로 Spring이 제어하며 개발자는 객체의 생성과 관리를 직접 처리하지 않는다
2. 의존성 관리
    - 객체 간의 의존성을 Spring이 주입한다. → DI
    - 객체가 필요로 하는 다른 객체에 대해 Spring 컨테이너가 의존성을 주입해준다.
3. 제어 흐름의 역전
    - 개발자가 코드의 제어 흐름을 결정하지 않고, 프레임워크가 객체의 라이프사이클 및 실행 흐름을 관리한다.

[IOC 필요성]

- 객체 간 결합도를 낮출 수 있다
- IOC는 객체의 생성 및 관리 책임을 외부로 넘겨 재사용성, 유연성, 테스트 가능성을 높인다

  [IOC 장점]

- 결합도 감소 : 코드 변경에 유연하게 대응, 유지보수성 향상 및 시스템의 확장성 높아짐
- 유연성 증가
- 재사용성
- 테스트용이성 : 단위 테스트 쉽게 만들어줌
- 확장성

[IOC 단점]

- 복잡성 증가
- 초기 학습 필요
- 디버깅 어려움

### 4. 생성자 주입 vs 수정자, 필드 주입 차이

1. 생성자 주입 : 객체 생성 시점에 필요한 의존성을 생성자를 통해 주입하는 방식
    - 생성자를 통해 주입된 의존성을 final로 선언할 수 있어 불변성을 보장할 수 있다
    - 생성자를 통해 필요한 의존성을 명시적으로 선언하기 때문에 클래스가 어떤 의존성을 필요로 하는지 명확히 알 수 있다
    - 테스트 시 new 생성 가능해서 단위 테스트에 용이하다

```
@Component
public class OrderService {
    private final OrderRepository orderRepository;

    // 생성자가 하나면 @Autowired 생략 가능
    public OrderService(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }
}
```

2. 세터(수정자) 주입 : 기본 생성자로 객체 생성 후 setter 메서드를 통해 의존성 주입하는 방식
    - 변경 가능성이 있는 의존 관계에 사용한다
    - 생성자 호출 이후에 필드 변수에 변경이 일어나기 때문에 final 붙일 수 없다
    - 생성 이후에도 의존성을 설정하거나 변경할 수 있어 유연하다

```
@Component
public class PaymentService {
    private PaymentGateway gateway;

    @Autowired
    public void setGateway(PaymentGateway gateway) {
        this.gateway = gateway;
    }
}
```

3. 필드 주입 : 멤버 변수에 직접 @Autowired를 붙여 주입받는 방식
    - 코드 간결하지만 테스트 불가능
    - 프레임워크 의존도 가장 높음
    - 간편하게 의존 관계 주입이 가능하지만 참조관계를 눈으로 확인하기 어렵다

```
@Component
public class NotificationService {
    @Autowired
    private MessageSender sender;
}
```

#### 3가지 방식 중에서 생성자 주입 방식이 권장되는 이유!

1. 순환 참조 방지
    - 애플리케이션 구동 시점(객체의 생성 지점)에 순환 참조 에러 예방 가능
    - 세터와 필드 주입에서 실행 결과의 차이가 나는 이유는 Bean을 주입하는 순서가 다르기 때문이다
        - @Autowired를 이용한 필드주입, 세터 주입을 했을 때 애플리케이션 구동 시점에 에러가 발생하지 않은 이유는 빈의 생성과 @Autowired 시점이 분리되어 있기 떄문이다. 그렇기 때문에 @Autowired는 모든 Bean 생성이 완료된 후에 의존관계 주입이 처리돼 호출이 되고 나서야 순환 이슈 즉 순환 참조를 확인할 수 있다.
        - 생성자 주입은 객체의 생성과 @Autowired가 동시에 실행되서 오류를 잡을 수 있다
2. 테스트 용이
    - 단순 POJO를 이용한 테스트 코드 작성 가능
3. 스프링에 독립적인 코드 작성
4. 객체 불변성 확보

순환참조 : 두 개 이상의 객체가 서로를 직접 또는 간접적으로 참조하여 의존성 사이클이 형성되는 현상으로 무한 루프를 유발하거나 메모리 누수를 발생시킴


### 5. AOP

AOP : 관점지향 프로그래밍

(애플리케이션의 핵심적인 기능에서 부가적인 기능을 분리해서 Aspect라는 모듈을 만들어 설계하고 개발한느 방법)

→ 프로그램을 핵심기능과 공통기능으로 나누어 관리하는 프로그래밍 패러다임

→ 공통 관심사를 분리하여 횡단 관심사로 따로 정의하고, Spring Aop가 런타임 시점에 핵심 로직과 합쳐서 실행되도록 한다.

[AOP 개념 및 용어]

- Target : 부가기능을 부여할 대상을 의미
- Aspect : 핵심 기능에 부가되어 의미를 갖는 특별한 모듈(Advice + PointCut, 부가기능 정의하는 Advice와 Advice를 어디에 적용할지 결정하는 PointCut)
- Advice : Aspect에서 실질적으로 어떤 일을 해야할지에 대한 부가기능 담은 구현체(’무엇’을 ‘언제’할지 정의)
- JoinPoint : Advice가 적용될 위치
- PointCut : 부가기능을 적용될 대상을 선정하는 방법. Advice를 적용할 JoinPoint를 선별하는 기능을 정의한 모듈
- Proxy : Target을 감싸서 Target에 들어오는 요청을 대신 받아주는 Wrapping 오브젝트

[AOP의 장점]

- 공통 관심 사항을 핵심 관심사항으로부터 분리시켜 핵심 로직을 깔끔하게 유지할 수 있다
- 코드의 가독성, 유지보수성 높일 수 있다
- 각각의 모듈에 수정이 필요하면 다른 모듈의 수정 없이 해당 로직만 변경하며 된다
- 공통 로직을 적용할 대상을 선택할 수 있다.

### 6. 서블릿

서블릿(Servlet)이란 동적 웹 페이지를 만들 때 사용되는 자바 기반의 웹 애플리케이션 프로그래밍 기술이다. 서블릿은 웹 요청과 응답의 흐름을 간단한 메서드 호출만으로 체계적으로 다룰 수 있게 해준다

[WS와 WAS]

- WS(Web Server) : 정적인 웹 리소스를 서비스하는데 특화된 서버 소프트웨어. 웹 서버는 클라이언트의 HTTP 요청을 받아 해당 요청에 맞는 정적 컨텐츠를 반환한다.
- WAS(Web Application Server) : 웹 애플리케이션을 실행하기 위한 서버 소프트웨어. WAS는 클라이언트의 요청에 따라 동적인 웹 페이지를 생성하고 데이터베이스와의 상호작용, 트랙잭션 처리, 보안, 세션 관리 등 웹 어플리케이션의 핵심 비지니스 로직을 수행하는 역할을 담당.

[서블릿을 사용해야하는 이유]

예시) 회원의 이름과 나이를 DB에 저장하는 간단한 WAS

```
// 웹 브라우저가 생성한 요청 HTTP 메시지 - 회원 저장
POST /save HTTP/1.1
Host: localhost:8080
Content-Type: application/x-www-form-urlencoded
username=kim&age=20

// 서버에서 HTTP 응답 메시지 생성
HTTP/1.1 200 OK
Content-Type: text/html;charset=UTF-8
Content-Length: 3423
<html>
 <body>...</body>
</html>
```

이러한 작업을 수행할 때 서버에서는 서버 TCP/IP 연결 대기 및 소켓 연결, 
HTTP 요청 메시지를 파싱해서 읽기, 
POST 방식 /save URL 인지, 
Content-Type 확인, 
HTTP 메시지 바디 내용 피싱, 
저장 프로세스 실행, 
**비지니스 로직 실행(데이터 베이스에 저장 요청)**, 
HTTP 응답 메시지 생성 시작, 
TCP/IP에 응답 전달 및 소켓 종료 과정을 거쳐야 한다. 

하지만 서블릿을 이용할 경우 비지니스 로직 실행만 집중해서 구현하면 된다.

[서블릿의 특징]

- 클라이언트의 Request에 대해 동적으로 작동하는 웹 어플리케이션 컴포넌트
- 기존의 정적 웹 프로그램의 문제점을 보완하여 동적인 여러가지 기능 제공
- JAVA의 스레드를 이용하여 동작
- MVC 패턴에서 컨트롤러로 사용된다
- 서블릿 컨테이너에서 실행
- 보안 기능을 적용하기 쉽다

[서블릿의 동작]

```
@WebServlet(name = "helloServlet", urlPatterns = "/hello")
public class HelloServlet extends HttpServlet{
        @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) {
        //애플리케이션 로직
    }
}
```

- urlPatterns의 URL이 호출되면 서블릿 코드가 실행된다
- HTTP 요청 정보를 편리하게 사용할 수 있는 HttpServletRequest, HTTP 응답 정보를 편리하게 제공할 수 있는 HttpServletResponse
- HttpServletRequest, HttpServletResponse 객체를 생성 후 Web.xml이 어느 서블릿에 대해 요청한 것인지 탐색
- 해당하는 서블릿에서 service() 메소드 호출
- doGet() 또는 doPost() 호출
- 동적 페이지 생성 후 ServletResponse 객체에 응답 전송 후 HttpServletRequest, HttpServletResponse 객체 소멸

[서블릿 컨테이너]

- 톰캣처럼 서블릿을 지원하는 WAS를 서블릿 컨테이너라고 한다.
- 서블릿 컨테이너는 서블릿 객체를 생성, 초기화, 호출, 종료하는 생명주기( 초기화 : init(), 작업수행 : doGet(), doPost(), 종료 : destroy() ) 관리
- 서블릿 객체는 싱글톤으로 관리된다.
- 웹 서버와의 통신을 지원한다
- 멀티쓰레드 지원 및 관리

! 싱글톤 패턴

- 객체 지향 프로그래밍에서 특정 클래스가 단 하나만의 인스턴스를 생성하여 사용하기 위한 패턴
- 생성자를 여러 번 호출하더라도 인스턴스가 하나만 존재하도록 보장하여 애플리케이션에서 동일한 객체 인스턴스에 접근할 수 있도록 한다.
- 인스턴스를 여러 개 만들게 되면 불필요한 자원을 사용하게 되고, 프로그램이 예상치 못한 결과 낳을 수 있음

### 7. Stream API

: Stream API는 컬렉션, 배열 등의 데이터 소스로부터 데이터를 받아와서 원하는 형태로 가공하거나 필터링할 수 있는 기능을 제공(데이터 처리 과정을 직관적으로 표현)

[Stream 특징]

- 원본 데이터를 변경하지 않음 : 스트림 연산은 원본 데이터 소스를 변경하지 않고 새로운 결과 스트림을 생성한다.
- 지연 연산 : 중간 연산은 바로 실행되지 않고 최종 연산이 호출될 때 모든 연산이 한번에 실행된다.
- 일회성 : 한 번 사용된 스트림을 재사용할 수 없다.
- 내부 반복 : 스트림 API는 내부에서 반복을 처리하기 때문에 병렬 처리가 용이하다.

[Stream 처리과정]

**Stream의 객체를 구성할 때 Stream 생성 → 중간연산 → 최종 연산의 세 단계의 과정을 통하여 처리가 이루어진다.** : 객체.Stream생성( ).중간연산.최종연산( )

1. Stream 생성하기 : 스트림을 사용할 데이터 소스에서 스트림을 생성한다.
- 컬렉션 스트림 생성

```
List<String> strList = new ArrayList<>(Arrays.asList("a", "b", "c", "d"));
Stream<String> strStream = strList.stream();
```

- 배열 스트림 생성

```
String[] strArr = {"a", "b", "c", "d"};
// [Type1]
Stream<String> strStream2 = Arrays.stream(strArr);

// [Type2]
Stream<String> strStream3 = Stream.of(strArr);

```

- 빈 스트림 생성

```
Stream<Object> emptyStream = Stream.empty();
```

2. 중간 연산 : 데이터를 가공하는 연산들로 여러 개의 중간 연산을 연결하여 파이프라인을 만든다. 중간 연산은 새로운 스트림을 반환한다.

- filter() : 조건에 맞는 요소만 걸러낸다.

```
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);
List<Integer> evenNumbers = numbers.stream()
        .filter(n -> n % 2 == 0)
        .collect(Collectors.toList());
System.out.println(evenNumbers); // 출력: [2, 4, 6]
```

- map() : 각 요소를 변환한다.

```
List<String> words = Arrays.asList("hello", "world", "java");
List<Integer> wordLengths = words.stream()
        .map(String::length)
        .collect(Collectors.toList());
System.out.println(wordLengths); // 출력: [5, 5, 4]
```

- sorted() : 요소를 정렬한다.

```
List<Integer> numbers2 = Arrays.asList(3, 1, 4, 2, 5);
List<Integer> sortedNumbers = numbers2.stream()
        .sorted()
        .collect(Collectors.toList());
System.out.println(sortedNumbers); // 출력: [1, 2, 3, 4, 5]
```

- distinct() : 중복된 요소 제거한다.

```
List<Integer> numbers3 = Arrays.asList(1, 2, 2, 3, 3, 4, 5);
List<Integer> distinctNumbers = numbers3.stream()
        .distinct()
        .collect(Collectors.toList());
System.out.println(distinctNumbers); // 출력: [1, 2, 3, 4, 5]
```

3. 최종연산 : 스트림 파이프라인의 끝을 맺는 연산으로 최종 결과를 반환하거나 부수 효과를 일으킨다.

- foreach() : 각 요소를 순회하면 연산을 반복 처리할 수 있다

```
numbers.stream()
        .filter(n -> n % 2 != 0) // 홀수 필터링
        .forEach(oddNumbers::add); // 홀수를 리스트에 추가
System.out.println("Odd numbers: " + oddNumbers); // [1, 3, 5]
```

- reduce() : 요소를 결합하여 하나의 값으로 산출

```
int number = 10;
List<Integer> numbers6 = Arrays.asList(1, 2, 3, 4, 5);
int sum2 = numbers6.stream().reduce(number, Integer::sum);
System.out.println(sum2); // 출력: 25
출처: https://sjh9708.tistory.com/191#google_vignette [데굴데굴 개발자의 기록:티스토리]
```

- collect() : 스트림의 요소를 다양한 종류의 결과로 모으고 싶을 때 사용

```
List<String> strings = Stream.of("a", "b", "c")
                            .collect(Collectors.toList());
                            
Set<Integer> numbers = Stream.of(1, 2, 3, 4, 5)
                            .collect(Collectors.toSet());

Map<String, Integer> map = Stream.of("apple", "banana", "cherry")
                                .collect(Collectors.toMap(Function.identity(), String::length));
```

- sum(), count(), average() : 숫자 데이터의 합, 개수, 평균을 구한다

[Stream API의 단점]

- 성능 문제 : 작은 규모의 데이터 처리에서는 일반 for 루프가 더 빠를 수 있다
- 재사용 불가 : 스트림은 일회성이기 때문에 재사용하려면 데이터 소스로부터 새로운 스트림을 생성해야만 한다
- 명령형 코드보다 복잡한 경우도 있다.


### 8. Optional 클래스

Optional은 Java 8에서 도입된 클래스로 Null Pointer Exception을 방지하고 Null을 보다 명확하게 처리하도록 돕는 기능을 제공한다(Null이 될 수 있는 객체를 감싼다)

[사용하는 경우]

- 반환 결과값이 없음을 명확하게 표현할 때
- null을 반환할 때 발생하는 위험을 처리할 때

[Optional 생성하기]

1. ‘Hello’라는 문자열 가진 Optional 객체 생성

```
Optional<String> optional = Optional.of("Hello");
```

2. Null 값을 가진 Optional 객체 생성

```
Optional<String> optional = Optional.empty();

// Null이 될 수 있는 값을 Optional 객체로 감싸려면 Optional.ofNullable()

String str = null;
Optional<String> optional = Optional.ofNullable(str);
```

[Optional 값 접근]

Optional 객체의 값을 얻으려면 get() 메소드 사용한다

```
Optional<String> optional = Optional.of("Hello");
if (optional.isPresent()) {
    String value = optional.get(); // "Hello"
}

// orElse() 메소드를 사용해 Optional 객체가 비어있을 때의 기본 값 설정 가능

Optional<String> optional = Optional.empty();
String value = optional.orElse("Default Value"); // "Default Value"
```

[Optional 사용 시 주의사항]

- Optionaldl null이 될 수 있는 경우
- 클래스 필드로서 Optional이 사용하는 것
- 컬렉션과 벼열에서 Optional 사용하는 것
- null 반환하게 하는 경우