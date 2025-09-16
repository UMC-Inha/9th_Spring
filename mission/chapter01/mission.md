# 미션기록

## 1. 리뷰 작성하는 Query

        INSERT INTO review (
        restaurant_id,
        member_id,
        content,
        rating,
        created_at
        )
        VALUES (
        :restaurant_id,
        :member_id,
        :content,
        :rating,
        NOW()
        );
        

## 2. 마이 페이지 화면 쿼리<br>
마이페이지 화면 쿼리 관련하여 member table에 닉네임 컬럼이 없어서 ERD 수정 (nickname추가)

        ```sql
        SELECT nickname, email, phone_number, point
        FROM member
        WHERE id = :memberId;
        ```

위처럼 마이페이지 화면 Query는 생성했지만 memberId 값을 어떻게 가져올 수 있는지 모름

→ 관련내용 공부

일반적으로 쓰이는 방법 중 토큰을 이용한 방법에 대해 공부

JWT (Json Web Token) 같은 인증 토큰을 사용하여 로그인시 사용자에게 토큰을 부여

HTTP는 stateless이기 때문에 로그인 세션을 유지하려면 사용자는 이후 요청마다 Token을 첨부해 로그인 된 상태임을 증명

이 때 토큰의 Payload 부분에 memberId 그리고 이외의 추가 정보들을 첨부될 수 있기 때문에 이 토큰을 이용해 memberId 값을 적절히 넣어 조회



## 3. 내가 진행중, 진행 완료한 미션 모아서 보는 쿼리(페이징 포함)

멤버미션 테이블에 is_complete가 없어서 진행중, 진행완료 판단불가 → ERD수정 (is_complete추가)

미션을 도전하는 경우에만 멤버미션 테이블에 추가하기 때문에 멤버미션 테이블에 없는 경우 도전중, 완료 상태에 포함되지 않음 → 따라서 is_complete만으로 해결가능

포인트, 가게이름, 미션내용을 성공여부(0 or 1)에 따라 SELECT

        
        SELECT
        m.id
        m.reward,
        r.name,
        m.content
        
        FROM member_mission mm
        
        JOIN mission m
        ON m.id = mm.mission_id
        
        JOIN restaurant r
        ON r.id = m.restaurant_id
        
        WHERE mm.member_id = :memberId
        AND m.is_complete = 0 
        -- 커서 페이징 - 첫 페이지에서는 사용하지 않음
        AND (
        		m.created_at < :cursorCreatedAt
        		OR (m.created_at = :cursorCreatedAt AND m.id < :cursorId)
        		)
        ORDER BY m.created_at DESC, m.id DESC
        LIMIT :limit;
        

mission 테이블에 포함된 보상 포인트, 미션내용

restaurant 테이블에 포함된 가게이름에 대한 데이터가 필요해 적절한 조건을 통해 JOIN하였고

그 중 로그인 된 사용자의 데이터를 가져와야 하기 때문에 member_mission 테이블 또한 JOIN하였다.

미션은 완료 된 미션과 진행중인 미션에 대한 탭이 따로 존재하도록 UI가 설계되어 있어 m.is_complete = 0 or 1만 수정하여 두가지 상황에 대해 모두 사용할 수 있다.

미션이 만들어진 날짜를 기준으로 최신순으로 정렬하였고 mission의 id까지 이용해서 페이징을 했다.

:cursorCreatedAt 은 현 페이지 마지막 행 CreatedAt 값

:cursorId 는 현 페이지 마지막 행 id 값

이 두 값을 다음페이지에 넘겨주면 현재 보고있는 페이지 기준 가장 마지막 행 이후를 페이징 할 수 있다.


## 4. 홈 화면 쿼리 (현재 선택 된 지역에서 도전이 가능한 미션 목록, 페이징 포함)

UI를 통해 홈 화면에는 다음과 같은 정보가 표시되야 한다는 것을 확인했다.

위쪽 - 지역당 완료 미션 수 % 10 (10개 채우면 포인트 획득 후 0으로 초기화됨)

아래쪽 - 도전할 수 있는 미션 (선택된 지역, 이미 도전중이거나 완료된 것 제외)

지역당 완료 미션 수를 가져오기 위해서는 미션이 어느 지역에 속하는지에 대한 정보가 필요하고 이에따라 음식점(restaurant) 테이블까지 JOIN을 했다.

        SELECT MOD(COUNT(*),10)
        
        FROM member_mission mm
        JOIN mission m ON m.id = mm.mission_id
        JOIN restaurant r ON r.id = m.restaurant_id
        
        WHERE m.is_complete = 1
        AND r.address = :address 
        --ERD설계에서 지역명(주소)을 address 상세주소를 detail_address로 설정
        --따라서 위의 :address는 지역명(ENUM)이다.
        AND mm.member_id = :memberId


현재 선택된 지역의 미션들 중 이미 도전중이거나 완료된것을 제외해야 하는데 이는 멤버미션 테이블에 자신의 member_id로 존재하는 모든 미션이 완료되거나 도전중이기 때문에 이를 제외하면 된다.

        SELECT
        m.id,
        m.content,
        m.reward,
        m.deadline,
        r.id,
        r.name,
        r.address,
        fc.id,
        fc.name --한식 중식 일식 등 food_category의 name
        
        FROM mission m
        JOIN restaurant r ON r.id = m.restaurant_id
        JOIN food_category fc ON r.food_category_id = fc.id
        
        WHERE r.address = :address
        AND m.id NOT IN (
                        SELECT mm.mission_id
                        FROM member_mission mm
                        WHERE mm.member_id = :memberId
                        )
        AND m.deadline >= NOW()
        
        --커서 첫 페이지에서는 커서 없음							  
        AND (
        		m.deadline > :cursorDeadline
        		OR (m.deadline = :cursorDeadline AND m.id > :cursorId)
        		)
        							  
        ORDER BY m.deadline ASC, m.id ASC
        LIMIT :limit
        
        

mission의 내용(조건), 보상포인트, 마감일

restaurant의 식당명, 주소(지역)

food_category의 카테고리명

을 가져오기 위해 3개의 테이블을 JOIN하고 :address 은 ENUM 타입 지역명이 들어가 특정 지역의 미션에 대해서만 조회한다.

이 중 내가 이미 도전하기 버튼을 눌러 member_mission 테이블에 포함된 미션들을 제외한다.

커서 페이지는 현재 도전을 시도하지 않은 미션임을 고려해 deadline이 짧은 미션