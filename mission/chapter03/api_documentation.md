# API 명세서
API 명세서를 표 형식으로 정리하여 각 엔드포인트와 HTTP 메서드를 표시하였으며, 
요청 Body와 Response 등의 상세 내용은 노션 링크에서 확인할 수 있습니다.

| 분류      | 기능              | Method | Path                                       | Auth |
|-----------|-------------------|:------:|--------------------------------------------|:----:|
| 로그인     | 회원 가입 하기      | POST   | `/api/auth/signup`                         |      |
| 메인 화면  | 지역 조회          | GET    | `/api/regions/resolve`                     |      |
| 메인 화면  | 지역 미션 조회      | GET    | `/api/regions/{regionId}/missions`         |      |
| 메인 화면  | 미션 성공 개수 조회 | GET    | `/api/users/me/missions/success/count`     | ✅   |
| 메인 화면  | 포인트 조회        | GET    | `/api/users/me/points`                     | ✅   |
| 메인 화면  | 알림 개수 조회      | GET    | `/api/users/me/notifications/count`        | ✅   |
| 마이페이지 | 마이 페이지 리뷰 작성 | POST   | `/api/reviews/{restaurantId}`              | ✅   |
| 미션      | 미션 목록 조회      | GET    | `/api/users/me/missions`                   | ✅   |
| 미션      | 미션 성공 누르기    | POST   | `/api/missions/{missionId}/success`        | ✅   |

👉 각 엔드포인트별 **Request/Response 상세 내용**은 [여기에서 확인하세요](https://www.notion.so/makeus-challenge/API-27cb57f4596b80fb9334f4220d26461f?source=copy_link).
