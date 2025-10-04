# 홈 화면 조회
## end point
- /users/me/home

## method
- GET

## Query String

- region=안암동

## Request Header

- Authorization: Bearer <access_token>

## Request Body

```json
필요 없음.
```

## Response Body
```JSON
{
		"completed_mission" : 7,
		"missions" : [
					{
							"mission_id" : 1,
							"mission_description" : "10000원이상시어쩌구",
							"end_days" : 7,
							"status" : "PENDING",
							"store" : {
										"store_id" : 1,
										"store_name" : "반이학생마라탕",
										"store_type" : "중식당",
							}
					},
					{
							"mission_id" : 2,
							"mission_description" : "10000원이상시어쩌구",
							"end_days" : 7,
							"status" : "PENDING",
							"store" : {
										"store_id" : 2,
										"store_name" : "반이학생마라탕",
										"store_type" : "중식당",
							}
					}
		]
}
```

# 마이페이지 리뷰 작성
## end point
- /users/reviews/{store-id}

## method
- POST

## Query String

- 필요 없음

## Request Header

- Authorization: Bearer <access_token>
- Content-type: multipart/form-data

## Request Body
| key | value |
| --- | --- |
| content  | 맛있었어요~ |
| star_rating | 5 |
| photo | 파일url |

## Response Body
```JSON
{
		"status" : "success",
		"code" : 201,
		"message" : "리뷰가 생성되었습니다"
}
```

# 미션 목록 조회(진행 중, 진행 완료)
## end point
- /users/missions

## method
- GET

## Query String

- status=pending
- status=completed

## Request Header

- Authorization: Bearer <access_token>

## Request Body

```json
필요 없음.
```

## Response Body
```JSON
{
		"missions" : [
				{
							"mission_id" : 1,
							"mission_description" : "10000원이상시어쩌구",
							"end_days" : 7,
							"status" : "PENDING",
							"points" : 500,
							"store" : {
										"store_id" : 1,
										"store_name" : "반이학생마라탕",
										"store_type" : "중식당",
							}
				},
				{
							"mission_id" : 2,
							"mission_description" : "10000원이상시어쩌구",
							"end_days" : 7,
							"status" : "COMPLETED",
							"points" : 500,
							"store" : {
										"store_id" : 3,
										"store_name" : "반이학생마라탕",
										"store_type" : "중식당",
							}
				},
				...
		]
}
```

# 미션 성공 누르기
## end point
- /users/missions/{mission-id}/complete

## method 
- PATCH

## Query String
- 필요 없음.

## Request Header
- Content-type: application/json

## Request Body

```json
{
		"status" : "COMPLETED"
}
```

## Response Body
```json
{
		"status" : "success",
		"code" : 200,
		"message" : "성공이 반영되었습니다."
}
```

# 회원 가입 하기
## end point
- auth/users/signup

## method
- POST

## Query String

- 필요 없음

## Request Header

- Content-type: application/json

## Request Body

```json
{ 
		"id" : "zeoueon",
		"password" : "1229",
		"name" : "윤서연",
		"gender" : "여",
		"birth" : "2004-01-03",
		"address" : "인천광역시 어쩌구",
		"preference_food" : [
						"한식",
						"일식"
		]
}	
```

## Response Body
```json
{
		"status" : "success",
		"code" : 201,
		"message" : "회원 가입이 완료되었습니다.",
		"token" : {
				"refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0b2tlbl90eXBlIjoicmVmcmVzaCIs
				ImV4cCI6MTc1NjM2MzYyMSwiaWF0IjoxNzU1NzU4ODIxLCJqdGkiOiI5OGYxY2Y2OGRjNDg0N2Y4OGIyMDRlODE3NDFiZDg5MCIsInVzZXJfaWQiOjF9.woPVeuH-IMvx3uK_Cqlbr3uH3XKR7eik_CVmKsMOR3c",
		    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0b2tlbl90eXBlIjoiYWNjZXNzIiw
		    iZXhwIjoxNzU1NzYyNDIxLCJpYXQiOjE3NTU3NTg4MjEsImp0aSI6ImMwNGM1ODc4ZjI3OTQyYTFiN2YxZjE4NDViYTk1NjI1IiwidXNlcl9pZCI6MX0.nFClYelMz33uS9OQu6QgteAvkWQImmN07PJV-SiiL3I"
	   }
}
```

## End point에 무조건 명사만 들어가야 할까?

### 자원 VS 컨트롤 자원

- 자원이란?
    
    → RESTful API의 핵심 개념 중 하나로, 웹에서 고유하게 식별할 수 있는 것을 의미한다.
    
- **컨트롤 자원이란?**
    
    → 특정 자원에 대한 조작이나 상태 변화를 관리하기 위해 디자인된 API 엔드포인트이다. 이는 자원의 생성, 조회, 수정, 삭제와 같은 표준 작업을 넘어서, 자원의 특정 행위나 상태 관리를 위한 동작을 정의한다.
    
    → 컨트롤 자원은 일반적으로 **동사**를 사용하여 표현할 수 있다.
    
- “미션 성공 누르기” API에서
    - “성공 누르기” 행위는 자원에 대한 CRUD와 같은 표준 작업이 아닌 특정 상태를 관리하기 위한 동작이다.
    - 따라서 “completion”이 아닌, “**complete**”를 사용해 end point를 구성할 수 있따!!