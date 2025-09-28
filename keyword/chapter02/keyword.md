- **SOLID**
    
    SOLID : 객체 지향 설계의 5원칙이다. 이 원칙을 지키면 시간이 지나도 변경이 용이하고 유지보와 확장이 쉬운 소프트웨어를 개발할 수 있다.
    
    S : 단일 책임 원칙 ( Single Responsibility Principle)
    
    O : 개방 폐쇄 원칙 (Open Closed Priciple)
    
    L : 리스코프 치환 원칙 (Liskov Substitution Principle)
    
    I : 인터페이스 분리 원칙 (Interface Segregation Principle)
    
    D : 의존 역전 원칙 (Dependency Inversion Principle)
    
    ## 1. 단일 책임 원칙
    
    - 클래스(객체)는 단 하나의 책임만 가져야 한다는 원칙이다. 여기서 책임은 하나의 기능 담당이다. 하나의 클래스는 하나의 기능을 담당하여 하나의 책임을 수행할 수 있도록 클래스를 따로 설계하라는 원칙이다.
    - 만일 하나의 클래스에 여러 책임이 있다면 기능 변경이 일어났을 때 수정 사항이 많아지고 책임이 순환되는 형태가 될 수 있다. 단일 책임 원칙을 적용하면 한 책임의 변경으로부터 다른 책임의 변경으로의 연쇄 작용을 극복할 수 있다.
    
    ## 2. 개방 폐쇄 원칙
    
    - 확장에 열려있어야 하며 수정에는 닫혀있어야 한다는 원칙이다. 기능의 추가가 발생했을 때, 클래스의 확장을 통해 구현이 가능하고, 확장에 따른 클래스 수정은 최소화하도록 프로그램을 설계하는 기법이다.
    - 즉, 새 요구가 생겨도 기존 코드를 고치지 않고 새 코드를 덧붙여서(확장) 해결하라는 설계 원칙이다.
    - 요구사항이 바뀔 때 마다 기존 클래스를 고치면 연쇄적인 수정이 필요할 수 있기 떄문에 기존의 안정 영역을 고정시키고, 변화는 확장하는 방식.
    
    ## 3. 리스코프 치환 원칙
    
    - 서브 타입은 언제나 부모 타입으로 교체할 수 있어야 한다는 원칙이다. 즉, 다형성 원리를 이용하기 위한 원칙이라고 생각하면 된다. 다형성의 특징을 이용하기 위해 상위 클래스타입으로 객체를 선언하여 하위 클래스의 인스턴스를 받으면, 업캐스팅된 상태에서 부모의 메서드를 사용해도 동작이 의도대로 흘러가야한다는 것을 의미한다.
    - 부모 클래스 객체를 사용하는 코드에서 자식 클래스로 바꿔도 프로그램의 기능이 깨지지 않아야 한다.
    - “사각형은 직사각형이다” ← 이런 식으로 겉보기에는 맞는 것 같아도 상속 관계가 어긋난 경우를 주의해야 한다. 한다는 것이다. 부모가 가진 계약을 자식이 반드시 지켜야 한다. ( 사각
    - 쉽게 말해서 자식은 부모의 규칙을 지켜야 한다.
    
    ## 4. 인터페이스 분리 원칙
    
    - 한마디로, “자신이 사용하지 않는 메서드에 의존하지 않아야 한다”는 것이다.
    - 하나의 거대한 인터페이스에 너무 많은 메서드가 있으면, 구현체는 쓰지도 않는 메서드를 억지로 구현해야 한다. → 불필요한 의존성이 증가하고, 변경에 따른 영향 범위가 커진다.
    - 작은 인터페이스 여러개가 하나의 거대한 인터페이스보다 백배 낫다!  잘 쪼개자
    
    ## 5. 의존 역전 원칙
    
    - 고수준의 정책 또는 비즈니스 규칙은 저수준에 의존하면 안된다. 둘 다 인터페이스에 이존해야 한다는 원칙이다. 인터페이스는 구체적인 것에 의존하지 않고, 구체적인 것이 인터페이스에 의존해야 한다.인 것에
    - 즉, 정책 코드나 DB 같은 세부 구현에 의존하지 말고, 추상화한 인터페이스에 의존하라는 것이다. 이렇게 해야 구현이 바뀌어도 수정을 최소화할 수 있다.
    - 간단히 말하면 인터페이스를 사용하라는 것이다.
    
- **DI**
    
    ## DI (Dependency Injection) 이란?
    
    외부에서 두 객체 간의 관계를 결정해주는 디자인 패턴으로, 인터페이스를 사이에 둬서 클래스 레벨에서는 의존 관계가 고정되지 않도록 하고 런타임 시에 관계를 동적으로 주입해주는 것을 말한다.
    
     즉, 객체간의 의존 관계를 개발자가 직접 생성하지 않고, 스프링 컨테이너가 대신 주입해주는 것이다. 
    
    의존성이란? 
    
    어떤 클래스가 다른 클래스를 사용해야 할 때 그 객체를 의존한다고 말한다.
    
    DI가 왜 필요할까?
    
    - 결합도 낮추기 : 직접 new를 통해 객체를 만들면 클래스 간 결합도가 높아져, 테스트와 확장이 어렵다. 반면, 스프링이 객체 생성을 관리하면, 인터페이스 기반으로 코드를 짜고 쉽게 교체가 가능하다.
    - 재사용성 증가 : 같은 코드를 다양한 상황에서 활용이 가능하다.
    
    스프링은 의존성 주입을 도와주는 DI 컨테이너로서, 강하게 결합된 클래스를 분리하고, 애플리케이션 실행 시점에 객체 간의 관계를 결정해줌으로써 결합도를 낮추고 유연성을 확보해준다. 
    
    이러한 방법은 상속보다 훨씬 유연하다. 단, 한 객체가 다른 객체를 주입받으려면 반드시 DI 컨테이너에 의해 관리되어야 한다.  
    
    ```java
    @Component
    class Engine { void start(){ System.out.println("엔진 시동"); } }
    
    @Component
    class Car {
        private final Engine engine;
        Car(Engine engine) { this.engine = engine; } // ✅ 생성자 주입
        void drive() { engine.start(); }
    }
    
    @SpringBootApplication
    public class App {
        public static void main(String[] args) {
            var ctx = SpringApplication.run(App.class, args);
            ctx.getBean(Car.class).drive(); // new 없이 컨테이너에서 꺼냄
        }
    }
    ```
    
    - `@Component`를 스캔해 **빈(객체)를 생성한다.**
    - `Car`의 생성자를 보고 **필요한 Engine 빈을 자동 주입한다.**
    - `new` 대신 **컨테이너에서 꺼내(getBean)** 사용한다.
        
        → **객체 생성/연결의 제어권 = 스프링 컨테이너** (IoC), 
        
        →주입 동작 = DI
        
    
- **IoC**
    
    ## IoC (Inversion of Control) 이란?
    
    프로그램의 제어 흐름을 개발자가 직접 제어하는 것이 아니라, 외부(스프링 컨테이너) 가 제어하는 것을 말한다.
    
    원래대로라면 개발자가 직접 new를 통해 객체를 생성하고 관리해야 하지만, IoC를 적용하면 이런 객체 생성과 생명주기에 관한 관리 제어권이 개발자에서 스프링으로 넘어간다. 
    
    즉, 개발자는 “무엇을 할 지”에 집중하고, 언제/어떻게 실행할지는 스프링이 알아서 해주는 구조라고 생각하면 된다.
    
    DI는 IoC를 구현하는 방법 중 하나다. 스프링에서는 IoC를 DI를 통해 구현한다.
    
    스프링은 IoC 컨테이너를 통해 객체(빈)를 생성하고, 관리하며, 필요할 때 주입해준다.
    
    객체 생성과 주입을 스프링 컨테이너 즉, DI 컨테이너가 담당하며, 제어권이 개발자에서 스프링으로 넘어가게 되는 것이다. 
    
    ```java
    @Component
    class A {
        public void doSomething() {
            System.out.println("A 동작");
        }
    }
    
    @Component
    class B {
        private final A a;
    
        @Autowired   // 스프링이 알아서 A를 주입
        public B(A a) {
            this.a = a;
        }
    
        public void action() {
            a.doSomething();
        }
    }
    
    @SpringBootApplication
    public class IoCExample {
        public static void main(String[] args) {
            var context = SpringApplication.run(IoCExample.class, args);
            B b = context.getBean(B.class); // new 안 쓰고 컨테이너에서 꺼냄
            b.action();
        }
    }
    ```
    
    - `new` 키워드로 객체를 만들지 않는다
    - `@Component`와 `@Autowired`를 쓰면 **스프링 IoC 컨테이너가 자동으로 생성 + 주입한다.**
    - `main`에서도 `new B()` 안 하고 → `context.getBean(B.class)`로 가져온다
    
- **생성자 주입 vs 수정자, 필드 주입 차이**
    
    ## 1. 생성자 주입 방법
    
    필요한 의존성을 생성자 파라미터로 주입받는 방식이다.
    
    생성자의 호출 시점에 1회 호출되는 것이 보장되며, 그렇기 때문에 주입받은 객체가 변하지 않거나, 반드시 객체의 주입이 필요한 경우에 강제하기 위해 사용할 수 있다.
    
    스프링 프레임워크에서는 생성자 주입 방식을 권장하고 있다.
    
    ### 자세한 설명
    
    - final 필드를 사용하여 런타임에 바뀌지 않으므로 불변성을 보장한다.
    - 생성자가 존재하지 않으면 객체를 만들 수 없으니 컴파일/런타임 초기(조기)에 문제를 발견할 수 있다.
    - 순환 참조를 조기에 검출할 수 있다 → 서로의 생성자에서 요구하면 바로 확인이 가능하다.
    - 스프링 없이도 생성자에 목 데이터를 넣어 단위 테스트가 가능하다
    - 생성자 1개만 있을 경우에 @Autowired 를 생략해도 주입이 가능하도록 편의성을 제공한다. (뭔 말인지 잘 모르겠음)
    
    ## 2. 수정자 주입 방법 (Setter 주입 방법)
    
    필드 값을 변경하는 Setter 메서드를 이용해서 의존관계를 주입하는 방식이다. Setter 주입은 생성자 주입과는 다르게 주입받는 객체가 변경될 가능성이 있는 경우에 사용한다. 
    
    ### 자세한 설명
    
    - 스프링이 객체를 생성한 후 set~() 메서드를 호출해서 필요한 빈을 넣어준다.
    - 런타임에 교체 가능하기 때문에 동적으로 빈을 바꿔끼우는 테스트나 설정 상황에서 유용하다.
    - 객체가 완전히 생성된 이후에 의존성이 들어오기 때문에 불변성 보장이 어렵다.
    - 주입을 빼먹어도 컴파일 단계에서 확인하기 어렵다.→이 또한 객체가 생성되고나서 의존성이 들어오기 때문
    
    일반적으로 수정자 주입은 필수 의존성이 아닌 선택적 의존성에만 한정적으로 사용하고, 그 외에는 생성자 주입을 표준으로 삼는 것이 가장 안전하다. 
    
    ## 3. 필드 주입 방법
    
    의존성을 클래스 내부의 필드에 바로 의존 관계를 주입하는 방법이다. 
    
    @Autowired 해서 주입한다. 가장 코드가 짧고 단순하지만, 테스트/유지보수 측면에서 단점이 많아, 실무에서는 권장되지 않는 방식이다. 
    
    ### 자세한 설명
    
    - 가장 단순해서 코드의 양이 적고 바로 의존성을 사용할 수 있다.
    - final 키워드를 사용할 수 없으므로 불변성을 보장할 수 없다.
    - 외부에서 접근이 불가능하다.
    - @Autowired 키워드 없이는 객체가 정상 동작하지 않는다. → 반드시 DI 프레임워크가 존재해야 한다.
    - 생성자 주입은 애플리케이션 시작 시점에서 바로 에러가 나지만, 필드 주입은 런타임에 늦게 드러나기도 한다.
    - 빠르게 의존성을 연결할 때 간단하게 주입할 수 있기 때문에 데모 프로젝트나 테스트용으로 적합하다.
    
    ## 생성자 주입을 사용해야 하는 이유
    
    1. 객체의 불변성 확보
    
    의존 관계의 변경이 필요한 상황은 거의 없지만, 다른 주입방식을 이요하면 불필요하게 수정의 가능성을 열어 두기 때문에 유지보수성을 떨어뜨린다. 따라서 생성자 주입을 통해 변경의 가능성을 배제하고 불변성을 보장하는 것이 좋다.
    
    1. 테스트 코드의 작성
    
    테스트가 특정 프레임워크에 의존하는 것은 좋지 못하다. 생성자 주입이 아닌 코드로 작성하면, 순수 자바 코드로 단위테스트를 작성하는 것이 어렵다.
    
    1. final 키워드 작성 및 Lombok과의 결합
    
    생성자 주입을 사용하면 필드 객체에 final 키워드를 사용할 수 있으며, 컴파일 시점에 누락된 의존성을 확인할 수 있다. 반면에 다른 주입방법들은 final 키워드를 사용할 수 없다. 
    
    또한 final 키워드를 붙이면 Lombok과 결합되어 코드를 간결하게 작성할 수 있다. 
    
    - Lombok이란?
        
        어노테이션 기반으로 코드를 자동완성 해주는 라이브러리이다. Lombok을 활용하면 Getter, Setter, Equlas, ToString 등과 같은 다양한 방면의 코드를 자동완성 시킬 수 있다. 
        
        장점
        
        - 어노테이션 기반의 코드 자동 생성을 통한 생산성 향상
        - 반복되는 코드 다이어트를 통한 가독성 및 유지보수성 향상
        - 빌더 패턴이나 로그 생성 등 다양한 방면으로 활용 가능
    1. 스프링에 비침투적인 코드 작성
    
    필드 주입을 하려면 @Autowired를 이용해야 하는데, 이것은 스프링이 제공하는 어노테이션이다. 따라서 사용하면 스프링 의존성이 침투하게 되는데, 프레임워크가 언제 바뀔지도 모르고, 민감한 정보에 관한 책임을 지는 코드에 스프링이 박혀버리면 좋지 않다. 
    
    1. 순환 참조 에러 방지
    
    생성자 주입을 사용하면 어플리케이션 구동 시점에 순환 참조 에러를 예방할 수 있다.  
    
    ### 예제
    
    ```java
    import org.springframework.stereotype.Component;
    
    @Component
    class Engine {
        public void start() {
            System.out.println("엔진 시동");
        }
    }
    ```
    
    공통 의존 대상
    
    1. 생성자 주입
    
    ```java
    import org.springframework.stereotype.Component;
    
    @Component
    class Car {
        private final Engine engine;  // 불변성 유지 가능
    
        // 생성자를 통해 의존성을 주입받음
        public Car(Engine engine) {
            this.engine = engine;
        }
    
        public void drive() {
            engine.start();
        }
    }
    ```
    
    1. 수정자 주입
    
    ```java
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Component;
    
    @Component
    class Car {
        private Engine engine;  // null일 수도 있음
    
        @Autowired
        public void setEngine(Engine engine) {
            this.engine = engine;
        }
    
        public void drive() {
            engine.start();
        }
    }
    ```
    
    1. 필드 주입
    
    ```java
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Component;
    
    @Component
    class Car {
        @Autowired
        private Engine engine;  // 직접 주입
    
        public void drive() {
            engine.start();
        }
    }
    ```
    
- **AOP**
    
    ## AOP (Aspect Oriented Programming) 이란?
    
    AOP는 관점 지향 프로그래밍. 즉, 핵심 비즈니스 로직과 부가 기능을 분리하여 모듈화 하는 개념이다. 여기서 모듈화란 어떤 공통된 로직이나 기능을 하나의 단위로 묶는 것을 말한다. 
    
    예) 
    
    핵심 로직: 실제 비즈니스 목적 (주문 생성, 결제 처리) 
    
    부가 기능: 로깅 보안 검사, 트랜잭션 관리, 성능 측정 
    
    부가기능은 보통 여러 클래스와 메서드에 중복해서 들어가게 되는데, AOP에서는 이 중복되는 부가 기능을 따로 모듈화해서 공통으로 적용할 수 있도록 한다.
    
    ### 핵심 개념
    
    1. Aspect (관점) : 공통 기능을 모듈화 한 것.
    2. Target : Advice가 적용될 객체
    3. Join Point : 공통 기능을 적용할 수 있는 지점(메서드 호출 시점, 객체 생성 시점 등)
    4. Pointcut : 어디에 공통 기능을 적용할 지 지정하는 규칙
    5. Advice : 실제 실행될 부가 기능 코드 
    6. Weaving : pointcut에 따라 Aspect를 핵심 코드에 적용하는 과정
    
    ### 스프링에서의 AOP
    
    스프링 컨테이너가 대상 빈 주변에 **프록시**를 만들어, 외부 호출이 들어올 때 Advice를 실행한 뒤 원본 메서드를 호출한다.
    
    - **JDK 동적 프록시**: 인터페이스가 있으면 기본 사용
    - **CGLIB**: 인터페이스 없으면 하위 클래스를 만들어 프록시 (final 메서드는 가로채기 불가)
    
    > 중요 제약
    > 
    > - **self-invocation 문제**: 같은 객체의 내부 메서드 호출은 프록시를 못 거침 → AOP 미적용
    >     - 해결: 메서드를 다른 빈으로 분리하기, 혹은 `@EnableAspectJAutoProxy(exposeProxy = true)` + `((Type) AopContext.currentProxy()).otherMethod()`
    >     
    > - **private/final 메서드**: 프록시로 가로채기 어려움 → 공개 메서드 중심 설계
    
    - 프록시란 무엇인가?
        
        **프록시 = 대리인(대신 처리해주는 객체)**
        
        - 원래 객체(Real Object)를 직접 쓰는 대신, **중간에 하나를 더 두고**
        - 클라이언트는 이 프록시를 통해서만 원본 객체를 호출
        - 프록시는 요청을 가로채서 **추가 기능(로깅, 보안, 트랜잭션 등)** 을 수행한 다음 원본 객체를 실행
        
        👉 즉, **“대리 객체”** 라고 생각하면 된다.
        
        스프링에서의 프록시
        스프링 AOP, @Transactional, @Cacheable 같은 기능이 동작할 때 다 **프록시 객체**를 사용한다. 
        
        ```java
        @Service
        class OrderService {
            @Transactional
            public void placeOrder() {
                // 주문 로직
            }
        }
        ```
        
        - 스프링이 `OrderService`를 그대로 쓰지 않고 → **프록시 객체**를 만들어서 컨테이너에 등록함
        - 우리가 `getBean(OrderService.class)`로 꺼내 쓰는 건 사실상 “프록시 객체”
        - 그 프록시는 `placeOrder()`가 호출될 때 **트랜잭션 시작/커밋/롤백** 같은 기능을 추가로 수행함
        
        프록시의 장점?
        
        - 핵심 로직(비즈니스)과 공통 기능(로깅/트랜잭션 등)을 **분리**
        - 기존 코드를 바꾸지 않고 **기능 추가** 가능 (OCP 원칙)
        - 런타임에 유연하게 동작 (스프링이 동적 프록시를 자동으로 생성)
    
- 서블릿
    
    ## 서블릿이란?
    
    클라이언트의 요청을 처리하고, 그 결과를 다시 클라이언트에게 전송하는 Servlet 클래스의 구현 규칙을 지킨 자바 프로그램이다.
    
    동적인 페이지를 가공하기 위해 웹 서버가 다른 곳에 도움을 요청한 후 가공된 페이지를 넘겨주게 되는데, 이때 서블릿을 사용하게 되면 웹페이지를 동적으로 생성하여 클라이언트에게 반환해줄 수 있다. 
    
    개발자는 서블릿이 요구하는 구현 규칙을 지키면서 서블릿을 정의해주면 요청과 응답을 쉽게 처리할 수 있게 되었다. 
    
    일반 자바 객체와의 차이점?
    
    서블릿은 main() 메서드로 직접 호출되지 않고, 서블릿 컨테이너에 의해 실행된다. 컨테이너가 web.xml을 읽고 서블릿 클래스를 클래스 로더에 등록하는 절차를 밟는다. 
    
    ### 서블릿 컨테이너란?
    
    구현되어있는 servlet 클래스의 규칙에 맞게 서블릿 객체를 생성, 초기화, 호출, 종료하는 생명 주기를 관리한다.
    
    서블릿 컨테이너는 클라이언트의 요청을 받고 응답할 수 있도록 웹 서버와 소켓으로 통신한다.
    
    대표적인 컨테이너로는 Tomcat이 있다.
    
    여기서 DI 컨테이너와 헷갈렸는데, 한마디로 정리하면
    
    서블릿 컨테이너 = 웹 요청을 관리하는 관리자
    
    DI 컨테이너 = 객체(Bean)를 관리하는 관리자
    
    서블릿 객체와 일반 Bean으로 선언되는 객체는 다르다. 
    
    서블릿은 HTTP 요청과 응답을 처리하고, DI 컨테이너는 객체를 생성/주입하고, 객체의 생애주기를 관리한다. 
    
    - WAS란?
        
        WAS는 웹 서버와 웹 애플리케이션 실행 환경을 합친 개념이다.
        
        클라이언트의 요청을 받아서 단순한 정적 파일만 주는 게 아니라, 스프링, JSP, servlet과 같은 동적인 로직을 실행해서 응답을 만들어 줄 수 있는 서버다. 
        
        ### 톰캣의 특징
        
        - **Apache Tomcat**은 **자바 기반**의 오픈소스 WAS
        - 자바 EE(Java Enterprise Edition) 표준 중 일부(서블릿, JSP)를 구현한 서버
        - 주요 역할:
            1. HTTP 요청을 받아 서블릿 컨테이너를 통해 서블릿 실행
            2. JSP를 서블릿으로 변환 후 실행
            3. 세션 관리, 멀티스레딩 관리
            4. Spring 같은 프레임워크가 올라갈 실행 환경 제공
        
         즉, 톰캣은 **서블릿 컨테이너 기능 + WAS 역할**을 동시에 수행한다. 
        
    
    스프링에서는 서블릿을 직접 다루지 않고 어노테이션 기반 컨트롤러로 요청/응답을 처리함
    
    DispatcherServlet이 중간에서 모든 흐름을 관리함.
    
    ```java
    @Controller
    public class HelloController {
    
        @GetMapping("/hello")
        public String hello(Model model) {
            model.addAttribute("username", "홍길동");
            return "hello"; // → ViewResolver가 hello.html 찾음
        }
    }
    ```
    
    - 클라이언트 `/hello` 요청
    - DispatcherServlet → HandlerMapping → HelloController 실행
    - Controller가 `model + "hello"` 반환
    - ViewResolver → `hello.html` 뷰 선택
    - HTML 렌더링 후 응답
    
    ### 1. 톰캣(Tomcat)
    
    - **서블릿 컨테이너(Servlet Container)**
    - 자바 웹 애플리케이션을 실행시켜주는 환경
    - HTTP 요청을 받아서 **등록된 서블릿에게 전달**하는 게 주 역할
    - “운영체제 + JVM 위에서 돌아가는 서버 프로그램”이라고 보면 됨
    
    👉 톰캣은 “운영자” 같은 존재.
    
    ---
    
    ### 2. 디스패처서블릿(DispatcherServlet)
    
    - **스프링 프레임워크가 제공하는 하나의 서블릿 클래스**
    - `org.springframework.web.servlet.DispatcherServlet`
    - 모든 요청을 받아서 → 어떤 컨트롤러를 호출할지 결정하고 → 응답을 뷰로 돌려주는 **프론트 컨트롤러(Front Controller)** 역할
    
    👉 DispatcherServlet은 “톰캣에 등록된 서블릿 중 하나”일 뿐.
    
    ## 동작 순서 정리
    
    1. 브라우저가 `http://localhost:8080/hello` 요청
    2. **톰캣** (서블릿 컨테이너)이 요청을 받음
    3. 해당 URL을 처리할 서블릿으로 매핑 → `DispatcherServlet` 호출
    4. **DispatcherServlet** 이 내부에서
        - HandlerMapping → Controller 찾기
        - Controller 실행 → Model 반환
        - ViewResolver → HTML/JSON 렌더링
    5. 최종 응답을 톰캣이 브라우저로 전달
    
    톰캣이 Dispacherservlet을 실행시키는 구조임
    
- Optional 클래스
    
    ## Optional이란?
    
    Optional 클래스는 null이 올 수 있는 값을 감싸는 Wrapper 클래스로, 참조하더라도 NPE (NullPointerException)가 발생하지 않도록 도와준다. 
    
    Optional 클래스는 값이 null이라도 바로 NPE가 발생하지 않으며, 각종 메서드를 제공한다.
    
    값이 없을 수도 있다는 사실을 명확하게 표현할 수 있게 해준다.
    
    - NPE (NullPointerException)란?
        - **자바에서 가장 흔히 발생하는 런타임 예외** 중 하나
        - 말 그대로 (아무것도 참조하지 않는 상태)인 객체에 접근하려고 할 때 발생한다
        - 즉, 객체가 메모리에 없는데 마치 있는 것처럼 사용하려고 하면 터진다.
        
        언제 발생할까?
        
        1. 객체를 초기화하지 않고 사용했을 때
        2. 메서드 리턴 값이 null인데 체크하지 않고 사용했을 때
        3. 배열이나 컬렉션 안의 요소가 null일 때
    
    Optional은 메서드가 null을 반환할 수도 있을 때 쓰면 좋다.
    
    ## 주요 메서드
    
    - `Optional.of(value)`
        
        → 값이 절대 `null`이 아닐 때 생성
        
    - `Optional.ofNullable(value)`
        
        → 값이 `null`일 수도 있을 때 생성
        
    - `Optional.empty()`
        
        → 비어있는 Optional 생성
        
    - `isPresent()`
        
        → 값이 있는지 여부 확인
        
    - `ifPresent(Consumer)`
        
        → 값이 존재할 때만 실행
        
    - `orElse(defaultValue)`
        
        → 값이 없으면 기본값 반환
        
    - `orElseGet(Supplier)`
        
        → 값이 없으면 Supplier 실행
        
    - `orElseThrow()`
        
        → 값이 없으면 예외 발생
        
    
    ### 예제
    
    1. 옵셔널 생성
    
    ```java
    import java.util.Optional;
    
    public class OptionalExample {
        public static void main(String[] args) {
            // 값이 null이 아닌 경우
            Optional<String> opt1 = Optional.of("Hello");
            System.out.println(opt1.get()); // Hello
    
            // 값이 null일 수도 있는 경우
            Optional<String> opt2 = Optional.ofNullable(null);
            System.out.println(opt2.isPresent()); // false
    
            // 비어있는 Optional
            Optional<String> opt3 = Optional.empty();
            System.out.println(opt3.isPresent()); // false
        }
    }
    ```
    
    1. 값 꺼내기 (`orElse`, `orElseGet`, `orElseThrow`)
    
    ```java
    public class OptionalOrElseExample {
        public static void main(String[] args) {
            Optional<String> opt = Optional.ofNullable(null);
    
            String value1 = opt.orElse("기본값");
            System.out.println(value1); // 기본값
    
            String value2 = opt.orElseGet(() -> "Supplier 기본값");
            System.out.println(value2); // Supplier 기본값
    
            try {
                String value3 = opt.orElseThrow(() -> new IllegalArgumentException("값이 없음!"));
                System.out.println(value3);
            } catch (Exception e) {
                System.out.println(e.getMessage()); // 값이 없음!
            }
        }
    }
    
    ```
    
    1. ifPresent 활용
    
    ```java
    public class OptionalIfPresentExample {
        public static void main(String[] args) {
            Optional<String> opt = Optional.of("hello");
    
            opt.ifPresent(val -> System.out.println("값 있음: " + val));
            // 출력: 값 있음: hello
        }
    }
    ```
    
- Stream API
    
    ## Stream API란?
    
    데이터를 선언적이고 함수형 스타일로 처리할 수 있게 해주는 API이다.
    
    여기서 Stream은 데이터를 담는 저장소가 아니라 데이터를 처리하는 흐름이다. 
    
    한번 생성된 stream은 한 번만 소비할 수 있고, 재사용하려면 새로 만들어야 한다. 
    
    ### 특징
    
    1. 원본의 데이터를 변경하지 않는다.
    
    Stream API는 원본의 데이터를 조회해서 원본의 데이터가 아닌 별도의 요소들로 Stream을 생성한다. 그렇기 때문에 원본의 데이터로부터 읽기만 할 뿐이고, 정렬이나 필터링 작업은 별도의 Stream 요소들에서 한다.
    
    1. 일회용이다
    
    Stream API는 일회용이기 때문에 한 번 사용이 끝나면 재사용이 불가능하다. Stream이 또 필요한 경우에는 Stream을 다시 생성해 주여야 한다. 만약 닫힌 Stream을 다시 사용한다면 IlligalStrateExeption이 발생하게 된다.
    
    1. 내부 반복으로 작업을 처리한다.
    
    Stream을 이용하면 코드가 간결해지는 이유 중 하나는 내부 반복 때문이다. 기존에는 반복문을 사용하기 위해서 for이나 while같은 문법을 사용해야 했지만, stream에서는 그러한 반복 문법을 메서드 내부에 숨기고 있어서 보다 간결한 코드 작성이 가능하다.  
    
    ### Stream API의 세 가지 단계
    
    1. 생성하기
    - Stream 객체를 생성하는 단계이다.  재사용이 불가능하므로 닫히면 다시 생성해주어야 한다.
    1. 가공하기 
    - 원본의 데이터를 별도의 데이터로 가공하기 위한 중간 연산이다.
    - 연산 결과를 Stream으로 다시 반환하기 때문에 연속해서 중간 연산을 이어갈 수 있다.
    1. 결과 만들기
    - 가공된 데이터로부터 원하는 결과를 만들기 위한 최종 연산이다.
    - Stream 요소들을 소모하면서 연산이 수행되기 때문에 1번만 처리가 가능하다.
    
    Stream 연산들은 매개변수로 함수형 인터페이스를 받는다.
    
    - 함수형 인터페이스란?
        
        함수를 1급 객체처럼 다룰 수 있게 해 주는 어노테이션으로, 인터페이스에 선언하여 단 하나의 추상 메서드만을 갖도록 제한하는 역할을 한다. 
        
        1급 객체는 뭐지?
        
        프로그래밍 언어에서 어떤 요소(값, 함수, 객체 등)가 **다른 값들과 동등하게 취급될 수 있는 것**을 의미한다.
        
        즉, **변수에 할당할 수 있고, 함수의 인자로 넘길 수 있고, 함수의 반환값으로 돌려줄 수 있다면** 그 언어에서 그것은 "1급 객체"라고 부른다.
        
    
    일반적으로 스프링 자체가 Stream API를 강제하지는 않지만, 많은 스프링 프로젝트에서 Stream API가 쓰이는것임!
    
    보통 JPA/DB 조회 결과 처리에서 많이 쓰임.
    
    람다식이랑 Stream API는 짝꿍인데, Stream API는 전부 함수형 인터페이스를 받도록 설계가 되어있음,, 근데 이 함수형 인터페이스는 전부 람다식으로 구현이 가능함!
    
    예제
    
    1. 일반적인 for 문
    
    ```java
    public class ForLoopExample {
        public static void main(String[] args) {
            List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
    
            for (Integer n : numbers) {
                if (n % 2 == 0) {                // 짝수 조건
                    int squared = n * n;         // 제곱
                    System.out.println(squared); // 출력
                }
            }
        }
    }
    ```
    
    1. Stream + 람다를 사용한 방식
    
    ```java
    public class StreamLambdaExample {
        public static void main(String[] args) {
            List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
    
            numbers.stream()
                   .filter(n -> n % 2 == 0)     // 조건: 짝수만
                   .map(n -> n * n)             // 변환: 제곱
                   .forEach(System.out::println); // 출력
        }
    }
    ```