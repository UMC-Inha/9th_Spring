- 객체 그래프 탐색

  # 1. 객체 그래프란?

  객체 그래프(Object Graph)는 **객체들이 서로 연관 관계로 연결된 전체 구조**를 의미한다.
  예를 들어,

  ```java
  class Member {
      private Team team;
  }

  class Team {
      private List<Member> members;
  }
  ```

  `Member → Team → Members → ...`처럼 객체들이 서로 연결되어있음.
  이렇게 연결된 전체가 **객체 그래프이다.**2. 객체 그래프 탐색이란?

  ***

  연관 관계를 바탕으로 **점(.)을 따라 필요한 객체까지 접근하는 것**을 말한다.
  예)

  ```java
  member.getTeam().getLeader().getAddress().getCity();
  ```

  Member → Team → Leader → Address → City 순서로 “그래프를 따라가는 행위”가 **객체 그래프 탐색**.

  ***

  # 3. JPA에서 왜 중요한 개념인가?

  ## (1) 지연 로딩(LAZY)과 즉시 로딩(EAGER)

  객체 그래프 탐색 시 JPA는 필요한 객체를 DB에서 꺼내오기 위해 **프록시를 초기화하거나 즉시 조회**함.
  예)

  ```java
  Member member = em.find(Member.class, 1L);
  member.getTeam().getName(); // 여기서 Team은 Lazy였으면 조회 쿼리 실행
  ```

  그래프 탐색을 할 때마다 쿼리가 발생할 수 있음.

  ***

  ## (2) N+1 문제의 핵심

  특히 컬렉션을 탐색할 때 N+1 문제가 쉽게 발생한다.
  예)

  ```java
  List<Member> members = memberRepository.findAll();
  for (Member m : members) {
      System.out.println(m.getTeam().getName()); // 여기서 팀 조회 쿼리 N회 발생 가능
  }
  ```

  Member 리스트 1번 조회 → 각 Member의 Team 조회로 추가 N개의 쿼리
  👉 **총 N+1 쿼리 발생**

  ***

  ## (3) 영속성 컨텍스트 캐시 활용

  객체 그래프를 탐색할 때, 이미 영속성 컨텍스트에 있으면 DB 조회 없이 가져온다
  예)

  ```java
  Team team = em.find(Team.class, 1L); // DB 조회
  member.getTeam(); // 이미 1차 캐시에 있으므로 추가 쿼리 없음
  ```

  ***

  # 4. 객체 그래프 탐색 규칙(JPA에서 특별히 중요함)

  ### “언제든 객체 그래프 끝까지 탐색할 수 있어야 한다”

  JPA는 **연관된 엔티티를 모두 객체로 표현**하고, 개발자는 이를 자유롭게 탐색할 수 있어야 한다는 철학을 가짐.
  그래서 JPA는 다음을 보장함:

  - 연관된 객체가 있으면 객체로 접근 가능해야 한다
  - 필요한 경우 자동으로 프록시 초기화하여 DB 조회

  ***

  # 5. 문제점과 해결 방법

  ### 문제: 무분별한 객체 그래프 탐색 → 쿼리 폭발

  특히 LAZY가 많으면 점 하나 찍을 때마다 쿼리가 나올 수 있다.

  ### 해결 방법

  - **fetch join** 사용
    → 한 번의 쿼리로 필요한 그래프를 미리 당겨오기
  - **EntityGraph**
  - **BatchSize 설정(컬렉션에 효과적)**
  - **DTO Projection 활용**

  ***

  # 6. 예시로 정리

  ### 엔티티 구조

  ```java
  Member → Team → Organization
  ```

  ### 코드

  ```java
  Member m = memberRepository.findById(1L).get();
  String orgName = m.getTeam().getOrganization().getName();
  ```

  이 한 줄에서 다음이 벌어질 수 있음:

  - Team이 Lazy면 `getTeam()` 호출 시 쿼리 1번
  - Organization이 Lazy면 `getOrganization()` 호출 시 쿼리 1번
    결과적으로 **객체 그래프를 탐색하는 행위가 SQL 여러 번 발생**시키는 것.

  ***

  # 📌 결론 요약

  - **객체 그래프 탐색** = 연관된 엔티티를 점(.)으로 계속 접근하는 것
  - JPA는 엔티티 객체들을 통해 자유로운 그래프 탐색을 지원
  - LAZY 로딩, EAGER 로딩, N+1 문제와 깊이 관련
  - 탐색이 많아지면 쿼리 폭발 가능 → Fetch Join 등으로 최적화
  - JPA의 핵심 철학 중 하나가 "끝까지 탐색할 수 있어야 한다"라는 것
