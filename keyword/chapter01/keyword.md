- **DB Join이란?**

  데이터베이스에서 중복 데이터를 피하기 위해 데이터를 쪼개 여러 테이블에 저장한다. 이렇게 분리되어 있는 데이터에서 원하는 결과를 도출하기 위해선 테이블을 조합해야하는데, 이때 쓰는게 Join 연산이다.

  Join은 공통 컬럼을 기준으로 두 개 이상의 테이블의 행을 합쳐주는 연산이다.

  만약 어떤 유저와 유저가 쓴 리뷰 테이블이 있다면, 사용자 이름과 리뷰 내용을 함께 출력하고 싶을 때 사용할 수 있다.


- **Join 종류들**
    - INNER JOIN → 조인하는 테이블의 ON절의 조건이 일치하는 결과만 출력한다. 교집합이라고 생각하면 된다. from절에 콤마만 써도 inner join과 같이 동작한다.
      ![Image](https://github.com/user-attachments/assets/264edf45-4f95-47bb-92b7-80829c055e23)
    - **LEFT OUTER JOIN** → 조건으로 필터링은 하되, 한쪽 테이블의 데이터를 필터링하지 않고 보존하고 싶을 때 사용한다. 그 중 LEFT JOIN은 FROM (테이블1) LEFT OUTER JOIN (테이블2) 이렇게 있으면 테이블1을 기준으로 조합하는 연산이다. 이를 보면 알 수 있듯이 **테이블**의 순서가 굉장히 중요한 연산이다.

      ![Image](https://github.com/user-attachments/assets/51c974d2-587e-4d12-8a4d-5c35ab19b11b)
    - **RIGHT OUTER JOIN** → LEFT OUTER JOIN과 유사하되 그 기준이 테이블1이 아닌 테이블 2인 연산이다. 오른쪽 테이블에 맞춰 join한다.

      ![Image](https://github.com/user-attachments/assets/35f09e48-15e4-41ee-9ccb-5a7d9c65d18b)

    - **FULL OUTER JOIN** → 테이블1, 테이블2 모두를 보존하는 join 방법이나, MySQL에선 지원하지 않는 문법이다. 그렇다면 어떻게 구현하느냐,
        - **UNION →** 두 테이블의 합집합을 구한다. 다만, union 할 때 주의할 점은 select문으로 선택된 컬럼 개수와 타입, 순서까지 일치해야한다는 점이다. 기본적으로 중복제거기능이 포함되어 있는데, UNION ALL을 쓰게되면 중복을 허용한다.
          ![Image](https://github.com/user-attachments/assets/5a8d1cdb-8263-486e-ad59-1047edccd1e1)

    - **CROSS JOIN →** 한쪽 테이블의 모든 행과 다른 쪽 테이블의 모든 행의 모든 조합을 구한다. 전체 행 개수는 nxm이 된다.
    - **SELF JOIN →** 테이블 자기 자신과 join한다. 예를 들어, 같은 부서에서 일하는 상위 직원을 알고 싶을 때 사용한다.


- 트랜잭션이란?

  데이터베이스의 하나의 연속적인 업무 단위를 말한다. ALL OR NOTHING 개념이다. 모두 Commit 하거나 모두 Rollback 처리 해야한다는 것이다. 이를 왜 사용하냐했을 때 예시를 들어보자면, 만약 A에서 B로 돈을 보냈을 때, A에서 돈이 빠져나가는 것과 B에 돈이 입금되는 것이 보장되어야 한다. 하나라도 실패하면 돈은 빠져나가고 입금은 안된다던지 이런 치명적인 단점이 된다.

  트랜잭션의 특성으론 4가지가 있다.

    - 원자성 → 모두 반영하거나 전혀 반영하지 않는다.
    - 일관성 → 트랜잭션 실행 전 데이터베이스 내용이 잘못되어 있지 않다면 트랜잭션을 실행하고 이후에도 데이터베이스 내용의 잘못이 있으면 안된다. ex) 이체 전후 전체 잔고 합계가 동일, 모든 제약조건 만족
    - 고립성 → 트랜잭션 실행 도중 다른 트랜잭션의 영향을 받아 잘못된 결과를 만들어선 안된다.
    - 지속성 → 트랜잭션이 성공적으로 수행되면 갱신한 데이터베이스 내용이 영구저장 되어야한다.


- Join on 과 where의 차이
    - on: join 전에 조건을 필터링한다
    - where: join 후에 조건을 필터링한다.

  inner join을 할 때는 차이가 없다.

  outer join시에 on으로 조건을 적용시켜야 원하는 결과를 얻을 수 있다.

    ```jsx
    select u.id, u.name, r.score, r.content
    from users as u
    left join reviews r on r.user_id = u.id
    	and r.score >=4
    	
    -> 1 | haewon | 4 | 굿
    	 2 | jihyunii | null | null -> 별 2개를 달았으나 조건 불충족으로 NULL
    ```

    ```jsx
    select u.id, u.name, r.score, r.content
    from users as u
    left join reviews r on r.user_id = u.id
    **where** r.score >=4;
    
    -> 1 | haewon | 4 | 굿 
    -> 사실상 inner join처럼 동작한다. 
    ```

  📘ON은 조인할 때의 매칭 규칙이라고 생각하면 되고, WHERE은 조인이 끝난 후의 최종 필터라고 생각하면 된다.