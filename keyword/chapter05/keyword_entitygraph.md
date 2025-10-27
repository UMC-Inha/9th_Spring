## EntityGraph
`JOIN FETCH`를 매번 JPQL로 작성하는 것이 번거롭게 느껴질 수 있다. `@EntityGraph` 어노테이션을 사용하면, 쿼리문 변경 없이 **메서드 선언만으로** 함께 조회할 연관 엔티티를 지정할 수 있다.

```java
// UserRepository.java
@Override
@EntityGraph(attributePaths = {"orders"})
List<User> findAll();
```

이제 `userRepository.findAll()`을 호출하기만 해도, JPA는 `orders`가 함께 필요하다는 것을 인지하고 자동으로 `JOIN` 쿼리를 실행한다.