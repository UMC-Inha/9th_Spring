# 🍀 UMC Mission Database 설계 & 구현 기록

## 📌 0주차 Mission

### 1. ERD 초안 설계
- **회원(User)** 과 **선호하는 음식(Food)** 관계를 단순하게 설계.
- 하지만 한 명의 회원이 여러 음식을 선호할 수 있고, 동시에 하나의 음식이 여러 회원에게 선호될 수 있다는 점을 간과.
- 즉, 회원과 음식은 **다대다(N:M) 관계**임을 깨달음.

### 2. 개선된 설계
- 다대다 관계를 해결하기 위해 **연결 테이블(선호음식)** 추가.
- 정규화를 통해 데이터 무결성을 보장.
- 각 테이블에 맞는 데이터 타입 지정.


## 📌 1주차 Mission

### 1. 개선 사항
- 미션 진행 중 **지역 정보(Location)** 를 별도의 테이블로 분리해야 한다는 필요성을 깨달음.
- 회원과 가게 모두 지역과 연관 관계를 가짐.
- 이에 따라 ERD를 수정.


---

## 📌 SQL 쿼리 정리

### 1. 리뷰 조회
```sql
SELECT 리뷰.grade,
       리뷰.content,
       리뷰.photo,
       회원.name
FROM 리뷰 
JOIN 가게  ON 가게.id = 리뷰.shop_id
JOIN 회원  ON 회원.id = 리뷰.user_id
WHERE 가게.name = '점포관리';
```

### 2. 리뷰 등록
```sql
INSERT INTO 리뷰 (shop_id, user_id, grade, content, photo)
SELECT 가게.id,
       회원.id,
       5,
       '서비스가 훌륭합니다',
       '사진'
FROM 가게, 회원
WHERE 가게.name = '점포관리'
  AND 회원.id = '로그인한_사용자_id';
```

### 3. 사용자 정보 조회
```sql
SELECT name,
       email,
       phone,
       point
FROM 회원
WHERE id = '로그인한_사용자_id';

```

### 4. 미션 조회(offset 기반)
```sql
SELECT 미션.point,
       미션수행.status,
       가게.name,
       미션.content,
       DATEDIFF(미션.date, NOW()) AS days_left
FROM 미션수행
         JOIN 미션 ON 미션수행.mission_id = 미션.id
         JOIN 가게 ON 미션.shop_id = 가게.id
WHERE 미션수행.user_id = '로그인한_사용자_id'
ORDER BY days_left ASC
    LIMIT 10 OFFSET 100;

```

### 5-1. 홈 화면(페이징 처리 전)
```sql
SELECT 가게.name,
       미션.content,
       미션.point
FROM 미션
         JOIN 가게   ON 미션.shop_id = 가게.id
         JOIN 지역   ON 가게.location_id = 지역.id
         JOIN 회원   ON 지역.id = 회원.location_id
WHERE 회원.id = '로그인한_사용자_id'
ORDER BY 미션.id ASC
    LIMIT 10;

```

### 5-2. (커서 기반 페이징)
```sql
SELECT 가게.name,
       미션.content,
       미션.point
FROM 미션
         JOIN 가게   ON 미션.shop_id = 가게.id
         JOIN 지역   ON 가게.location_id = 지역.id
         JOIN 회원   ON 지역.id = 회원.location_id
WHERE 회원.id = '로그인한_사용자_id'
  AND 미션.id > 10
ORDER BY 미션.id ASC
    LIMIT 10;


```