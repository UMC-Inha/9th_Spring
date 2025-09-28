# 🌟 **SOLID**

**SOLID 원칙**이란 객체지향 설계에서 지켜줘야 할 5개의 소프트웨어 개발 원칙을 말한다.

- **S**RP(Single Responsibility Principle): 단일 책임 원칙
- **O**CP(Open Closed Principle): 개방 폐쇄 원칙
- **L**SP(Liskov Substitution Principle): 리스코프 치환 원칙
- **I**SP(Interface Segregation Principle): 인터페이스 분리 원칙
- **D**IP(Dependency Inversion Principle): 의존 역전 원칙

---

## 🏗 객체지향의 네 가지 특징

SOLID 설계 원칙이 객체지향의 네 가지 특징(추상화, 상속, 다형성, 캡슐화)과 닮아 있어서, 먼저 객체지향의 네 가지 특징을 설명해보려 한다.

- **추상화(Abstraction)**: 클래스들의 공통적인 특성들을 묶어 표현하는 것
- **캡슐화(Encapsulation)**: 데이터와 코드의 형태를 외부로부터 알 수 없게 하고, 데이터의 구조와 역할, 기능을 하나의 캡슐 형태로 만드는 방법
- **상속성(Inheritance)**: 부모 클래스에 정의된 변수 및 메서드를 자식 클래스에서 상속받아 사용하는 것
- **다형성(Polymorphism)**: 파생클래스 객체가 기본클래스가 가지고 있는 멤버함수를 커스터마이징을 하고 싶은 경우, 오버라이딩 및 오버로딩 등을 통하여 실현 가능

---

## 🎨 디자인 패턴

이러한 SOLID 원칙과 OOP의 4가지 특징들은 모두 디자인 패턴을 위한 필수 지식들이다.

※ **디자인 패턴**: OOP와 SOLID 원칙을 바탕으로, 선배 개발자들이 반복적으로 등장하는 문제를 해결하기 위해 정리해놓은 구체적인 설계 방법(템플릿)

---

## 📖 SOLID 원칙 설명

### 1️⃣ 단일 책임 원칙
- 각 클래스는 한 가지 책임(역할)만 담당하고, 변경해야 하는 이유는 단 하나여야 한다.
- 하나의 클래스는 하나의 역할을 담당하여 하나의 책임을 수행하는데 집중되어 있어야 한다는 의미
- 클래스의 설계가 변경되어야 할 경우는 한 클래스가 둘 이상의 역할을 책임질 경우이며, 이때 역할별로 클래스를 각각 설계해주는 것이 바람직함
- 즉, 클래스는 그 책임을 완전히 캡슐화 하는것

---

### 2️⃣ 개방 폐쇄 원칙
- 확장에는 열려 있어야 하고, 수정에는 닫혀 있어야 한다.
- 기능 추가 요청이 오면 클래스를 확장을 통해 손쉽게 구현하면서, 확장에 따른 클래스 수정은 최소화 하도록 프로그램을 작성해야 하는 설계 기법
- 추상화 사용을 통한 관계 구축을 권장한다는 의미
- 다형성과 확장을 가능케 하는 객체지향의 장점을 극대화

---

### 3️⃣ 리스코프 치환 원칙
- 상위 클래스의 객체는 언제나 자신의 하위 클래스 객체로 대체될 수 있어야 한다.
- 상위 클래스의 객체가 들어갈 자리에 하위 클래스의 객체를 넣어도 문제없이 잘 작동해야 함
- 상속과 재정의의 중요성을 강조함
- 상속 구조를 만들 때 클래스에 추상 메소드가 포함되어 있는 경우 반드시 클래스를 추상 클래스로 작성해야 함

---

### 4️⃣ 인터페이스 분리 원칙
- 클라이언트는 자신이 사용하지 않는 메서드와 의존 관계를 맺으면 안된다.
- 다수의 클라이언트가 일반적인 인터페이스 하나를 같이 사용하는 것보다 각각의 클라이언트가 필요한 대로 여러 개의 구체적인 인터페이스로 분리해 사용하는 것이 더 나음
- 각각의 클라이언트가 필요로 하는 메서드 군이 존재할 때 인터페이스를 분리

---

### 5️⃣ 의존 역전 원칙
- 상위 모듈이 하위 모듈에 의존해서는 안되며, 양쪽 모두 구체 클래스가 아닌 추상 클래스에 의존해야 한다.
- 서로 관계가 의존적이라는 것은 결합도가 높다는 것이고 이는 클래스 설계에서 좋은 관계가 아님
- 의존 역전 원칙에서는 구체 클래스에 의존하도록 클래스 설계를 하지 않고 자주 바뀌는 클래스를 상속 구조로 만들어 추상 클래스에 의존하도록 설계

---

## 🌟 SOLID의 장점

코드를 확장하고 유지 보수 관리하기가 더 쉬워지며, 불필요한 복잡성을 제거해 리팩토링에 소요되는 시간을 줄임으로써 프로젝트 개발의 생산성을 높일 수 있다.

※ **리팩토링**: 소프트웨어의 외부 동작은 그대로 유지하면서 내부 구조를 개선하는 과정

---

# 🌱 DI

**DI**는 Dependency Injection의 줄임말이고 해석하면 의존성 주입이다.

의존성 주입이란 스프링 프레임워크에서, 직접 객체를 생성하거나 참조하지 않고 외부에서 주입받는 것을 의미한다.

---

## 🔗 의존성이란?

먼저 의존성에 대해 설명하자면 의존성이란 **A클래스가 B클래스를 하용해야만 동작하는 관계**이다.

```java
class Engine {
    public void start() {
        System.out.println("엔진이 켜졌습니다!");
    }
}

class Car {
    private Engine engine = new Engine(); // Car는 Engine에 의존함

    public void drive() {
        engine.start();  // Car가 혼자서는 못 움직이고, Engine에 기대고 있음
        System.out.println("자동차가 달립니다!");
    }
}
```
즉 Car는 Engine 없이는 움직일 수 없다.
즉, Car는 Engine에 의존한다.

---

## 🌟 DI를 쓰는 이유

```java
// 추상화(인터페이스) 도입
interface Engine {
    void start();
}

class GasEngine implements Engine {
    public void start() {
        System.out.println("가솔린 엔진 시동!");
    }
}

class ElectricEngine implements Engine {
    public void start() {
        System.out.println("전기 엔진 시동!");
    }
}

class Car {
    private Engine engine;

    // ✅ 생성자 주입 (DI)
    public Car(Engine engine) {
        this.engine = engine;
    }

    public void drive() {
        engine.start();
        System.out.println("자동차가 달립니다!");
    }
}

public class Main {
    public static void main(String[] args) {
        Engine gas = new GasEngine();
        Engine electric = new ElectricEngine();

        Car car1 = new Car(gas);       // 가솔린 엔진 차
        Car car2 = new Car(electric); // 전기 엔진 차

        car1.drive();
        car2.drive();
    }
}
```
Car는 이제 Engine이라는 추상화(인터페이스)에만 의존하고,
실제 어떤 엔진을 쓸지는 외부(Main)에서 주입해 줌.

이게 바로 **의존성 주입**이다.

---

## ⚙️ DI 방식(주입 유형)

### 1️⃣ 생성자 주입

```java
@Service
public class MemberService {

    private final MemberRepository memberRepository;

    @Autowired // @Autowired 어노테이션은 생성자의 수가 한개라면 생략해도 된다. -> 즉 여기선 없어도 된다!
    public MemberService(final MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
}
```
---
### 2️⃣ Setter 주입

```java
@Service
public class MemberService {

    private MemberRepository memberRepository;

    @Autowired
    public void setMemberRepository(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
}
```
---
### 3️⃣ 필드 주입

```java
@Service
public class MemberService {

    @Autowired
    private MemberRepository memberRepository;
}
```
---
## ✨ 가장 권장되는 방식은 생성자 주입이다.

---

## 🌸 의존성 주입과 스프링
스프링은 의존성 주입을 통해 애플리케이션 내에서 객체들 간의 관계를 자동으로 관리한다.

개발자는 직접 객체를 생성하거나 의존성을 관리할 필요 없이,
스프링이 제공하는 설정만으로 객체의 생성과 주입을 처리할 수 있다.
---

# 🔄 IoC

**IoC**는 Inversion of Control을 줄인 표현으로 직역하면 제어의 역전이다.  
→ 객체가 스스로 제어권을 갖는 것이 아니라 **외부 컨테이너가 객체의 생성과 관리, 의존성 주입을 담당**하는 구조를 말한다.

---

## 💻 코드로 비교

### 1️⃣ IoC 없을 때

```java
class Engine {
    void start() { System.out.println("엔진 켜짐!"); }
}

class Car {
    private Engine engine;

    public Car() {
        this.engine = new Engine(); // Car가 Engine을 직접 생성 ❌
    }

    void drive() {
        engine.start();
        System.out.println("차 달림!");
    }
}

public class Main {
    public static void main(String[] args) {
        Car car = new Car();
        car.drive();
    }
}
```

### 2️⃣ IoC 있을 때

```java
interface Engine { void start(); }

class GasEngine implements Engine {
    public void start() { System.out.println("가솔린 엔진 켜짐!"); }
}

class Car {
    private Engine engine;

    public Car(Engine engine) { // 생성자 주입
        this.engine = engine;
    }

    void drive() {
        engine.start();
        System.out.println("차 달림!");
    }
}

public class Main {
    public static void main(String[] args) {
        Engine engine = new GasEngine();
        Car car = new Car(engine); // 외부에서 Engine을 주입 ✅
        car.drive();
    }
}
```
---
## 📊 IoC 비교표
| 구분       | IoC 없을 때 | IoC 있을 때 |
| -------- | -------- | -------- |
| 객체 생성 책임 | 직접(new)  | 외부 컨테이너  |
| 의존성 관리   | 직접 연결    | 컨테이너가 주입 |
| 유연성      | 낮음       | 높음       |
| 테스트      | 어려움      | 쉽고 유연    |

---

## ⚡ 결합도와 유연성
IoC 없을 때 유연성이 낮고 결합도가 높은 이유
만약 engine을 ElectricEngine으로 바꾸고 싶을 때
```java
public Car() {
    this.engine = new ElectricEngine(); // Car 코드 수정 필수
}

```
Car 코드를 직접 고쳐야 함


## 위에 나온 DI가 IoC를 구현하기 위해 사용하는 방법 중 하나이다.

---

# 🧩 생성자 주입 vs 세터 주입 vs 필드 주입

| 구분 | 생성자 주입 | 세터 주입 | 필드 주입 |
| --- | --- | --- | --- |
| 주입 시점 | 객체 생성 시 | 객체 생성 후 | 객체 생성 후 |
| 필드 final | 가능 ✅ | 불가 ❌ | 불가 ❌ |
| 테스트 용이성 | 매우 쉬움 ✅ | 쉬움 ⚖️ | 어려움 ❌ |
| 선택적 의존성 | 불가 ❌ | 가능 ✅ | 가능 ✅ |
| 순환 참조 처리 | 발견 시 실패 ✅ | 세팅 시 해결 가능 ⚖️ | 세팅 시 해결 가능 ⚖️ |
| 장점 | 안전, 명시적, 테스트 용이 | 선택적 주입 가능, 순환참조 해결 유리 | 코드 간결 |
| 단점 | 생성자 길어짐 | 불완전 객체 존재 가능 | 테스트 어려움, 의존성 불명확 |

---

# 🔹 AOP

**AOP**는 Aspect Oriented Programming을 줄인 표현으로 직역하면 관점 지향 프로그래밍이다.

→ 프로그래밍에 대한 관심을 핵심 관점, 부가 관점으로 나누어서 관심 기준으로 모듈화하는 것을 의미

![img.png](img.png)
위와 같이 흩어진 관심사를 **Aspect**로 모듈화하고 핵심적인 비즈니스 로직에서 분리하여 재사용하겠다는 것이 AOP의 취지다.

---

## 🔹 AOP 주요 개념

- **Aspect**: 위에서 설명한 흩어진 관심사를 모듈화 한 것, 주로 부가기능을 모듈화함
- **Target**: Aspect를 적용하는 곳 (클래스, 메서드 등)
- **Advice**: 실질적으로 어떤 일을 해야할 지에 대한 것, 실질적인 부가기능을 담은 구현체
- **JointPoint**: Advice가 적용될 위치, 끼어들 수 있는 지점, 매서드 진입 지점, 생성자 호출 시점, 필드에서 값을 꺼내올 때 등 다양한 시점에 적용 가능
- **PointCut**: JointPoint의 상세한 스펙을 정의한 것. ‘A란 메서드의 진입 시점에 호출한 것’과 같이 더욱 구체적으로 Advice가 실행될 지점을 정할 수 있음

---

## 🔹 Spring에서의 AOP

- Spring AOP는 **프록시 기반**으로 동작
- 프록시는 **스프링이 관리하는 빈**에만 생성됨 → 따라서 **AOP는 스프링 빈에만 적용 가능**
- 직접 `new`로 만든 객체나 스프링 컨테이너에 등록되지 않은 객체에는 적용 불가

※ 프록시는 실제 객체를 감싸고, 호출을 가로채서 부가 기능 실행하는 대리 객체

---

## 🔹 Spring AOP 실제 사용 사례

### 1️⃣ 트랜잭션 관리
- 핵심 로직에선 **비즈니스 기능**만 구현
- 트랜잭션 시작/커밋/롤백 같은 반복 코드를 **AOP가 프록시를 통해 자동 처리**

### 2️⃣ 성능 모니터링
- 특정 메서드 실행 시간 측정
- 핵심 로직 코드는 그대로, 부가 기능만 분리

### 3️⃣ 예외 로깅
- 공통적으로 발생하는 예외를 **AOP로 캡처**
- 서비스 코드에서는 로직에만 집중

---

## 🔹 AOP의 장단점

### ✅ 장점
- **코드 중복 감소**: 횡단 관심사를 한 곳에서 관리함으로써, 여러 곳에서 중복되는 코드를 줄일 수 있다.
- **유지보수성 향상**: 비즈니스 로직과 부가적인 관심사를 분리함으로써, 코드의 가독성과 유지보수성이 증가한다.
- **비즈니스 로직과 부가적인 관심사의 분리**: 로깅, 보안, 트랜잭션 관리 등과 같은 부가적인 관심사를 비즈니스 로직에서 분리하여 관리할 수 있다.

### ❌ 단점
- **코드 흐름의 복잡성**: AOP의 남용은 코드의 실행 흐름을 이해하기 어렵게 만들 수 있다.
- **성능에 미미한 영향**: AOP는 성능에 미미한 영향을 줄 수 있지만, 대부분의 경우 이는 크게 문제 되지 않는다.

---

# 🌐 서블릿(Servlet)

**서블릿**은 자바 어플리케이션에서 클라이언트의 요청을 처리하고 응답을 반환하는 역할을 하는 하나의 **클래스**이다.

---

### 🔹 서블릿 주요 기능

- **HTML 폼에 사용자가 입력할 수 있다.**
    - 사용자가 입력한 데이터가 폼을 통해 서버로 전송된다.
    - 사용자가 아이디와 비밀번호를 입력하고 확인 버튼을 누르면 HTTP 요청으로 **서블릿**에게 데이터가 전송되고, 서블릿은 데이터를 사용해 로그인 로직을 처리한다.

- **데이터베이스에 쿼리를 날린다.**
    - 서블릿은 외부 서버 중 하나인 데이터베이스와의 연결을 담당하기도 한다.
    - JDBC API를 사용해 `Connection` 객체를 생성하고 `PreparedStatement` 객체 등을 사용해서 SQL 쿼리를 데이터베이스에 전송한다.

- **동적 웹 환경을 구성한다.**
    - 서버 측에서 데이터를 기반으로 HTML을 생성하여 클라이언트에게 전송하면 클라이언트에서 동적인 웹 페이지를 확인할 수 있다.

---

### 🔹 서블릿 컨테이너(Servlet Container)

서블릿 컨테이너는 서블릿들의 생성, 실행, 삭제를 담당한다.  
서버가 요청을 전달받았을 때 서블릿 컨테이너에게 요청을 전달하고, 서블릿 컨테이너가 요청을 처리할 수 있는 서블릿에게 요청을 위임하는 구조이다.  
그래서 서블릿 컨테이너는 **서블릿 엔진(Servlet Engine)**이라고 불리기도 한다.

#### **서블릿 컨테이너의 역할**

1. **웹서버와의 통신 지원**
    - 서블릿 컨테이너는 서블릿과 웹서버가 손쉽게 통신할 수 있게 해줌
    - 일반적으로 소켓을 만들고 listen, accept 등을 해야하지만, 서블릿 컨테이너는 이러한 기능을 API로 제공하여 복잡한 과정을 생략할 수 있음
    - 개발자가 서블릿에 구현해야 할 비즈니스 로직에 집중 가능

2. **서블릿 생명주기(Life Cycle) 관리**
    - 서블릿 컨테이너는 서블릿의 탄생과 죽음을 관리
    - 서블릿 클래스를 로딩하여 인스턴스화, 초기화 메소드 호출
    - 요청이 들어오면 적절한 서블릿 메소드 호출
    - 서블릿 종료 시 Garbage Collection 지원

3. **멀티쓰레드 지원 및 관리**
    - 요청이 올 때마다 새로운 자바 쓰레드를 생성하고, HTTP 서비스 메서드 실행 후 쓰레드 종료
    - 개발자는 쓰레드 안정성 걱정 없이 멀티쓰레드 지원 가능

4. **선언적인 보안 관리**
    - 개발자는 보안 관련 내용을 서블릿 또는 자바 클래스에 구현할 필요 없음
    - 보안관리는 XML 배포 서술자에 기록
    - 자바 소스 코드를 수정하지 않고도 보안관리 가능

---

### 🔹 서블릿 동작 과정

클라이언트에서 HTTP 요청을 보내고 서블릿에서 받아 처리하여 응답을 보내는 과정:

1. 클라이언트는 웹 서버에게 HTTP 요청을 보낸다.
2. 웹 서버는 서블릿 컨테이너에게 요청을 위임하고, 서블릿 컨테이너는 `HttpServletRequest`와 `HttpServletResponse` 객체를 새로 생성한다.
3. 이 객체가 서블릿에게 전달되고, 서블릿은 요청의 HTTP Method에 해당되는 적절한 메서드에서 요청을 처리한다.
4. 응답값이 `HttpServletResponse`에 담겨 서블릿 컨테이너에게 전달된다.
    - 서블릿 컨테이너는 이 객체에서 응답 데이터와 헤더, 상태 코드를 추출하여 클라이언트에게 HTTP 응답을 보낸다.

---

# 🟢 Optional 클래스

**Optional은 Java8 버전부터 도입되었으며, ‘값이 없는 경우’를 표현하기 위한 용도로 사용되는 클래스**

Java8 전에는 Optional이라는 클래스가 없었으므로, 메서드의 리턴 타입으로 ‘결과 없음’을 표현하기 위한 용도로 null이 사용되었다.

그러나 객체 값이 null인 경우에, null 체크 없이 해당 객체의 메서드를 호출하거나 하는 등, 부주의하게 사용하는 경우에 **‘NPE(NullPointerException)’**가 발생하면서 프로그램이 죽어버리는 상황이 발생한다.

**이러한 상황을 방지하고 안전한 코딩을 위해 Optional이 만들어졌다.**

---

## 🔹 Java Optional 사용 이점

1. 빈 결과를 명확히 반환할 수 있다.
2. null을 반환하는 메서드보다 오류 가능성이 적다.
3. null일 때 예외를 던지는 메서드보다 더 사용이 용이한 코드를 만들 수 있다.

---

## 🔹 Optional 기본 문법

#### 1️⃣ Optional 생성 방법
Optional에서 제공하는 static 메서드를 사용하여 Optional을 만들어 낼 수 있다.

```java
Optional<String> opt1 = Optional.empty(); // 값이 비어있는 Optional 생성
Optional<String> opt2 = Optional.of("Hello"); // "Hello" 값이 담긴 Optional 생성
String temp = null;
Optional<String> opt3 = Optional.ofNullable(temp); // temp가 null일 수도 있는 상황에서 사용
```
주의: of()는 null 인자 불가. null일 가능성이 있는 값은 반드시 ofNullable() 사용.

#### 2️⃣ Optional 주요 메서드

- **isPresent()**: 값이 존재하면 `true`, 없으면 `false`
- **isEmpty()**: 값이 비어있으면 `true`, 존재하면 `false`
- **get()**: Optional이 감싸고 있는 실제 값을 반환. 값이 없으면 `NoSuchElementException` 발생
> `isPresent()` 또는 `isEmpty()`로 값 존재 여부 체크 후 `get()` 사용 권장

- **ifPresent() / ifPresentOrElse()**  
  값이 있는 경우 처리 함수를 직접 람다로 지정 가능하며, 값이 없을 때도 처리 가능

```java
Optional<String> opt = Optional.of("Hello");

opt.ifPresentOrElse(
    value -> System.out.println(value.toUpperCase()), // 값 있을 때
    () -> System.out.println("값 없음")               // 값 없을 때
);
```
- **orElse(), orElseGet(), orElseThrow()**  
```java
Optional<String> optionalString = Optional.of("Hello");

String value = optionalString.orElse("Default Value"); // 대체값 지정
String result = optionalString.orElseGet(() -> "Value from Supplier"); // 람다 대체값
String str = emptyOptional.orElseThrow(() -> new RuntimeException("Optional is empty")); // 값 없을 때 예외
```
- **or()**: 여러 Optional 중 첫 번째로 값이 있는 걸 선택
```java
Optional<String> firstOptional = Optional.empty();
Optional<String> secondOptional = Optional.of("Hello, world!");

String result = firstOptional.or(() -> secondOptional).orElse("Default Value");
System.out.println("Result: " + result);
```

- **filter()**  
```java
Optional<String> optionalString = Optional.of("Hello World");

optionalString.filter(s -> s.length() >= 5)
        .ifPresent(s -> System.out.println("Filtered Value: " + s));
```

- **map()**
```java
Optional<String> optionalString = Optional.of("Hello World");

Optional<String> upperCaseOptional = optionalString.map(String::toUpperCase);
upperCaseOptional.ifPresent(s -> System.out.println("Uppercase Value: " + s));
```

- **flatMap()**: map()과 달리 람다 함수 반환 값이 Optional 객체일 때 사용
```java
Optional<String> optionalString = Optional.of("Java Optional");
Optional<String> flatMappedOptional = optionalString.flatMap(s -> Optional.of(s + " Tutorial"));

flatMappedOptional.ifPresent(s -> System.out.println("FlatMapped Value: " + s));
```

---

## 🔹 Optional 사용 시 주의사항

- Optional 변수에 `null`을 할당하지 않는다.
- Optional을 필드 타입으로 사용하지 않는다.
- 메서드 인자로 Optional을 사용하지 않는다.
- 빈 컬렉션이나 배열을 나타낼 때 Optional을 사용하지 않는다.
- 원시 타입 값을 다룰 때는 원시 타입용 Optional(`OptionalInt`, `OptionalLong`, `OptionalDouble`)을 사용한다.

---

## 🔹 Optional 대표 사용처: Spring Data JPA

```java
public interface CrudRepository<T, ID> extends Repository<T, ID> {
    Optional<T> findById(ID id);
}
```
```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
    
    public void getUserById(Long userId) {

        Optional<User> userOptional = userRepository.findById(userId);
        
        if (userOptional.isPresent()) {

            User user = userOptional.get();
            System.out.println("User found: " + user.getName());

        } else {
            System.out.println("User not found with ID: " + userId);
        }
    }
}
```

---

# 🔹 Stream API

**스트림(Stream) API**는 람다식(Lambda Expression)을 이용한 기술 중 하나로,  
**데이터 소스(컬렉션, 배열, 난수, 파일 등…)를 조작 및 가공, 변환하여 원하는 값으로 반환해주는 인터페이스**를 의미한다.

> **람다식(Lambda Expression)이란?**  
> 함수를 하나의 식으로 표현한 함수형 인터페이스로, 메소드의 이름이 없기 때문에  
> **익명 함수(Anonymous Function)의 한 종류**라고도 한다.

---

## 🔹 Stream 타입별 객체

| **분류** | **타입** | **Stream Type** |
| --- | --- | --- |
| 정수형 | byte | Stream<Byte> |
| 정수형 | short | Stream<Short> |
| 정수형 | int | **IntStream** |
| 정수형 | long | **LongStream** |
| 문자형 | char | Stream<Character> |
| 실수형 | float | Stream<Float> |
| 실수형 | double | **DoubleStream** |
| 문자열 | String | **Stream<String>** |
| 논리형 | boolean | Stream<Boolean> |

---

## 🔹 Stream의 특징

1. **Stream은 원본 데이터를 변경하지 않는다.**  
   💡 Stream API는 원본 데이터를 조회하여 **별도의 Stream 객체로 생성**한다.

2. **Stream은 재사용이 불가능하여 일회용이다.**  
   💡 이미 사용되어 닫힌 Stream은 **재사용 불가**하며, `java.lang.IllegalStateException` 오류가 발생한다.  
   새 Stream 객체를 생성해야 한다.

3. **Stream은 내부 반복으로 작업을 처리한다.**  
   💡 Stream 내에서 반복문을 내부적으로 처리하므로 **간결한 코드 작성 가능**.

4. **병렬 처리 지원**  
   💡 `.parallelStream()` 사용 시 멀티코어 활용 가능.

5. **종료 연산이 필수적이다.**  
   💡 `filter()`, `map()`과 같은 **중간 연산**은 데이터를 변환하지만,  
   최종 결과를 얻으려면 **종료 연산**이 수행되어야 한다.

---

## 🔹 Stream 처리 과정

1. **Stream 생성 (Source)**  
   `stream()` 또는 `Arrays.stream()` 등으로 Stream 생성.

2. **중간 연산 (Intermediate Operations)**  
   데이터 변환, 필터링, 정렬 등 수행 후 **새로운 Stream 반환**.  
   여러 중간 연산을 체인처럼 연결 가능.  
   **지연 실행(Lazy Evaluation)**으로 최종 연산이 호출될 때 실제로 실행됨.  
   예: `filter()`, `map()`, `sorted()`, `distinct()`, `limit()`, `skip()`

3. **최종 연산 (Terminal Operation)**  
   결과 반환 연산. 최종 연산이 호출될 때 중간 연산이 실행됨.  
   예: `collect()`, `forEach()`, `reduce()`, `count()`

---

**💡 사용 예시**
```java
객체.stream()
     .중간연산()
     .최종연산();
```
**한 줄의 코드로 Stream 가능**
```java
names.stream() //생성
          .filter(name -> name.length() >= 3)   // 조건 필터링
          .map(String::toUpperCase)             // 대문자 변환
          .sorted()                             // 정렬
          .forEach(System.out::println);        // 출력
     // 출력
```