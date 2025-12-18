# Authentication & Authorization

## 1. 인증(Authentication)이란?

인증은 쉽게 말해 **“이 사용자가 누구인지 신원을 확인하는 과정”**입니다.

시스템이 “당신은 정말 당신이 주장하는 그 사람이 맞습니까?”라고 묻고, 사용자가 이에 대한 증거(비밀번호, 생체 정보 등)를 제시하는 단계입니다.

> 건물에 들어가기 위해 입구에서 **신분증**을 보여주는 것과 같습니다.
> 

로그인을 시도했을 때, 입력한 정보가 DB에 있는지 확인합니다.

**일치하면** 유저 정보를 반환하거나, 다음 통신에서 나를 증명할 수 있는 **토큰** 혹은 **세션**을 부여합니다.

**일치하지 않으면:** 접근을 차단하며 보통 **401 Unauthorized** 에러를 반환합니다.

## 2.  인가(Authorization)란?

인가는 인증을 마친 사용자가 **“어떤 권한을 가지고 있으며, 무엇을 할 수 있는지 판단하는 과정”**입니다.

단순히 로그인에 성공했다고 해서 모든 기능을 쓸 수 있는 것은 아니기에, 특정 요청을 수행할 자격이 있는지 검증하는 단계가 반드시 필요합니다.

> 건물에는 들어왔지만(인증),
> 

> **‘관리자 전용 구역’**에는 관리자 카드키가 있는 사람만 들어갈 수 있는 것과 같습니다.
> 

유저가 "내 블로그 설정 변경" 요청을 보냈을 때, 시스템은 두 가지를 확인합니다.

1. 이 유저가 로그인된 상태인가? (**인증 확인**)
2. 이 블로그의 소유주가 현재 로그인한 유저가 맞는가? (**권한 확인 = 인가**)

권한이 없다면 접근을 거부하며 **403 Forbidden** 에러를 반환합니다.



## 3. Spring Security에서 인증과 인가의 흐름

![image.png](./image/image2.png)

사용자가 로그인을 시도하고, 이후 권한이 필요한 페이지에 접근하는 전 과정을 5단계로 요약할 수 있습니다.

### 1단계: 인증 요청 (Authentication)

- 사용자가 아이디와 비밀번호를 입력하여 로그인을 시도합니다.
- **UsernamePasswordAuthenticationFilter**가 이 요청을 가로채서 `Authentication` 객체(미인증 상태)를 생성합니다.

### 2단계: 신원 검증 (AuthenticationManager & Provider)

- 필터는 **AuthenticationManager**에게 검증을 맡깁니다.
- 매니저는 **AuthenticationProvider**를 통해 `UserDetailsService`를 호출하고, DB에 저장된 유저 정보(`UserDetails`)와 입력된 비밀번호를 비교합니다.
- 비밀번호가 일치하면 '인증이 완료된' `Authentication` 객체를 반환합니다.

### 3단계: 인증 정보 보관 (SecurityContextHolder)

- 인증에 성공하면, 위에서 만든 '인증 완료 객체'를 **SecurityContextHolder** 내부의 **SecurityContext**에 저장합니다.
- 이로써 시스템은 "현재 접속 중인 이 유저는 '홍길동'이며 'USER' 권한을 가졌다"는 사실을 기억하게 됩니다.

### 4단계: 권한 확인 (Authorization / FilterSecurityInterceptor)

- 로그인한 유저가 권한이 필요한 페이지(예: `/admin`)에 접근합니다.
- 필터 체인의 가장 마지막 단계에 있는 **FilterSecurityInterceptor**가 작동합니다.
- 이 필터는 **SecurityContextHolder**에서 유저 정보를 꺼내와 "이 유저가 ADMIN 권한이 있는가?"를 확인합니다.

### 5단계: 최종 응답 (Success or Failure)

- **인가 성공:** 유저가 권한을 가지고 있다면 컨트롤러로 요청을 전달하여 페이지를 보여줍니다.
- **인가 실패:** 권한이 없다면 **ExceptionTranslationFilter**가 에러를 가로채서 `403 Forbidden` 응답을 보내거나 적절한 예외 처리를 수행합니다.