- **SOLID**
## 객체 지향 설계의 5원칙 : SOLID

### 1️⃣ SRP(Single Responsibility Principle): 단일 책임 원칙

- **클래스가 하나의 책임만 가져야 하며, 클래스가 변경되어야 하는 이유는 오직 하나여야 한다는 원칙**
- 여기서 ‘책임’이란, 하나의 기능을 말한다.
- 만약 하나의 클래스에 여러 책임(기능)을 가지고 있다면, 기능 변경이 일어났을 때 수정해야할 코드가 많아진다.
```
// SRP 위반 예제
class Student {
    private String name;
    private String department;

    public Student(String name, String department) {
        this.name = name;
        this.deparatment = department;
    }

    public void goClass() {
        // 수업 듣는 로직
    }

    public void generateReport() {
        // 보고서 생성 로직
    }
}
```
- 위의 코드에서 Student 클래스는 수업을 듣는 클래스와, 보고서 생성 로직 두 가지의 책임을 가지고 있다. 따라서 SRP를 위반하는 예제이다.
```
// SRP 준수 예제
class Student {
		private String name;
		private String department;
		
		public Student(String name, String department) {
				this.name = name;
				this.department = department;
		}
		
		public void goClass() {
				// 수업 듣는 로직
		}
}

class ReportGenerator {
		public void generateReport() {
				// 보고서 생성 로직
		}
}
```
- 위의 코드에서는 Student 클래스는 수업을 듣는 것과 관련된 책임만을 가지며, 보고서 생성 로직은 별도의 클래스로 분리되었다. 따라서 SRP를 준수하는 예제이다.
- SRP 원칙에서, 책임의 범위는 딱 정해져 있는 것이 아니라, 어떤 로직을 구현하느냐에 따라 개발자마다 기준이 달라질 수 있다.
### 2️⃣ OCP(Open Closed Principle): 개방-폐쇄 원칙

- **소프트웨어 개체(클래스, 모듈, 함수 등)는 확장에 대해서는 열려 있어야 하지만 변경에 대해서는 페쇄되어야 한다는 원칙**
- 기능 추가할 때, 클래스를 확장을 통해 손쉽게 구현하면서, 확장에 따른 클래스 수정은 최소화 하도록 구현해야하는 설계 기법이다.
```
// 도형을 그리는 인터페이스
interface Shape {
    void draw();
}

// 원 클래스
class Circle implements Shape {
    public void draw() {
        System.out.println("원을 그립니다.");
    }
}

// 사각형 클래스
class Rectangle implements Shape {
    public void draw() {
        System.out.println("사각형을 그립니다.");
    }
}

// 그림 그리는 클래스
class Drawing {
    public void drawShape(Shape shape) {
        shape.draw();
    }
}
```
- 위의 코드에서 Shape 인터페이스를 구현한 클래스들은 확장에 열려 있고, 수정에 닫혀있다. → 새로운 도형 클래스를 추가할 때 기존 코드를 수정하지 않고도 추가할 수 있다는 의미이다.
```
// 삼각형 클래스
class Triangle implements Shape {
    public void draw() {
        System.out.println("삼각형을 그립니다.");
    }
}
```
- 예를 들어, 삼각형 클래스를 추가하려면 위와 같이 기존 코드를 수정하지 않고, 기능을 확장할 수 있다.
- OCP원칙은 추상화 사용을 통한 관계 구축을 권장 → 다형성과 확장을 가능하게 하는 객체지향의 장점을 극대화하는 기본적인 설계 원칙
### 3️⃣ LSP(Listov Subtitution Principle): 리스코프 치환 원칙

- **서브 타입은 언제나그것의 슈퍼타입으로 대체할 수 있어야 한다는 원칙**
- 어떤 클래스의 인스턴스가 있을 때, 그 클래스의 하위 클래스의 인스턴스로 대체해도 프로그램은 정상적으로 동작해야 한다는 의미이다.
```
class Bird {
    public void fly() {
        System.out.println("날아갈 수 있습니다.");
    }
}

class Sparrow extends Bird {
		// 참새는 날 수 있음.
    // Sparrow는 Bird를 확장하면서 추가적인 행위를 정의하지 않음
}

class Ostrich extends Bird {
    public void fly() {
        // 타조는 날지 못하므로 오버라이딩하여 구현을 변경
        System.out.println("날지 못합니다.");
    }
}
```
- 위의 코드에서 Bird 클래스는 fly 메서드를 가지고 있으며, Sparrow 클래스는 이 매서드를 오버라이딩 하지 않고 상속한다.
- Ostrich는 날지 못하므로 fly 메서드를 오버라이딩하여 Bird클래스와 다르게 구현한다.
- 여기서, Sparrow와 Ostrich클래스 모두 Bird 타입의 객체로 대체될 수 있으며, 프로그램은 정상적으로 작동한다.
```
public class Main {
    public static void main(String[] args) {
        Bird sparrow = new Sparrow();
        Bird ostrich = new Ostrich();

        sparrow.fly(); // "날아갈 수 있습니다." 출력
        ostrich.fly(); // "날지 못합니다." 출력
    }
}
```
- 위의 코드에서 다형성의 특징을 이용하기 위해, 상위 클래스 타입으로 객체를 선언하여 하위 클래스의 인스턴스를 받는다.
- LSP원칙은 이러한 상황에서 업캐스팅된 상태에서 부모의 메서드를 사용해도 동작이 의도대로 흘러가야 하는 것을 의미한다.
### 4️⃣ ISP(Interface Segregation Principle): 인터페이스 분리 원칙

- **클라이언트가 자신이 사용하지 않는 메서드에 의존하지 않아야 한다는 원칙**
- SRP 원칙이 클래스의 단일 책임을 강조한다면, ISP 원칙은 인터페이스의 단일 책임을 강조하는 것이다.
```
// ISP 위반 예제
interface Worker {
    void work();
    void eat();
}

class Human implements Worker {
    public void work() {
        // 일하는 로직
    }
    
    public void eat() {
        // 식사하는 로직
    }
}

class Robot implements Worker {
    public void work() {
        // 일하는 로직
    }
    
    public void eat() {
        // 로봇은 먹지 않는데도 먹는 메서드를 구현해야 함
    }
}
```
- 위의 코드에서 Worker 인터페이스는 work와 eat 매서드를 가지고 있다.
- 하지만, Worker 인터페이스를 상속받아 Robot 클래스를 구현하려면 로봇은 먹지 못함에도 불구하고 eat 매서드를 구현해야 한다.
```
// ISP 준수 예제
interface Workable {
    void work();
}

interface Eatable {
    void eat();
}

class Human implements Workable, Eatable {
    public void work() {
        // 일하는 로직
    }
    
    public void eat() {
        // 식사하는 로직
    }
}

class Robot implements Workable {
    public void work() {
        // 일하는 로직
    }
}
```
- ISP 원칙을 준수하려면, 위와 같은 코드로 인터페이스를 분리해서 클라이언트가 필요로 하는 인터페이스만 상속받도록 해야 한다.
- 이때, 인터페이스를 한번 분리해 구성해 놓고, 이후에 수정 사항이 생겨 다시 인터페이스를 분리하는 행위를 가하지 않도록 주의해야 한다.
### 5️⃣ DIP(Dependency Inversion Principle): 의존 역전 원칙

- **고수준 모듈은 저수준 모듈에 의존해서는 안 되며, 둘 모두 추상화에 의존해야 한다는 원칙**
- 소스 코드의 의존성이 추상화에 의해 정의되어야 하며, 구체적인 구현에 의존해서는 안된다는 것을 의미한다.
- 고수준 모듈: 어떠한 의미 있는 단일 기능을 제공하는 모듈. 시스템의 정책, 규칙, 흐름을 정의하는 쪽
- 저수준 모듈: 고수준 모듈의 기능을 구현하기 위해 필요한 기능들을 구현한 모듈
```
// 저수준 모듈
class LightBulb {
    public void turnOn() {
        System.out.println("전구가 켜집니다.");
    }

    public void turnOff() {
        System.out.println("전구가 꺼집니다.");
    }
}

// 고수준 모듈
interface Switch {
    void operate();
}

class RemoteControl implements Switch {
    private LightBulb bulb; // LightBulb 클래스에 직접 의존

    public RemoteControl(LightBulb bulb) {
        this.bulb = bulb;
    }

    public void operate() {
        bulb.turnOn();
    }
}
```
- 위의 코드에서 RemoteControl 클래스가 LightBulb 클래스에 직접 의존하고 있으므로 DIP를 위반하는 예제이다.
```
// 추상화
interface Switchable {
    void turnOn();
    void turnOff();
}

// 저수준 모듈
class LightBulb implements Switchable {
    public void turnOn() {
        System.out.println("전구가 켜집니다.");
    }

    public void turnOff() {
        System.out.println("전구가 꺼집니다.");
    }
}

// 고수준 모듈
interface Switch {
    void operate();
}

// 고수준 모듈
class RemoteControl implements Switch {
    private Switchable device; // Switchable 추상화에 의존

    public RemoteControl(Switchable device) { // 생성자 주입
        this.device = device;
    }

    public void operate() {
        device.turnOn();
    }
}
```
- 위의 코드에서는 RemoteControl 클래스는 Switchable 인터페이스에 의존하고 있으며, LightBulb 클래스가 이 인터페이스를 구현하고 있다.
- Fan, Heater와 같은 새로운 장치를 추가해도 RemoteControl을 수정할 필요가 없다.
- 이렇게 하면 고수준 모듈이 저수준 모듈에 직접 의존하지 않고 추상화에 의존하게 되어, 시스템이 더 유연하고 확장 가능해진다.
<aside>
💡

**RemoteControl**은 “리모콘을 눌렀을 때 장치를 켠다.” 라는 동작의 규칙/정책을 담고 있음. → 고수준 모듈
**LightBulb**는 단순히 “켜지고 꺼진다’는 세부 구현만 담당 → 저수준 모듈

</aside>

- **DI**
    
    ## DI : Dependency Injection 의존성 주입
    
    의존성 주입이란? 어떤 객체나 함수가 필요로 하는 다른 객체를 내부에서 직접 생성하지 않고, 외부에서 주입 받는 프로그래밍 기법이다.
    
    ### DI의 구성요소 (Roles)
    
    - Service (서비스)
        - 기능을 제공하는 쪽
        - “나는 불을 켤 수 있다.”, “나는 결제를 처리할 수 있다” 와 같은 실제 기능을 갖고 있다.
    - Client (클라이언트)
        - 서비스를 사용하는 쪽
        - 혼자 동작하지 못하고, Service가 필요함.
    - Interface (인터페이스 = 추상화)
        - 클라이언트가 서비스에 직접 의존하지 않고, “중간에 있는 약속”으로 쓰는 것
        - SOLID의 DIP원칙과 관련되어 있음. (DIP는 설계 원칙, DI는 원칙을 실제로 코드에서 구현하는 기법)
    - Injector (인젝터)
        - 클라이언트와 서비스를 실제로 연결해 주는 것
        - 생성자, 수정자, 필드 주입 방법이 있다.
    
    ### DI의 장점
    
    1. 결합도 감소
        - 클라이언트가 의존성이 어떻게 구현되어있는 지를 알 필요가 없어지기 때문에, 클래스 간 결합도가 낮아진다 → **결과적으로 코드를 재사용하기 쉽고, 테스트가 용이하고, 유지보수가 쉽게 구현할 수 있다.**
        1. 유연성 증가
            - 클라이언트는 인터페이스를 구현해 자신이 기대하는 어떤 객체든 사용할 수 있다. → **다양한 구현체를 상황에 따라 바꿔 끼울 수 있다.**
        2. 중복 코드 감소
            - 의존성 생성  및 초기화를 한 곳에서 처리하므로 반복 코드가 줄어든다.
        3. 병렬 개발 가능
            - 두 개발자가 동시에 개발 가능하다.
            - 각자 상대방의 구현체는 몰라도, 인터페이스만 알면 독립적으로 개발할 수 있다.
    
    ### DI의 단점
    
    1. 코드 추적 어려움
        - 객체 생성과 사용이 분리되기 때문에, 코드만 보면 객체를 추적하기 어려울 수 있다.
    2. 프레임워크 의존성
        - 보통 DI는 프레임워크(Spring 등)에 의존해서 사용해서, 해당 프레임워크에 종속될 위험이 있다.
    - 
    
    ### 의존성 주입하는 방식
    
    1. 생성자 주입
        1. 의존성을 생성자 파라미터로 받음. 
        2. 객체가 생성될 때 필요한 의존성이 모두 주입된다. → 불변성 보장, NULL 위험성 줄어듦.
    2. 매서드 주입
        1. 특정 기능을 수행할 때, 매서드 파라미터로 의존성을 받는다.
    3. 세터 주입
        1. Setter 메서드를 통해 의존성을 주입한다.
        2. 객체 생성 후에도 변경 가능하다. → 유연하지만, 의존성이 주입되지 않은 상태에서 사용될 위험이 있다.
    4. 인터페이스 주입
        1. 의존성이 자신을 클라이언트에 주입할 수 있는 메서드를 인터페이스로 제공한다.
        2. 클라이언트가 특정 인터페이스를 구현하야하고, 인젝터가 그 인터페이스를 통해 의존성을 넣어준다.
        3. 객체가 꼭 특정 인터페이스를 구현해야 한다는 강제성이 생기기 때문에, 실무에서는 거의 쓰이지 않는다.
    
- **IoC**
    
    ## IoC: Inversion of Control (제어의 역전)
    
    제어의 역전이란? 프로그램의 흐름을 기존의 방식과 반대로 뒤집는다는 설계 원칙이다.
    
    전통적 프로그래밍에서는 개발자(A) → 라이브러리(B)를 호출해서 기능을 사용하는 것이 기본 흐름이지만, IoC에서는 외부 프레임워크나 환경이 필요할 때 개발자(A)한테 호출해주는 방식으로 흐름이 바뀌는 것이다.
    
    ### IoC의 두 가지 관점
    
    1. 제어 흐름 관점(이벤트 중심)
        1. 브라우저 이벤트 리스너, 스프링 MVC의 URL매핑 같은 경우
        2. 개발자가 직접 요청을 분기하지 않고, 프레임워크가 알아서 함수를 호출해준다.
        3. 이때의 IoC는 프로그램 실행 중 특정 이벤트/상황이 발생했을 때, 프레임워크가 내 코드를 호출하는 것을 말한다.
        4. 예: @GetMapping(”/hello”) 붙은 메서드는 개발자가 직접 호출하는 것이 아니라, Spring이 URL매칭되면 알아서 호출 → IoC
    2. 객체 관리 관점(DI/컨테이너 중심)
        1. Spring IoC 컨테이너(ApplicationContext)가 빈(Bean)을 스캔하고, 생성자 주입/세터 주입으로 의존성 주입
        2. 객체 생성, 연결, 라이프사이클 관리를 코드가 하지 않고 외부 컨테이너가 제어한다.
        3. 이때의 IoC는 객체 생성/연결의 제어권을 외부가 가져간다는 뜻이다.
    
    ### Spring의 IoC컨테이너 = ApplicationContext
    
    1. @Configuration이 붙은 클래스들을 설정 정보로 등록해두고, @Bean이 붙은 메소드의 이름으로 빈 복록을 생성한다.
    2. 클라이언트가 해당 빈을 요청한다.
    3. ApplicationContext는 자신의 빈 목록에서 요청한 이름이 잇는지 찾는다.
    4. ApplicationContext는 설정 클래스로부터 빈 생성을 요청하고, 생성된 빈을 돌려준다.

- **생성자 주입 vs 수정자, 필드 주입` 차이**
    - 생성자 주입
        - 객체의 생성자를 통해서 의존성을 주입하는 방식이다.
        - **가장 흔하게 쓰이는 의존성 주입 방식이다.**
        - 객체를 생성할 때 한 번 생성자를 호출해 의존 관계를 정의하고, 불변으로 설계할 수 있다.
        - null을 주입하지 않는한 NullPointerException은 발생하지 않는다.
        
        ```java
        public class Client {
            private Service service;
        
            // The dependency is injected through a constructor.
            Client(final Service service) {
                if (service == null) {
                    throw new IllegalArgumentException("service must not be null");
                }
                this.service = service;
            }
        }
        ```
        
    - 수정자 주입
        - 수정자 함수(set Method)를 이용해서 의존성을 주입하는 방식이다.
        - 수정자 주입 방식은 런타임 시에 할 수 있도록 낮은 결합도를 가지게 구현되어 있어서, 변경 가능성이 있는 의존 관계에 사용한다.
        - 객체를 생성한 후 Setter를 호출하지 않은 채로 다른 메서드를 호출하게 되면, 누락된 필드의 변수는 null로 남아있어 NullPointerException이 발생할 수 있다.
        
        ```java
        public class Client {
            private Service service;
        
            // The dependency is injected through a setter method.
            public void setService(final Service service) {
                if (service == null) {
                    throw new IllegalArgumentException("service must not be null");
                }
                this.service = service;
            }
        }
        ```
        
    - 필드 주입
        - 클래스 필드에 그대로 주입하는 방식이다.
        - 코드가 매우 간결하다는 장점이 있다.
        
        ```java
        public class Client {
        
        		@Autowired // 순수 자바 언어로 설명 불가능
            private Service service;
        
            ...
        }
        ```
        
    
    ### 왜 생성자 주입을 사용해야 할까?
    
    - 대부분의 의존 관계는 애플리케이션 종료까지 거의 변하지 않는다. → 생성자 주입 방식으로 객체를 생성할 때 불변(final 키워드)으로 설계할 수 있다.
    - DI 프레임워크에 의존하지 않고, 순수 자바 언어로도 잘 작동하며 자바 언어의 객체 지향이라는 특징을 잘 살릴 수 있다.
    - 생성자 주입은 필수 의존성을 강제하고, 의존성 주입이 명시적으로 보이기 때문에 수정자 주입/필드 주입 보다 테스트 하기에 더 용이하다.

- **AOP**
    
    ## AOP: Aspect-Oriented Programming (관점 지향 프로그래밍)
    
    **AOP란?** 프로그래밍 패러다임 중 하나로, 관심사의 분리를 위해 사용되는 기술이다. 
    
    ### AOP의 목적
    
    - 비지니스 웹 애플리케이션에는 핵심 비즈니스 로직과 부가 기능 로직이 있다.
    - 이때, 애플리케이션 전체를 관통하는 부가 기능 로직인 횡단 관심사의 코드를 핵심 비즈니스 로직의 코드와 분리하여 코드의 간결성을 높이고 변경에 유연하도록 하는 것이다.
    
    ### AOP 용어 및 개념
    
    - Aspect: 공통 기능(보통 횡단 관심사)을 모아둔 모듈
    - Join Point: 공통 기능을 삽입할 수 잇는 위치
    - Advice: Join Point에서 실행되는 코드
        - @Before: 메서드 실행 전에 실행
        - @After: 메서드 실행 후에 실행
        - @AfterReturning: 메서드가 정상 종료된 뒤 실행
        - @AfterThrowing: 예외가 던져진 경우 실행
        - @Around: 메서드 실행 전/후 모두 제어
    - Pointcut: 실제로 공통 기능을 적용할 Join Point를 선택하는 조건
    - Weaving: Pointcut + Advice를 실제 코드 실행에 연결하는 과정
    
    ### 스프링에서의 AOP 동작
    
    1. 스프링 IoC 컨테이너 초기화
        - @EnableAspectJAutoProxy: AspectJ 스타일 애노테이션(@Aspect, @Before 등)을 인식해서 프록시를 자동 생성하는 설정을 켜주는 역할
        - spring-boot-starter-aop: 스트링 부트에서 제공하는 스타터 라이브러리로, 이 의존성을 추가하면 AspectJ관련 라이브러리가 자동 추가되고, @EnableAspectJAutoProxy가 기본으로 켜진다.
        
        → AnnotationAwareAspectJAutoProxyCreator라는 빈후처리기(BeanPostProcessor)를 컨테이너에 등록한다.
        
        - 컨테이너가 Bean을 생성할 때 끼어들어서 @Aspect 클래스들을 스캔하고, Bean이 Pointcut에 해당되면 원본 Bean 대신 프록시 객체를 만들어서 컨테이너에 등록한다.
        - 이때 프록시 객체는 Target(실제 원본 Bean 객체), Advice Chain(적용할 Advice목록), Invocation Handler를 담고 있다.
            - Invocation Handler: Adivce를 순서대로 실행 → 필요할 때 Target메서드 호출 → 결과/예외를 Advice로 다시 전달
    2. 런타임 호출(메서드 실행) 시점
        - 애플리케이션 코드가 빈의 메서드를 호출하면 실제로는 프록시 메서드가 먼저 실행 된다.
        - 프록시는 포인트 컷에 매칭되는지 검사하고, 매칭되면 Advice를 실행한 뒤 원본 메서드를 호출한다.
    3. Advice 실행 흐름
        - @Around → @Before → Target 메서드 실행 → @AfterReturning/@AfterThrowing → @After → @Around
    
    ```java
    // Target Bean
    
    @Service
    public class HelloService {
        public String sayHello(String name) {
            System.out.println("실제 로직 실행: Hello " + name);
            return "Hello " + name;
        }
    }
    
    // AOP Aspect 클래스 
    @Aspect
    @Component
    public class LoggingAspect {
    
        // com.example.demo.service 패키지의 모든 메서드 실행 전 실행
        @Before("execution(* com.example.demo.service.*.*(..))")
        public void logBefore(JoinPoint joinPoint) {
            System.out.println("[AOP] 실행 전: " + joinPoint.getSignature().getName());
        }
    }
    ```
    
    - 애플리케이션이 시작할 때 빈 후처리기(AnnotationAwareAspectJAutoProxyCreator)가 컨테이너에 등록됨 → 컨테이너가 LoggingAspect 클래스를 빈으로 등록할 때, @Aspect 붙었네? 내부 포인트컷 파싱해서 저장 → HelloService가 빈으로 등록될 때 포인트 컷에 걸리네? → 원본 HelloService 그대로 저장하지 않고 프록시 객체로 컨테이너에 넣음 → 이 프록시 객체는 내부적으로 HelloService를 참조하고 있음. → 만약 HelloService가 포인트 컷에 걸리지 않는다면 원본 객체로 컨테이너에 넣음

- **서블릿**
    
    ## 서블릿(Servlet)이란?
    
    - 자바를 사용하여 웹 페이지를 동적으로 생성하는 서버 측 프로그램을 말한다.
    
    ### 서블릿의 특징
    
    - 자바 언어로 작성되고, JVM(Java Virtual Machine)위에서 동작한다. → JVM만 있으면 어디든 실행 가능
    - 클라이언트의 요청에 의해서 동적으로 실행된다. → 다양한 클라이언트 요구 사항을 처리할 수 있다.
    - 하나의 서블릿 객체를 여러 요청이 공유한다.
    - 요청이 들어올 때마다 새 스레드가 생성되어 동시에 처리된다.
    - 객체 생성, 초기화, 실행, 소멸까지 lifecycle을 서블릿 컨테이너가 관리한다. → 개발자는 로직만 작성해도 되며, 실행 환경은 컨테이너가 보장한다.
    
    ### 서블릿 컨테이너와 동작 과정
    
    - 서블릿 컨테이너는 서블릿 객체들의 생성, 초기화, 실행, 소멸을 담당한다.
    - WAS와 같은 개념이라고 볼 수 있다.
    
    ![image.png](attachment:46b61992-61ed-4ea5-9ef0-6a9edb8ada15:image.png)
    
    1. 클라이언트가 웹 서버에 요청을 한다.
    2. 웹 서버는 클라이언트의 요청을 Tomcat과 같은 서블릿 컨테이너에 위임한다.
    3. 서블릿 컨테이너는 각 요청마다 HttpServletRequest와 HttpServletResponse객체를 생성한다.
    4. 서블릿 컨테이너는 각 요청에 해당하는 서블릿 객체의 service()를 호출한다. 이때, 서블릿 객체는 보통 하나씩 생성되어있고, 요청을 받을 때마다 스레드를 생성한다. (멀티스레드)
    5. HTTP메서드에 따라 doGet()/doPost()를 호출해 HttpServletResponse객체에 응답을 담는다.
    6. 서블릿 컨테이너가 HttpServletResponse를 클라이언트로 전송한다.
    
    ### Servlet의 생명주기
    
    1. 서블릿 클래스 로딩 및 인스턴스화: 요청이 오면 Servlet 클래스가 로딩되어 요청에 대한 Servlet 객체 생성 (Servlet 객체 생성 시점은 서버 설정에 따라서 약간 다를 수 있음)
    2. 초기화: init() 메서드 호출로 Servlet초기화
    3. 서비스: service() 메서드 호출로 Servlet에게 요청 처리
    4. 종료: destroy() 메서드 호출로 Servlet제거, GC 진행, 디폴트로는 서버 종료시에만 실행된다.

- **자바의 Optional 클래스**
    
    ## Optional 이란?
    
    - 값이 없는 경우를 표현하기 위한 클래스
    - Optional 객체는 값이 존재할 수도 있고, 없을 수도 있다. → NullPointerException 예외를 방지할 수 있다.
    
    ### Optional의 필요성
    
    - 어떤 메서드가 null을 반환할지 확신할 수 없거나, null 처리를 놓쳐서 발생하는 예외를 피할 수 있다.
    - Optional을 사용하게 되면 코드를 더 명확하게 작성할 수 있고, 예외 처리를 더 쉽게 할 수 있어 코드 가독성이 높아지며 유지 보수도 더 편리해진다.
    
    ### Optional 객체 생성하기
    
    ```java
    // 빈 Optional
    Optional<Car> optCar1 = Optional.empty();
    
    // null이 아닌 값을 담는 Optional
    // car가 null이면 NullPointerException 발생
    Optional<Car> optCar2 = Optional.of(car);
    
    // null값을 담을 수 있는 Optional
    // car가 null이면 빈 Optional 객체 반환
    Optional<Car> optCar3 = Optional.ofNullable(car);
    ```
    
    ### Optional이 제공하는 메서드
    
    - isPresent(): Optional의 값이 존재하면 true, 해당 값이 없으면 false를 리턴한다.
    - isEmpty(): isPresent() 메서드와 반대로 작용한다.
    - get(): Optional이 감사고 있는 실제 값을 얻기 위해서 사용하며, 만약 Optional의 값이 비어있는 경우, NoSuchElementException이 발생한다.
    - ifPresent()/ifPresentOrElse(): 값이 있는 경우이거나,  값이 있거나 없는 경우를 처리할 수 있다.
    
    ```java
    Optional<String> optionalString = Optional.of("Hello");
    optionalString.ifPresent( val -> System.out.println("Optional contains a value: " + val),
    () -> System.out.println("Optional is empty") );
    ```
    
    - orElse()/orElseGet/orElseThrow: 값이 없을 경우를 처리하는 메서드이다.
    
    ```java
    Optional<String> opt = Optional.ofNullable(null);
    
    // 값이 없으므로 "default" 반환
    String result = opt.orElse("default");  
    System.out.println(result); // default
    
    // 값이 없으므로 람다 실행 -> "computed" 반환
    String result2 = opt.orElseGet(() -> "computed");
    System.out.println(result2); // computed
    
    // 값이 없으므로 IllegalArgumentException 발생
    String result3 = opt.orElseThrow(() -> new IllegalArgumentException("값 없음"));
    ```

- **Stream API**
    
    ## Stream API란?
    
    - 람다식을 이용한 기술 중에 하나로 데이터 소스(컬렉션, 배열 등)를 조작 및 가공, 변환하여 원하는 값으로 변환해주는 인터페이스이다.
    - Stream API는 데이터를 추상화하고, 처리하는 데 자주 사용되는 함수들을 제공한다.
    
    ### 람다식(Lambda Expression)이란?
    
    - 함수를 하나의 식으로 표현한 함수형 인터페이스 함수로, 람다식으로 표현하면 메서드의 이름이 없기 때문에 익명 함수의 한 종류이기도 하다.
    
    ### Stream Type
    
    - byte, short, char,float, string, boolean → Stream<Type>
    - int, long, double → ‘type’+Stream
    
    ### Stream API 특징
    
    - 원본의 데이터를 변경하지 않는다.
        - Stream APi는 원본의 데이터를 조회하여 별도의 Stream을 생성한다.
        - 원본의 데이터로부터 읽기만 할 뿐이며, 정렬이나 필터링 등의 작업은 별도의 Stream요소들에서 처리 된다.
        
        ```java
        String[] nameArr = {"IronMan", "Captain", "Hulk", "Thor"}
        List<String> nameList = Arrays.asList(nameArr);
        
        // 별도의 스트림을 생성함.
        Stream<String> nameStream = nameList.stream();
        Stream<String> arrayStream = Arrays.stream(nameArr);
        
        // 복사된 데이터를 정렬하여 출력함
        nameStream.sorted().forEach(System.out::println);
        arrayStream.sorted().forEach(System.out::println);
        ```
        
    - 일회용 이다.
        - Stream API는 일회용이기 때문에 한번 사용이 끝나면 재사용이 불가능하다.
        - Stream이 또 필요한 경우에는 Stream을 다시 생성해야 한다.
        - 만약 닫힌 Stream을 다시 사용한다면 IllegalStateException이 발생한다.
        
        ```java
        userStream.sorted().forEach(System.out::print);
        
        // 스트림이 이미 사용되어 닫혔으므로 에러 발생
        int count = userStream.count();
        ```
        
    - 내부 반복으로 작업을 처리한다.
        - for이나 while문 같은 반복 문법을 메서드 내부에 숨기고 있기 때문에 간결한 코드 작성이 가능하다.
        
        ```java
         // 반복문이 forEach라는 함수 내부에 숨겨져 있다.
        nameStream.forEach(System.out::println); 
        ```
        
        ### Stream 연산
        
        1. Stream 생성
            1. empty stream 생성 → Stream.empty();
            2. Collection 생성 → ArrayList.stream();
        2. 중간 연산
            1. filter() → 결과 값 필터링하기 위해 사용된다.
            2. distinct() → 중복을 제거한 결과값을 반환하기 위해 사용된다.
            3. map() → 요소들을 원하는 값으로 변화하여서 반환하기 위해 사용된다.
            4. sorted() → 요소들에 대해서 오름/내림 차순을 수행하여 반환하기 위해 사용된다.
            5. 등 … 많습니다.
        3. 최종 연산
            1. forEach() → 배열 혹은 리스트 내에서 순회하며 요소에 대한 값을 출력하거나 새로운 형태로 변환하여 구성하기 위한 목적으로 사용된다.
            2. findFirst() → 스트림 내에서 가장 앞에 잇는 요소를 리턴하는 함수
            3. findAny() → 스트림 내에서 먼저 탐색되는 요소를 리턴하는 함수
            4. count() → 요소들의 개수를 리턴하는 함수
            5. sum() → 요소들의 합을 리턴하는 함수