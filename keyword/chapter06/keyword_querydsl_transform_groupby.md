# QueryDSL : transform - groupBy

`transform()`과 `groupBy()`의 조합은 **QueryDSL에서 DTO 내부에 컬렉션(List)이나 중첩 구조를 채워 넣을 때 매우 유용한 기능**이다. 

이 기능은 이름만 보면 SQL의 `GROUP BY`와 비슷해 보이지만, 실제 동작 방식은 다르다.

SQL의 `GROUP BY`가 `COUNT`, `SUM`, `AVG` 같은 **집계 함수**를 수행하는 반면, 

QueryDSL의 `groupBy()`는 **쿼리 결과 행(Row)을 자바 객체 구조로 묶어주는 역할**을 한다.

## 사용 예시를 통해서 알아보자!

### 예시 상황

Member와 Team Entity의 모든 필드를 반환하는 DTO 가 있다고 해보자. 

실제로는 이렇게 하지 않는다.

팀 별로 멤버를 담도록 결과를 반환하고 싶은 상황이다.

```java
@Getter 
@AllArgsConstructor
public class TeamDto {
    private Long id;
    private String name;
    private List<MemberDto> members;
}

@Getter
@AllArgsConstructor
public class MemberDto {
    private Long id;
    private String name;
}
```

### 해결 방안

모든 `Member`와 `Team`의 필드를 한 DTO에 몽땅 담는 대신, **팀별로 멤버를 묶어 반환**하는 구조가 더 적합한 상황이다. 이를 QueryDSL의 `transform()` + `groupBy()`로 깔끔하게 구현할 수 있다.

```java
Map<Long, TeamDto> result = queryFactory
    .from(team)
    .leftJoin(team.members, member)
    .transform(groupBy(team.id).as(
        Projections.constructor(TeamDto.class,
            team.id,
            team.name,
            list(Projections.constructor(MemberDto.class,
                member.id,
                member.name))
        )
    ));
```

### `.transform(groupBy(team.id).as(...))`

**조인 결과처럼 행 단위의 결과를 자바 코드에서 계층 구조로 재구성하는 과정**을 수행한다.

여기서 `groupBy(team.id)`는 말 그대로 **“팀 id를 기준으로 결과를 묶어라”** 는 의미다.

즉, 여러 멤버가 같은 팀에 속해 있다면, 그 팀의 `id`를 키로 하여 하나의 그룹으로 모은다.

이렇게 변환된 결과의 반환 타입이 `Map<Long, TeamDto>`인 이유도 알 수 있다!

- **Key**: `team.id` 는  그룹핑의 기준이 되는 팀의 식별자
- **Value**: `as(...)`에서 정의한 형태로 생성된 `TeamDto` 객체

그리고 `as(...)`는 **“그렇게 묶인 한 팀을 어떤 형태의 객체로 만들 것인지”** 를 정의하는 부분이다.

보통 이 안에서 `Projections.bean()`이나 `Projections.constructor()` 등을 사용하여 DTO를 생성한다.

### 원래 데이터

| team_id | team_name | member_id | member_name |
| --- | --- | --- | --- |
| 1 | A팀 | 10 | 철수 |
| 1 | A팀 | 11 | 영희 |
| 2 | B팀 | 20 | 민수 |

### 변환한 데이터

```java
{
  1L = TeamDto(1, "A팀", [MemberDto(10, "철수"), MemberDto(11, "영희")]),
  2L = TeamDto(2, "B팀", [MemberDto(20, "민수")])
}
```