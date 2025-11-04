### 1. SRP: 단일 책임 원칙 (Single Responsibility Principle)

단일 책임 원칙은 한 클래스는 하나의 책임만 가져야 한다는 의미예요. 여기서 ‘책임’이란 변경의 이유를 뜻하는데, 클래스가 여러 책임을 가지면 변경 시 그 영향을 예측하기 어려워지고, 오류 가능성이 커집니다. 따라서 클래스의 책임을 명확히 나누면 코드 변경이 필요할 때 수정 범위를 최소화할 수 있어요.

**나쁜 설계**: UI 변경과 데이터 처리를 한 클래스에서 모두 처리

```java
class UserManager {
      public void renderUI() {
          // 화면 출력
      }
      public void handleData() {
          // 데이터 처리
      }
  }
```

**좋은 설계**: UI와 데이터를 분리

```java
class UIHandler {
    public void renderUI() {
        // 화면 출력
    }
}

class DataHandler {
    public void handleData() {
        // 데이터 처리
    }
}
```

### 2. OCP: 개방-폐쇄 원칙 (Open/Closed Principle)

개방-폐쇄 원칙은 **소프트웨어 요소는 확장에는 열려 있고, 변경에는 닫혀 있어야 한다**는 것을 뜻합니다. 즉, 새로운 기능을 추가할 때 기존 코드를 수정하지 않고 확장할 수 있도록 설계하라는 의미예요. 이를 실현하기 위해 다형성을 활용하거나 인터페이스를 통해 구체적인 구현을 분리하는 방식이 필요합니다.

### 문제와 해결

**문제점**: 아래처럼 구현체를 직접 다루면, 새로운 구현체를 추가할 때 기존 코드를 수정해야 해요.

```java
public class MemberService {
    private MemberRepository memberRepository = new MemoryMemberRepository();
}```
```

**해결 방법**: 인터페이스를 사용하고, 구현체는 외부에서 주입받도록 설계하면 됩니다!

```java
public class MemberService {
    private MemberRepository memberRepository;

    public MemberService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
}
```

### 3. LSP: 리스코프 치환 원칙 (Liskov Substitution Principle)

리스코프 치환 원칙은 상위 타입의 객체를 하위 타입으로 대체하더라도 프로그램이 문제없이 동작해야 한다는 뜻입니다.

다형성을 활용하려면 하위 클래스가 상위 클래스의 계약(규약)을 지켜야 하며, 논리적인 동작이 일관성을 가져야 해요. 즉, 단순히 컴파일이 성공한다고 끝나는 것이 아니라, 실제 동작도 일관성을 유지해야 한다는 의미입니다.

**나쁜 설계**: 엑셀을 밟으면 차가 뒤로 가는 자동차

```java
class Car {
    void accelerate() {
        // 차가 앞으로 간다
    }
}

class BackwardCar extends Car {
    void accelerate() {
        // 차가 뒤로 간다
    }
}
```

부모 클래스(`Car`)의 `accelerate()` 메서드는 **“앞으로 간다”**는 동작을 나타냅니다.

그런데 자식 클래스(`BackwardCar`)는 이를 재정의하여 **뒤로 가도록** 구현해버렸습니다.

`Car myCar = new BackwardCar(); myCar.accelerate();` 와 같이 코드를 작성하면, 개발자는 "차가 앞으로 갈 것"이라고 기대하지만 실제로는 뒤로 가는 동작이 발생하게 되고 실제 상황이라면 큰일날 수 있겠죠..?

이처럼 상위 타입의 규약을 깨뜨리기 때문에 **리스코프 치환 원칙(LSP) 위반**이라고 할 수 있어요

**좋은 설계**: 모든 자동차의 엑셀은 차를 앞으로 가게 한다는 규약을 지켜야 해요.

```java
class Car {
    void accelerate() {
        System.out.println("차가 앞으로 간다");
    }
}

class Sedan extends Car {
    @Override
    void accelerate() {
        System.out.println("세단이 앞으로 간다");
    }
}

class Truck extends Car {
    @Override
    void accelerate() {
        System.out.println("트럭이 앞으로 간다");
    }
}
```

자동차를 상속받은 자동차 종류는 accelerate() 라는 함수는 모두 앞으로 간다는 로직을 수행하게 되죠! 상위 객체인 Car와 그 하위 객체인 Truck Sedan의 accelrate()는 모두 동일한 로직을 수행합니다.

### 4. ISP: 인터페이스 분리 원칙 (Interface Segregation Principle)

인터페이스 분리 원칙은 특정 클라이언트를 위한 인터페이스 여러 개가 범용 인터페이스 하나보다 낫다는 의미예요.

인터페이스를 작게 나누면 클라이언트가 자신이 사용하지 않는 기능에 의존하지 않게 되고, 변경 시 영향 범위를 줄일 수 있어요.

> 쉽게 말하면 거대한 하나의 인터페이스보다는 기능별 여러 인터페이스가 낫다!
> 

**나쁜 설계**: 모든 기능을 포함한 자동차 인터페이스

```java
interface Car {
    void drive();
    void repair();
}
```

자동차를 사용하는 `Driver`구현체를 정의한다고 가정해봅시다.

`Driver`는 `Car`인터페이스의`drive()` 메서드를 통해 운전할 수 있습니다. 그런데 실제로 운전자는 차량을 수리할 수 없는데도, `Car` 인터페이스에 `repair()` 메서드가 포함되어 있기 때문에 `Driver` 클래스 역시 `repair()` 메서드를 구현해야 합니다.

**좋은 설계**: 운전과 정비를 분리

```java
interface Driveable {
    void drive();
}

interface Repairable {
    void repair();
}
```

### 5. DIP: 의존관계 역전 원칙 (Dependency Inversion Principle)

의존관계 역전 원칙은 구체적인 구현이 아니라 추상화(인터페이스)에 의존해야 한다는 의미입니다.

구현 클래스에 직접 의존하면 유연성이 떨어지고, 코드 변경 시 클라이언트 코드도 수정해야 할 가능성이 커집니다. 이를 해결하려면 인터페이스를 활용하고, 의존성 주입(DI) 같은 기법을 사용해야 합니다.

**문제점**: 구현 클래스에 직접 의존

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

위 코드에서 `OrderService`는 주문 서비스에 할인 정책을 적용하기 위해 **고정 할인 정책**(`FixDiscountPolicy`)을 직접 사용하고 있습니다. 하지만 할인 정책은 얼마든지 바뀔 수 있습니다. 고정 금액 자체가 달라질 수도 있고, 아예 비율 할인 정책으로 교체될 수도 있죠.

이 경우 매번 `OrderService` 내부 코드를 수정해야 하므로, 구현체에 직접 의존하는 문제가 발생합니다.

**해결 방법**: 인터페이스에 의존하고, 구현체는 외부에서 주입받기

```java
public interface DiscountPolicy {
    int discount(int price);
}

public class FixDiscountPolicy implements DiscountPolicy {
    @Override
    public int discount(int price) {
        return 1000;
    }
}

public class RateDiscountPolicy implements DiscountPolicy {
    @Override
    public int discount(int price) {
        return (int) (price * 0.1); // 10% 할인
    }
}
```

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

위 코드에서 주문 서비스는 **`DiscountPolicy` 인터페이스**에만 의존합니다. 따라서 새로운 할인 정책을 추가하고 싶다면, `DiscountPolicy`를 구현한 새로운 클래스를 만들기만 하면 됩니다.

그리고 실제로 어떤 정책을 사용할지는 서비스 내부가 아니라 **설정 파일(config)에서 주입**을 통해 결정합니다. 즉, 서비스 로직은 그대로 두고, 정책만 교체할 수 있어 **DIP를 지키고 OCP도 만족**하게 되는 거죠