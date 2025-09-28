회원 가입
POST /auth/signup

### Request Body

```jsx
{
		"username" : String,
		"acceptedTerms": [ "SERVICE", "PRIVACY", "MARKETING" ],
		"gender" : String,
		"birth" : Date,
		"address" : String, 
		"food" : [
			{
				"name": String
			},
			...
		]
		
}

```

### Response

```jsx
201 Created -> 정상적으로 회원가입 완료
400 Bad Request -> 요청 데이터가 잘못됨
										(request body 데이터 일부 누락, 데이터 타입 불일치)
409 Conflict -> username 중복

```

### Response Body
없습니다 




미션 단건 조회
GET  /api/v1/mission/${id}

### Request Body

```jsx

```

### Response

```jsx
200 OK -> 리소스가 성공적으로 조회됨 
404 Not Found -> 해당 아이디의 미션을 찾을 수 없음
400 Bad Request -> 받는 데이터의 타입이 잘못됨
```

### Response Body

```jsx
{
	"id": Integer,
	"name": String,
	"category": String,
	"countDown": Integer,
	"description": String,
	"point": Integer,
	"status": String
}
```




미션 조회
GET  /api/v1/missions

### **Query Parameter**

location = String

location에 나타낼 mission의 위치를 적어주세요.

status = String

진행중인 미션인 경우 “inProgress”

진행을 완료한 미션인 경우 “finished”

를 적어주세요

### Request Body

```jsx
없습니다 
```

### Response

```jsx
200 OK -> 리소스가 정상적으로 조회됨

400 Bad Request -> 받는 데이터의 타입이 잘못됨
```

### Response Body

```jsx
{
  "missions": [
    { 
		  "id": Integer,
			"name": String,
			"category": String,
			"countDown": Integer,
			"description": String,
			"point": Integer,
			"status": String
	},
    { 
		  "id": Integer,
			"name": String,
			"category": String,
			"countDown": Integer,
			"description": String,
			"point": Integer,
			"status": String
	}...
	  ]
}

```




리뷰 작성
POST  /api/v1/restaurant/${restaurant}

### Request Body

```jsx

{
	"rate": Integer,
	"description": String,
	"profileImage": String
}	
```

### Response

```jsx
201 Created -> 리소스가 정상적으로 만들어짐

400 Bad Requset -> Requst Body에서 잘못된 데이터 형식을 주었다
```

### Response Body

```jsx
{
	"reviewId"" Integer
}
```

미션 성공 요청
POST  /api/v1/missions/success

### Request Body

```jsx

{
	"missionId": Integer
}
```

### Response

```jsx
200 OK -> 성공적으로 인증이 되었다

400 BAD REQUEST -> 인증이 되지 않았다 
```

### Response Body

```jsx
없습니다 
```
