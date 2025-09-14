## DB Join이란?
    
DB에서 **Join**이란 두 개 이상의 테이블을 서로 연결해 하나의 결과 집합을 만드는 연산이다. 테이블 간의 **관계(Relationship)**를 활용해 원하는 데이터를 합쳐 조회할 수 있다.
    
조인 연산은 기본적으로 두 릴레이션의 **카티션 곱(Cartesian Product)**을 기반으로 하며, 그 결과 중에서 **일정 조건을 만족하는 튜플(행)만 선택**한다. 또한, 결과 집합에 포함할 **속성(컬럼)**도 지정할 수 있다. 
    
주의할 점은, **JOIN은 실제로 새로운 테이블을 생성하는 것이 아니라 단지 조회 시점에만 결과를 보여주는 것**이라는 점이다.

## Join 종류들
**1. INNER JOIN**

- 두 테이블에서 **조건이 일치하는 행만** 결과로 가져온다.
- 집합으로 비유하면 **교집합**과 유사하다.

    ```sql
    SELECT s.name, d.dept_name
    FROM student s
    INNER JOIN department d
    ON s.dept_id = d.id;
    ```

- `ON` 또는 `USING`을 통해 **조인 조건을 반드시 명시하는 것이 일반적**이다.
- 만약 조건을 생략하면, 두 테이블의 **카티션 곱(Cartesian Product)**이 발생하여 모든 조합이 출력된다. 사실상 `CROSS JOIN`과 동일한 결과를 출력한다.

**2. LEFT JOIN (LEFT OUTER JOIN)**

- 왼쪽 테이블의 **모든 행을 기준**으로 가져오고, 오른쪽 테이블에 조건이 맞는 값이 있으면 함께 보여준다.
- 오른쪽 값이 없으면 `NULL`로 채워진다.

    ```sql
    SELECT s.name, d.dept_name
    FROM student s
    LEFT JOIN department d
    ON s.dept_id = d.id;
    ```

**3. RIGHT JOIN (RIGHT OUTER JOIN)**

- LEFT JOIN의 반대.
- 오른쪽 테이블의 **모든 행을 기준**으로 가져오며, 왼쪽 값이 없으면 `NULL`로 표시된다.

**4. FULL OUTER JOIN**

- 두 테이블의 모든 행을 다 가져온다.
- 조건이 맞지 않으면 `NULL`이 들어간다.
- MySQL은 직접 지원하지 않고 `UNION`으로 구현한다.

**5. CROSS JOIN**

- 두 테이블의 모든 행을 **곱집합(카티션 곱)**으로 만든다.
- 조건이 없을 경우 `A 테이블 행 수 × B 테이블 행 수` 만큼 결과가 나온다.

    ```sql
    SELECT s.name, c.course_name
    FROM student s
    CROSS JOIN course c;
    ```


## 트랜잭션이란?
트랜잭션은 데이터베이스에서 하나의 작업을 이루는 **여러 SQL 연산을 하나로 묶은 것**이다. 여러 쿼리를 실행하더라도 사용자는 그것을 **하나의 작업 단위**처럼 다룬다.

예를 들어 **송금 기능**을 생각해보자. A 계좌에서 10만 원을 출금하고, B 계좌에 10만 원을 입금한다고 할 때 이 두 동작이 모두 성공해야만 “송금”이라는 하나의 작업이 완료된다. 만약 하나라도 실패하면 전체가 취소되어야 하며, 이를 보장해주는 것이 **트랜잭션**이다.

전체 흐름을 간단히 보자면, WAS나 데이터베이스 접근 도구에서 DB에 접속했다면, 이때 DB 서버와 접근 도구 사이에 **커넥션(Connection)**이 생성되고, 이 커넥션을 통해 DB 서버에 쿼리가 전달된다.

DB 서버는 이에 대응되는 **세션(Session)**을 생성하여 실제 쿼리를 수행한다. 세션은 쿼리를 받는 순간 트랜잭션을 시작하며, 이후 `COMMIT` 혹은 `ROLLBACK`을 통해 트랜잭션을 종료한다.

<aside>

- **커밋(commit)**: SQL의 변경사항을 데이터베이스에 **적용**

</aside>

<aside>

- **롤백(rollback)**: SQL 수행 중 문제가 생기면 **변경사항을 되돌림**

</aside>

대부분의 데이터베이스에서는 기본적으로 **자동 커밋(auto-commit)** 모드가 설정되어 있어, SQL 문이 실행되면 곧바로 **자동으로 커밋**된다. 하지만 이 방식은 트랜잭션을 세밀하게 제어하기 어렵기 때문에, **트랜잭션을 제대로 활용하려면 자동 커밋을 꺼야 한다**.

```sql
set autocommit true; //자동 커밋 모드 켜기
set autocommit false; //자동 커밋 모드 키기
```

자동 커밋을 끄면, **수동 커밋** 모드로 전환되며, 개발자가 명시적으로 `commit` 또는 `rollback`을 호출해야 트랜잭션이 종료된다. 한 번 커밋 모드를 설정하면 해당 **세션에서는 계속 유지**되며, 필요에 따라 중간에 **다시 자동 커밋으로 변경**하는 것도 가능하다.

---

트랜잭션은 ACID라 하는 원자성(Atomicity), 일관성(Consistency), 격리성(Isolation), 지속성(Durability)을 보장해야 한다!

<aside>

- **원자성(Atomicity)**

    트랜잭션을 성공적으로 끝내면 그 결과가 항상 기록되어야 한다. 중간에 시스템에 문제가 발생해도 데이터베이스 로그 등을 사용해서 성공한 트랜잭션 내용을 복구해야 한다

- **지속성(Durability)**

    동시에 실행되는 트랜잭션들이 서로에게 영향을 미치지 않도록 격리한다. 예를 들어 동시에 같은 데이터를 수정하지 못하도록 해야 한다. 격리성은 동시성과 관련된 성능 이슈로 인해 트랜잭션 격리 수준(Isolation level)을 선택할 수 있다.

- **격리성(Isolation)**

    모든 트랜잭션은 일관성 있는 데이터베이스 상태를 유지해야 한다. 예를 들어 데이터베이스에서 정한 무결성 제약 조건을 항상 만족해야 한다.

- **일관성(Consistency)**

    트랜잭션 내에서 실행한 작업들은 마치 하나의 작업인 것처럼 모두 성공 하거나 모두 실패해야 한다.

</aside>

## Join on 과 where의 차이
### **`JOIN ... ON`**

- **조인 조건**을 지정하는 문법.
- 두 테이블을 어떻게 연결할지 DB에게 알려준다.
- **조인 단계에서** 이미 걸러지기 때문에, 어떤 행을 합칠지가 결정된다.

    ```sql
    SELECT s.name, d.dept_name
    FROM student s
    JOIN department d
    ON s.dept_id = d.id;
    ```

### **`WHERE`**

- **결과 집합에 대한 필터 조건**.
- 조인이 끝난 뒤, 만들어진 결과 집합에서 조건을 만족하는 행만 남긴다.
- 따라서 조인 조건이 아니라 **추가 필터링**에 주로 사용된다.

    ```sql
    SELECT s.name, d.dept_name
    FROM student s
    JOIN department d
    ON s.dept_id = d.id
    WHERE d.dept_name = '컴퓨터공학과';
    ```

둘 다 **필터링**을 한다는 점에서는 같지만 적용 시점에서 차이가 있다. SQL은 일반적으로 `FROM → ON → JOIN → WHERE` 순으로 실행되므로 `ON` 절이 먼저 적용된다. 겉보기에는 같은 작업처럼 보일 수 있지만 실제 결과는 달라질 수 있다. **WHERE** 절은 조건을 만족하지 않는 행을 완전히 제거해 버리는 반면, **ON** 절은 조인 시점에 조건을 적용하므로 **OUTER JOIN**의 경우 조건을 만족하지 않아도 행 자체는 유지되며 해당 컬럼 값이 `NULL`로 채워진다. 따라서 INNER JOIN에서는 `ON`과 `WHERE`의 차이가 거의 없지만 OUTER JOIN에서는 결과가 크게 달라질 수 있다.

#### **ON 절에 조건을 둔 경우**

```sql
SELECT s.name, d.dept_name
FROM student s
LEFT JOIN department d
  ON s.dept_id = d.id
  AND d.dept_name = '컴퓨터공학과';
```

조인 시점에 `dept_name = '컴퓨터공학과'` 조건을 검사한다. 검사했을 때, 조건이 맞으면 학과명이 붙고, 맞지 않으면 `NULL`이 들어간다. 따라서 **모든 학생이 출력**되며 컴퓨터공학과 학생만 학과명이 표시된다.

#### **WHERE 절에 조건을 둔 경우**

```sql
SELECT s.name, d.dept_name
FROM student s
LEFT JOIN department d
  ON s.dept_id = d.id
WHERE d.dept_name = '컴퓨터공학과';
```

조인된 결과에서 다시 `WHERE` 조건을 검사한다. 이때 `dept_name`이 `NULL`인 행은 조건을 만족하지 못해 제거된다. 결국 **컴퓨터공학과 학생만 출력**된다.