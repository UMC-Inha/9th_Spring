1. 뷰 작성하는 쿼리,
* 사진의 경우는 일단 배제

review_id는 auto_increment 로 자동 생성

리뷰 작성 날짜는 timestamp를 사용해 자동으로 들어가게 한다

INSERT INTO review (store_id, member_id, rate, explanation)
VALUES (1, 1, 5, '맛있어요!');


2. 마이 페이지 화면 쿼리

마이 페이지에서 해당되는 유저의 정보를 제공한다

SELECT m.name, m.email, m.phone_num

FROM MEMBER m

WHERE m.id = 3



3. 내가 진행중, 진행 완료한 미션 모아서 보는 쿼리(페이징 포함)

해당하는 유저의 미션을 찾는다

이후 해당 미션을 이행하는 가게를 찾는다

SELECT *
FROM (SELECT *

FROM MEMBER_MISSION member_mission

where member_mission.member_id = ?) mem_mission

JOIN MISSION mission

ON mem_mission.mission_id = mission.mission_id

LEFT JOIN STORE store

ON mission.store_id = store.store_id

WHERE mission.status = ‘DONE’

LIMIT 5


4. 홈 화면 쿼리
   (현재 선택 된 지역에서 도전이 가능한 미션 목록, 페이징 포함)

SELECT m.name, m.price. m.point, m.due_date, s.name, s.category

FROM MISSION m

JOIN STORE s

ON m.store_id = s.store_id

where s.address LIKE ‘%?%’

AND m.status = ‘IN_PROGRESS’

AND m.due_date > = 0

LIMIT 5