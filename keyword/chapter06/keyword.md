# QueryDSL에서 FetchJoin 하는 법

## QueryDSL에서의 JOIN

- `innerJoin, leftJoin, rightJoin, fullJoin, fetchJoin`모두 사용 가능하다.
- fetchJoin을 어떻게 적용할까?
    
    → join(조인 대상, 별칭) 함수 뒤에 `.fetchJoin()` 를  붙여주면 fetchJoin 적용됨!!
    

## 예시) MemberMission 테이블에서 Member를 같이 가져오도록!

```java
QMember member = QMember.member;
QMemberMission memberMission = QMemberMission.memberMission;

query.select(memberMission, member.name)
		.from(memberMission)
		.innerJoin(memberMission.member, member).fetchJoin() // join 뒤에 fetch를 붙여줌!!
		.fetch();
```

## 🧩 QueryDSL의 DTO Projection

### DTO Projection

- **쿼리 결과로 필요한 필드를 묶어 DTO를 생성하고 쿼리의 결과로 받는 방법**

### Projections.constructor

- DTO의 생성자를 기반으로 projection한다.

```java
public List<MemberDto> getAllMembersById() {
		return query.select(
												Projections.constructor(
																MemberDto.class,
																member.id,
																member.name
																)
												)
									.from(member)
									.fetch();
}
```

- 생성자 파라미터의 순서를 고려해야 한다는 단점이 있다.
- DTO 객체에 `@AllArgsConstructor` 을 사용하면 편리하다.

### Projections.bean

- DTO의 기본 생성자와 setter를 사용해 projection한다.

```java
public List<MemberDto> getAllMembersById() {
		return query.select(
												Projections.bean(
																MemberDto.class,
																member.id,
																member.name
																)
												)
									.from(member)
									.fetch();
}
```

- 생성자의 파라미터의 순서를 고려하지 않아도 되지만, DTO에 setter가 정의되어 있어야 한다.
- DTO의 필드와 필드명이 다를 경우 setter가 호출되지 않기 때문에 as를 사용하여 별칭을 지정해야 한다.
- 보통 DTO는 불변 객체로 사용되기 때문에 불필요한 setter를 정의하면, 값이 변경될 위험이 있다.

### Projections.fields

- 필드 이름이 DTO와 정확히 일치하는 속성과 매핑한다.

```java
 public List<MemberDto> getAllMembersById() {
		return query.select(
												Projections.fields(
																MemberDto.class,
																member.id,
																member.name
																)
												)
									.from(member)
									.fetch();
}
```

- bean과 다른 점은 DTO에 setter를 정의하지 않아도 작동한다.
- setter를 정의할 필요가 없기 때문에 DTO객체의 값이 변경될 위험이 낮다.
- DTO 객체의 필드명과 QClass의 필드명이 동일해야 한다. beans와 마찬가지로 as를 사용하여 별칭을 지정해야 한다.

### Query Projection

- DTO에 생성한 @QueryProjection을 통해 매핑한다.

```java
 public List<MemberDto> getAllMembersById() {
		return query.select(
												new QMemberDto (
																member.id,
																member.name
																)
												)
									.from(member)
									.fetch();
}
```

- DTO 객체 내에  `@QueryProjection` 어노테이션을 사용해서 생성자를 정의한다.

```java
		@QueryProjection
    public MemberDto(Long id, String name) {
        this.id = id;
        this.name = name;
    }
```

- 이때, DTO의 QClass가 생성되며, 컴파일 시점에 에러를 잡아낼 수 있다는 장점을 가지고 있다.
- 하지만 DTO가 결국은 QueryDSL에 대한 의존성을 갖게 된다는 단점이 있다.

## DTO안에 DTO?

- 아마… DTO객체 안에 다른 DTO객체를 필드로 사용하는 경우를 말하는 것 같다!!

```java
public class MemberDto {
		private Long id;
		private String name;
		private String nickname;
		
		@QueryProjection
		public MemberDto(Long id, String name, String nickname) {
				this.id = id;
				this.name = name;
				this.nickname = nickname;
		}
		
		...
		
}
```

```java
public class MemberMissionDto {
		private Long id;
		private MemberDto memberDto; // Dto 객체를 필드로 사용함
		
		@QueryProjection
		public MemberMissionDto(Long id, MemberDto memberDto) {
				this.id = id;
				this.memberDto = memberDto;
		}
		
		...
		
}
```

- 여기서 MemberMissionDto로 projection할 때

```java
return query.select(
										new QMemberMissionDto (
														memberMission.id,
														new QMemberDto (
																	member.id,
																	member.name,
																	member.nickname
																	)
												)
						.from(memberMission)
						.join(memberMission.member, member)
						.where(member.id.eq(?L))
						.fetch();
```

- transform-groupby를 사용해서 원하는 형태로 mapping도 가능함!

## Pagination

- pagination은 검색 결과를 가져올 때 데이터를 쪼개서 일부만 가져오는 기법이다.
- 개발자가 서비스에 특화된 offset과 limit을 설정해서 pagination을 구현할 수 있다.

### Offset paging (QueryDSL)

- Pageable 객체를 사용해서 설정한 offset과 limit 정보를 꺼내 사용할 수 있다.
- 만약 사용자가 조회하는 동안, 새로운 행이 추가된다면 게시글을 중복해서 조회할 수 있다.
- offset값을 크게 설정하면 앞에 있는 모든 데이터를 읽어와야 하기 때문에 성능 저하 문제가 생길 수 있다.

```java
public Page<MemberDto> getAllMembersById(Pageable pageable) {
		List<MemberDto> content = query.select(
																Projections.constructor(
																				MemberDto.class,
																				member.id,
																				member.name
																				)
																)
													.from(member)
													.offset(pageable.getOffset())
													.limit(pageable.getPageSize())
													.fetch();
		
		long totalCount = query.select(member.count())
														.from(member)
														.fetchOne();
														
		return new PageImpl<>(content, pageable, totalCount);
													
	}
```

- QueryDSL은 쿼리 생성의 역할만 담당하기 때문에 Spring Data JPA가 직접 페이지 구현체를 생성해주지 않기 때문에 개발자가 직접 생성해서 반환해야한다.

### Cursor paging (QueryDSL)

- 사용자에게 응답해준 마지막 데이터를 기준으로 다음 값들을 보여주는 방식이다.

```java
public Slice<MemberDto> getAllMembersById(Long cursorId, Pageable pageable) {
		List<MemberDto> content = query.select(
																	Projections.constructor(
																					MemberDto.class,
																					member.id,
																					member.name
																					)
																	)
														.from(member)
														.orderby(member.id.asc())
														.where(cursorId(cursorId) // 커서 적용!!!
														.limit(pageable.getPageSize() + 1)
														.fetch();
														
			boolean hasNext = false;
			if(content.size() > pageable.getPageSize()) { // 다음 데이터 존재
					content.remove(pageable.getPageSize());
					haxNext = true; // N+1 기법 사용
			}
			
			return new SliceImpl<>(content, pageable, hasNext);
}
```

```java
private BooleanExpression cursorId(Long cursorId){
		return cursorId == null ? null : member.id.gt(cursorId);
}
```

- Page객체로 반환하면 count 쿼리를 추가적으로 실행한다.
    
    → cursor paging 기법을 사용할 때에는 전체 행의 수를 조회할 필요가 없기 때문에, 다음 행 존재 여부만 판단하는 Slice 구현체를 생성해 반환하는 방식으로 구현했다.

## transform - groupBy

- 쿼리의 결과를 원하는 결과로 가공하여 `Map<key, value>` 형태로 한번에 반환할 수 있는 기능이다.
- 이때의 groupBy는 SQL의 groupBy와는 의미가 다르며, transform과 함께 사용해야 의미가 있다.
    - groupBy로 사용한 필드는 Key로 매핑되며 as를 통해 매핑될 value값을 선언한다.

```java
@Override
public Map<String, List<Member>> getMembersByFoodName {
		return query.from(memberFood)
								.join(memberFood.member, member)
								.transform(GroupBy.groupBy(memberFood.food.type).as(list(member)));
}
```

### 여기서 transform-groupBy를 사용하지 않으면?!

```java
@Override
public List<MemberFoodDto> getMemberFoodDtos {
		return query.select(
												new QMemberFoodDto(
													member,
													memberFood.type
												)
									.from(memberFood)
									.join(memberFood.member, member).fetchJoin()
									.fetch();
}
```

```java
public Map<String, List<Member>> groupingMembersByFoodName {
		return MemberFoodRepository.getMemberFoodDtos()
																.stream()
																.collect(Collectors.groupingBy(MemberFoodDto::getType,
																Collectors.mapping(
																		MemberFoodDto::getMember,
																		Collectors.toList()
																)));
}
```

- transform과 groupBy를 함께 사용하면 이 코드들을 한번에 구현할 수 있다.!

