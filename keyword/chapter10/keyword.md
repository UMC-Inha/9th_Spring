                         ## Spring Security

### 1-1. Spring Security란?

Spring Security는 Spring 기반 애플리케이션에서 인증과 인가를 담당하는 보안 프레임워크이다.

웹 애플리케이션은 다음과 같은 보안 위협에 항상 노출되어 있다.

- 로그인 하지 않은 사용자의 접근
- 권한이 없는 사용자의 기능 사용
- 세션 탈취
- CSRF, XSS 공격

Spring Security는 이러한 문제를 표준화된 방식으로 해결하도록 도와준다.

### 1-2. Spring Security의 핵심 역할

Spring Security는 크게 아래 역할을 수행한다.

1. 인증(Authentication) → “이 사용자가 누구인가?”
2. 인가(Authorization) → “이 사용자가 이 기능을 사용할 권한이 있는가?”
3. 보안 필터 체인(Filter Chain) 관리 → 요청이 컨트롤러에 도달하기 전에 보안 검사 수행
4. 보안 공격 방어
    - CSRF, Session Fixation, Clickjacking, Password Encoder 제공

### 1-3. Spring Security의 동작 구조

Spring Security는 Filter 기반으로 동작한다.

```java
[Client 요청] -> [Spring Security Filter Chain] -> [DispatcherServlet] -> [Controller]
```

→ 즉 모든 요청은 컨트롤러에 도달하기 전에 반드시 Security 필터를 거친다.

### 1-4. Security Filter Chain 구성

대표적인 필터 흐름은 다음과 같다.

1. SecurityContextPersistenceFilter
    - 이전 요청에서 저장된 인증 정보를 불러옴
2. UsernamePasswordAuthenticationFilter
    - 로그인 요청 처리
    - 아이디/비밀번호 검증
3. AuthenticationFilter (JWT 사용 시 )
    - 토큰 검증
4. ExceptionTranslationFilter
    - 인증/인가 실패 시 예외 처리
5. FilterSecurityInterceptor
    - 접근 권한 최종 검사

### 1-5. 핵심 객체 개념

1. Securiy Context
    - 현재 요청의 보안 정보 저장소
    - 인증된 사용자 정보 보관
2. Authentication
    - 사용자의 인증 상태를 나타내는 객체
    - 포함정보:
        - principal (사용자 정보)
        - credentials (비밀번호)
        - authorities (권한 목록)
3. UserDetails
    - Spring Security가 이해하는 사용자 정보 인터페이스
    - 만든 User 엔티티를 감싸서 사용

### 1-6. Spring Security의 인증 흐름 (Form Login 기준)

1. 사용자가 로그인 요청
2. UsernamePasswordAuthenticationFilter가 요청 가로챔
3. AuthenticationManager에게 인증 요청
4. UserDetailsService에서 사용자 조회
5. 비밀번호 비교(Password Encoder)
6. 성공 시 Authentication 객체 생성
7. SecurityContext에 저장
8. 이후 요청은 인증된 사용자로 처리

### 1-7. Spring Security 설정 방식

**과거 방식**

- WebSecurityConfigurerAdapter 상속

**현재 권장 방식**

- SecurityFilterChain Bean 등록

## 인증(Authentication)과 인가(Authorization)

웹 애플리케이션의 보안은 크게 인증과 인가라는 두 단계로 나뉜다. 이 두 개념은 자주 함께 사용되지만, 역할과 목적은 명확히 다르다.

### 2-1. 인증(Authentication)

### 2-1-1. 인증이란?

인증(Authentication)이란 사용자가 누구인지 확인하는 과정이다.

즉, “당신은 정말 이 계정의 주인인가?”를 검증하는 단계이다.

### 2-1-2. 인증의 예시

- 아이디 + 비밀번호 로그인
- 소셜 로그인(카카오, 구글)
- 토큰 (JWT) 검증
- OTP(일회용 비밀번호)

→ 인증이 성공하면 시스템은 이 사용자는 신뢰할 수 있다고 판단한다.

### 2-1-3. 인증에 성공하면 발생하는 일

- 사용자의 신원 정보가 시스템에 저장된다.
- 이후 요청에서 다시 로그인하지 않아도 된다.
- Spring Security에서는 이 정보를 SecurityContext에 저장한다.

### 2-1-4. Spring Security에서의 인증

Spring Security에서 인증은 다음 객체들을 중심으로 이루어진다.

**핵심 객체 흐름**

1. Authentication
    - 인증 전: 아이디/비밀번호만 있음
    - 인증 후: 사용자 정보 + 권한 포함
2. AuthenticationManager
    - 인증 전체를 총괄하는 관리자
3. AuthenticationProvider
    - 실제 인증 로직 담당
    - 비밀번호 비교, 사용자 존재 여부 검증
4. UserDetailsService
    - 사용자 정보를 DB에서 조회

### 2-1-5. 인증 과정 요약(로그인 기준)

1. 사용자가 로그인 요청
2. Spring Security 필터가 요청 가로챔
3. AuthenticationManager에게 인증 위임
4. UserDetailsService에서 사용자 조회
5. 비밀번호 검증
6. 성공 시 Authentication 객체 생성
7. Security Context에 저장

### 2-1-6. 인증 실패 시

- 비밀번호 불일치
- 존재하지 않는 사용자
- 계정 잠김 / 비활성화

→ 이 경우 401 Unauthorized 응답 발생

### 2-2. 인가(Authorization)

### 2-2-1. 인가란?

인가란 인증된 사용자가 특정 자원에 접근할 권한이 있는지 확인하는 과정이다.

즉, → “이 사용자가 이 기능을 사용할 수 있는가?”를 판단한다.

### 2-2-2. 인가의 전제 조건

인가를 하기 위해서는 반드시 인증이 선행되어야 한다.

- 인증 X → 인가 불가
- 인증 O → 인가 판단 가능

### 2-2-3. Spring Security에서의 인가 방식

Spring Security는 권한 또는 역할(Role)을 기준으로 인가를 처리한다.

**Role**

- `ROLE_USER`
- `ROLE_ADMIN`

**Authority**

- `READ_POST`
- `WRITE_POST`

### 2-2-4. 인가 설정 예시

```java
http.authorizeHttpRequests(auth -> auth
    .requestMatchers("/admin/**").hasRole("ADMIN") // /admin으로 시작하는 모든 URL은 ADMIN권한 있어야 접근 가능
    .requestMatchers("/user/**").hasRole("USER")
    .anyRequest().authenticated()
);
```

### 2-2-5. 메서드 단위 인가

```java
@PreAuthorize("hasRole('ADMIN')") //이 메서드는 ADMIN 권한을 가진 사용자만 실행 가능
public void deleteUser() { }
```

### 2-2-7. 인가 실패 시

- 인증은 되었지만 권한이 없음
- 403 Forbidden 응답 발생

### 2-3. 실세 서비스 흐름으로 이해하기!

1. 사용자가 로그인 요청 → 인증
2. 인증 성공 → 사용자 정보 저장
3. 특정 API 요청
4. 해당 API 접근 권한 확인 → 인가
5. 권한이 있으면 처리, 없으면 차단

## 세션과 토큰

인증이 완료되면 다음 문제가 생긴다 → “매 요청마다 다시 로그인해야 하나?”

이 문제를 해결하기 위해 인증 상태를 유지하는 방식이 필요하고, 그 대표적인 방법이 세션과 토큰이다.

### 3-1. 세션

### 3-1-1. 세션이란?

세션은 서버가 사용자별 인증 정보를 저장하고 관리하는 방식이다.

- 로그인 성공 시
- 서버가 사용자 정보를 세션 저장소에 저장
- 사용자에게는 세션 ID만 전달

### 3-1-2. 세션 기반 인증 흐름

1. 사용자가 로그인 요청
2. 서버가 사용자 인증 성공
3. 서버가 세션 생성
4. 세션 ID 발급
5. 세션 ID를 쿠키에 담아 클라이언트에 전달
6. 이후 요청마다 쿠키에 세션 ID 포함
7. 서버는 세션 ID로 사용자 정보 조회

### 3-1-3. Spring Security에서의 세션

Spring Security 기본 설정은 세션 기반 인증이다.

- SecurityContext가 세션에 저장됨
- 세션을 통해 인증 상태 유지

```java
SecurityContextHolder.getContext().getAuthentication();
```

### 3-1-4. 세션의 장점

- 구현이 단순
- 서버에서 직접 인증 정보 관리 → 보안 제어 쉬움
- 서버 로그아웃 처리 간단

### 3-1-5. 세션의 단점

- 서버 메모리 사용
- 서버 확장 시 세션 공유 필요
- 모바일 / SPA 환경에 불리

### 3-2. 토큰

### 3-2-1. 토큰이란?

토큰은 서버가 발급한 인증 정보를 클라이언트가 직접 보관하는 방식이다.

- 서버는 상태를 저장하지 않음
- 인증 정보가 토큰 안에 포함됨

### 3-2-2. 토큰 기반 인증 흐름

1. 사용자가 로그인 요청
2. 서버가 인증 성공
3. 서버가 토큰 발급
4. 클라이언트가 토큰 저장
5. 요청 시 토큰을 헤더에 담아 전송
6. 서버는 토큰 검증만 수행

### 3-2-3. 토큰의 핵심 특징

- 서버에 인증 상태를 저장하지 않는다.
- 매 요청마다 토큰 검증

### 3-2-4. 토큰의 장점

- 서버 확장에 유리
- 모바일 / SPA / MSA 환경 적합
- 세션 공유 문제 없음

### 3-2-5. 토큰의 단점

- 토큰 탈취 시 위험
- 강제 로그아웃 처리 어려움
- 토큰 말료 전략 필수

정리하자면 세션은 서버가 인증 상태를 관리하는 방식이며, 토큰은 클라이언트가 인증 정보를 보관하는 방식이다.

## 액세스 토큰(Access Token)과 리프레시 토큰(Refresh Token)

토큰 기반 인증(JWT)를 사용하면 다음 문제가 생긴다.

→ 토큰이 탈취되면 어떻게 하지?

→ 토큰을 너무 짧게 하면 매번 로그인해야 하지 않나?

-→ 이 문제를 해결하기 위해 Access Token과 Refresh Token이라는 두 종류의 토큰을 사용한다.

### 4-1. Access Token

### 4-1-1. Access Token이란?

Access Token은 API 요청 시 사용되는 실제 인증 토큰이다.

- 서버가 발급
- 클라이언트가 요청마다 전송
- 사용자 권한 정보 포함

→ 즉, “이 요청은 인증된 사용자가 보낸 요청입니다”를 증명하는 토큰이다.

### 4-1-2. Access Token의 특징

- 유효 기간 짧음
- 탈취되더라도 피해 최소화
- Authorization 헤더에 포함

### 4-1-3. AccessToken에 들어가는 정보 (JWT 기준)

- 사용자 ID
- 권한
- 발급 시간
- 만료 시간

→ 이 정보로 인가 판단까지 수행

### 4-1-4. AccessToken 사용 시점

- 모든 보호된 API 요청
- Spring Security 필터에서 검증
- 성공 시 Authentication 객체 생성

### 4-1-5. Access Token 만료 시

- 서버는 요청을 거부
- **401 Unauthorized** 응답
- 이때 등장하는 게 **Refresh Token**

### 4-2. Refresh Token

### 4-2-1. Refresh Token이란?

Refresh Token은 Access Token을 재발급하기 위한 토큰이다.

- API 요청엔 사용 X
- 오직 재발급 요청에만 사용
- 유효기간이 김

### 4-2-2. Refresh Token의 특징

- 노출 위험이 더 큼
- 그래서 서버에서 관리하는 경우가 많음
- DB / Redis에 저장

### 4-2-3. Refresh Token 사용 흐름

1. Access Token 만료
2. 클라이언트가 Refresh Token 전송
3. 서버가 Refresh Token 검증
4. Access Token 발급

### 4-2-4. Refresh Token은 어디에 저장?

Access Token → 메모리 / JS 상태

Refresh Token → HttpOnly Cookie

### 4-2. 전체 인증 흐름 한 번에 보기

```java
[로그인] -> Access Token + Refresh token 발급 -> [API 요청]: Authorization: Bearer AccessToken
[Access Token 만료] -> Refresh Token으로 재발급 요청 -> 새 Acces Token 발급
```

### 4-3. 왜 토큰을 두 개로 나누는가

- 보안 강화
    - Access Token 짧게 → 탈취 피해 최소화
    - Refresh Token 길게 → 로그인 유지
- 사용자 경험
    - 토큰 만료되어도 자동 재발급. 사용자는 다시 로그인할 필요 없음
- 서버 제어 가능 → Refresh Token 삭제, 강제 로그아웃