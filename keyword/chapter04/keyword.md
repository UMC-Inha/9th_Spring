- 계층형 구조 vs 도메인형 구조
    
    ## 계층형 구조
    
    ```
    controller
    	- MemberController
    	- StoreController
    	- MissionController
    	- ...
     
    service
    	- MemberService
    	- StoreService
    	- MissionService
    	- ...
     
    domain
    	- Member
    	- Store
    	- Mission
    	- ...
    ```
    
    - **애플리케이션에서 사용하는 계층 별로 패키지를 구성하는 방식.**
    - 보통 Layered Architecture의 컴포넌트 및 관련 요소들이 패키지가 된다.
    - Layered Architecture란?
        - 소프트웨어 시스템을 여러개의 논리적인 계층을 분리하는 방법.
        - 각 게층은 특정한 역할과 책임을 가지며, 상위 계층은 하위 계층을 사용하여 기능을 수행한다.
        1. Presentation Layer(User Interface Layer) : 사용자의의 상호작용을 처리 ex) Service
        2. Application Layer: 비즈니스 로직을 처리하고, 사용자 요청을 해석하여 하위 계층에 전달. ex) Controller
        3. Domain Layer: 시스템의 핵심 비즈니스 규칙과 개념을 포함하며, 데이터 유효성 검증 및 엔터디 관리 등을 수행 ex) Repository
        4. Infrastructure Layer: 기술 종속성이 강한 구현체를 제공하는 계층
    - **계층형 구조의 장점**
        - 프로젝트의 전반적인 이해도가 낮아도, 패키지 구조만 보고 전체적인 구조를 파악할 수 있다.
            
            → 현재 로직이 어떤 흐름으로 흘러가는지 패키지만 보고 알 수 파악할 수 있다.
            
        - 계층 별 응집도가 높아진다.
            
            → 계층 별 수정이 일어날 때, 하나의 패키지만 보면 된다.
            
    - **계층형 구조의 단점**
        - 도메인 별 응집도가 낮다.
            
            → 도메인 별 흐름을 파악하려면, 모든 패키지를 봐야 파악할 수 있다.
            
        - 도메인 관련된 기능이 변경되었을 때, 변경 범위가 크다.
            
            → 하나의 도메인을 수정하려면 여러 패키지에서 변경이 일어날 수 있다.
            
        - 하나의 패키지에 많은 클래스들이 모이게 된다.
        - 사용자의 행위 표현이 어렵다.
            
            → 하나의 기능을 구현하면, 모든 패키지에 흩어져 있게 된다.
            
    
    ## 도메인형 구조
    
    ```
    domain
    	- Member
    		- controller
    		- service
    		- entity
    		- dto
    		- ...
    	
    	- Store
    		- controller
    		- service
    		- entity
    		- dto
    		- ...
    		
    	- Mission
    		- controller
    		- service
    		- entity
    		- dto
    		- ...
    		
    global
    
    ```
    
    - **도메인을 기준으로 패키지를 나눈 방식**
    - 도메인이란?
        
        → 소프트웨어 시스템이 해결하고자 하는 특정 문제 영역 도는 비즈니스 활동 범위
        
    - 프로젝트 전반적으로 사용되는 클래스들(auth, config, BaseEntity 등)은 domain과 같은 레벨에서 global이라는 패키지 안에 구성한다.
    - **도메인형 구조의 장점**
        - 도메인별 응집도가 높아진다. → 도메인의 흐름을 파악하기 쉽다.
        - 도메인과 관련된 기능이 변경되었을 때, 변경 범위가 작다.
        - 사용자 행위별로 세분화해서 표현이 가능하다.
    - **도메인형 구조의 단점**
        - 애플리케이션의 전반적인 흐름을 한눈에 파악하기 어렵다.
        - 개발자의 관점에 따라 어느 도메인에 둘지 애매한 클래스들이 존재한다.
    
    ## 계층형 구조 VS 도메인형 구조
    
    - 계층형 구조는
        - 규모가 작고 도메인이 적은 경우에 많이 사용한다.
        - 도메인이 적으면 하나의 패키지안에 클래스들이 많아질 가능성이 적다.
        - 도메인의 변경이 일어나도, 규모가 작은 만큼 변경 범위가 그렇게 크지 않을 수 있다.
    - 도메인형 구조는
        - 규모가 크고 도메인이 많은 경우에 많이 사용한다.
        - 도메인이 많을 수록 도메인의 응집도가 높은 것이 좋다.
        - 규모가 큰 만큼 사용자 행위 별로 클래스를 분리하는 경우가 많을 수 있다.

- JPA
    
    ## JPA(Java Persistence API)
    
    <aside>
    💡
    
    자바 진영에서 ORM(Object-Relational Mapping) 기술 표준으로 사용되는 인터페이스의 모음이다.
    
    </aside>
    
    - ORM(Object-Relational Mapping) 이란?
        - 애플리케이션 Class와 RDB(Relational DataBase)의 테이블을 매핑하는 것이며, 기술적으로는 애플리케이션의 객체를 RDB테이블에 자동으로 영속화 해주는 것이다.
        - ORM을 이용하면, SQL문이 아닌 Method를 통해 DB를 조작할 수 있으며, 개발자는 객체 모델을 이용하여 비즈니스 로직을 구성하는 데만 집중할 수 있다.
        - 또, 객체 지향적인 코드 작성이 가능하고, 부수적인 코드가 줄어들어 코드의 가독성을 높일 수 있다.
        - 하지만, 복잡하고 무거운 Query는 속도를 위해 별도의 튜닝이 필요하기 때문에 결국 SQL문을 직접 써야하는 경우가 있을 수 있다.
    - 자바 어플리케이션에서 관계형 데이터베이스를 사용하는 방식을 정의한 인터페이스이다.
    - 대표적인 JPA 구현체로는, **Hibernate, OpenJPA**와 같은 것들이 있다.
    
    ## EntityManager
    
    - `@Entity`  어노테이션을 달고 있는 Entity객체들을 관리하며, 실제 DB테이블과 매핑하여 데이터를 조회/수정/저장 하는 중요한 기능을 수행한다.
    - find, persist, remove 등의 메서드가 정의되어 있다.
    - EntityManager는 PerisistenceContext라는 논리적 영역을 두어, 내부적으로 Entity의 생애주기를 관리한다.
    
    ## Spring Data JPA
    
    - Spring 프레임워크에서 JPA에 대한 Repository를 제공하는 것으로, JPA를 이용한 데이터 접근을 일관된 프로그래밍으로 애플리케이션 개발을 용이하게 한다.
    - JPA를 추상화 시킨 인터페이스만 정의하면, Spring Data JPA가 구현체를 자동으로 생성해주는 기능을 제공한다. 이를 통해 CRUC와 같은 중복 코드를 줄일 수 있다.
    - 메소드 이름을 분석하여 SQL 쿼리를 자동으로 생성하는 기능을 제공한다.
    - 페이징과 정렬을 간단하게 적용할 수 있는 기능을 제공한다.
    - JpaRepository<T, ID>
        - JpaRepository를 상속받는 인터페이스를 정의하면, 해당 타입의 엔티티의 쿼리 메서드를 자동을 구현해준다.
        - JpaRepository는 CurdRepository와 PagingAndSortingRepository를 상속 받기 때문에, CRUD 메서드는 물론이고 페이징과 정렬 기능까지 지원한다.
        - 정의한 인터페이스에 원하는 기능을 하는 메서드를 선언하면, Spring Data JPA가 메서드 이름을 분석해서 자동으로 구현체를 생성한다.

- N+1 문제
    
    ## 먼저, 지연 로딩(Lazy Loading)이란?
    
    - JPA가 하나의 Entity를 조회할 때, 연관 관계에 있는 객체들을 필요한 시점에 불러오는 것이다.
    - `@xxToxx(fetch = fetchType.LAZY)` : 해당 객체는 필요할 때 불러오겠다는 선언!
    - 지연 로딩의 원리
        - 엔티티 매니저가 엔티티를 로드할 때, JPA는 참조하고 있는 객체를 실제 엔티티 대신 프록시 객체로 반환한다.
        - 프록시 객체 사용시점까지는 실제 데이터를 로드하지 않는다.
        - 프록시 객체의 메서드 호출 시, 실제 데이터가 로딩되고, 프록시 객체가 초기화 된다.
        - 한 번 프록시 객체가 초기화되면(실제 데이터가 로드되면), 그 이후에는 프록시 객체 대신에 실제 객체가 사용된다.
    
    ## JPA의 N + 1 문제란?
    
    <aside>
    💡
    
    N + 1 문제는 연관 관계가 설정된 엔티티 사이에서 한 엔티티를 조회하였을 때, 조회된 엔티티의 개수(N개)만큼 연관된 엔티티를 조회하기 위해 추가적인 쿼리가 발생하는 문제를 의미한다.
    
    </aside>
    
    - 즉 N + 1에서, 1은 한 엔티티를 조회하기 위한 쿼리의 개수이며, N은 조회된 엔티티의 개수만큼 연관된 데이터를 조화하기 위해 발생되는 추가적인 쿼리의 수를 의미한다.
    
    ## N + 1 문제 해결 방법
    
    - outer join fetching
        1. fetch join : JPQL문법인 fetch join을 사용하자!!
        
        ```sql
        -- INNER JOIN
        select t from Team t join fetch t.members
        
        -- OUTER JOIN
        select t from Team t left join fetch t.members
        ```
        
        ```java
        public interface TeamRepository extends JpaRepository<Team, Long> {
        	
        	@Query("select t from Team t join fetch t.members")
        	List<Team> findAllWithInnerFetchJoin();
        	
        	@Query("select t from Team t left join fetch t.members")
        	List<Team> findAllWithOuterFetchJoin();
        	
        }
        ```
        
        - Spring Data JPA를 사용하면, `@Query`를 이용해서 JPQL을 직접 사용할 수 있습니다.
        
        → SQL join문법과 같이 작용해서, Team 엔티티를 가져올 때 Member엔티티도 함꼐 가져오기 때문에 Member를 가져오는 추가 조회 쿼리가 발생하지 않는다.
        
    - batch & subselect fetching
        - 두 방법 모두 N+1 문제에서 발생하는 추가 조회 쿼리를 없애는 방향이 아니라, 추가 조회 쿼리를 1개의 쿼리로 줄이는 방향으로 문제를 해결한다..
        1. batch fetching 
            - 추가 쿼리로 조회 되는 Entity위에 @BatchSize를 추가하면 Entity가 조회될 때 IN절을 통해 한번에 조회할 수 있다.
            
            ```java
            @Entity
            public class Team {
            		...
            		
            		@OneToMany(mappedBy = "team", fetch = FetchType.LAZY)
            		@BatchSize(size = 3)
            		private List<Member> members = new ArrayList<>();
            		
            		...
            		
            }
            ```
            
            - size를 3으로 설정했기 때문에, Member들을 조회할 때 where절 안에 in (team1, team2, team3)이 들어가 세 개의 팀의 멤버들을 한번에 조회할 수 있다.
            - BatchSize를 전역적으로 구성파일에서도 지정 가능하다.
        2. subselect fetching
            - `@Fetch(value = FetchMode.SUBSELECT)` 을 사용하면, IN절과 서브쿼리를 사용하여 한번에 조회할 수 있다.
            
            ```java
            @Entity
            public class Team {
            		...
            		
            		@OneToMany(mappedBy = "team", fetch = FetchType.LAZY)
            		@Fetch(value = FetchMode.SUBSELECT)
            		private List<Member> members = new ArrayList<>();
            		
            		...
            		
            }
            ```

- 기본 키 생성 전략
    
    ## 직접 할당 방식
    
    - PK를 직접 할당 방식으로 할당하려는 경우, Entity를 생성할 때 Key Column에 `@Id`만 사용해 주어도 된다.
    - 직접 할당 방식을 사용하면, em.persist()로 entity를 저장 하기 전, 직접 기본 키를 할당해주는 코드를 넣어야 한다.
    
    ## 자동 생성 방식
    
    - 자동 생성 방식을 사용할 경우, `@Id`와 `@GeneratedValue`를 사용한다.
    - strategy를 설정해 자동 생성 방식을 선택할 수 있다.
    - ***IDENTITY***
        - Sequence가 아닌, AutoIncrement기능을 이용해 기본 키 값을 자동으로 생성하는 MySQL과 같은 DBMS에서 사용한다.
        - IDENTITY(AutoIncrement)전략은 Data를 DB에 insert하기 전에는 기본 키 값을 알 수 없다.
        - 따라서 Entity에 식별자 값을 할당하려면 JPA는 추가로 DB를 조회해야 한다.
            
            → Statement.getGeneratedKeys() 함수를 사용하면 Data를 저장하면서 생성된 기본 키 값을 가져올 수 있다.
            
        - Entity가 영속성 컨텍스트에 저장되려면, 식별자가 반드시 필요하기 때문에 쓰기 지연이 불가능하다.
        
        ```java
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private long id;
        ```
        
    - ***SEQUENCE***
        - Sequence전략을 지원하는 Oracle, PostgreSQL, DB2, H2 DB와 같은 DBMS에서 주로 사용한다.
        - 별도로 Sequence Object를 DB에 따로 만들어 두고, 하나씩 값을 뽑아서 PK로 할당한다.
        - 하나의 Sequence Object를 여러 객체가 공유할 수 있으며, 이미 한번 할당된 값은 재사용이 불가능하다.
        - Sequence전략은 Entity를 영속성 컨텍스트에 반영하기 전에, DB sequence를 먼저 조회한 후 조회한 식별자를 Entity에 할당하고 영속상태로 저장한다.
        - 따라서 영속성 컨텍스트에 넣을 때 이미 pk를 알고 있기 때문에, 쓰기 지연이 가능하다.
        
        ```java
        @Entity
        @SequenceGenerator(
        	name = "member_seq_generator",
        	sequenceName = "member_seq",
        	initialValue = 1,
        	allocationSize = 50
        )
        public class member {
        	
        	@Id
        	@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "member_seq_generator")
        	private long id;
        	
        	...
        	
        }
        ```
        
        - allocationSize의 default값은 50이다.
            
            → 새로운 객체를 삽입할 때마다 DB를 호출하면 너무 많은  쿼리가 발생하기 때문에, 각 generator마다 미리 50 만큼의 size를 할당 받고, JPA 내부에 캐싱해둔다. 따라서 50번의 INSERT는 DB접근 없이 PK를 가지고 올 수 있다.
            
    - ***TABLE***
        - Key 생성 Table을 사용하는 전략이다.
        - Key 생성 전용 Table을 생성하고, name, value로 사용할 Column을 생성하여 DB Sequence를 흉내내는 전략이다.
        - 값을 조회하면서 Select 쿼리 수행 후, 값 증가를 위해 Update 쿼리를 한번 더 사용하기 때문에 SEQUENCE 전략과 비교해 DB와 한번 더 통신한다.
        - SEQUENCE 전략을 지원하지 않는 DB에서도 사용 가능하다.
        
        ```java
        @Entity
        @TableGenerator(
        	name = "member_table_generator",
        	table = "member_table",
        	pkColumnName = "seq_name",
        	valueColumnName = "next_val",
        	pkColumnValue = "member_seq",
        	allocationSize = 50
        )
        public class Member{
        	
        	@Id
        	@GeneratedValue(strategy = GenerationType.TABLE, generator = "member_table_generator")
        	private long id;
        	
        	...
        	
        }
        ```
        
    - ***AUTO***
        - AUTO전략은 선택한 DB 방언에 따라 ITENDITY, SEQUENCE, TABLE 전략을 자동으로 선택한다.
        - DB의 종류도 많고, 기본 키 생성 방식이 다양하기 때문에, AUTO전략을 사용하면 DB를 변경해도 코드를 수정하지 않아도 되는 장점이 있다.
        - 하지만, 하이버네이트가 늘 올바르 전략을 선택하다는 보장이 없기 때문에 직접 DBMS에 맞는 전략을 지정하는 것이 낫다.
