## spring security 란?
- spring 기반의 보안을 (인증, 권한, 인가 등) 담당하는 모듈입니다. 
- 인증과 권한에 대한 부분을 Filter 흐름에 따라 처리하고 있습니다.
  - Filter 는 request 요청이 들어왔을 때, DispatcherServlet 에 접근하기 전에 적용됩니다.
- spring security 는 크게 인증과 인가를 담당하고 있습니다.
  - 인증 : ID, PW 입력하여 본인인증 확인하는 절차
  - 인가 : 사용자가 자원에 접근가능한지를 확인하는 절차

## spring security 상세

#### Authentication
- 접근 주체의 정보와 권한을 담고 있습니다.
- 인증 후, SecurityContextHolder 에 저장됩니다.

#### SecurityContext
- Authentication 를 보관하고 있습니다.

#### SecurityContextHolder
- SecurityContext 를 보관하는 역할입니다. 
- SecurityContext 가 ThreadLocal 로 선언돼있기에 여러 스레드들이 동시에 접근해도 됩니다.

#### WebSecurityConfigurerAdapter
- spring security 관련 configuration 을 담당하는 adapter 입니다.
  - 어떤 자원에 접근을 허용할지, 권한을 줄지 등을 설정할 수 있습니다.
  - AuthenticationFilter 를 설정할 수 있습니다.

#### UserDetail
- user 정보를 가지고 있습니다. (ID, PW)
  - UsernamePasswordAuthenticationToken 만드는데 사용됩니다.
  - UsernamePasswordAuthenticationToken 이란 Principal(접근하는 주체), Credential (비밀번호) 정보를 가지고 있습니다.


#### Principal
- 사용자(접근 주체)
  - Spring Security 에서 Principal (접근 주체), Credential (비밀번호) 사용하는 Credential 기반 인증 방식 사용합니다.

#### CSRF
- Cross Site Request Forgery (사이트 간 요청 위조)
- 요청을 위조하여 서버에 잘못된 요청을 보내는 방법입니다.
- 해결방안은 Referrer 검증, Token 기반 인증방식이 있습니다.
- Spring Security 에서 @EnableWebSecurity 어노테이션을 지정하면 CSRF 보호 기능이 자동으로 활성화 됩니다.
  - POST, PUT 등의 데이터 변조가 가능한 method 에만 적용됩니다.

  
## JWT 란?
- 인코딩하여 만든 토큰이며, 주로 인증할 때 사용됩니다.
- [JWT 상세 참고](https://insanelysimple.tistory.com/333)

## Spring Security JWT 플로우
1. 요청 —> 인증 —> 인가
  - 로그인을 통해 인증해서 token 을 받아오고 해당 토큰으로 리소스에 접근할 (인가) 수 있습니다.
2. 요청이 들어오면 인증 절차를 수행합니다. (Filter 부분에서)
  - AuthenticationManager 에서 authenticate 메소드를 호출하며, AuthenticationProvider 를 통해 인증을 수행합니다.
  - AuthenticationManager 는 AuthenticationProvider 담고 있고, AuthenticationProvider 는 인증을 담당하는 Provider 입니다. AuthenticationProvider 는 교체 가능합니다.
  - Request 담겨져있는 JWT 토큰을 가져와서 유효한지 검사합니다.
3.인증이 완료되면 JVM 전역 내에서 접근 가능하도록 SecurityContextHolder 에 SecurityContext 저장합니다.

* 토큰에 대한 재발급이 필요할 경우 RefreshToken 을 서버로 보내서 재발급을 받습니다.

## reference
- https://wildeveloperetrain.tistory.com/m/57
- https://mangkyu.tistory.com/76