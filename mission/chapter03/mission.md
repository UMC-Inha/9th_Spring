## API 명세서 작성하기

홈화면, 마이 페이지 리뷰 작성, 미션 목록 조회(진행중, 진행완료), 미션 성공 누르기, 회원 가입 하기

````
API Endpoint : API가 서버에서 리소스에 접근하게 해주는 URL(HTTP 메소드, URL)
ath Variable : 단건 조회, 단 하나 특정 대상 지목
Query String : 게시글 중 이름에 ~~~가 포함된 게시글 조회(검색 조회)
Request Body : json 담아 서버 전송
Request Header : 전송에 관련된 기타 정보(ex : accessToken, content-type)
````

### 1. 홈화면 : 로그인 후 첫 화면, 이미 로그인 한 상태이기 때문에 id 필요없다.

- API EndPoint : GET /api/home
- Path Variable : 이미 지목 되어있기 때문에 필요x
- Query String : 필요x
- Request Body

  ```
  {
      "region_id" = "안암동" 
  }
  ```

- Request Header

  ```
  {
      Authorization : Bearer <token>
      Content-Type : application/json
  }
    
  ```

### 2. 마이페이지 리뷰 작성 : 신규 리소스 생성 , POST 사용
- API EndPoint : POST /api/my-page/reviews
- Path-Variable : 필요x
- Query-String : 필요x
- Requeset-Body : 별점, 리뷰내용, 리뷰사진

  ```json
  {
      "star" : "별점",
      "text" : "리뷰내용",
      "photo" : "리뷰사진"
  }
  ```

- Request Header

  ```
  {
      Authorization : Bearer <token>
  }
  ```

### 3. 미션 목록 조회(진행중 or 진행완료)
- API EndPoint : GET /api/missions
- Path-Variable : x
- Query String (status가 IN_PROGRESS or SUCCESS)

  ```
  GET /missions?status = IN_PROGRESS // 진행중
  GET /missions?status = SUCCESS // 진행완료
  ```

- Request-Body : GET 요청이기 때문에 x
- Request Header

  ```
  {
      Authorization : Bearer <token>
  }
  ```

### 4. 미션 성공 누르기 : 상태 업데이트, POST 사용
- API End Point : POST /api/missions/{mission_id}
- Path-Variable

  ```
  mission_id
  ```

- Query String : x
- Request Body

  ```json
  {
      "status" : "SUCCESS"
  }
  ```

- Request Header

  ```
  {
      Authorization : Bearer <token>
      Content-Type : application/json
  }
  ```


### 5. 회원가입하기
- API EndPoint : POST /api/users/sign-up
- Path-Variable : x
- Query String : x
- Request Body : 이름, 성별, 생년월일, 주소, 선호음식, 이메일

  ```json
  {
      "name" : "이름",
      "gender" : "성별",
      "birthdate" : "생년월일",
      "address" : "주소",
      "favorite_food" : "선호음식",
      "email" : "이메일"
  }
  ```

- Request Header
    ````
    {
         Content-Type : application/json
    }
    ````