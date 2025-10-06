- 미션내용
    
    로그인 필요한 기능
    
    Request Header
    
    Authorization : Bearer {accessToken}
    
    3. 회원탈퇴   post
    
    /api/auth/withdraw   
    
    ```json
    {
     "password" : "string"
    }
    ```
    
    4. 로그아웃  post
    
    /api/auth/logout
    
    5. 마이페이지 조회  get
    
    /api/users/me
    
    내 정보 수정  patch
    
    /api/users/me
    
    ```json
    {
    "nickname" : "string"
    }
    ```
    
    6. 내 포인트 내역 조회  get
    
    /api/users/mypoint
    
    7. 내 문의 목록 조회  get
    
    /api/ask
    
    내 문의 상세 조회  get
    
    /api/{askId}
    
    내 문의 작성  post
    
    /api/ask
    
    ```json
    {
    "title" : "string",
    "content" : "content",
    "images" : ["image1",
             "image2",
             ...]
    }
    ```
    
    내 문의에 대한 답변 조회  get
    
    /api/{askId}/{answerId}
    
    8. 내가 진행중인 미션목록 get
    
    api/current-mission
    
    내가 완료한 미션목록 get
    
    api/completed-mission
    
    1. 내 알림설정  patch
    
    api/users/notice
    
    ```json
    {
    "new_event" : "true",
    "review_answer" : "false",
    "ask_answer" : "true"
    }
    ```
    
    1. 리뷰 작성하기  post
    
    api/review
    
    ```json
    {
    "star" : "int",
    "content" : "content",
    "images" : ["image1",
             "image2",
             ...]
    }
    ```
    
    내가 작성한 리뷰 목록 get
    
    api/review 
    
    사장님이 리뷰 답글달기  post
    
    api/{reviewId}/answer
    
    ```json
    {
    "content" : "content"
    }
    ```
    
    로그인 필요없는 기능이라고 보자
    
    - 일단 지금은 헤더에 아무것도 안넣음
    
    1. 로그인  post
    
    /api/auth/login
    
    ```json
    {
    "email" : "example@naver.com",
    "password" : "string"
    }
    ```
    
    2. 회원가입  post
    
    /api/auth/signup
    
    ```json
    {
    "name" : "string",
    "sex" : "male",
    "birth_date" : "date",
    "address" : "어쩌구저쩌구",
    "food_category" : ["한식", "양식"...]
    }
    ```
    
    1. 가게 정보 보기  get
    
    api/{storeId}
    
    2. 가게 리뷰 보기  get
    
    api/{storeId}/review 
    
    3. 가게 미션 보기  get
    
    api/{storeId}/mission
    
    4. 가게 사진 보기  get
    
    api/{storeId}/photo
    
    5. 가게 이름 검색 쿼리필요  get
    
    api/store-search