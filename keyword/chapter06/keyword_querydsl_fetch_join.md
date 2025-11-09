## QueryDSL에서 FetchJoin 하는 법

Querydsl에서 **fetch join**을 사용하려면 일반적인 `join` 구문 뒤에 `.fetchJoin()`을 붙이면 된다.

fetch join은 연관된 엔티티를 지연 로딩 대신 **즉시 로딩**으로 가져와 **N+1 문제를 방지하고 성능을 향상**시키는 데 도움이 된다.

```java
List<Member> result = queryFactory
    .selectFrom(member)
    .leftJoin(member.team, team).fetchJoin()
    .where(team.name.eq("teamA"))
    .fetch();

```

위 예시처럼 **`.leftJoin(연관 엔티티, alias).fetchJoin()`** 패턴으로 사용하면 된다.

이 방식은 `innerJoin()`이나 `rightJoin()`에도 동일하게 적용할 수 있다.