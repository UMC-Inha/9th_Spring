## 🔐 Spring Security란?

> Spring 기반 애플리케이션의 인증(Authentication)과 인가(Authorization)를 담당하는 보안  프레임워크
> 
- 필더(Filter)기반으로 동작하며 스프링 MVC와 분리되어 관리 및 동작한다.

### 인증 (Authentication)

- “누가 들어왔지?”
- 사용자의 신원을 확인하는 것을 말한다.

### 인가 (Authorization)

- “사용자가 무엇을 할 수 있는가?”
- 사용자의 접근 권한을 판단한다.

### 동작 구조 (요청 흐름)

- Client
    - 사용자가 요청을 보낸다.
- Security Filter Chain
    - 요청을 가로채어 보안 관련 필터들을 순차적으로 실행한다.
- Authentication Filter
    - 여러 Security Filter 중에서 UsernamePasswordAuthenticationFilter에서 인증을 처리한다.
    - HttpServletRequest에서 아이디와 비밀번호를 추출하여 UsernameAuthenticationToken을 발급한다.
    - AuthenticationManager에게 인증 객체를 전달한다.
- AuthenticationManager
    - 전달받은 Authentication 객체를 기반으로 실제 인증을 수행할 수 있는지 관리, 판단한다.
    - UsernameAuthenticationToken이 올바른 유저인지 확인한다.
- AuthenticationProvider
    - 사용자 정보와 자격 증명을 검증하여 인증 성공 여부를 결정한다.
- UserDetailsService
    - 사용자 식별 정보를 바탕으로 DB등에서 사용자 정보를 조회해 반환한다.

### 주요 Component

- Authentication
    - 현재 접근하는 주체의 정보와 권한을 담는 인터페이스
    
    ```java
    getCredentials();
    getDetails();
    getPrincipal();
    isAuthenticated();
    setAuthenticated(boolean isAuthenticated);
    ```
    
- SecurityContext
    - Authentication을 보관하는 역할을 한다.
    - SecurityContext를 통해 Authentication객체를 꺼내올 수 있다.
- SecurityContextHolder
    - 응용프로그램의 현재 보안 컨텍스트에 대한 세부 정보가 저장된다.
    - 기본적으로 ThreadLocal 방식으로 인증 정보를 유지한다.
    
    ```cpp
    인증에 성공 
    -> principal + credential 을 Authentication에 담음.
    -> Spring Security에서 Authentication을 SpringContext에 담아 보관
    -> 이 SpringContext를 SecurityContextHolder에 담아 보관함.
    ```
    
- UserDetails
    - Authentication 객체를 구현한 UsernamePasswordAuthenticationToken을 생성하기 위해 사용된다.
    - UserDetails를 implements 하여 구현체로 만들 수 있다.
    
    ```cpp
    public class CustomUserDetails implements UserDetails {
    		private User user;
    		
    		public User getUser() {
    				return user;
    		}
    		
    		public CustomUserDetails(User account) {
    				this.user = account;
    		}
    		
    		@Override
    		.
    		.
    		.
    }
    ```
    
- UserDetailsService
    - UserDetails 객체를 반환하는 하나의 메서드만 가지고 있다.
    
    ```java
    UserDetails loadUserByUsername(String username);
    ```
    
    - 인증 과정에서 UserDetailService → UserDetails → Authentication 생성 흐름
- PasswordEncoder
    - 비밀번호를 단방향 암호화하고 검증하는 컴포넌트
    - 평문 비밀번호를 직접 비교하지 않고, 암호화된 값끼리 비교하여 보안 강화
    
    ```java
    String encode(CharSequence rawPassword);
    boolean matches(CharSequence rawPassword, String encodedPassword);
    ```
    
    - 비밀번호는 DB에 반드시 암호화된 상태로 저장해야 한다!

    ## 인증 (Authentication)

- “누구인가???”
- 로그인한 사용자의 신원을 확인하는 과정이다.
- 인증이 되지 않으면, 서버 입장에서는 현재 요청자가 누구인지 확인할 수 없다.

## 인가 (Authorization)

- “사용자가 무엇을 할 수 있나??”
- 로그인한 사용자의 접근 권한을 확인하는 과정이다.
- 인증이 되어있어도 인가가 없으면, 접근할 수 없다.
- 서버 입장에서는 누가 로그인을 했는지 알고 있으나, 권한이 없어 해당 요청을 차단한 것이다.

## 세션(Session)과 토큰(Token)

- 세션과 토큰은 사용자를 인증한 이후 생성된 인증 정보를 어떤 방식으로 저장하고 전달할 것인지에 관한 방법이며, 이를 통해 자원 접근에 대한 인가를 수행할 수 있도록 한다.

### 세션

- 세션 기반 로그인은 사용자의 인가 정보가 세션 저장소에 저장되는 방식이다.
- 사용자가 로그인을 하면, 서버는 해당 인가 정보를 세션 저장소에 저장한 후 세션 저장소의 식별자인 Session ID를 클라이언트 측에 전달한다.
- 클라이언트 측은 받은 Session ID를 웹 스토리지 혹은 쿠키에 저장하고 인가가 필요한 요청시 함께 전달하게 된다.

### 토큰

- 토큰 기반 로그인은 인가 정보를 서버가 추적하지 않고, 토큰만으로 권한을 판별하는 방식이다.
- 사용자가 로그인을 하면, 서버는 인가 정보를 담은 Token을 발행한 후 클라이언트 측에 전달한다. 클라이언트 측은 전달받은 Token을 웹 스토리지 혹은 쿠키에 저장한다.
- 대표적인 토큰인 JWT의 경우 디지털 서명이 존재해 토큰의 내용이 위변조 되었는지 서버측에서 확인할 수 있다.

### 세션 vs 토큰

1. 사이즈
    - 세션의 경우 Session ID만 실어 보내면 되므로 트래픽을 적게 사용한다.
    - 하지만 JWT는 사용자 인증 정보와 토큰의 발급 시각, 만료 시각, 토큰 ID 등 담겨있는 정보가 Sesstion ID에 비해 크기 때문에 세션 방식보다 훨씬 더 많은 네트워크 트래픽을 사용한다.
2. 안정성과 보안
    - 세션의 경우 모든 인증 정보를 서버 측에서 관리하기 때문에 보안 측면에서 조금 더 유리하다.
    - 세션 ID가 해커에게 탈취당한 경우, 서버 측에서 해당 세션을 무효 처리하면 된다.
    - 하지만 토큰의 경우 서버가 토큰을 추적하지 않기 때문에 클라이언트가 모든 인증 정보를 가지고 있다.
    - 따라서 토큰이 한번 해커에게 탈취되면 해당 토큰이 만료되기 전까지는 피해를 입을 수 있다.
3. 확장성
    - 현재 거의 모든 웹 애플리케이션이 토큰의 ‘확장성’때문에 토큰 기반 인증 방식을 사용한다.
    - 일반적으로 웹 애플리케이션의 서버는 수평적으로 확장한다. 이때, 여러대의 서버가 요청을 처리하게 되는데 별도의 작업을 해주지 않는다면 세션 기반 인증 방식은 세션 불일치 문제를 겪게 된다.
    - 하지만 토큰 기반 인증 방식의 경우 직접 인증 방식을 저장하지 않고 클라이언트가 저장하기 때문에 세션 불일치 문제로부터 자유롭다!
4. 서버의 부담
    - 앞에서 말했 듯이, 세션은 서버가 직접 세션 데이터를 저장해야하는 반면 토큰은 서버가 인증 데이터를 가지고 있지 않기 때문에 서버에서 관리해야하는 부담이 확실히 적다.

    ## Access Token 기반 JWT 토큰 문제

- Access Token만을 이용해서 인증 인가를 처리한다면, 유효 기간 내에 토큰을 탈취 당할 위험에서 벗어날 수 없다.
- 서버는 클라이언트와 탈취한 사람을 구분하지 못하기 때문에 해커가 토큰을 악용한다면 보안상 문제가 생길 수 있다!

→ 이런 문제를 해결하기 위해 Access Token의 유효 기간을 짧게 해두고, Refresh Token을 사용하는 방식을 사용한다.

## Refresh Token

- 통신이 빈번한 Access Token은 탈취당할 가능성이 높기 때문에, 유효기간이 긴 Access Token을 같이 발급해 클라이언트가 로컬에 저장해 두도록 한다.
- 클라이언트는 헤더에 Access token을 두고 통신을 하는데, Access Token의 유효기간이 만료되었다면 헤더에 Refresh Token을 넣고 재요청을 보내 새로운 Access Token을 발급받는다.

### 서버-클라이언트 통신

1. 로그인 인증에 성공하면 클라이언트는 Refresh Token과 Access Token 두 개를 서버로부터 받는다.
2. 클라이언트는 로컬에 두 개의 토큰을 저장해둔다.
3. 클라이언트는 헤더에 Access Token을 넣고 API 통신을 한다.
4. 일정 기간이 지나 Access Token이 만료되었다면, 클라이언트는 서버에 Refresh Token을 보내 새로운 Access Token을 발급받는다. (별도의 API)
5. 만약 Refresh Token도 만료되었다면 서버는 401 Error Code를 보내고, 클라이언트는 재 로그인을 해야한다.