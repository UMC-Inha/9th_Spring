- QueryDSL에서 FetchJoin 하는 법


    QueryDSL에서 fetch join 방식
    
    ```jsx
    //1. 일반 조인인 경우
    @Override
        public List<Member> findAllOrderByNameAsc() {
            return queryFactory
    				    .selectFrom(member)
    				    .join(member.team, team).fetchJoin()
    				    .fetch();
        }
        
    
    //2. left join인 경우 
    @Override
        public List<Member> findAllOrderByNameAsc() {
            return queryFactory
    				    .selectFrom(member)
    				    .leftJoin(member.team, team).fetchJoin()
    				    .fetch();
        }
        
    ```
    
    이처럼 QueryDSL 에서 fetch join을 할때
    
    join 메소드를 호출한 뒤에 마지막에 fetchjoin()을 호출해서 사용한다

- DTO 매핑 방식 (+DTO안에 DTO)


    DTO 매핑 방식을 사용하면 엔티티 대신에 DTO를 조회할 수 있다 
    
    ### 매핑 방식
    
    1. projections.bean()
        1. 이때 반드시 DTO에 기본 생성자와 setter가 필요하다 
        2. 
        
        ```jsx
        @Override
        public List<MemberDto> findAllAsDto(){
        	return  queryFactory
                    .select(Projections.bean(MemberDto.class,
                            member.name,
                            member.email))
                    .from(member)
                    .fetch();
        }
        
        ----------------------------------------
        
        @Getter
        @Setter
        @NoArgsConstructor
        @AllArgsConstructor
        public class MemberDto{
        	private String name;
        	private String email;
        }
        
        ```
        
    2. Projections.fields()
        1. 이때 DTO의 필드명이 SELECT의 컬럼명과 동일해야 한다
        2. setter가 없어도 가능하다
        3. 
        
        ```jsx
        
        @Override
        public List<MemberDto> findAllAsDto(){
        	return  queryFactory
                    .select(Projections.fields(MemberDto.class,
                            member.name,
                            member.email))
                    .from(member)
                    .fetch();
        }
        
        ----------------------------------------
        
        @Getter
        //@Setter   없어도 된다 
        @NoArgsConstructor
        @AllArgsConstructor
        public class MemberDto{
        	private String name;
        	private String email;
        }
        
        ```
        
    3. Projections.constructor()
        1. DTO에 생성자만 있으면 사용할 수 있다
        2. 
        
        ```jsx
        
        @Override
        public List<MemberDto> findAllAsDto(){
        	return  queryFactory
                    .select(Projections.constructer(MemberDto.class,
                            member.name,
                            member.email))
                    .from(member)
                    .fetch();
        }
        
        ----------------------------------------
        
        //@Getter   없어도 된다 
        //@Setter   없어도 된다 
        @NoArgsConstructor
        @AllArgsConstructor
        public class MemberDto{
        	private String name;
        	private String email;
        }
        
        ```
        
    4. @QueryProjection
        1. DTO에 해당 애너테이션 추가
        2. 빌드 하면 해당 DTO의 Q클래스가 자동으로 생긴다 
        3. 장점
            1. 런타임 리플렉션을 사용하지 않아 성능이 뛰어나다
            2. 컴파일 시점에 오류를 찾을 수 있다
            
        
        ```jsx
        @Override
        public List<MemberDto> findAllAsDto(){
        	return  queryFactory
                    .select(new QMemberDto(
                            member.name,
                            member.email))
                    .from(member)
                    .fetch();
        }
        
        ----------------------------------------
        
        public class MemberDto{
        	private String name;
        	private String email;
        	
        	@QueryProjection
        	public MemberDto(String name, String email){
        		     this.name = name;
                this.email = email;
            }
            
            
        }
        ```
        
    
    ## DTO 안에 DTO
    
    이는 쿼리 결과를 객체 안에 객체 구조로 리턴하는 방식이다 
    
    1. Projections.constuctor() 사용
    
    ```jsx
    @Override
    public List<MemberDto> findAllAsDto(){
    
    List<MemberDto> result =  queryFactory
                .select(Projections.constructer(MemberDto.class,
                        member.id,
                        member.name,
                        Projections.constructor(MissionDto.class,
    			                    mission.id,
    			                    mission.name)
    			                    )
                .from(member)
                .join(member.mission, mission)
                .fetch();
    
    }
    
    ----------------------------------------------------
    
    @Getter
    @Setter
    @NoArgsConstructor
    @AllArgsConstructor
    public class MissionDto {
    
        private Long id;
        private String name;
    }
    
    @Getter
    @Setter
    @NoArgsConstructor
    @AllArgsConstructor
    public class MemberDto {
    
        private Long id;
        private String name;
        private MissionDto mission;
    }
    
    ```
    
    1. Projections.fields()
    
    예시 코드
    
    ```jsx
    @Override
    public List<MemberDto> findAllAsDto(){
    List<MemberDto> result = 
    				 queryFactory
                .select(Projections.fields(MemberDto.class,
                        member.id,
                        member.name,
                        Projections.fields(MissionDto.class,
    			                    mission.id,
    			                    mission.name)
    			                    .as("mission") 
    			                    ))
                .from(member)
                .join(member.mission, mission)
                .fetch();
    }
    ```

1. Projections.bean()


    예시 코드
    
    ```jsx
    @Override
    public List<MemberDto> findAllAsDto(){
    List<MemberDto> result = 
    				 queryFactory
                .select(Projections.bean(MemberDto.class,
                        member.id,
                        member.name,
                        Projections.bean(MissionDto.class,
    			                    mission.id,
    			                    mission.name)
    			                    .as("mission") 
    			                    ))
                .from(member)
                .join(member.mission, mission)
                .fetch();
    }
    ```


- 커스텀 페이지네이션


    ## Offset 기반
    
    예시 코드
    
    ```jsx
    @Override
        public Page<Member> findMembersByKeyword(String keyword, Pageable pageable) {
            List<Member> content = queryFactory
                    .selectFrom(member)
                    .where(member.name.containsIgnoreCase(keyword)
                            .or(member.email.containsIgnoreCase(keyword)))
                    .offset(pageable.getOffset())
                    .limit(pageable.getPageSize())
                    .fetch();
    
            Long total = queryFactory
                    .select(member.count())
                    .from(member)
                    .where(member.name.containsIgnoreCase(keyword)
                            .or(member.email.containsIgnoreCase(keyword)))
                    .fetchOne();
    
            return new PageImpl<>(content, pageable, total);
        }
        
       /*
       PageImpl은 Page 인터페이스의 구현체이다
       매개변수로
       1. content 현재 페이지의 데이터 리스트
       2. pageable 요청된 페이징 정보 
       3. total 전체 데이터 개수
       
       이를 통해 스프링이 알아서 페이징 정보를 제공한다
       
     */
        
        
        
        
        
        
     -----------------------------------------------------------------------------
     
     
     @Service
    @RequiredArgsConstructor
    public class MemberService {
    
       private final MemberRepository memberRepository;
    
        public Page<Member> searchMembers(String keyword, int page, int size) {
            Pageable pageable = PageRequest.of(page, size, Sort.by("name").ascending());
    // 이걸로 몇페이지 부터 몇개씩 어떻게 정렬할지에 대한 정보를 넣어둔다 
    
            return memberRepository.findMembersByKeyword(keyword, pageable);
        }
    }
    
    ```
    
    - offset()
        - 시작인덱스를 지정한다
    - limit()
        - 한 페이지 내 크기를 제한한다
    - fetchOne()
        - count 쿼리를 실행한다
        - 전체 개수를 조회한다
        
    - Pageable
        - 스프링이 제공하는 페이징 정보 객체
    - Page<>
        - 스프링 데이터의 표준 Page 결과
    
    - count 쿼리를 직접 작성하기에 느리다
    
    ## Cursor 기반
    
    - 특징
        - 속도가 offset 기반 보다 빠르다
        - 주로 무한 스크롤이 필요한 경우 사용한다
        - 특정 키를 기준으로 페이징 한다
    
    예시 코드
    
    ```jsx
    public List<Member> findNextPage(Long lastId, int pageSize) {
            return queryFactory
                    .selectFrom(member)
                    .where(lastId == null ? null : member.id.lt(lastId))
                    .orderBy(member.id.desc())
                    .limit(pageSize)
                    .fetch();
        }
    ```


- transform - groupBy


    transform - groupby를 통해 데이터를 그룹화 해주고 그  그룹화된 데이터를 특정 형태로 변환하는 역할을 한다 
    
    ```jsx
    Map<Long, List<Member>> result = queryFactory
                    .from(member)
                    .join(mission).on(mission.name.eq(member.name))
                    .transform(
                            groupBy(mission.id).as(list(member))
                    );
                    
                    
    ```
    
    - 해당 코드에서 mission id 로 그룹화를 진행하고
    - 반환 타입은 Map<Long, List<Member>> 로 member 들이 리스트로 나온다
    
    집계 함수
    
    ```jsx
    Map<Long, Long> result = queryFactory
                    .from(member)
                    .join(mission).on(mission.name.eq(member.name))
                    .transform(
                            groupBy(mission.id).as(member.count())
                    );
                    
                    
    ```
    
    - 해당 예시에서 각 미션들을 완료한 사용자들의 수가 나오게 된다
    
    DTO 매핑
    
    ```jsx
    Map<Long, MemberMissionDto> result = queryFactory
                    .from(member)
                    .join(mission).on(mission.name.eq(member.name))
                    .transform(
                            groupBy(mission.id).as(
    		                        Projections.constructor(MemberMissionDto.class,
    					                        member.id, member.title, list(mission))
                            )
                    );
                    
                    
    ```
    
    - 결과를 Dto로 내보낼 수도 있다

- order by null


    해당 기능을 통해서 특정 조건이 있으면 정렬하고 없으면 정렬하지 않게 한다 
    
    → orderby(null) 을 쓰면 orderBy 절이 만들어지지 않는다 
    
    코드 예시
    
    ```jsx
    queryFactory
                    .selectFrom(member)
                    .orderBy(
                            sortByName ? member.name.asc() : null,
                            sortByEmail ? member.email.desc() : null
                    )
                    .fetch();
                    
                    
    ```
    
    - 해당 코드에서 sortByName과 sortByEmail이 false면 정렬은 일어나지 않는다
    - 둘중 하나라도 true 인 경우 해당 정렬 쿼리가 나타난다
    
    ## orderby NULL first/last
    
    - 해당 기능은 orderBy 절에서 null 값을 어디에 배치할지 제어하는 기능이다
    
    예시 코드
    
    ```jsx
    					queryFactory
                    .selectFrom(member)
                    .orderBy(
                           member.name.desc.nullsFirst()
                    )
                    .fetch();
    ```
    
    - 이 경우 멤버의 이름이 null 인 경우 그 행이 맨 위에 위치한다
    
    ```jsx
    	queryFactory
                    .selectFrom(member)
                    .orderBy(
                    
                           member.age.asc.nullsLast()
                    )
                    .fetch();
    ```
    
    - 멤버의 나이가 null 인 경우 맨 밑에 위치한다