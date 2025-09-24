# SOLID
SOLID는 객체 지향 설계원칙 5가지
### 1. 단일 책임 원칙 (Single Responsibility Principle)
- 하나의 클래스는 하나의 책임만을 가져야 한다. (=변경 사유가 하나여야 한다)
- 하나의 책임은 추상적일 수 있음.
- 하나의 클래스가 여러개의 책임을 가지면, 유지보수등의 상황에서 책임소재를 명확히 알 수 없음.

### 2. 개방-폐쇄 원칙 (Open/Closed Principle)
- 확장에는 열려 있고, 변경에는 닫혀 있어야 한다.
- 새로운 기능을 추가할 떄 기존 코드의 변경없이 새로운 코드를 추가해 기능 추가를 할 수 있다.

### 3. 리스코프 치환 원칙 (Liskov Substitution Principle)
- 하위 클래스는 상위 클래스의 규약을 따라야함.
- 따라서 상위 클래스(부모)에 하위 클래스(자식)을 할당해도 동작해야 함.
- 이 원칙을 지켜야 객체 지향 특징인 다형성 보장.

### 4. 인터페이스 분리 원칙 (Interface Segregation Principle)
- 하나의 큰 인터페이스보다는 여러 개의 구체적인 인터페이스가 낫다.
- 클라이언트는 자신이 사용하지 않는 메서드에 의존하면 안 됨.
- 즉 클라이언트가 필요한 인터페이스만 제공할 수 있도록 인터페이스를 잘게 나눠라.

### 5. 의존 역전 원칙 (Dependency Inversion Principle)
- 구체 클래스에 의존X, 추상에 의존 O (인터페이스에 의존)
- 인터페이스를 구현한 클래스는 클라이언트의 어떠한 변경도 없이 교체할 수 있음.


<br><br>


# DI
### 1. Dependency Injection란? (의존성 주입)
- 객체가 직접 의존성(객체)을 생성하지 않고 외부(spring container)가 대신 생성 및 주입해줌.
- 객체가 직접 의존성 생성시 결합도가 높아져 유지보수 어려움

### 2. 장점
- 결합도가 낮아져 교체가 쉬움
- 재사용성 (다양한 구현체로 대체 하면서 기존 코드를 재사용)
- 확장성, 유지보수성

### 3. 단점
- 구현 복잡도 증가 (작은 프로젝트에서 인터페이스와 구현체 모두 필요)


<br><br>


# IoC
Inversion of Control (제어의 역전)
### 1. IoC란?
- 객체 생성과 의존성 관리의 제어권을 개발자가 아닌 외부 컨테이너가 가지는 것.
- 제어권이 개발자 -> 프레임워크.
- IoC의 구현 방법이 Dependency Injection.
- 스프링에서는 ApplicationContext가 IoC 컨테이너의 역할

### 2. 장점 
(DI와 동일)
- 결합도 감소
- 재사용성 (다양한 구현체로 대체 하면서 기존 코드를 재사용)
- 확장성, 유지보수성


<br><br>


# 생성자, 수정자, 필드 주입 차이
DI를 구현하는 세가지 방법
### 1. 생성자 주입 (스프링 권장)
- 스프링이 Bean 생성 시점에 생성자를 호출 하면서 의존관계 주입
- 불변성 보장 
- NULL 값이 들어갈 일 없음
- 테스트 용이성 (Repository를 DB대신 자바내부 자료구조로 구현 및 테스트)


### 2. Setter 주입
- Setter 메서드를 통해 의존관계 주입
- 선택적 의존성에 적합
- 런타임에 변경가능
- NULL 가능성 (객체는 생성되고 setter가 호출되지 않을경우)

### 3. 필드 주입
- 필드에 직접 @Autowired를 통해 주입
- 코드가 간결하고 구현이 편리함
- 불변성X
- 테스트 불편 (테스트 시 직접 값을 넣어줘야함)
- DI 컨테이너에 강하게 결합 (순수 Java객체를 할당하기 어려움)


<br><br>


# AOP
Aspect Oriented Programming (관점 지향 프로그래밍)

### 1. 정의
- 프로그램을 관심사(관점) 단위로 나누어 보는 패러다임
- 보통 프로그램은 핵심 관심사(비즈니스 로직)와 횡단 관심사 (여러 모듈에 걸쳐 반복되는 기능)으로 구성된다.
- AOP는 이 두 관심사를 분리하여 개발하고 런타임이나 컴파일 단계에서 끼워넣는 방식으로 동작 (스프링에서는 런타임 프록시 기반)

### 2. 장점
- 코드 중복 제거: 로깅, 보안, 트랜잭션 같은 코드 반복 제거
- 유지보수 용이: 공통 정책을 한 곳에서 관리 가능
- 가독성 향상: 핵심 로직은 본연의 역할만 집중

### 3. 런타임 프록시 기반
- 프록시 객체를 만들고 프록시가 메서드 호출을 가로채 부가기능 수행 후 원본객체 호출




<br><br>



# 서블릿
### 1. 정의
- 서블릿(Servlet)이란 동적 웹 페이지를 만들 때 사용되는 자바 기반의 웹 애플리케이션 프로그래밍 기술 
- 서블릿은 웹 요청과 응답의 흐름을 간단한 메서드 호출만으로 체계적으로 다룰 수 있게 해준다.

### 2. 흐름
- 클라이언트가 웹 서버에 HTTP 요청
- 정적 파일은 웹 서버가 직접 읽어서 응답 가능 (html, css, js, image 등)
- 동적 요청 (URL 매핑된 자바 클래스)을 만나면 서블릿 컨테이너에 넘김
- 서블릿 컨테이너에서 매핑된 인스턴스를 찾아 service 메서드 실행
- 서블릿이 비즈니스 로직 실행 및 응답 데이터 작성
- 서블릿이 HttpServletResponse 객체에 쓴 데이터가 HTTP 응답 메시지가 됨
- 웹 서버는 이 응답 메시지를 그대로 브라우저에 전달

### 3. 스프링에 적용
- Spring MVC의 Controller는 서블릿 위에서 동작하는 계층
- Spring의 DispatcherServlet이 FrontController의 역할
- DispatcherServlet이 적절한(매핑된) Controller 호출
- Controller가 결과 반환
- ViewResolver나 HttpMessageConverter로 가공해서 응답으로 돌려줌



<br><br>



# Optional 클래스
자바의 Optional<T> 클래스는 자바 8에서 도입된 Null 안전 처리 유틸리티 클래스

### 1. 개념
- 값이 있을수도 있고 없을수도 있음을 명시적으로 표현
- Optional이라는 객체로 감싸서 NullPointException위험을 줄임 
- Null값이 들어가도 객체가 이를 감싸고 있기 때문에 NPE 방지

### 2. 사용
    public Optional<String> findNameById(int id) {
        if (id == 1) {
            return Optional.of("영준");
        } else {
            return Optional.empty(); // 값 없음
        }
    }

    //orElse
    String name = findNameById(1).orElse("손님");
    String name2 = findNameById(2).orElse("손님");

    //ifPresent
    findNameById(1).ifPresent(n -> System.out.println("회원: " + n));
    findNameById(2).ifPresent(n -> System.out.println("회원: " + n));



<br><br>



# Stream API
### 1. 개념
- 데이터 컬렉션(배열, 리스트, 맵 등)을 SQL 쿼리처럼 선언적으로 처리하는 도구

### 2. 특징
- 재사용 불가
- 데이터 소스를 변경하지 않음 = 새로운 결과를 생성
- 중간 연산 + 최종 연산 구조
- 지연 실행 -> 최종 연산이 호출될 때 실제로 중간 연산이 수행

### 3. 장점
- 코드가 간결해져 가독성
- 컬렉션 가공 로직을 데이터 흐름처럼 표현 가능

### 4. 이해를 위한 간단한 사용예시
    import java.util.*;
    import java.util.stream.*;
    
    public class StreamExample {
        public static void main(String[] args) {
            List<String> names = Arrays.asList("영준", "철수", "영희", "준호");

            // 1) "영"으로 시작하는 이름 필터링
            List<String> result = names.stream()
                    .filter(n -> n.startsWith("영"))   // 중간 연산
                    .map(String::toUpperCase)          // 중간 연산
                    .sorted()                          // 중간 연산
                    .collect(Collectors.toList());     // 최종 연산
    
            System.out.println(result); // [영준, 영희] → 대문자로 [영준, 영희]

        }
    }

### 재사용 불가?
    Stream<String> stream = name.stream();
- 4번 예시에서 stream을 한번 꺼냈다고 가정
- stream.forEach(System.out::println); //최종연산
- stream.forEach(System.out::println); //두번 소비 불가능

### 지연 실행?
- 최종 연산 전까지 실제 연산이 일어나지 않음

        public static void main(String[] args) {
            List<String> names = Arrays.asList("영준", "철수", "영희");

            Stream<String> stream = names.stream()
                .filter(n -> {
                    System.out.println("filter 실행: " + n);
                    return n.startsWith("영");
                })
                .map(n -> {
                    System.out.println("map 실행: " + n);
                    return n.toUpperCase();
                });

            System.out.println("== 아직 최종 연산 전 ==");

            // 최종 연산
            List<String> result = stream.collect(Collectors.toList());

            System.out.println("== 최종 연산 후 결과 ==");
            System.out.println(result);
        }
- 위 예제에서 중간 연산만 명시한 부분에서는 아직 stream 자료형으로 실행이 되지않음
- 최종 연산 부분을 실행하면서 모든 중간 연산들이 수행됨
- 이를 통해 불필요한 연산을 줄일 수 있음. (최종 연산이 뭐냐에 따라 중간 과정이 달라질 수 있기 때문에 최종 연산까지 정해진 시점에서 수행하는게 효율적)