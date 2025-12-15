# Spring Security
자바/스프링 기반 애플리케이션의 인증과 인가를 위한 프레임워크

### 1. 핵심기능

Authentication : 사용자가 누구인지 확인

Authorization : 인증된 사용자가 특정 리소스에 접근할 권한이 있는지 확인

공격 방어 : CSRF(Cross-Site Request Forgery), XSS, 세션 고정등 일반적인 웹 취약점으로부터 보호

세션 관리 : 인증된 사용자의 정보를 저장하고 관리

### 2. 흐름이해 (세션)

1. 로그인 요청 발생
2. Security Filter Chain 진입
3. 로그인 요청이면 UsernamePasswordAuthenticationFilter가 잡음
4. 인증 요청 객체 생성

    ```java
    new UsernamePasswordAuthenticationToken(
        username,
        password   // 아직 검증 안 됨
    )
    ```

5. AuthenticationManager (interface)에게 위임 실제 구현체는 ProviderManager

    ```java
    authenticationManager.authenticate(authentication)
    ```

6. ProviderMangager 내부에 AuthenticationProvider 선택

   for loop 통해 전달받은 authentication을 supports(authentication) 확인

7. DaoAuthenticationProvider → 인증 로직 수행되는 지점

   UserDetailsService 호출해서 DB에서 사용자 조회, PasswordEncoder로 비밀번호 비교해서 Exception 이나 통과

8. 인증 성공시 → 인증 완료된 Authentication 객체 생성

    ```java
    new UsernamePasswordAuthenticationToken(
        userDetails,
        null,
        authorities
    )
    ```

9. 8에서 생성된 결과를 spring security가 자동으로 저장

    ```java
    SecurityContextHolder.getContext()
        .setAuthentication(authentication);
    ```


### 2. 흐름이해 (토큰)

1. 로그인 요청 발생 (email, pwd만 전달하는 과정)
2. Security Filter Chain 진입
3. JwtAuthFilter 실행

   로그인 시점에서 위 단계는 아무 인증도 하지 않고 그대로 통과

4. 로그인 Controller에서 수동 인증 수행

   직접 구현한 코드로 email 이용해 DB에서 멤버 조회하고 passwordEncoder이용해 password 일치하는지 체크

5. 인증 성공 시 JWT 발급

1. 이후 인증이 필요한 요청 발생
2. Security Filter Chain 진입
3. JwtAuthFilter 실행

   기존 로그인 시점에는 여기서 인증이 일어나지 않았지만 지금은 Authorization 헤더에서 JWT 추출해서 토큰 유효성 검증 후 사용자 정보 추출

4. UserDetailsService로 사용자 조회
5. Authentication 객체 직접 생성
6. SecurityContext에 저장

    ```java
    SecurityContextHolder.getContext()
        .setAuthentication(authentication);
    ```

7. 요청이 끝나면 context는 폐기되고 이후에 다시 요청할 때 다시 JWT 검증

### 3. 차이점

session은 로그인 시점에 인증 후 서버에서 해당 상태를 유지 stateful

따라서 이후 요청에서는 서버에 저장된 정보를 이용

token은 로그인 시점에 인증이 수행된 토큰을 발급 서버에서는 토큰을 유지하지 않음stateless

따라서 이후 요청에서 클라이언트는 토큰을 포함한 요청을 하고 서버는 해당 토큰 유효성만 검증



# 인증(Authentication)과 인가(Authorization)


### 인증 
: 사용자가 누구인지 확인하는 과정

- 로그인
- 신원 확인

### 인가
: 인증된 사용자가 특정 자원에 접근할 권한이 있는지 확인하는 과정

- ROLE 확인
- 권한 확인
- URL/메서드 접근 제어

---

### Spring Security 에서의 인증

- Authentication 객체 생성
- 자격 증명 검증
- Security Context에 저장

### Spring Security에서의 인가

- 이미 인증된 Authentication 객체 사용
- 권한 비교
- 접근 허용 / 거부 결정

---

### 세션 기반

인증은 로그인 시점에 이루어지고

인가는 자원을 요청하는 시점에 이루어짐

### JWT 기반

인증은 로그인 시점에 이루어지고 (워크북 과정 중 email, pwd확인)

요청시점에 토큰을 검증하고 권한을 확인하는 과정이 인가

---

### HTTP 상태 코드

401 Unauthorized : 이름으로 인해 인가되지 않음으로 생각할 수 있지만 인증이 이루어지지 않음을 뜻함 (로그인이 필요한 상황)
403 Forbidden : 인증은 되었으나, 인가가 거부됨 (권한이 해당 자원 접근하기에 부족)

---


# 세션과 토큰
### 세션기반 인증

저장 위치 : 서버의 메모리

동작 원리 :

- 사용자가 로그인하면 서버는 세션 ID를 생성하여 서버 메모리에 유저 정보와 함께 저장
- 클라이언트는 세션 ID만 쿠키에 담아서 보관하도록 함
- 이후 요청에서 클라이언트는 세션ID를 서버에 보여주고 서버는 메모리에서 해당 정보를 찾음

장점

- 보안성 : 중요한 정보 (유저정보)는 서버만 가지고 클라이언트는 의미 없는  ID만을 가짐
- 제어 가능 : 서버가 로그인 정보를 유지하므로 강제 로그아웃, 의심스러운 접속 즉시 차단 가능

단점

- 서버 부하
- 확장성 : 서버에서 정보를 유지하기 때문에 서버를 여러대로 늘리면 서버간 요청시에 로그인을 알아보지 못함

### 토큰기반 인증

저장 위치 : 클라이언트

동작 원리 :

- 로그인시 서버는 유저 정보를 암호화하고 서명하여 토큰을 만들어 클라이언트에게 제공
- 클라이언트는 요청할 때마다 헤더에 토큰을 포함하여 요청
- 서버는 서명만 확인 후 통과시킴

장점

- 확장성 : 서버가 로그인 정보를 기억할 필요가 없음 (토큰만 검사하면 됨)

단점

- 한 번 발급해주면 유효기간이 끝날 때 까지는 서버가 강제로 만료시킬 수 없음

  (토큰 탈취 시 상황이 복잡해짐. 이를 보완하기 위해 Refresh token 전략 사용)

- 데이터 크기 : 세션ID 보다는 토큰 길이가 길어서 네트워크 트래픽이 약간 증가할 수 있음

---

# 액세스 토큰(Access Token)과 리프레시 토큰(Refresh Token)

### JWT의 치명적인 단점

한번 발급되면 만료될 때까지 막을 수 없음, 만료를 자주 시키면 (기간을 짧게 설정) 자주 로그인 하는 귀찮음

### Refresh Token의 목적

보안을 위해 유효기간을 짧게 줄이면서, 로그인을 자주 하는 귀찮음을 없애기 위한 전략으로

Access Token 만료시에 Refresh Token을 이용해 새로운 토큰을 발급받음

---

### 동작 흐름

1. 로그인 성공 시 AccessToken(짧은 시간), RefreshToken(긴 시간)을 둘 다 발급

   이 때 서버는 Refresh Token을 DB에 저장

2. Access Token을 이용해 서비스 이용
3. Access Token이 만료된 상태로 요청 시 401 에러 발생

기존에는 이 지점에서 다시 로그인 요청을 해야 하는데 Refresh Token을 이용해 이 부분을 해결

1. 클라이언트는 사용자에게 다시 로그인을 시키지 않고 아직 만료 되지 않은 Refresh Token 을 서버에 전달
2. 서버는 Refresh Token을 DB에 저장된 것과 일치하는지, 유효기간이 남았는지 확인해서 AccessToken을 재발급
3. 재발급 받은 Access Token 이용해서 다시 요청 및 서비스 이용

→ 이를 통해 토큰 탈취 상황에도 해당 토큰은 금방 만료되기 때문에 피해를 최소화 할 수 있음

→ 짧은 수명으로 인한 불편함은 Refresh Token으로 해결해 사용자의 불편함 최소화

→ Refresh Token은 평소에 사용하지 않고 오직 재발급에만 사용하기 때문에 노출 빈도가 적어서 상대적으로 안전하고 DB에 저장되기 때문에 서버가 통제할 수 있음

---

### RTR (Refresh Token Rotation)

Refresh Token을 한 번 사용할 때 마다 폐기하고 새로 발급하는 방식

만약 refresh token 까지 털리면 어떻게 될까? 해커는 해당 기간동안 access token을 계속 재발급 받으며 활동할 수 있음 이러한 상황을 방지하기 위해 도입한 개념으로 Refresh Token을 한 번 사용하면 폐기하고 새로 발급. 서버측 DB에는 새로 발급된 Refresh Token만 저장되어있음.

계정의 원래 사용자가 refresh token을 이용해 access token을 재발급 받는 과정에서 refresh token 또한 재발급되고 이후에 해커가 가지고 있는 refresh token을 제시해 access token을 재발급 받으려고 할 때  DB에 저장된 값과 해커가 제시한 refresh token 값이 다르기 때문에 재발급을 막을 수 있음.

(이 때 해커가 먼저 요청하는 상황까지 고려해 refresh token에 대해 일치하지 않는 요청이 오면 두 refresh token 모두 페기함)
