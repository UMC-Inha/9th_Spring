- RestContollerAdvice
    
    # **RestControllerAdvice란?**
    
    > 프로젝트 전체 컨트롤러에서 발생하는 에러를 한 군데에서 처리해주는 관리자 클래스임.
    > 
    
    즉, 컨트롤러마다 try/catch 넣지 않아도 되고, 공통 처리 로직을 한 곳에서 관리할 수 있다.
    
    ---
    
    # 왜 필요할까?
    
    ### RestControllerAdvice 없으면
    
    - 컨트롤러마다 try-catch 넣어야 함.
    - 동일한 에러 처리 로직이 여기저기 중복됨
    - 유지보수가 어렵고 코드 더러워짐
    
    ### RestControllerAdvice 사용하면
    
    - **전역 예외 처리** 가능
    - **API 응답 형식 통일** 가능
    - **로깅/검증/에러 메시지** 한 곳에서 관리 가능
    - 컨트롤러 코드가 깨끗해짐
    
    ---
    
    # RestControllerAdvice에 자주 쓰는 기능
    
    ## 1) **예외 처리 기능**
    
    컨트롤러에서 던진 예외를 한 곳에서 받아서 응답 만들어줌.
    
    ```java
    @RestControllerAdvice
    public class GlobalExceptionHandler {
    
        @ExceptionHandler(IllegalArgumentException.class)
        public ApiResponse<?> handleIllegalArgument(IllegalArgumentException e) {
            return ApiResponse.onFailure(GeneralErrorCode.BAD_REQUEST, null);
        }
    }
    ```
    
    ➡ IllegalArgumentException이 발생하면
    
    프로젝트 어디서든 **자동으로 이 메서드가 실행된다.**
    
    ---
    
    ## 2) **모든 예외 공통 처리**
    
    예외를 분류하지 못했을 때 사용:
    
    ```java
    @ExceptionHandler(Exception.class)
    public ApiResponse<?> handleException(Exception e) {
        return ApiResponse.onFailure(GeneralErrorCode.INTERNAL_SERVER_ERROR, null);
    }
    ```
    
    ---
    
    ## 3) **특정 패키지/컨트롤러만 적용도 가능**
    
    ```java
    @RestControllerAdvice(basePackages = "umc.domain.member")
    ```
    
    ➡ 멤버 관련 컨트롤러에만 적용
    
    ---
    
    # RestControllerAdvice + @ExceptionHandler 구조 이해
    
    컨트롤러에서 예외 발생 ↓
    
    스프링이 잡음 ↓
    
    적합한 @ExceptionHandler가 있는지 확인 ↓
    
    해당 메서드를 실행해 JSON 응답 반환
    
    ---
    
    # 예시
    
    ```java
    @RestControllerAdvice
    public class GeneralExceptionAdvice {
    
        @ExceptionHandler(GeneralException.class)
        public ResponseEntity<ApiResponse<Void>> handleException(GeneralException ex) {
            return ResponseEntity
                    .status(ex.getCode().getStatus())
                    .body(ApiResponse.onFailure(ex.getCode(), null));
        }
    
        @ExceptionHandler(Exception.class)
        public ResponseEntity<ApiResponse<String>> handleException(Exception ex) {
            BaseErrorCode code = GeneralErrorCode.INTERNAL_SERVER_ERROR;
            return ResponseEntity
                    .status(code.getStatus())
                    .body(ApiResponse.onFailure(code, ex.getMessage()));
        }
    }
    
    ```
    
    ➡ 프로젝트 전체에서 던진 `GeneralException`은 다 여기로 들어옴
    
    ➡ 예상 못한 다른 예외들도 아래 메서드가 처리함
    
    ---
    
    # 결론
    
    `@RestControllerAdvice`는
    
    - **전역 예외 처리기**
    - **공통 응답 포맷 통일**
    - **컨트롤러 코드 깔끔하게 유지**
    - **유지보수 쉬움**
    
    이라는 점에서 스프링 프로젝트에서 거의 필수로 쓰는 기능이다.
    
- lombok
    
    Lombok은 **자바에서 반복적으로 작성해야 하는 보일러플레이트(boilerplate) 코드를 자동으로 생성해주는 라이브러리**
    
    쉽게 말하면,
    
    > “귀찮은 getter/setter, 생성자, toString 같은 코드 직접 안 써도 되게 해주는 자동 코드 생성 도구”
    > 
    
    ---
    
    # Lombok이 왜 필요한가?
    
    ### Lombok 없을 때 (전통 자바)
    
    클래스 하나 만들면 이런 코드 다 써야 함:
    
    ```java
    public class Member {
        private String name;
        private int age;
    
        public Member() {}
    
        public Member(String name, int age) {
            this.name = name;
            this.age = age;
        }
    
        // getter/setter
        public String getName() { return name; }
        public void setName(String name) { this.name = name; }
    
        // toString
        @Override
        public String toString() {
            return "Member(name=" + name + ", age=" + age + ")";
        }
    }
    ```
    
    필요 없는 코드가 너무 많고 보기 불편함.
    
    ---
    
    ### Lombok 사용하면
    
    ```java
    @Getter
    @Setter
    @NoArgsConstructor
    @AllArgsConstructor
    @ToString
    public class Member {
        private String name;
        private int age;
    }
    ```
    
    ➡ 필요한 메서드를 **자동 생성**해줘서 클래스가 엄청 깔끔해짐.
    
    ---
    
    # Lombok의 주요 어노테이션
    
    ## 1) **Getter / Setter**
    
    ```java
    @Getter
    @Setter
    private String name;
    ```
    
    → 자동으로 getName(), setName() 메서드 생성
    
    ---
    
    ## 2) **@NoArgsConstructor / @AllArgsConstructor / @RequiredArgsConstructor**
    
    ### 기본 생성자
    
    ```java
    @NoArgsConstructor
    ```
    
    ### 모든 필드 생성자
    
    ```java
    @AllArgsConstructor
    ```
    
    ### final 필드만 생성자로
    
    ```java
    @RequiredArgsConstructor
    ```
    
    ---
    
    ## 3) **@Builder**
    
    객체를 "빌더 패턴"으로 직관적으로 생성하게 해줌.
    
    ```java
    @Builder
    public class Member {
        private String name;
        private int age;
    }
    ```
    
    사용:
    
    ```java
    Member m = Member.builder()
                     .name("Tommy")
                     .age(20)
                     .build();
    ```
    
    ➡ 실무에서 DTO 만들 때 매우 자주 사용함!
    
    ---
    
    ## 4) **@Data**
    
    ```java
    @Data
    ```
    
    → Getter + Setter + toString + equals/hashCode + RequiredArgsConstructor
    
    (단, entity에서는 추천하지 않음 — equals/hashCode 문제 가능)
    
    ---
    
    ## 5) **@Slf4j (로그 클래스)**
    
    ```java
    @Slf4j
    public class TestClass {
        public void test(){
            log.info("Hello Lombok!");
        }
    }
    ```
    
    → 자동으로 `log` 객체 생성해줌.
    
    ---
    
    # Lombok의 핵심 장점 정리
    
    | 장점 | 설명 |
    | --- | --- |
    | 코드 단순화 | 불필요한 getter/setter 제거 |
    | 가독성 향상 | 핵심 로직에 집중 가능 |
    | 유지보수 용이 | 변경 시 자동 생성 코드가 따라감 |
    | 빌더 패턴 지원 | 가독성 좋은 객체 생성 가능 |
    
    ---
    
    # Lombok 사용 시 주의점
    
    | 주의사항 | 이유 |
    | --- | --- |
    | Entity에서 @Data는 지양 | equals/hashCode, Setter 남발 위험 |
    | 생성자와 Builder 중복 사용 조심 | 혼동될 수 있음 |
    | 롬복이 실제 코드를 만들어주는 건 컴파일 타임 | IDE 설정 필요 |
    
    ---
    
    # 결론
    
    > Lombok = 귀찮은 코드 자동 생성기
    > 
    > 
    > 스프링과 JPA 개발에서 거의 필수로 사용되는 필수 편의 라이브러리다.
    > 
    
- dto 형식 public static VS record 비교하기
    
    # 1. 두 방식 예시로 비교
    
    ## **① 기존 방식 — public static class (Lombok 포함)**
    
    ```java
    public class MissionResDTO {
    
        @Getter
        @AllArgsConstructor
        public static class MissionInfo {
            private Long id;
            private String name;
            private Integer point;
        }
    }
    ```
    
    ---
    
    ## **② record 방식**
    
    ```java
    public record MissionInfo(
            Long id,
            String name,
            Integer point
    ) {}
    ```
    
    ---
    
    # 🔥 핵심 차이 한눈 정리
    
    | 비교 항목 | public static class DTO | record DTO |
    | --- | --- | --- |
    | **불변성(Immutable)** | ❌ 기본적으로 불변 아님 (Setter 가능) | ✅ 완전 불변 객체 |
    | **코드 길이** | 길고 Lombok 필요 | 매우 짧고 Lombok 불필요 |
    | **생성자** | 직접 작성하거나 Lombok 필요 | 자동 생성 |
    | **필드 접근** | getter/setter 메서드 | accessor 메서드(id(), name()) |
    | **Java 버전** | Java 8부터 가능 | Java 16+ 필요 |
    | **상속 가능?** | 가능 | ❌ 상속 불가능 |
    | **JSON 직렬화/역직렬화** | 완벽 지원 | 대부분 지원 (Spring Boot 3+ 완벽 지원) |
    | **JPA Entity로 사용?** | 가능 | ❌ 불가능 |
    | **DTO 목적에 적합성** | 유연함 | 명확하고 깔끔함 |
    
    ---
    
    # 2. DTO로써 어떤 차이가 있나?
    
    ## ① 불변성(Immutable)
    
    **record = 자동 불변 객체**
    
    - 모든 필드가 final
    - setter 없음
    - 스레드 안전
    
    → DTO는 원래 상태가 바뀌지 않아야 하므로 record가 더 이상적임.
    
    **public static class는**
    
    - setter 있으면 값이 계속 바뀔 수 있음 → 사이드 효과 위험
    
    ---
    
    ## ② 코드 간결성
    
    record는 코드가 극단적으로 짧아짐.
    
    ### 기존 방식
    
    ```java
    @Getter
    @AllArgsConstructor
    public static class Info {
        private Long id;
        private String name;
    }
    ```
    
    ### record
    
    ```java
    public record Info(Long id, String name) {}
    ```
    
    → getter/setter/equals/hashCode/toString builder 등 필요 없음
    
    ---
    
    ## ③ DTO 특성에 잘 맞는가?
    
    DTO는 보통 **읽기 전용 데이터 전달용 객체**
    
    그래서 불변성이 기본인 record가 가장 이상적임.
    
    ---
    
    # 3. 언제 public static DTO를 사용해야 할까?
    
    다음 상황에서는 record보다 **static DTO class**가 더 적합함:
    
    ### 1) DTO 안에 Builder 쓰고 싶을 때
    
    record는 빌더 지원이 없음
    
    ### 2) 필드가 너무 많은 경우 → builder가 편함
    
    ### 3) nested DTO 구조를 유지하고 싶을 때
    
    예:
    
    ```java
    public class UserResponse {
        public static class Info {}
        public static class Detail {}
    }
    ```
    
    record는 이러한 구조 표현이 불편함.
    
    ### 4) 일부 필드는 nullable / 일부는 optional 같은 제어가 필요할 때
    
    ### 5) Java 8 프로젝트에서 Lombok만 사용하는 경우
    
    record는 JDK 16+ 필요
    
    ---
    
    # 4. 언제 record DTO를 쓰는게 더 좋은가?
    
    ### ✔ 응답 DTO가 간단하다
    
    ### ✔ 불변 데이터 전달이 목적일 때
    
    ### ✔ Getter만 있으면 충분할 때
    
    ### ✔ Lombok 의존도를 줄이고 싶을 때
    
    ### ✔ 단일 DTO 파일이 더 직관적일 때
    
    ### ✔ 값 비교를 자주 해야 할 때 (record는 equals/hashCode 자동 생성)
    
    ---
    
    # 5. 실제 스프링에서의 사용 권장 패턴
    
    ### **요청 DTO(Request DTO) → public static class 추천**
    
    - 검증(Validation) (@NotBlank 등)
    - Builder 필요 가능성
    - 생성자가 복잡한 경우
    
    ### **응답 DTO(Response DTO) → record 적극 추천**
    
    - 대부분 읽기 전용
    - 구조가 단순함
    - JSON 직렬화 완벽 지원 (Spring Boot 3+)
    
    ➡ 요즘 실무 흐름은
    
    **"응답 DTO는 record로, 요청 DTO는 class로"** 많이 씀.
    
    ---
    
    # 결론
    
    | 상황 | 추천 |
    | --- | --- |
    | **단순한 응답 DTO** | 🟢 record |
    | **불변성 중요** | 🟢 record |
    | **코드 짧게 유지하고 싶음** | 🟢 record |
    | **요청 DTO(검증 필요)** | 🔵 static class (with Lombok) |
    | **Builder 필요** | 🔵 static class |
    | **Java 8/11 쓰는 경우** | 🔵 static class |