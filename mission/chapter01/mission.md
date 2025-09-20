![image (3).png](image(3).png)

### 1. 리뷰 작성하는 쿼리

SQL의 INSERT INTO
: 데이터 INSERT INTO문은 데이터베이스 테이블에 새로운 행을 추가하는데 사용

내 ERD review 테이블에는 mission_id, store_id, star, text가 있다

[기본 형식]

1. 지정된 컬럼에 값 삽입
   ````
   INSERT INTO review(mission_id, store_id, star,text)
   VALUES(1,100,5,’음 너무 너무 맛있어요 포인트도 얻고 맛있는 
   맛집도 알게 된 것 같아 너무나도 행복한 식사였답니다. 다음에 또 올게요!!’)
   ````

2. 모든 컬럼에 값 삽입
   ````
   INSERT INTO review
   VALUES(1,100,5,’음 너무 너무 맛있어요 포인트도 얻고 맛있는 
   맛집도 알게 된 것 같아 너무나도 행복한 식사였답니다. 다음에 또 올게요!!’)
   ````

### 2. 마이 페이지 화면 쿼리

필요한 기능

1. 이름 변경

   SQL의 UPDATE문 사용
       : UPDATE문은 특정 조건을 만족하는 하나 이상의 데이터베이스 행에 대해 데이터를 수정

   [기본 구조]
   ````
   UPDATE 테이블명
   SET 컬럼1 = 값1, 컬럼2 = 값2

   예제
   
   UPDATE Employees
   SET Salary = 50000
   WHERE EmployeeID = 4;
   -> Employees라는 테이블에서 EmployeeID가 4인 직원의 Salary를 50000으로 변경
   ````
   현재 name이 nickname012이고 변경할 수 있어야 한다.

   ````
   UPDATE member
   SET name = ‘nickname’
   WHERE member_id = 1
   ````

2. 유저/멤버의 정보 조회

    SQL의 SELECT문 사용
       : 데이터베이스에서 저장된 데이터를 조회하는 쿼리
       (작성하는 순서와 실행되는 순서가 다르다)

   ````
   SELECT [DISTINCT] 필드명 -> (처리된 데이터에서 어떤 칼럼을 출력할지 선택)
   
   FROM 테이블명 -> (조회할 대상 테이블 지정)
   
   WHERE 조건 -> (FROM 절에서 가져온 테이블의 데이터를 필터링)
   
   GROUP BY 컬럼명 -> (지정한 컬럼을 기준으로 앞서 반환된 데이터 결과 그룹핑)
   
   HAVING 조건 -> (그룹별로 집계된 결과에 대한 필터링)
   
   ORDER BY 컬럼명 [ASC || DESC] -> (지정한 컬럼을 기준으로 정렬)
   ````
   
   실행순서
   FROM -> WHERE -> GROUP BY -> HAVING -> SELECT -> ORDER BY
    <br><br>
    #### 현재 member 테이블에 name, email, phone_number, phone_certification, point 모두 있다

    - 이름(nickname012)
    - 이메일(dlapdlf@naver.com)
    - 휴대폰번호
    - 휴대폰번호인증여부(미인증)
    - 내 포인트(2,500)
   
    ````
    SELECT name, email, phone_number, phone_certification, point

    FROM member

    WHERE member_id = 1
    ````

### 3. 내가 진행중이거나 진행 완료한 미션 모아서 보는 쿼리
   

   member_id인 사용자가 진행중(IN_PROGRESS)이거나 성공(SUCCESS)한 미션들 중에서 updated_at 시간을 기준으로 최신인 것부터 5개를 보여준다.

   그리고 각 미션에 연결	된 가게 정보(미션내용, 포인트, 가게 id, 가게 이름)를 함께 보여준다.

   INNER JOIN을 사용해서 각 가게에 해당하는 미션들을 불러온다.
   
   ````
   SELECT
   ms.mission_id, ms.text AS mission_content, ms.point,
   ms.price, st.store_id, st.store_name
   
   FROM mission AS ms
   
   JOIN store AS st ON st.store_id = ms.store_id
   
   WHERE ms.member_id = ?
   
   AND ms.status IN ('IN_PROGRESS','SUCCESS')
   
   ORDER BY ms.updated_at DESC, ms.mission_id DESC
   
   LIMIT 5;
   ````

### 4. 홈 화면 쿼리(현재 선택 된 지역에서 도전이 가능한 미션 목록)


   (이 UI는 어떻게 짜야할지 잘 모르겠어서 최대한 아는만큼 해봤습니다,,,,,)

   4-1) (안암동, 7/10, 미션 10개 달성시 1,000P) 이 UI에서는 지역 이름, 달성한 미션개수 등이 필요하다

   COUNT(*)는 조건을 만족하는 행 전체의 수를 세는 것이다.

   	 SELECT 
     COUNT(*) AS count

   	 FROM mission

   	 WHERE member_id = ?

   	 AND success = true;
     

   4-2) 식당이름, 식당유형, 미션기한, 미션내용, 포인트, 미션 성공 여부 등이 필요하다

member_id에 해당하는 회원이 아직 성공하지 않은 미션들을 선택해야하고,
그 미션이 있는 가게와 지역을 기준으로 필터링해야 한다. 
그리고 정렬기준은 deadline이 빠른 순으로 한다	

    
   	SELECT
    ms.mission_id,
    ms.text	AS mission_content,
    st.store_id, st.store_name, st.store_type,
    ms.deadline, ms.point, ms.success, rg.region_id

   	FROM mission AS ms

   	JOIN store AS st ON st.store_id = ms.store_id

   	JOIN region AS rg ON rg.store_id = st.store_id

   	WHERE ms.member_id = ?
   		AND (ms. success = false OR ms.success IS NULL)
   		AND rg.region_id = ?
   	ORDER BY ms.deadline ASC, ms.mission_id DESC
   	LIMIT 5;
    
			
			

