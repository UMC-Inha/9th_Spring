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

---

### **2. `@EntityGraph`**

`JOIN FETCH`를 매번 JPQL로 작성하는 것이 번거롭게 느껴질 수 있습니다. `@EntityGraph` 어노테이션을 사용하면, 쿼리문 변경 없이 **메서드 선언만으로** 함께 조회할 연관 엔티티를 지정할 수 있습니다.

```java
// UserRepository.java
@Override
@EntityGraph(attributePaths = {"orders"})
List<User> findAll();
```

이제 `userRepository.findAll()`을 호출하기만 해도, JPA는 `orders`가 함께 필요하다는 것을 인지하고 자동으로 `JOIN` 쿼리를 실행합니다.

---

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