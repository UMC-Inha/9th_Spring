## 제어의 역전 IoC(Inversion of Control)

**기존 방식 (제어 흐름 직접 관리)**

```java
public class OrderService {
    private FixDiscountPolicy discountPolicy = new FixDiscountPolicy(); 

    public int createOrder(int price) {
        return discountPolicy.discount(price);
    }
}

class FixDiscountPolicy {
    public int discount(int price) {
        return 1000; // 고정 1000원 할인
    }
}
```

위 예시에서 `OrderService`는 주문을 받는 역할을 한다. 주문을 받을 때 반드시 할인 정책을 반영해야 하므로, 내부적으로 `FixDiscountPolicy` 객체를 직접 생성해서 사용하고 있다.

즉, `OrderService`의 주된 목적은 "주문을 처리하면서 할인 정책을 반영하는 것"인데, 여기서 한 발 더 나아가 **할인 정책의 구체적인 구현체까지 직접 선택**해버린다.

결과적으로 주문 서비스는 자신의 본래 책임(주문 처리)뿐 아니라, **어떤 할인 정책을 사용할지 결정하는 책임까지 떠안고 있는 셈**이다.

**AppConfig 도입 후 (제어 흐름 분리)**

이제 AppConfig를 통해 제어의 흐름을 분리해보자!

`AppConfig`에서 `OrderService`를 생성할 때, 사용할 할인 정책을 외부에서 선택(주입)해줬다.

```java

public class OrderService {
    private final DiscountPolicy discountPolicy; // 추상에 의존

    // 구현체는 외부에서 주입
    public OrderService(DiscountPolicy discountPolicy) {
        this.discountPolicy = discountPolicy;
    }

    public int createOrder(int price) {
        return discountPolicy.discount(price);
    }
}
```

그 결과 `OrderService`는 자신이 어떤 할인 정책을 쓰는지 알 필요가 없다. 그저 주어진 정책을 이용해 주문 시 할인 금액을 반영하기만 하면 된다. 즉, 프로그램의 흐름을 더 이상 `OrderService`가 직접 주도하지 않는다. “어떤 정책을 쓸지”라는 선택은 외부(AppConfig)가 결정하고, `OrderService`는 오직 **자신의 본래 책임(주문 처리)** 에만 집중하게 된다.

이처럼 프로그램의 흐름을 직접 제어하지 않고, **외부(AppConfig)에 맡기는 방식**을 **제어의 역전(Inversion of Control, IoC)**이라고 한다.

## 의존성 주입(Dependency Injection)
IoC(제어의 역전) 예시를 보면, `AppConfig`를 통해 `OrderService`에서 사용할 할인 정책을 외부에서 넣어주었다. 이렇게 객체의 의존성을 외부에서 주입해주는 방식을 **DI(Dependency Injection, 의존성 주입)** 이라고 한다.

원래 `OrderService`는 할인 정책 구현체에 직접 의존하고 있었다. 하지만 이제는 외부(AppConfig)가 할인 정책 객체를 생성해서 **생성자**를 통해 주입해준다.

따라서 `OrderService`는 여전히 **할인 정책이라는 추상(인터페이스)에 의존**하지만, 구체적인 구현체를 스스로 선택하지 않고 **외부에서 제공받는 구조**가 된다.

`OrderService`는 "할인 정책이 필요하다"는 사실만 알고, 실제 어떤 정책이 들어올지는 외부가 결정한다. 이것이 바로 **IoC(제어의 역전)** 이며, 그 구체적인 구현 방법이 **DI(의존성 주입)** 인 것이다.

### 그럼 의존성을 주입해주는 AppConfig는..?

의존성을 주입해주는 설정 클래스인 `AppConfig`도 결국 하나의 객체다.  그렇다면 여러 객체에 DI를 해주는 `AppConfig` 역시 결국은 **개발자가 직접 제어해야 하는 것일까?**

순수 자바 환경에서는 그렇다. 하지만 스프링을 도입하면 이야기가 달라진다.

```java
public class AppConfig {
    public OrderService orderService() {
        // 어떤 정책을 쓸지는 여기서 결정
        // return new OrderService(new FixDiscountPolicy());
        return new OrderService(new RateDiscountPolicy());
    }
}

// 실행
public class Main {
    public static void main(String[] args) {
        AppConfig appConfig = new AppConfig();
        OrderService orderService = appConfig.orderService();

        int discount = orderService.createOrder(20000);
        System.out.println("할인 금액: " + discount);
    }
}
```

순수 자바 방식에서는 개발자가 직접 `AppConfig`를 실행하고, 그 안에서 `orderService()`를 호출해 객체를 생성한다. 이렇게 내부적으로는 DI를 적용했더라도, 여전히 `AppConfig` 자체를 개발자가 직접 제어하고 있으므로 IoC나 DI를 스프링 컨테이너 수준에서 달성한 것은 아니다.

**스프링 버전**

```java
@Configuration
public class AppConfig {

    @Bean
    public DiscountPolicy discountPolicy() {
        // 여기서 정책 선택
        // return new FixDiscountPolicy();
        return new RateDiscountPolicy();
    }

    @Bean
    public OrderService orderService() {
        return new OrderService(discountPolicy());
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class); 
        OrderService orderService = ac.getBean(OrderService.class); // 컨테이너에서 꺼내오기
        int discount = orderService.createOrder(20000);
        System.out.println("할인 금액: " + discount);
    }
}
```

스프링을 사용하면 개발자가 직접 `AppConfig`를 실행할 필요가 없다.

스프링 컨테이너가 `@Configuration` 설정을 읽고, 필요한 객체를 **생성·관리·주입까지 모두 담당**한다. 위 코드에서 개발자가 한 일은 단지 `AnnotationConfigApplicationContext`를 통해 컨테이너를 띄운 것뿐이다.

이후 객체 생성은 전부 컨테이너가 알아서 처리하며, 우리는 `getBean()`으로 꺼내 쓰기만 한다.

즉, **객체 생성과 의존성 관리의 제어 흐름이 개발자에서 프레임워크로 넘어가 IoC(제어의 역전)가 실현된 것**이다.

이 예제에서는 `AnnotationConfigApplicationContext`를 직접 생성했지만, **스프링 부트**를 사용하면 이 작업조차 필요 없다. 스프링 부트 프로젝트를 시작하면 보이는 `@SpringBootApplication` 어노테이션이 바로 그 역할을 한다. 이 어노테이션이 붙은 메인 클래스에서 `SpringApplication.run()`을 실행하면, 내부적으로 스프링 컨테이너가 자동으로 생성되고 필요한 기본 설정도 함께 적용된다.