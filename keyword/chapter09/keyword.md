- 객체 그래프 탐색


    엔티티와 엔티티가 연관관계로 연결이 되어 있을때 연결을 따라가면서 객체에 계속 접근하는 것을 의미한다 
    
    예시
    
    ```jsx

    // MemberMission 이라는 객체 안에 Mission과 Member 이 존재 하는 경우
    
    memberMission.getMember().getName()
    
    memberMission.getMember().getAge()
    
    memberMission.getMission().getTitle()

    ```
    
    장점
    
    SQL 조인을 직접 사용하지 않고 연관된 객체를 객체 지향적인 코드로 작성 할 수 있다 

    
    주의점
    
    - 연관 엔티티를 탐색 할 때 아직 DB 에서 해당 데이터를 불러오지 않았고 fetch 타입이 LAZY 라면 LazyInitializationException이 터진다

        - Lazy 에서는 연관 객체가 실제 객체가 아닌 프록시가 반환된다
        이로 인해 트랜잭션이 끝난 이후 프록시 객체의 필드에 접근하면 실제 데이터를 가져올 수 없어 해당 예외가 터진다

        - 해결책
            - Fetch Join을 통해 사용할 데이터를 미리 넣어둔다

    - 연관관계가 과도하게 깊어지는 경우 에러를 발견하기 힘들 수 있다

            예시
            ```jsx

            memberMission.getMember().getSchool().getGrade().getAverage();

            ```

        - N + 1 문제가 발생할 수도 있다

        - 예시
            - 멤버 미션에서 100개를 조회하는 쿼리를 1개 실행
            해당 멤버미션에서의 상태를 찾을 때 멤버미션의 갯수 만큼 데이터베이스에 접근해 총 100개의 쿼리가 나간다
            이로 인해 총 101개의 쿼리가 발생한다

        - 해결책
            - Fetch Join 등을 사용해 나가는 쿼리 수를 줄인다




        Fetch Join과 객체 그래프 탐색
        
        Fetch Join
        
        - 연관된 데이터를 한번에 효율적으로 조회한다
          - 성능 최적화에 사용된다

          - 쿼리 발생 시점
              - 쿼리 실행 되는 당시에 모든 데이터가 들어온다
                  - N + 1 문제를 방지 할 수 있다

          - 항상 실제 객체를 반환 한다

        - 사용 시기
            - 즉각적인 비즈니스 로직 수행
        
                ```jsx
                
                String address = memberMission.getMember().getAddress();

                ```
        
            - 데이터의 양이 많아 즉시 로딩보다 지연로딩이 효과적인 경우
          - 연관관계가 간단한 경우



        
        객체 그래프 탐색
        
        - 객체 간의 참조 관계로 데이터 접근

          - 쿼리 발생 시점
              - 연관 객체를 사용하는 시점에 쿼리가 나간다 ( 지연 로딩 )

          - 프록시 또는 실제 객체를 반환한다

        - 사용 시기
            - N + 1 문제를 해결하고 싶은 경우
            - LazyInitializeException을 방지하고 싶은 경우
            - 성능을 최적화하고 싶은 경우
            - 연관관계가 복잡한 경우


        -> 대부분의 경우 기본적으로 지연로딩을 사용한다
            이후 필요 따라 Fetch Join을 사용해 성능을 고려한다 