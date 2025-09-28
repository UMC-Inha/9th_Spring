# **회원가입**

### **API Endpoint**

POST/users/signup

### RequestHeader

```json
Content-Type: application/json
```

클라이언트가 RequestBody를 어떤 형식으로 보냈는지 서버에게 알려주는 헤더

application/json이면 “바디가 JSON이니 JSON으로 파싱하세요” 라는 뜻

### RequestBody

```json
{
	"name": "haewon",
	"gender": "female",
	"birth_date":"2003-05-26",
	"address":"인천 미추홀구",
	"email": "godnjs5870@naver.com",
	"phone_num": "010-2772-5870",
	"terms": [{"term_id":1,"is_agree": true},{"term_id":2, "is_agree":true}]
	"food_preferences": {
        "category_ids": [3, 5, 7]
  }
}
```

# 홈화면

## 1) 지역 목록 가져오기

### API Endpoint

GET/regions

### Request Headers

```json
Authorization : Bearer {accessToken}
```

## 2) 지역 진행도 가져오기

### API Endpoint

GET/regions/{regionId}/progress

### Request Headers

```json
Authorization : Bearer {accessToken}
```

### Query String

```json
status = success
```

## 3) 지역의 도전 가능 미션 목록

### API Endpoint

GET/regions/{regionId}/missions

### Request Headers

```json
Authorization : Bearer {accessToken}
```

### Query String

```json
availability=available 
page = 1 
size = 15
```

## 4) 현재 포인트 조회

### API Endpoint

GET/users/me/point

### Request Headers

```json
Authorization : Bearer {accessToken}
```

## 5) 미션 도전(내 미션 생성)

### API Endpoint

POST/user-missions

### Request Headers

```json
Authorization : Bearer {accessToken},
Content-Type: application/json
```

### RequestBody

```json
{"mission_id": 1 }
```

# 미션 목록 조회(진행중, 진행 완료)

### API Endpoint

GET/user-missions

### Request Headers

```json
Authorization: Bearer {accessToken}
```

### Query String

```json
status = in_progress | success
page = 1
size = 15
```

# 미션 성공 누르기 
### API Endpoint

PATCH/user-missions/{userMissionId}

### Request Headers

```json
Authorization: Bearer {accessToken}
Content-Type: application/json
```

### Request Body

```json
{"status": "SUCCESS"}
```

# 마이페이지 리뷰 작성 
## 1) 리뷰 생성

### API Endpoint

POST/reviews

### Request Headers

```json
Authorization: Bearer {accessToken}
Content-Type: application/json
```

### Request Body

```json
{
	"store_id": 123,
	"rating": 5,
	"content": "음식 마시땅~"
}
```

→ 사진은 다음 API로 업로드

## 2) 리뷰 사진 업로드

### API Endpoint

POST/reviews/{reviewId}/photos

### Request Headers

```json
Authorization: Bearer {accessToken}
Content-Type: multipart/form-data
```

이미지 등 바이너리 파일을 전송할 때는 데이터 타입을 명시하는 `Content-Type` 헤더에 `multipart/form-data` 값을 설정해야 한다.

### Request Body

```json
images[0]=<file1>
images[1]=<file2>
images[2]=<file3>
```

## 3) 내가 작성한 리뷰 목록 조회

### API Endpoint

GET/users/me/reviews

### Request Headers

```json
Authorization: Bearer {accessToken}
```

## 4) 리뷰 수정

### API Endpoint

PATCH/reviews/{reviewId}

### Request Headers

```json
Authorization: Bearer {accessToken}
Content-Type: application/json
```

### Request Body

```json
{
	"rating":4,
	"content": "내용바꿈용"
}
```

## 5) 리뷰 사진 삭제(단건)

### API Endpoint

DELETE /reviews/{reviewId}/photos/{photoId}

### Request Header

```
Authorization: Bearer {accessToken}
```