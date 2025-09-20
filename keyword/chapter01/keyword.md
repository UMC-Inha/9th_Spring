

  # DB JOIN

  [정의]

  JOIN은 두 개 이상의 테이블을 연결하여 원하는 데이터를 하나의 결과 집합으로 조회하는 SQL 연산

  [추가설명]

  관계형 데이터베이스에서는 데이블 간의 관계 (외래키 등) 을 활용해 데이터를 조합한다.

  [예]

    sql
    SELECT [m.name](http://m.name/), o.order_date
    FROM Member m
    JOIN Orders o ON [m.id](http://m.id/) = o.member_id;
    

  # JOIN의 종류

  ### -INNER JOIN

  [정의] 두 테이블 모두에 존재하는 데이터를 조회 (= 매칭되는 행만 조회)

  ### -LEFT JOIN

  [정의] (=LEFT OUTER JOIN) 왼쪽 테이블의 모든 행을 포함하고 오른쪽에는 매칭되는 값 없는경우 NULL로 채움

  ### -RIGHT JOIN

  [정의] (=RIGHT OUTER JOIN) 오른쪽 테이블의 모든 행을 포함하고 왼쪽에는 매칭되는 값 없는경우 NULL로 채움

  ### -FULL OUTER JOIN

  [정의] INNER JOIN 결과인 매칭되는 행 + LEFT JOIN, RIGHT JOIN에서 매칭되지 않아 한쪽이 NULL 로 채워진 모든 행

  ### -CROSS JOIN

  [정의] Cartesian Product를 통해 모든 조합을 반환, 두 테이블 각각의 행의 곱만큼의 결과가 만들어진다.

  [추가설명] INNER JOIN에서 항상 참인 조건을 설정하여 JOIN을 하면 모든 행의 조합이 생성되므로 CROSS JOIN과 동일한 결과를 낸다.

  ### -NATURAL JOIN

  [정의] 두 테이블에서 같은 이름을 가진 컬럼을 자동으로 매칭해서 JOIN 수행

  [추가설명] 가독성에서 장점을 가지나 의도치 않은 결과를 생성할 수 있기에 보통은 INNER JOIN으로 조건을 명시해 사용

  # Transaction

  [정의]

  데이터베이스에서 하나의 논리적 작업 단위를 의미한다.

  여러 SQL 문장이 하나의 Transaction 안에서 실행되고, 전부 성공하면 Commit 도중에 실패하면 전체 작업이 Rollback 되어 데이터의 일관성을 보장한다.

  [추가설명] 트랜잭션은 보통 ACID 특성을 지켜야 함

  ### ACID

  Atomicity(원자성) : 트랜잭션이 작업의 단위가 되기 때문에 트랜잭션 안의 작업은 모두 실행되거나 모두 실행되지 않아야 함

  Consistency(일관성) : 트랜잭션이 완료되면 데이터는 일관된 상태여야 함 (=데이터베이스 무결성 유지, =데이터베이스의 규칙을 만족)

  Isolation(고립성) : 동시에 여러 트랜잭션이 실행되더라도 서로 간섭하지 않아야 함

  Durability(지속성) : 트랜잭션이 Commit 되면 그 결과는 DB에 영구적으로 저장되어야 함



  # JOIN ON, WHERE의 차이

  JOIN ON은 두 테이블을 JOIN 할 때 어떤 조건에 따라 JOIN 하는지에 대해 결정

  WHERE은 JOIN이 있는 상황이라면 JOIN 후 결과에서 어떤 값을 필터링 할지 결정