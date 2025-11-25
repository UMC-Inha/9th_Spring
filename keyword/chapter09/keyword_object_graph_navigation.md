# 객체 그래프 탐색이란?
JPA를 공부하다 보면 자주 등장하는 개념이 바로 **객체 그래프 탐색**이다. 처음 들으면 뭔가 어려운 용어 같지만, 실제로는 우리가 매일 자연스럽게 사용하는 기능이다.


## 1. 객체 그래프란?

애플리케이션에서는 엔티티들이 서로 연관관계를 맺고 있다.

예를 들어, `Member`는 여러 `Order`를 가지고 있고, `Order`는 `Store`를 참조하며, `Store`는 `Address`를 가진다.

```java
class Member {
    List<Order> orders;
}

class Order {
    Store store;
}

class Store {
    Address address;
}

class Address {
    String city;
}
```

이렇게 **객체가 객체를 참조하고, 또 그 객체가 다른 객체를 참조하는 구조 전체**를 

우리는 **객체 그래프(object graph)**라고 부른다.

쉽게 말해, 객체들끼리 연결된 하나의 **그래프(네트워크)**라고 보면 된다.


## 2. 객체 그래프 탐색이란?

> 연결된 객체들을 계속 따라가면서 내부 객체까지 접근하는 것
> 

예를 들어,

```java
member.getOrders().get(0).getStore().getAddress().getCity();
```

이 한 줄의 코드는 **Member → Order → Store → Address → City** 로 이어지는 객체의 연결 고리를 따라가서 최종 데이터를 꺼내오는 과정이다.

이걸 바로 **객체 그래프 탐색**이라고 부른다.

자바에서 객체를 다루는 사람이라면 누구나 자연스럽게 하고 있는 동작이다.


## 3. JPA에서 특히 중요한 이유

단순히 객체를 탐색하는 것 자체는 문제가 아니다.

하지만 JPA에서는 **연관된 객체가 실제로 언제 로딩되는지**가 성능에 영향을 준다.

LAZY 로딩일 때 연관 객체는 처음에는 가져오지 않는다.

필요한 순간(`getStore()`)에 쿼리를 날려서 DB에서 가져온다.

```java
List<Member> members = memberRepository.findAll();
for (Member m : members) {
    m.getOrders().size();
}
```

그래서 이런 코드가 아래와 같이 동작할 수 있다.

- `findAll()` → Member만 조회 (쿼리 1번)
- 반복문에서 Order를 꺼낼 때마다 order 조회 쿼리 실행 → N번

즉, 객체 그래프 탐색이 많아질수록 불필요한 쿼리가 폭발할 수 있다.


## 객체 그래프 탐색 중 자주 생기는 문제들

### 1) **N+1 문제**

연관 객체를 탐색하는 순간 쿼리가 추가로 발생해 쿼리가 기하급수적으로 증가.

### 2) **LazyInitializationException**

Session이 닫힌 이후에 LAZY 로딩 객체를 접근하면 터지는 예외.

```java
store.getAddress().getCity();
// 트랜잭션 끝나고 탐색하면 예외 발생
```

### 3) 예측 불가능한 쿼리

코드 한 줄 때문에 쿼리가 몇 번 실행될지 예측하기 어려워짐.

---

### JPA에서 객체 그래프 탐색 최적화 방법

객체 그래프 탐색 자체는 좋은 것이다. 단, **JPA가 언제 데이터를 가져오는지** 통제해야 한다.

- Fetch Join 사용하기
- EntityGraph 적용하기
- 필요 없는 양방향 관계 제거
- DTO Projection으로 필요한 데이터만 조회