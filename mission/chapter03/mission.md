# 📑 API 명세서

## 🍔 Drone API

| 기능 | 설명 | 메소드 | API Endpoint | Request Header | Request Body | Query String | Path Variable | Response |
|------|------|--------|--------------|----------------|--------------|--------------|---------------|----------|
| 홈 화면 | 유저 정보 조회 | GET | `/umc/user/info` | `Authorization: Bearer {token}`<br>`Content-Type: application/json` | - | - | - | ```json { "region": "서울", "completedMissions": 5, "points": 1200 } ``` |
| 홈 화면 | 지역별 미션 조회 | GET | `/umc/missions` | `Authorization: Bearer {token}`<br>`Content-Type: application/json` | - | `region={user.region}&page={page}&size={size}` | - | ```json [ { "missionId": 1, "name": "미션1", "content": "내용1" }, { "missionId": 2, "name": "미션2", "content": "내용2" } ] ``` |
| 마이 페이지 | 포인트 내역 조회 | GET | `/umc/user/point` | `Authorization: Bearer {token}`<br>`Content-Type: application/json` | - | - | - | ```json [ { "point": 100, "missionId": 1, "missionName": "미션1" }, { "point": 200, "missionId": 2, "missionName": "미션2" } ] ``` |
| 마이 페이지 리뷰 작성 | 리뷰 작성 | POST | `/umc/shop/{shop-id}/review` | `Authorization: Bearer {token}`<br>`Content-Type: application/json` | ```json { "userId": "지금 로그인한 유저의 id", "rating": 5, "comment": "맛있어요!", "images": [ "https://cdn.myapp.com/images/review/file1.jpg", "https://cdn.myapp.com/images/review/file2.jpg" ] } ``` | - | `shop-id`: 리뷰가 쓰여질 shop | ```json { "reviewId": 123, "status": "success" } ``` |
| 미션 목록 조회 | 진행 중인 미션 목록 조회 | GET | `/umc/missions` | `Authorization: Bearer {token}`<br>`Content-Type: application/json` | - | `mission.status=ongoing&page={page}&size={size}` | - | ```json [ { "missionId": 1, "name": "미션1", "content": "내용1" } ] ``` |
| 미션 목록 조회 | 진행 완료된 미션 목록 조회 | GET | `/umc/missions` | `Authorization: Bearer {token}`<br>`Content-Type: application/json` | - | `mission.status=completion&page={page}&size={size}` | - | ```json [ { "missionId": 2, "name": "미션2", "content": "내용2" } ] ``` |
| 미션 성공 | 미션 성공 누르기 | POST | `/umc/missions/{mission-id}/complete` | `Authorization: Bearer {token}`<br>`Content-Type: application/json` | ```json { "userId": "지금 로그인한 유저의 id" } ``` | - | `mission-id`: 성공 요청할 미션 | ```json { "authCode": "ABC123" } ``` |
| 회원 | 회원 가입 하기 | POST | `/umc/auth/users/login` | `Content-Type: application/json` | ```json { "email": "abba210@naver.com", "password": "1234", "name": "배민", "gender": "남", "birth": "2001-02-10", "address": "부천시 상일로72", "preferredFoods": ["한식", "치킨", "일식"] } ``` | - | - | ```json { "userId": 1, "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." } ``` |
