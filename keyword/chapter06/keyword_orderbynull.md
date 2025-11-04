# SQL: order by null

`ORDER BY NULL` 구문은 말 그대로 **“정렬하지 않겠다”** 는 의미다.

즉, 명시적으로 **정렬 기준이 없음을 선언**하는 구문이다.

일반적으로 SQL 엔진은 특정 상황에서 내부적으로 정렬을 수행한다. 예를 들어, `GROUP BY`, `DISTINCT`, `UNION` 같은 연산을 사용할 때 자동으로 정렬이 이루어지는 경우가 많다.

하지만 `ORDER BY NULL`을 명시하면 이러한 **자동 정렬 과정조차 완전히 생략하라**는 의미가 되어,

불필요한 정렬 연산을 방지할 수 있다.

## 어디서 사용할 수 있을까?

사실 `ORDER BY NULL`은 **굳이 정렬이 필요하지 않은 상황에서 사용하면 좋은 구문**이다.

정렬 연산 자체가 상당히 많은 시스템 리소스를 소모하기 때문이다.

데이터베이스 엔진은 정렬을 수행할 때 정렬 버퍼를 사용하거나, 필요에 따라 디스크에 임시 테이블을 생성해 데이터를 재배치한다. 이 과정은 CPU와 메모리를 모두 사용하는 **비용이 큰 연산**이다. 

따라서 결과 순서가 중요하지 않은 쿼리라면,

`ORDER BY NULL`을 명시적으로 추가해 **불필요한 정렬 단계를 생략**하도록 하는 것이 좋다.

**✅ 예시 1 — `GROUP BY` 후 불필요한 정렬 제거**

```sql
SELECT team_id, COUNT(*) 
FROM member
GROUP BY team_id
ORDER BY NULL;
```

MySQL이나 MariaDB에서는 `GROUP BY`를 사용할 때 자동으로 그룹 기준(`team_id`)에 따라 정렬이 수행된다. 하지만 결과의 순서가 중요하지 않다면, `ORDER BY NULL`을 추가하여 이 **자동 정렬을 비활성화**할 수 있다.

✅ **예시 2 — `UNION` / `UNION ALL`에서 정렬 제거**

```sql
SELECT name FROM member_a
UNION
SELECT name FROM member_b
ORDER BY NULL;
```

`UNION`은 기본적으로 **중복 제거와 함께 정렬**을 수행한다.  

하지만 `ORDER BY NULL`을 사용하면 “최종 결과를 정렬하지 마라”는 의도를 명시할 수 있다. 

또한 `UNION ALL`과 함께 사용하면 **결과를 단순 병합만 수행하고 정렬은 생략**하게 된다.

**✅ 예시 3 — `DISTINCT` / `LIMIT`과 함께 최적화**

```sql
SELECT DISTINCT name
FROM member
ORDER BY NULL
LIMIT 10;
```

`DISTINCT` 또한 내부적으로 정렬을 수행하는 경우가 많다.

그러나 정렬 순서가 중요하지 않다면 `ORDER BY NULL`을 붙여 **엔진이 불필요한 정렬을 건너뛰도록 힌트를 제공**할 수 있다. 결과적으로 **쿼리 실행 속도를 소폭 개선**할 수 있다.