계층형 구조 vs 도메인형 구조

계층형 구조

- 기능 또는 역할에  의해서 코드를 계층별로 그룹화하는 구조이다
- 예시
    - 서비스
    - 리파지토리
    - 엔티티
    - dto
    - 컨트롤러
- 장점
    - 전체 구조를 빠르게 파악할 수 있다
    - 단순하고 직관적이다
- 단점
    - 각각의 디렉터리에 클래스들이 몰리게 되어 복잡해진다
    - 프로젝트가 복잡해질 수록 유지보수하기 어려워 질 수 있다

```jsx
com.demo
	- controller
	- service
	- repository
	- entity
	- dto 
```

도메인형 디렉터리 구조

- 비즈니스의 도메인을 중심으로 코드를 그룹화하는 구조입니다
  각 도메인 별로 패키지를 만들며 그 안에서 관련된 컨트롤러, 서비스, 엔티티등이 존재합니다

- global 패키지에는 모든 코드에서 사용하는 설정, 예외 처리, 인증과 같은 코드가 위치하게 됩니다

- 장점
    - 도메인 별로 코드가 분리되어 있기 때문에 코드의 응집도가 높아진다

- 단점
    - 전체적인 구조를 이해하기에 어려울 수 있다
    - 도메인간 공통 기능 분리가 필요하다

```jsx
com.demo
	- user
		- controller
		- service
		- repository
		- entity
		- dto 
		
	- food
		- controller
		- service
		- repository
		- entity
		- dto 
		
	- order
		- controller
		- service
		- repository
		- entity
		- dto 
		
	- global
		- controller
		- service
		- repository
		- entity
		- dto 
```

- JPA
    - JPA는 자바 진영에서 ORM을 위해서 사용하는 기술이다
        - 이를 통해 개발자가 SQL을 직접 쓰지 않아도 자바 코드로 DB를 손쉽게 다룰 수 있게 해준다

    - 엔티티
        - DB 테이블과 1:1로 매핑이 되는 클래스이다
        - @Entity 를 붙여서 사용한다

        - 엔티티의 생명주기
            - 비영속
            - 영속
                - 영속성 컨테이너가 관리하는 상태
            - 준영속
                - 영속성 컨텍스트가 더 이상 관리하지 않는 상태
                - 이후 엔티티를 수정하더라도 update가 되지 않는다 (JPA가 관리하지 않으니까)
                - 메소드
                    - em.detach()
                        - 특정 엔티티만 영속성 컨텍스트에서 분리
                    - em.clear()
                        - 영속성 컨텍스트 전체 초기화(모든 엔티티가 준영속 상태가 된다)
                    - em.close()
                        - 컨텍스트 자체 종료
            - 삭제

        - 엔티티의 병합
            - merge()를 통해 준영속 엔티티를 새로 복사해서 영속 엔티티로 바꾸어 줄수 있다
            - 기존에 있는거를 다시 영속 상태로 만들어주는것이 아니라 복사해서 새롭게 관리를 시작하는 것이다

            ```jsx
            Member m1 = em.find(Member.class, 1L);  // 영속
            em.detach(m1);                          // 준영속 상태
            
            m1.setName("gildong");                   // 변경은 반영 안 됨
            
            Member m2 = em.merge(m1);               // 새로운 영속 엔티티 생성
            
            //여기서 m1은 여젼히 준영속이고
            //m2는 영속 객체이다 
            ```


    - 엔티티 메니저
        - 실제로 DB에 접근하는 JPA 핵심 인터페이
        - 기능
            - persist()
                - 엔티티를 DB에 저장
            - find()
                - 엔티티를 조회
            - remove()
                - 엔티티를 삭제
            - merge()
                - 엔티티를 수정
            - createQuery()
                - JPQL 쿼리 실행
        
    - 영속성 컨텍스트
        - 이는 엔티티 메니저가 관리하는 엔티티 저장소이다
            - 객체를 관리하는 컨테이너
            
        
        - 영속성 컨테이너의 특징
            - 식별자 값이 존재해야 한다
                - @Id로 지정된 기본키를 통해 JPA가 엔티티를 구분한다 따라서 반드시 필요하다
                
            - 1차 캐시
                - Map 구조로 되어 있다
                    - key(식별자) , value (엔티티 객체)
                - em.find() 를 하는 경우 DB에서 찾는 것이 아니라 1차 캐시에서 찾는다
                    - 이를 통해 동일성을 보장할 수 있다
                    - 객체를 == 으로 비교할 수 있다
                    
            - 변경 감지
                - 원본 엔티티의 스냅샷 활용해서 달라진 필드만 골라서 UPDATE를 진행한다
                
            - 쓰기 지연
                - 내부 동작
                    1. em.persist()나 save()를 호출하면 DB에 바로 Insert 하지 않고 쓰기 지연 SQL 저장소에 저장해 둔다
                    
                    2. 트랜잭션이 끝나는 시점에 flush() 호출해서 모아둔 SQL을 한번에 전송한다
                        - flush란?
                            
                            영속성 컨텍스트의 변경된 내용을 데이터베이스에 반영하는 것을 의미한다 
                            
                    3. 실제 commit을 수행한다 
                    
                
                ```jsx
                em.persist(user1); //아직 SQL 실행 X
                em.persist(user2); //아직 SQL 실행 X
                
                em.flush();   // SQL 실행
                tx.commit();  //실제 커밋
                ```
                
            - 지연 로딩
                - 매핑된 엔티티를 바로 불러오는것이 아니라 필요할때만 부른다
                - 그 전까지는 프록시를 반환하고 실제로 그 값이 필요할때 DB를 조회한다
                
                ```jsx
                User user = em. find(User.class, 1L);
                user.getUser().getAddress() //여기서 실제 조회 쿼리가 실행
                ```

- N+1 문제

  한번의 조회 쿼리로 인해서 N번의 추가 쿼리가 발생하는 문제

  예시

    ```jsx
    @Entity
    public class Food {
    	@Id @GeneratedValue
    	Long id;
    	
    	@OneToMany(mappedBy = "food", fetch = FetchType.LAZY)
        private List<Order> orders = new ArrayList<>();
        
    }
    
    @Entity
    public class Order {
        @Id @GeneratedValue
        private Long id;
        private String product;
    
        @ManyToOne(fetch = FetchType.LAZY)
        @JoinColumn(name = "food_id")
        private Food food;
    }
    
    List<Food> foods = em.createQuery("select f from Foods f", Food.class)
                             .getResultList();
    
    for (Food f : foods ) {
        System.out.println(f.getOrders().size());
    }
    
    // 여기에서 select f from Food f 에서 회원 전체 조회가 일어난다
    
    //이후 for 문에서 f.getOrders()를 호출할 떄마다 각 회원의 주문을 가져오게 되어서 N번 SELECT 발생 
    
    ```

  원인

    - 지연로딩으로 인해서 처음에는 프록시를 가져오고 실제로 그 값을 읽을 때마다 쿼리를 실행한다
    - 이로 인해 JPA가 쿼리가 N번 날리게 된다

  해결방법

    - Fetch Join 사용
        - 한번의 쿼리로 연관된 엔티티까지 전부 조인해서 가져온다

        ```jsx
            
        List<Food> foods = em.createQuery(
        		"select f from Foods f join fetch f.orders", Foo.class)
            .getResultList();
        ```

    - Batch Fetch Size 설정
        - 여러 쿼리를 한번에 묶어서 실행할 수 있다
        - 즉, 쿼리 수를 줄여서 성능을 높일 수 있다

        ```jsx
        //application.yml
        
        spring:
        	jpa:
        		properties:
        			hibernate.default_batch_fetch_size: 100
        			
        
        // 해당 설정을 통해 
        select * from orders where food_id in (1,2,3,4,5,6,7,8,9,10);
        
        //위와 같이 여러개의 쿼리를 한번에 묶을 수 있다 
        
        ```


- 기본 키 생성 전략


    기본키 
    
    - @Id로 생성이 가능하다
    - 데이터베이스에서 각 행을 구분할 수 있는 유일한 값이다
    - 기본키를 매핑할 때 직접 할당이나 자동 생성이라는 두가지 방법이 있다
    
    - GenerationType.IDENTITY
        - JPA가 insert 할때 ID를 지정하지 않는다
        - DB가 자동으로 키를 생성한다
            - MySQL의 경우 auto_increment, PostgreSQL의 SERIAl을 사용
        - 즉시 insert 쿼리가 실행된다
            - 쓰기 지연 불가
        
        ```jsx
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;
        ```


- GenerationType.SEQUENCE
    - DB의 시퀀스 객체를 이용해서 PK를 생성한다
    - @SequenceGenerator을 사용한다
        - 속성
            - name
                - 식별자 생성기 이름
            - sequenceName
                - DB에 등록되는 이름
            - initialValue
                - DDL 생성에만 사용
                - 처음 시작하는 값을 지정한다
            - allocationSize
                - 시퀀스를 한번 호출할 때마다 증가하는 수

    - 내부 동작 방식
        - em.persist()를 호출할 때 DB 시퀀스를 통해 식별자를 조회한다
        - 이후 조회한 식별자를 엔티티에 할당하고 엔티티가 영속성 컨텍스트에 들어가게 된다

    ```jsx
    @Entity
    @SequnenceGenerator(
    	name = "mem_seq_generator",
    	sequenceName = "mem_seq",
    	inititialValue =1,
    	aloocationSize = 1
    )
    
    public class Member{
    	@Id
    	@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "mem_seq_generator")
    	private Long id;
    }
    
    ```

- GenerationType.TABLE
    - DB에 키 관리용 테이블을 따로 만들어 사용한다
    - 모든 DB에서 동일하게 동작한다
    - 성능이 느리다

    ```jsx
    @Id
    @GeneratedValue(strategy = GenerationType.TABLE)
    private Long id;
    ```


- GenerationType.AUTO
    - JPA가 사용하는 DB에 따라서 자동으로 선택해준다
    - 개발 초기에는 편하지만, 운영 DB가 바뀔 경우 문제가 발생할 수 있다