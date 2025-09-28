
# 3주차 미션 API 설계
참고 : 미션의 상태를
IN_PROGRESS,
COMPLETED,
NOT_STARTED,
EXPIRED 네가지로 설정.

## 1.홈 화면
### (1-1) 개별 리소스 API 호출
장점 : 리소스 단일 책임 원칙, 재사용성 높음.<br>
단점 : 홈 진입 시 네트워크 요청이 많아짐. (모든 리소스들에 대해 각각 요청을 보내기 때문)<br>

### (1-2) 한번에 묶은 API
장점 : 네트워크 요청 최소화<br>
단점 : 데이터 구조가 화면 종속적 → 재사용 어려움, 관리 잘못하면 API가 화면마다 늘어남<br>

### (1-3) 개별로 설계
- 사용자의 포인트<br><br>
[API Endpoint] <br>
GET /members/me <br><br>
[Request Header] <br>
Authorization : Bearer {accessToken}


- 미션 N/10 부분<br><br>
[API Endpoint] <br>
GET /members/me/missions<br><br>
[Query String] <br>
region = 안암동<br>
status = COMPLETED<br><br>
[Request Header] <br>
Authorization : Bearer {accessToken}


- 현재 지역 도전가능 미션<br><br>
  [API Endpoint] <br>
  GET /members/me/missions<br><br>
  [Query String] <br>
  region = 안암동<br>
  status = NOT_STARTED<br><br>
  [Request Header] <br>
  Authorization : Bearer {accessToken}

### (1-4) 한번에 묶어서 설계
[API Endpoint] <br>
GET /home <br>

[Query String]
region = 안암동<br>

[Request Header] <br>
Authorization : Bearer {accessToken}





## 2. 마이 페이지 리뷰 작성
### (2-1) 리뷰 생성
[API Endpoint] <br>
POST /restaurants/{restaurantId}/reviews  <br>

[Request Body] <br>
{<br>
"content" : "맛집이에요~" <br>
"rating" : 4.5 <br>
}<br>

[Request Header] <br>
Authorization : Bearer {accessToken}


### (2-2) 리뷰 이미지 등록
리뷰 생성 이후에 reviewId를 받아와서 진행 <br>
사진이 여러개인 경우 여러개 묶어서 보내기 or 여러번 요청 <br>
=> 여러번 요청하는 방식으로 구현<br>

[API Endpoint] <br>
POST /reviews/{reviewId}/photos  <br>

[Request Body] <br>
{ "photo_url" : "~~~~~~~"}<br>

[Request Header] <br>
Authorization : Bearer {accessToken}








## 3. 미션 목록 조회

[API Endpoint] <br>
GET /members/me/missions <br>

[Query String] <br>
status =  IN_PROGRESS(진행중) or COMPLETED(완료)

[Request Header] <br>
Authorization : Bearer {accessToken}






## 4. 미션 성공 누르기

### (4-1) 미션 상태를 성공으로 변경
[API Endpoint] <br>
PATCH /members/me/missions/{missionId}<br>

[Request Body] <br>
{<br>
"status" : "COMPLETED"<br>
}<br>

[Request Header] <br>
Authorization : Bearer {accessToken}






## 5. 회원 가입 하기 (소셜 로그인 고려X)
[API Endpoint]<br>
POST auth/members/signup<br>

[Request Body] <br>
{ <br>
"name" : "박영준"<br>
"gender" : "Male"<br>
"birth" : "2002-03-20"<br>
"address" : "부평구"<br>
"detail_address" : "부평6동 ~"<br>
"category" : [1,2,3,5]<br>
}<br>


