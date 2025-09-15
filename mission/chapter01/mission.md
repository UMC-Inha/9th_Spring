## 🧩 리뷰 작성하는 쿼리
```sql
INSERT INTO Review (id, member_id, store_id, star_rating, content, created_at)
VALUES (1, 2, 3, 4.0, "너무 맛있었어요~~~", 2025-09-15) 
```

## 🧩 마이페이지 화면 쿼리
```sql
SELECT nickname, email, phone_number, point
FROM Member
WHERE id = 1
```

## 🧩 진행중, 진행 완료한 미션 모아서 보는 쿼리 (페이징 포함)
```sql
SELECT MS.point, MS.description, MM.status
FROM Member_Mission as MM INNER JOIN Mission as MS on MM.mission_id = MS.mission_id
WHERE MM.member_id = 1 AND MM.mission_id < ?
ORDER BY MM.mission_id DESC
LIMIT 10
```

## 🧩 홈 화면 쿼리 (현재 선택 된 지역에서 도전이 가능한 미션 목록, 페이징 포함)
```sql
아직 미완성입니당..
SELECT  
FROM Store as S JOIN Rejoin as R on S.region_id = R.region_id
WHERE member_id = 1