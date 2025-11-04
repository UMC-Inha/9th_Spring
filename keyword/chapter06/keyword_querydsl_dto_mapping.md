# QueryDSL에서 DTO 매핑하는 방법

QueryDSL 은 JPQL 처럼 dto 에 대한 처리를 하는 방법이 여러가지다. 따라서 상황과 목적에 맞게 선택해서 적용할 필요가 있다.

## 예시 상황

### Member

```java
@Entity 
class Member { 
	@Id 
	Long id; 
	
	String name;
	
	@ManyToOne 
	Team team; 
}
```

### Team

```java
@Entity 
class Team { 
	@Id 
	Long id; 
	
	String name; 
}
```

### MemberDto

```java
@Getter 
@AllArgsConstructor
public class MemberDto {
  private Long id;
  private String name;
  private String teamName;
}
```


## 1. 생성자 기반 DTO

DTO를 생성과 동시에 그 생성자에 값을 바로 넣어주는 방식이다.

```java
List<MemberDto> list = queryFactory
  .select(Projections.constructor(MemberDto.class,
      member.id,
      member.name,
      team.name))
  .from(member)
  .join(member.team, team)
  .where(team.id.eq(teamId))
  .orderBy(member.id.asc())
  .fetch();
```

이 코드는 **QueryDSL이 쿼리 결과를 가져온 다음**, 그 결과 컬럼들을 **순서대로 해당 DTO의 생성자 파라미터에 넣어주어 인스턴스를 생성**한다.

```java
Projections.constructor(YourDto.class, expr1, expr2, expr3, ...)
```

### 주의 사항

- **생성자 파라미터 순서와 타입이 정확히 일치해야 한다.**
    
    순서가 다르면 런타임에 `IllegalArgumentException` 발생할 수 있다.
    
    따라서 컴파일은 통과하지만 실행 시 깨질 수 있다.
    
- **필드명이 달라도 상관 없다.**
    
    생성자의 멤버 면수 필드의 타입과 순서가 동일하면 된다.
    

- **불변 DTO와 잘 맞습니다.**
    
    세터(setter) 없어도 괜찮다.
    
    `record` 타입 DTO에도 사용 가능하다.
    


## 2. **`Projections.bean()`**

`Projections.bean()`방식은 **생성자를 통해 값을 주입하는 것이 아니라**, **기본 생성자로 생성된 DTO 객체에 `setter`를 사용해 값을 주입하는 방식**이다.

```java
List<MemberDto> result = queryFactory
    .select(Projections.bean(MemberDto.class,
        member.id.as("id"),
        member.name.as("name"),
        team.name.as("teamName")))
    .from(member)
    .join(member.team, team)
    .fetch();
```

이때 **주의해야 할 점**은 `Projections.bean()`이 필드명 기반으로 매핑된다는 것이다.

즉, **DTO의 필드명과 쿼리에서 조회한 컬럼명이 반드시 동일해야 한다**

**.**

만약 이름이 다를 경우, `.as("필드명")`을 사용하여 DTO의 필드명과 동일하게 맞춰줘야 한다.

예를 들어, 위 코드에서는 `team.name`을 DTO의 `teamName` 필드에 매핑해야 하므로 `team.name.as("teamName")`과 같이 별칭(alias)을 지정한다. 즉, 실제 쿼리에서는 `team` 테이블의 `name` 컬럼을 조회하지만, DTO에서는 `teamName`이라는 필드로 값을 받고자 하기 때문에 `as("teamName")`으로 이름을 맞춰주는 것이다.


## 3. `Projections.fields()`

이 방식은 **리플렉션(reflection)**을 활용하는 방식이다. 기본 생성자, 즉 매개변수가 없는 생성자만 있어도 객체를 생성한 뒤, 리플렉션을 통해 각 필드에 값을 채워 넣는 방식으로 동작한다.

```java
List<MemberDto> result = queryFactory
    .select(Projections.fields(MemberDto.class,
        member.id.as("id"),
        member.name.as("name"),
        team.name.as("teamName")))
    .from(member)
    .join(member.team, team)
    .fetch();
```

### 내부 상황

```java
MemberDto dto = new MemberDto(); // 객체 생성

// 리플랙션
Field idField = MemberDto.class.getDeclaredField("id");
idField.setAccessible(true);
idField.set(dto, 1L);
```

### 언제 사용하면 좋은가..?

- DTO가 **세터 없이 getter만 있는 불변 객체**일 때 (단, `final` 필드는 주입 안 됨)
- 조회용 임시 DTO나 Projection 전용 객체를 빠르게 만들고 싶을 때
- `@QueryProjection`을 쓰기 애매할 때 (QueryDSL 의존 피하고 싶을 때)

### 주의 사항

- **DTO 필드명과 컬럼 alias가 다르면 값이 들어가지 않음**
- **`final` 필드에는 값 주입 불가**
    
    런타임에 예외 발생 (`IllegalArgumentException`)
    
- **리플렉션 사용**이라 성능이 약간 더 느릴 수 있지만 대부분의 경우 체감 차이는 미미하다.


## 4. `@QueryProjection`

`@QueryProjection`은 DTO 클래스의 생성자에 어노테이션을 붙이면, QueryDSL이 컴파일 시점에 해당 DTO를 위한 Q타입 클래스를 자동으로 생성해준다. 

이렇게 생성된 Q타입을 통해 쿼리에서는 `new QMemberDto(...)`와 같은 형태로 DTO를 안전하게 생성할 수 있다. 

### DTO에 직접 적용한다!

```java
@Getter
public class MemberDto {
    private final Long id;
    private final String name;
    private final String teamName;

    @QueryProjection
    public MemberDto(Long id, String name, String teamName) {
        this.id = id;
        this.name = name;
        this.teamName = teamName;
    }
}
```

DTO의 생성자에  `@QueryProjection` 을 붙이면 알아서 Q 타입의 클래스를 생성해준다.

```java
List<MemberDto> result = queryFactory
    .select(new QMemberDto(
        member.id,
        member.name,
        team.name))
    .from(member)
    .join(member.team, team)
    .fetch();
```

이 방식을 사용하면 QueryDSL이 **컴파일 타임에 DTO 생성자의 파라미터 타입과 순서까지 검증**한다.

따라서 `MemberDto`의 생성자 시그니처가 변경되면, **컴파일 시점에 즉시 에러가 발생**하여 런타임 이전에 문제를 확인할 수 있다.

### 장점

- **컴파일 시점 검증**
    
    생성자의 파라미터 타입과 개수를 컴파일 단계에서 확인할 수 있다.
    
- **리팩토링 안정성**
    
    DTO 생성자가 변경되면 즉시 컴파일 에러로 감지되어 오류를 미리 방지할 수 있다.
    
- **개발 생산성**
    
    IDE 자동 완성과 타입 힌트를 제공받을 수 있어 코딩이 훨씬 편리하다.
    

### 단점

- **QueryDSL 의존성**
    
    DTO가 `com.querydsl.core.annotations.QueryProjection`에 의존하게 되어, QueryDSL 없는 환경에서는 재사용이 어렵다.
    
- **빌드 설정 필요**
    
    DTO용 Q클래스(`QMemberDto`)를 생성하기 위해 Gradle에 annotation processor 설정이 필요하다.
    
- **계층 의존성 문제**
    
    API 계층의 DTO가 QueryDSL에 종속되면, 계층 간 의존성에 대한 설계적 논의가 필요하다.