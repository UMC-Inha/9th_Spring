- **지연로딩과 즉시로딩의 차이**


    프록시 
    
    - JPA에서 식별자로 엔티티를 조회할 때 EntityManager.find()를 사용한다
    이 경우 실제 사용 여부와 관계없이 해당 데이터베이스를 조회하게 된다
    이를 미루고 싶은 경우 em.getReference를 사용하면 된다
    이 경우 실제 엔티티 객체가 반환되는게 아닌 데이터베이스 접근을 위임한 프록시 객체가 나온다
    - 프록시 클래스는 실제 클래스를 상속 받아서 만들어, 실제 클래스와 겉 모양이 같다
    - 프록시 객체는 실제 객체에 대한 참조를 보관한다, 프록시 객체의 메소드를 호출하면 프록시 객체는 실제 메소드를 호출한다
    
    프록시 객체의 초기화
    
    ```jsx
    Book book = em.getReference(Book.class, 10L);
    System.out.println(book.getTitle());
    
    class BookProxy extends Book{
    	Book target = null;
    	
    	public String getTitle(){
    	
    		if(target == null) {
    			//영속성 컨텍스트에 초기화 요청
    			//1차 캐시에 찾는 엔티티가 있으면 실제 엔티티를 그대로 반환
    			//없다면 DB에서 찾는다
    			//찾은 참조 값은 target에 넣어준다 
    			this.target = ...;
    		}
    		
    		
    		return target.getTitle()
    	}
    ```
    
    1. 프록시 객체에 실제 데이터를 조회하는 메소드를 호출한다
    2. 프록시 객체에 만약 실제 객체가 생성되어 있지 않는다면 영속성 컨텍스트에 실제 엔티티를 생성해달라고 요청한다, 이를 초기화라고 한다
    3. 영속성 컨텍스트는 DB를 조회해서 실제 엔티티 객체를 생성한다 
    4. 프록시 객체는 실제  생성된 엔티티 객체의 참조를 보관한다
    5. 프록시 객체는 해당 참조를 이용해 결과를 반환한다 
    
    - 프록시의 특징
        - 처음 사용할 때만 초기화가 된다
        - 프록시를 초기화한다고 해서 실제 엔티티로 바뀌는 것은 아니다. 그저 프록시 객체를 통해서 실제 엔티티에 접근이 가능할 뿐이다
        - 영속성 컨텍스트에 이미 찾는 엔티티가 있으면 DB를 조회하지 않고 em.getReference()여도 실제 엔티티를 반환한다
        - 초기화는 반드시 영속성 컨텍스트의 도움이 필요하다 따라서 영속성 컨텍스트 밖의 준영속 상태에서 프록시를 초기화하면 문제가 발생한다
    
    앞에서 설명한 프록시 객체를 사용해서 지연로딩이 이루어진다
    
    즉시 로딩
    
    - 엔티티를 조회할때 연관된 엔티티도 함께 조회한다
    - fetch 타입을 EAGER로 설정한다
    
    ```jsx
    Member member = em.find(Member.class, "member1");
    Team team = member.getTeam();
    ```
    
    `em.find(Member.class, "member1");` 에서 회원을 조회하는 순간 팀도 함께 조회가 된다 
    
    이 경우 회원과 팀을 조회하는 쿼리가 2개가 나가는 것이 아니라 **최적화를 위해 대부분 조인 쿼리를 사용해 한번에 나간다** 
    
    - JPA 조인 쿼리
        
        일반적인 경우 조인 쿼리는 외부조인으로 나간다
        
        그 이유는 (위 예시 사용)
        
        - 회원 테이블에서 TEAM_ID 는 NULL 값이 가능하다 따라서 내부 조인을 실행하는 경우 팀에 소속되지 않은 회원이 조회되지 않을 수도 있기에 외부 조인을 사용하는 것이다
        
        그러나 외부조인보다 내부 조인이 성능 면에서 더 유리하다
        
        내부 조인을 사용하고 싶다면 
        
        - 외래 키에 NOT NULL 제약
        - @JoinColumn에 nullable = false
        
        이 두가지 조건을 만족하는 경우 JPA는 내부 조인을 사용한다 
        
    
    지연 로딩
    
    - 연관된 엔티티를 실제 사용할 때 조회한다
    - fetch 타입을 LAZY로 설정한다
    
    ```jsx
    Member member = em.find(Member.class, "member1"); //멤버만 조회
    Team team = member.getTeam(); //프록시 객체 반환
    team.getName() // 실제 객체 사용 
    ```
    
    컬렉션 래퍼
    
    지연 로딩을 구현하기 위한 가짜 객체이다 
    
    - 프록시 와의 공통점과 차이점
        
        공통점
        
        - 지연 로딩을 구현하기 위해서 등장한 가짜 객체이다
        
        차이점
        
        - 프록시는 단일 엔티티 필드를 대신(연관 객체 1개)
            - @ManyToOne, @OneToOne 등..
        - 컬렉션 래퍼는 연관된 엔티티의 집합(리스트, 셋 등)을 대신
            - @OneToMany, @ManyToMany 등..
    
    동작 원리
    
    ```jsx
    Team team = em.find(Team.class, 1L);
    ArrayList<Member> members = team.getMembers(); // 아직 DB 안 감
    
    ```
    
    여기서 members에 실제 ArrayList가 아니라 HibernatePersistentBag와 같은 JPA 내부 래퍼 객체가 저장된다
    
    여기서 add(), size(), 와 같은 메서드가 호출되면 그때 진짜 객체를 찾기 위해서 DB에서 불러온다 
    
    컬렉션 래퍼 EAGER
    
    - 이 경우 항상 외부 조인이 실행된다
        
        프록시를 사용할 경우 NOT NULL이 보장되는 경우 내부조인으로 바뀌지만 컬렉션 래퍼는 모든 상황에서 외부조인이 발생한다


- **JPQL**


    테이블이 아닌 객체를 대상으로 검색하는 객체지향 쿼리이다 
    
    SQL을 추상화하여 특정 데이터베이스에 의존하지 않는다 
    
    해당 JPQL은 SQL로 변환된다
    
    문자열 기반 쿼리로 타입 안전성이 보장되지 않는다 
    
    - 컴파일 시점 오류 검출 불가
    - 이를 보완하기 위해 QueryDSL이 나왔다
    
    SELECT 절
    
    - JPQL 키워드 (SELECT, FROM ,AS) 같은 경우 대소문자를 구분하지 않고 엔티티와 속성은 대소문자를 구분한다
    - JPQL에서 사용하는 이름은 클래스 명이 아닌 엔티티 명으로 @Entity(name = ???)로 지정 할 수 있다
        - 지정하지 않으면 클래스명이 기본으로 사용된다
    - **JPQL에서 별칭은 필수이다**
    
    FROM 절
    
    - 엔티티 이름을 대상으로 한다
    
    WHERE  절
    
    - 엔티티 필드를 기준으로 조건을 만든다
    - 대부분의 연산자가 SQL과 동일하다
    
    JOIN 절
    
    - 조인 예시
        - 내부 조인
            - select m from Member m join [m.team](http://m.team) t
        - 외부 조인
            - select m from Member m LEFT  join [m.team](http://m.team) t
    
    서브쿼리
    
    - FROM 절에서는 불가능
    - WHERE이나 HAVING 절에서는 가능하다
    
    파라미터 바인딩
    
    1. 위치 기반 바인딩
    2. 이름 기반 바인딩
    
    → 이름 기반 바인딩이 선호된다 

- **Fetch Join**


    연관된 엔티티나 컬렉션을 한번에 같이 조회하는 기능이다 
    
    페치 조인 :: = [LEFT | INNER] JOIN FETCH 조인 경로
    
    select m from Member m join fetch m.team
    
    여기서 회원과 팀을 함께 조회한다 
    
    - 페치 조인은 별칭을 사용할 수 없다 (하이버네이트는 가능)
    - 특징
        - 회원과 팀을 지연로딩으로 설정 하더라도 페치 조인을 사용했기에 연관된 객체가 프록시가 아닌 실제 객체로 지연 로딩이 일어나지 않는다
        - 실제 엔티티이므로 회원엔티티가 영속성 컨텍스트에서 분리되어 준영속 상태가 되어도 연관된 팀을 조회할 수 있다
            - 엔티티에 프록시가 아닌 실제 객체가 메모리에 올라왔기에 순수한 자바 객체로서 영속성 컨텍스트가 없어도 자유롭게 그래프 탐색이 가능하다
            - 지연 로딩에서 실제 객체를 불러서 초기화가 된 시점에서 해당 객체를 준영속으로 바꾸면 그래프 탐색이 가능한가???
                
                
                가능하다
                
                이미 메모리에 올라왔기에 초기화를 위해 영속성 컨텍스트가 필요없다 따라서 가능하다 
                
    
    컬렉션 페치 조인
    
    1대다 관계에서 한번의 쿼리로 모두 불러온다
    
    일반조인
    
    SELECT t FROM Team t JOIN t.members m
    
    - 단순히 조인 조건만 존재 실제로 가져오는 Team 엔티티 뿐이다
    - t.getMembers() 에서 지연로딩 발생
    
    SELECT t FROM Team t JOIN FETCH t.members
    
    - Team을 조회하면서 속한 Members 리스트도 한번에 로딩이 된다
    - t.getMembers() 에서 추가 쿼리가 발생하지 않는다
        - 이미 Member이 메모리에 있다
    
    중복제거
    
    DISTINCT 사용 
    SELECT DISTINCT t FROM Team t JOIN FETCH t.members
    저
    
    - 전부 다 다른 행일 텐데 어떻게 중복 제거가 가능 할까?
        
        
        DISTINCT의 단계 구분
        
        1. SQL 단계의 DISTINCT
            
            DB 입장에서 완전히 동일한 행만 제거한다
            
        2. JPA의 엔티티 중복 제거 로직
            
            JPA는 SQL 실행 결과를 받을 때 행 단위가 아니라 **엔티티 단위로 재조립한다** 
            
            이때 SQL 결과에 같은 Team  정보가 여러번 등장하는 경우 “같은 팀”이라고 인식해서 하나의 엔티티 인스턴스로 합친다 
            
        
        즉, DISTINCT의 역할은 **JPA 레벨에서 중복 엔티티를 제거하는 역할이다** 
        
    
    페치 조인의 한계
    
    - 페치 조인 대상에는 별칭을 줄수 없다
    - 둘 이상의 컬렉션을 페치할 수 없다
    - 컬렉션을 조인하면 페이징 API를 사용할 수 없다

- **@EntityGraph**


    엔티티를 조회할 때 연관된 엔티티를 함께 조회하는 기능이다 
    
    엔티티 그래프 
    
    ```jsx
    @EntityGraph(attributePaths = {"team", "orders"})
    List<Member> findAll();
    ```
    
    - 재사용 불가
    - 인라인 사용
    
    Named 엔티티 그래프
    
    - @NamedEntityGraph 로 정의한다
        - name으로 엔티티 그래프 이름 정의
        - attributeNodes로 함께 조회할 속성 선택
        - 재사용 가능
        
        ```jsx
        @Entity
        @NamedEntityGraph(
        		name = "Member.withTeams",
        		attributeNodes = {
        				@NamedAttributeNode("team")
        			}
        )
        
        public class Member {...}
        
        ---------------------------------------------------
        
        @EntityGraph (value = "Member.withTeam")
        List<Member> findAll();
        
        ```
        
    
    특징
    
    - 페이징이 일부 가능하다
    - 재사용이 가능하다
    - JPA 단에서 로딩 그래프를 제어할 수 있다
    - 이미 로딩이 된 엔티티에서는 엔티티 그래프가 적용되지 않는다

- **commit과 flush 차이점**

  플러시

    - 영속성 컨텍스트의 변경 내용을 데이터베이스에 반영한다
    - 플러시 발생시
        1. 변경 감지가 동작해 영속성 컨텍스트에 있는 모든 엔티티를 스냅샷과 비교해 수정된 엔티티를 찾는다, 수정 된 부분은 수정 쿼리를 만들어서 쓰기 지연 SQL 저장소에 등록한다
        2. 쓰기 지연 SQL 저장소에 있는 쿼리를 데이터 베이스에 전송한다
    - 플러시 호출 방법
        - em.flush() 호출
        - 트랜잭션 커밋 시 플러시가 자동 호출된다
        - JPQL 쿼리 실행시 플러시가 자동 호출된다
    - 롤백 가능
    - SQL이 DB로 나가 반영은 되지만  트랜잭션이 끝나지 않아 확정은 되지 않는다

  커밋

    - 트랜잭션을 완료하면서 실제 DB 반영을 확정하는 단계
    - 트랜잭션이 끝날때 자동으로 실행된다
    - 롤백이 불가능하다

  즉 ,

  플러시는 DB에 SQL을 보내는 것

  커밋은 보낸 SQL을 확정짓는 것이다

- **QueryDSL, OpenFeign의 QueryDSL**


    QueryDSL
    
    JPQL을 타입 안전하게 작성하기 위한 자바 전용 쿼리 빌더 라이브러리이다
    
    컴파일시에 문법을 체크할 수 있다
    
    예시
    
    ```jsx
    QMember m = QMember.member
    
    List<Member> result = queryFactory
    	.selectFrom(m)
    	.where(m.age.gt(20)
    				.and(m.username.startWith("길동")))
    	.fetch();
    ```
    
    동작 원리
    
    - 컴파일시 Q클래스 자동 생성
    - 개발자가 Q클래스 필드를 통해 쿼리 작성
    - QueryDSL 내부에서 JPQL로 변환 → SQL로 변환
    
    OpenFeign의 QueryDSL
    
    본래 배포되던 QueryDSL을 OpenFeign이 가져가서 유지 보수 하는 버전이다
    
    기존에 존재하던 취약점에 대해서도 개선이 이루어지고 있다
    
    - QueryDSL의 보안 취약점
        
        공격자가 악의적으로 SQL 인젝션을 발생시켜서 의도하지 않은 행동이 나타날 수 있다 


- **N+1 문제 해결할 수 있는 여러 방안들**


    1. 페치 조인 
        
        SQL 조인을 통해 연관된 엔티티를 한번에 조회하기 떄문에 해당 문제가 발생하지 않는다
        
    2. @BatchSize
        
        연관된 엔티티를 조회할때 지정한 크기 만큼 SQL의 IN절을 통해 조회한다 
        
    3. 하이버네이트 @Fetch(FetchMode.SUBSELECT)
        
        이 경우 여러 부모 엔티티를 조회한 뒤
        
        그 부모들과 연관된 컬렉션을 한번의 서브 쿼리로 묶어서 조회하는 방식이다 
        
        예를 들어
        
        ```jsx
        SELECT * FROM team;                   (부모)
        SELECT * FROM member WHERE team_id=1; 
        SELECT * FROM member WHERE team_id=2;
        SELECT * FROM member WHERE team_id=3;
        ...
        
        //다음과 같이 하나의 팀에 여러 멤버를 조회할때
        
        ---------------------------------
        
        SELECT * FROM team;
        
        SELECT * FROM member 
        WHERE team_id IN (SELECT id FROM team);
        
        // 다음과 같이 서브 쿼리를 사용해서 효율적으로 사용할 수 있다 
        
        ```


- **영속 상태의 종류**


    영속성 컨텍스트
    
    엔티티를 영구 저장하는 환경
    
    영속성 컨텍스트의 특징
    
    - 영속성 컨텍스트는 엔티티를 식별자의 값으로 구분한다
        - 엔티티는 테이블인데 왜 식별자로 구분하지??
            
            JPA에서 @Entity 클래스는 DB 테이블 구조와 1대1로 매핑이 된다. 하지만 실제로는 “하나의 엔티티 객체”는 테이블의 한행을 표현하는 인스턴스이다 
            
            예시)
            
            ```jsx
            @Entity
            class User {
            		@Id
            		private Long id;
            		private String name;
            }
            ```
            
            | id | name |
            | --- | --- |
            | 1 | 지아 |
            | 2 | 지민 |
            
            이렇게 User 테이블이 있을 때 
            
            테이블은 ‘엔티티 클래스’와 매핑되고
            
            그 테이블의 각 행은 ‘엔티티 객체’로 매핑이 된다 
            
            이중에서 영속성 컨텍스트는 엔티티 객체를 관리한다 
            
    - 영속성 컨텍스트 얻을 수 있는 장점
        - 1차 캐시
        - 동일성 보장
        - 쓰기 지연
        - 변경 감지
        - 지연로딩
    
    엔티티의 생명 주기
    
    - 비영속
        - 영속성 컨텍스트와 관련이 없다
        - 순수한 객체 상태이다
    - 영속
        - 엔티티 메니저를 통해 엔티티를 영속성 컨텍스트에 저장한 상태
        - 즉, 영속성 컨텍스트가 관리하는 엔티티를 의미한다
    - 준영속
        - 영속성 컨텍스트가 관리하던 영속 상태의 엔티티가 영속성 컨텍스트로 부터 분리되면 준영속이 된다
        - 관련 메소드
            - detach()
                - 특정 엔티티를 준영속 상태로 전환한다
                - 이를 통해 해당 엔티티와 관련된 1차 캐시와 쓰기 지연 SQL 저장소에 있는 관련 SQL들이 전부 제거 된다
            - clear()
                - 영속성 컨텍스트 초기화
                - 해당 영속성 컨텍스트에 있는 모든 엔티티를 준영속 상태로 만든다
            - close()
                - 영속성 컨텍스트 종료
                - 해당 영속성 컨텍스트에 있는 모든 엔티티를 준영속 상태가 된다
        - 준영속 상태의 특징
            - 비영속에 가깝다
            - 식별자 값을 가지고 있다
            - 지연로딩을 할 수 없다
    - 삭제
        - 엔티티를 영속성 컨텍스트와 데이터베이스에서 삭제한다