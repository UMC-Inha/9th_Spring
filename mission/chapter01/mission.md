## 리뷰 작성하는 쿼리

```sql
INSERT INTO review 
VALUES (1, '음 너무 맛있어요 포인트도 얻고 맛있는 맛집도 알게 된 것 같아 너무나도 행복한 식사였답니다. 다음에 또 올게요!!', 5, '2022-05-14', 2);
```

```sql
INSERT INTO review (user_id, content, star)
VALUES (2, '음 너무 맛있어요 포인트도 얻고 맛있는 맛집도 알게 된 것 같아 너무나도 행복한 식사였답니다. 다음에 또 올게요!!', 5);
```

리뷰의 PK가 AUTO_INCREMENT로 자동 생성되고, 생성 시간이 CURRENT_TIMESTAMP로 자동 기록되도록 설정되어 있다면, `user_id`, `content`, `star`만 지정해도 된다.

## 마이 페이지 화면 쿼리
### **이름 변경**

```sql
UPDATE user 
SET name = 'nickname012' 
WHERE user_id = 1;
```

### **유저 정보 조회**

```sql
SELECT name, email, point, phone_number
FROM user 
where user_id = 1
```


## 내가 진행중, 진행 완료한 미션 모아서 보는 쿼리
```sql
SELECT
  um.mission_id,
  um.status,
  ms.point,
  ms.content,
  st.name AS store_name
FROM user_mission AS um
JOIN mission AS ms
  ON ms.mission_id = um.mission_id
JOIN store AS st
  ON ms.store_id = st.store_id
WHERE um.user_id = ?
  AND um.status IN ('IN_PROGRESS','SUCCESS')
  AND (
        um.accepted_at < ?
     OR (um.accepted_at = ? AND um.mission_id < ?)
  )
LIMIT ?;

```


## 홈 화면 쿼리

```sql
SELECT
  ms.mission_id,
  ms.content,
  ms.point,
  ms.deadline,
  st.name    AS store_name,
  st.type    AS store_type
FROM mission AS ms
JOIN store   AS st ON st.store_id = ms.store_id
WHERE st.address LIKE '%안암동%'             
  AND ms.deadline >= NOW()
  AND NOT EXISTS (
        SELECT 1
        FROM user_mission AS um
        WHERE um.user_id    = ?
          AND um.mission_id = ms.mission_id
          AND um.status IN ('IN_PROGRESS','SUCCESS')
  )
  AND (
        ms.deadline >  ?                          
     OR (ms.deadline = ? AND ms.mission_id < ?)   
  )
ORDER BY ms.deadline ASC, ms.mission_id DESC
LIMIT ?;                                          
```