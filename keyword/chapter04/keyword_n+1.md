# N+1 문제

JPA(Java Persistence API)는 개발자가 SQL을 직접 작성하지 않고도 객체를 통해 데이터베이스를 다룰 수 있게 해주는 강력한 도구입니다. 하지만 이 편리함 뒤에는 성능을 저하시킬 수 있는 **N+1 쿼리 문제**라는 함정이 숨어있습니다.

이번 글에서는 JPA에서 N+1 문제가 왜 발생하는지, 그리고 어떻게 해결할 수 있는지 알아보겠습니다.

## JPA에서 N+1 문제는 왜 발생할까요?

결론부터 말하면, N+1 문제는 대부분 **JPA의 지연 로딩(Lazy Loading) 전략** 때문에 발생합니다.

JPA는 엔티티 간의 관계(`@OneToMany`, `@ManyToOne` 등)를 설정할 수 있습니다. 이때, 연관된 엔티티를 언제 데이터베이스에서 조회할지를 결정하는 '페치(Fetch) 전략'을 사용합니다.

- **즉시 로딩 (Eager Loading)**: 엔티티를 조회할 때 연관된 엔티티도 함께 조회합니다.
- **지연 로딩 (Lazy Loading)**: 엔티티를 조회할 때는 우선 해당 엔티티만 가져오고, 연관된 엔티티는 **실제로 접근(사용)하는 시점**에 별도의 쿼리로 조회합니다.

### 잠깐! 지연 로딩(Lazy Loading)이란?

**지연 로딩**은 '정말로 필요할 때까지 데이터 로딩을 미루는' 효율적인 방식입니다.

`User`를 조회할 때, 당장 사용하지 않을 수도 있는 `Order` 데이터까지 한 번에 가져오는 것은 낭비일 수 있습니다. 지연 로딩은 이때 **우선 `User` 정보만 가져오고,** `Order` 데이터는 **코드에서 실제로 접근하는 순간**에 별도의 쿼리를 통해 조회합니다.

이처럼 꼭 필요한 시점에만 데이터를 가져와 불필요한 쿼리와 메모리 낭비를 막아주는 똑똑한 기능이지만, 반복문 안에서 사용될 경우 N+1 문제의 원인이 됩니다.

## **지연 로딩으로 인한 N+1 발생 시나리오**

'모든 회원의 주문 목록 확인하기' 예시를 다시 보겠습니다. 

`User` 엔티티는 `Order` 엔티티와 `@OneToMany` 관계를 맺고 있습니다.

**1. 첫 번째 쿼리: `findAll()` 실행**

먼저, `userRepository.findAll()`을 호출하여 모든 회원을 조회합니다.

```java
// 이 시점에는 User 정보만 가져오는 쿼리 1번이 실행됩니다.
List<User> users = userRepository.findAll(); // SELECT * FROM user;
```

JPA는 지연 로딩 전략에 따라, `User` 엔티티만 조회하고 `Order` 정보는 아직 조회하지 않습니다.

**2. 추가 쿼리: 연관 엔티티(`orders`) 접근**

이제 반복문을 돌며 각 회원의 주문 정보를 사용(`user.getOrders().size()`)하려고 합니다.

```java
for (User user : users) {
    // user.getOrders()에 처음 접근하는 순간, 해당 유저의 주문 정보를 가져오는 추가 쿼리가 실행됩니다.
    System.out.println(user.getName() + "의 주문 개수: " + user.getOrders().size());
}
```

바로 이 `getOrders()`가 호출되는 순간, JPA는 "아, 이제 진짜 주문 정보가 필요하구나!"라고 판단하고, 해당 회원의 주문 목록을 가져오는 추가 쿼리를 실행합니다. 이 과정이 회원 수(N)만큼 반복되면서 **`1 + N`개의 쿼리**가 발생하게 됩니다.

## JPA N+1 문제 해결 전략

사실 이번 시나리오에서 N+1 문제가 발생한 근본 원인은 `User` 객체를 반복하면서 그 안의 `getter`를 통해 연관된 주문을 조회했기 때문입니다. 즉, 회원 목록을 조회한 뒤 각 회원을 기준으로 주문에 접근했기 때문에 매번 추가 쿼리가 실행된 것입니다. 따라서 단순히 회원의 주문 내역이 필요하다면, 굳이 `User`를 거치지 않고 애초에 JPA 쿼리에서 회원의 주문을 직접 조회하는 방식으로 문제를 피할 수 있습니다.

하지만 이 방식이 모든 상황을 해결해주지는 않습니다. 복잡한 연관관계나 다양한 조회 패턴 속에서 비슷한 문제가 언제든 다시 발생할 수 있기 때문입니다. 따라서 JPA는 N+1 문제가 나타나기 전에 애초에 원천 차단할 수 있는 방법을 제공합니다.

### **1. Fetch Join (패치 조인)**

가장 대표적이고 확실한 해결책입니다. **JPQL**을 사용하여 쿼리를 작성할 때 `JOIN FETCH` 키워드를 사용하면, 처음 엔티티를 조회하는 시점에 연관된 엔티티 데이터까지 **하나의 `JOIN` 쿼리**로 함께 가져올 수 있습니다.

```java
// UserRepository.java
@Query("SELECT u FROM User u JOIN FETCH u.orders")
List<User> findAllWithOrders();
```

위와 같이 작성하면, `findAllWithOrders()` 호출 시 JPA는 아래와 같은 단 하나의 쿼리를 실행하여 N+1 문제를 원천적으로 차단합니다.

```sql
SELECT u.*, o.* FROM user u INNER JOIN orders o ON u.user_id = o.user_id;
```

### **2. `@EntityGraph`**

`JOIN FETCH`를 매번 JPQL로 작성하는 것이 번거롭게 느껴질 수 있습니다. `@EntityGraph` 어노테이션을 사용하면, 쿼리문 변경 없이 **메서드 선언만으로** 함께 조회할 연관 엔티티를 지정할 수 있습니다.

```java
// UserRepository.java
@Override
@EntityGraph(attributePaths = {"orders"})
List<User> findAll();
```

이제 `userRepository.findAll()`을 호출하기만 해도, JPA는 `orders`가 함께 필요하다는 것을 인지하고 자동으로 `JOIN` 쿼리를 실행합니다.

### **3. Batch Size 조절**

Fetch Join이나 `@EntityGraph`를 사용하기 어려운 경우가 있습니다. (예: 두 개 이상의 `@OneToMany` 관계를 동시에 페치 조인할 때 등)

이때는 **배치 사이즈(Batch Size)**를 조절하는 방법을 사용할 수 있습니다. 이 방법은 쿼리를 1개로 줄여주지는 않지만, N개의 추가 쿼리를 **`IN` 절을 사용하는 몇 개의 쿼리로** 크게 줄여줍니다.

`application.yml`에 다음과 같이 글로벌 설정을 추가하거나,

```yaml
spring:
  jpa:
    properties:
      hibernate:
        default_batch_fetch_size: 100 # 여기에 원하는 사이즈 지정
```

엔티티의 컬렉션 필드에 `@BatchSize` 어노테이션을 직접 붙여줄 수도 있습니다.

```java
// User.java
@BatchSize(size = 100)
@OneToMany(mappedBy = "user", fetch = FetchType.LAZY)
private List<Order> orders = new ArrayList<>();
```

이렇게 하면 지연 로딩 시점에 주문 정보를 조회할 때, 한 번에 100명의 회원에 대한 주문 정보를 `WHERE user_id IN (?, ?, ...)` 구문을 통해 가져오므로, 쿼리 수를 획기적으로 줄일 수 있습니다. (1 + N/100)