## 🧩 리뷰 작성하는 쿼리
```sql
INSERT INTO Review (id, member_id, store_id, star_rating, content, created_at)
VALUES (1, 2, 3, 4.0, "너무 맛있었어요~~~", 2025-09-15) 
```

## 🧩 마이페이지 화면 쿼리
```sql
SELECT nickname, email, phone_number, points
FROM Member
WHERE id = 1
```

## 🧩 진행중, 진행 완료한 미션 모아서 보는 쿼리 (페이징 포함)
```sql
SELECT MS.points, MS.description, MM.status, S.store_id, S.name
FROM Store as S INNER JOIN Mission as MS on MS.store_id = S.store_id
    INNER JOIN Member_Mission as MM on MS.mission_id = MM.mission_id
WHERE MM.member_id = 1 AND MM.mission_id < ?
ORDER BY MM.mission_id DESC
LIMIT ?
```
## 🧩 홈 화면 쿼리 (현재 선택 된 지역에서 도전이 가능한 미션 목록, 페이징 포함)
**윗부분 쿼리**
```sql
SELECT count(*)
FROM Store as S
    JOIN Region as R on S.region_id = R.region_id
    JOIN Mission as MS on S.store_id = MS.store_id
    JOIN Member_Mission as MM on MS.mission_id = MM.mission_id
WHERE R.name = '안암동' and MM.member_id = 1 and MM.status = 'SUCCESS'
```
**아랫부분 쿼리**
```sql
SELECT S.name, S.type, MS.description, DATEDIFF(MS.due_date, CURDATE()) as end_days, MS.mission_id, CONCAT(LPAD(DATEDIFF(Ms.due_date, CURDATE()), 10, '0'), LPAD(POW(10, 10) - MS.mission_id, 10, '0')) as cursor_value
FROM Store as S
    JOIN Region as R on S.region_id = R.region_id
    JOIN Mission as MS on S.store_id = MS.store_id
WHERE DATEDIFF(MS.due_date, CURDATE()) >= 0 
    AND NOT EXISTS (
        SELECT 1
        FROM Member_Mission as MM
        WHERE MM.mission_id = MS.mission_id
            AND MM.member_id = 1
            AND MM.status in ('SUCCESS', 'PENDING')
    )
HAVING cursor_value > ?
ORDER BY end_days ASC, MS.mission_id DESC
LIMIT ?
```