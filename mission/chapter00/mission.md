1. **엔티티 추출 작업**

   회원가입에서 나올 엔티티: user, food_category, user_food_reference, terms, user_terms

   홈화면(미션)에서 나올 엔티티: missions, user_missions, stores, regions

   리뷰에서 나올 엔티티: reviews, reviews_photos, reviews_comments

   1:1문의에서 나올 엔티티: customer_inquiries, inquery_photos,inquery_replies

   포인트 관리 엔티티: point_history

2. **필요한 컬럼 작업(구체화 전 정리 작업)**

   ![ERD](./image/chapter00_erd.jpg)
3. **erd cloud 이용하여 관계, 타입 명시**

   ![ERD](./image/before_refactoring.png)
4. **피드백 반영 알람, 가게 영업 시간 테이블 추가**
   ![ERD](./image/chapter01_erd.png)