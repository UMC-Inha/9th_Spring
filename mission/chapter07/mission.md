# RestControllerAdvice의 장점
RestControllerAdvice의 동작 흐름에 대해서는 키워드에서 설명.

### 1. 관심사 분리
예외처리 로직을 컨트롤러의 메인 로직과 분리할 수 있음

### 2. 코드의 간결성 및 가독성 향상
관심사를 분리했기 때문에 코드의 간결성 및 가독성이 향상됨

### 3. 유지보수 및 확장성
새로운 예외타입 추가 등의 상황에서 각 컨트롤러 코드를 수정하지 않고
@RestControllerAdvice 클래스를 수정하면 됨

### 4. 전역처리
키워드에서 설명한 RestControllerAdvice가 동작하는 흐름을 생각해보면
각 컨트롤러 내부에 예외처리 로직을 두지 않고 전역에서 처리할 수 있고,
이를 통해 프로젝트 구조에 맞게 exception과 관련된 코드들만 모아둘 수 있음.

### 5. @ControllerAdvice + @ResponseBody
= 자동으로 JSON 직렬화 진행해주기 때문에 REST API에서 유용<br>
이와 비슷하게 에러 응답 포멧을 통일하는데에 도움을 줌


# RestControllerAdvice가 없다면?

### 컨트롤러에 try catch 작성으로 에러처리
- 예외처리 로직이 분산되어서 코드가 복잡
- 유지보수가 어려워지고 비용이 증가 (여기저기 찾아가며 try catch 수정)
- JSON 형태의 일관된 응답을 보내려면 직접 모든 catch문에 대해서 ResponseEntity를 생성하는 코드를 일일히 작성해야 함

