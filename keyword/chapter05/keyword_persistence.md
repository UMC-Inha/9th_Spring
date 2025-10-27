### 비영속 (Transient)

> 아직 JPA가 모르는 상태
> 

```java
Member member = new Member("민혁");
```

- 단순히 `new`로 생성된 객체로, **영속성 컨텍스트와 전혀 관련이 없음**
- DB에 저장되지 않았고, JPA가 관리하지 않음
- 따라서 `em.persist()`, `em.find()` 등으로 관리 대상이 되기 전까지는 아무 일도 안 일어남
- 따라서 해당 데이터에 대한 수정/ 삭제 / 추가 DB 반영 X

---

### 영속 (Persistent)

> JPA가 관리하는 상태 (1차 캐시에 올라온 상태)
> 

```java
em.persist(member);
```

- `em.persist()`를 호출하면, 해당 엔티티가 **영속성 컨텍스트에 저장됨**
- 이후부터 JPA가 이 객체를 **트랜잭션 단위로 관리**
    - 변경 감지(Dirty Checking)
    - flush 시 SQL 자동 반영
- 1차 캐시에 존재
- 동일성 보장 (`em.find()`로 같은 객체 조회 시 같은 인스턴스 반환)
- 트랜잭션이 커밋되면 DB에 반영됨

---

### 준영속 (Detached)

> 한때 영속이었지만, 더 이상 JPA가 관리하지 않는 상태
> 

```java
em.detach(member);
// 또는 em.clear(), em.close() 호출 시 포함된 모든 엔티티가 준영속
```

- 영속성 컨텍스트에서 분리된 상태
- 더 이상 변경 감지나 flush의 대상이 아님
- 하지만 객체 자체는 여전히 존재함 (DB 반영은 안 됨)
- DB와 동기화되지 않음
- 변경해도 `update` SQL이 실행되지 않음

---

### 삭제 (Removed)

> 삭제 명령이 내려진 상태
> 

```java
em.remove(member);
```

- 영속성 컨텍스트에는 남아있지만, **삭제 예약 상태**
- 트랜잭션 커밋 시 실제로 DB `DELETE` 쿼리 실행
- flush 혹은 commit 시 DB에서 삭제됨
- 이후 더 이상 영속성 컨텍스트에서 관리되지 않음