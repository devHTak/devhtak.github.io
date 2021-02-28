---
layout: post
title: JWT 사용 이전에 Spring Session
summary: JWT 인증
author: devhtak
date: '2021-02-23 21:41:00 +0900'
category: Spring
---

#### Overview

- Spring Security에 JWT 연동을 목적으로 JWT에 대해 공부한 것을 정리합니다.

#### Spring Session

- HTTP 는 상태가 유지하지 않는 statless한 비접속형 프로토콜이기 때문에 서버는 클라이언트의 이전 상태를 기억하기 위한 다양한 방법이 존재한다.

- JSESSIONID
  - 서버는 전달받은 세션 ID를 서버에 이미 저장되어 있는 정보와 비교해서 클라이언트를 식별하며 클라이언트의 상태를 지속적으로 유지하게 된다.
  - 서버는 클라이언트의 첫 접속에 JSESSIONID를 할당해주고, 그 다음 접속에서는 JSESSIONID로 식별한다.
    - 서버에서는 클라이언트의 고유한 세션 ID(JESSIONID)를 웹 브라우저에 쿠키로 전달
    - 클라이언트에서는 서버에 요청 시 해당 세션 ID(JESSIONID)를 함께 전달  

- Spring Session
  - Spring에서는 클라이언트의 세션 정보를 관리하기 위한 API를 제공한다.
  - 특징
    - HttpSession
      - RESTful API와 함께 작동하도록 헤더에 Session ID를 제공하므로써 WAS에 영향없이 HttpSession을 대체할 수 있도록 한다.
    - WebSocket
      - WebSocket 메시지를 수신할 때 HttpSession을 활성 상태로 유지하는 기능 제공
    - WebSession
      - Spring WebFlux의 WebSession을 대체할 수 있다.
  - 출처: https://spring.io/projects/spring-session

- HttpSession
  - Controller에서 HttpSession을 매개변수로 정의하면, HttpSession 인터페이스의 구현체인 StandardSession 클래스 객체를 주입받는다.
  - HttpSession로 선언한 인스턴스에서 getId()를 호출하면 JESSIONID를 확인할 수 있다.
  ```java
  @GetMapping("/hello")
  public String hello(HttpSession session, HttpServletRequest request) {
      log.info(session.getId()); // 세션 ID를 확인할 수 있다.
      Cookie[] cookies = request.getCookies();
      Arrays.stream(cookies).forEach(cookie -> {
          if(cookie.getName().equals("JSESSIONID")) {
              log.info(cookie.getValue()); // JSESSIONID를 확인할 수 있다. sessoin.getId()와 같다
          }
      });
      return "hello spring";
  }
  ```
  
  - HttpSession의 주요 메소드
    - void setAttribute(String name, Object value)
    - Object getAttribute(String name)
    - void removeAttribute(String name)
    - Enumeration<String> getAttributeNames()
    - setMaxInactiveinterval(int second)
      - Specifies the time, in seconds, between client requests before the servlet container will invalidate this session.
    - String getId()
      - Returns a string containing the unique identifier assigned to this session. The identifier is assigned by the servlet container and is implementation dependent.
  
  - 출처: https://tomcat.apache.org/tomcat-9.0-doc/servletapi/javax/servlet/http/HttpSession.html

#### Spring Session Clustering

- 만약, 1대의 Server를 운영하던 도 중 서버의 부하를 줄이기 위해 서버를 증설했다.
  - Client -> L4 or L7 Switch -> Server, Server2
  - L4 또는 L7에 백엔드 API를 요청하고, L4는 트래픽을 분산하여 두 대의 백엔드 서버에 분산해서 호출한다.
  - 각각의 백엔드 서버에서 관리하는 세션 정보는 서로 공유하고 있지 않기 때문에 세션 정보가 일치하지 않는 상황이 발생한다.
  
- 해결책 1. AWS ELB의 Sticky Session
  - Sticky Session은 로드밸런서에서 클라이언트가 동일한 서버에 접속할 수 있도록 한다.
  - 클라이언트는 자신의 세션 정보를 갖고 있는 특정 서버에만 접속하기 때문에 세션이 일치하지 않는 상황이 발생하지 않는다.
  - 다만, 분산 역할을 완벽하게 해결할 수 없다.

- 해결책2. 세션 저장소 구축
  - Redis와 같은 것을 활용하여 세션 저장소 구축
    - Redis는 Remote Dictionary Server의 약자로서, "키-값" 구조의 비정형 데이터를 저장하고 관리하기 위한 오픈 소스 기반의 비관계형 데이터베이스 관리 시스템(DBMS)이다. 
    - 모든 데이터를 메모리로 불러와서 처리하는 메모리 기반 DBMS이다. 
  - Spring Session은 세션 공유 저장소를 단순하게 구축할 수 있는 기능을 제공한다.

#### Spring Session 인증 & 인가 인터셉터 구현

- 소스 저장소: https://github.com/devHTak/SpringAuth/tree/main/UsingSession

- Interceptor 등록
  - AuthInterceptor.class
    ```java
    @Slf4j
    @Component
    @RequiredArgsConstructor
    public class AuthInterceptor implements HandlerInterceptor {
        @Override
        public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
            log.info("AuthInterceptor preHandle");

            if(request.getSession().getAttribute(SecurityConstants.KEY_ROLE) != null
                && request.getSession().getAttribute(SecurityConstants.KEY_ROLE).equals(Role.USER.name())) {
                return true;
            }
            throw new CustomAuthenticationException();
        }
    }
    ```
    - 인증이 완료되면 true를 리턴하고 아닌 경우 CustomAuthenticationException을 발생시킨다.
  - WebMvcConfigurer.class
    ```java
    @Configuration
    @RequiredArgsConstructor
    public class WebMvcConfig implements WebMvcConfigurer{
        private final AuthInterceptor authInterceptor;
        @Override
        public void addInterceptors(InterceptorRegistry registry) {
            // TODO Auto-generated method stub
            registry.addInterceptor(authInterceptor)
                .addPathPatterns("/api/v1/coffees/**")
                .excludePathPatterns("/api/vi/login/**");
        }
    }
    ```
    - 인증이 필요없는 path를 excludePathPatterns에 설정하고, 인증이 필요(session확인)한 path는 addPathPatterns에 설정했다.

- 인증 실패: Exception
  - CustomAuthenticationException.class
    ```java
    public class CustomAuthenticationException extends RuntimeException {
	      public CustomAuthenticationException() {
            super(ErrorCode.AUTHENTICATION_FAILED.getMessage());
        }

        public CustomAuthenticationException(Exception ex) {
            super(ex);
        }
    }
    ```
  - GlobalExceptionHandler.class
    ```java
    @Slf4j
    @ControllerAdvice
    public class GlobalExceptionHandler {
        @ExceptionHandler(CustomAuthenticationException.class)
        protected ResponseEntity<CommonResponse> handleCustomAuthenticationException(CustomAuthenticationException e) {
            log.info("CustomAuthenticationException", e);
            CommonResponse response = CommonResponse.builder()
                .code(ErrorCode.AUTHENTICATION_FAILED.getCode())
                .message(e.getMessage())
                .status(ErrorCode.AUTHENTICATION_FAILED.getStatus()).build();

            return new ResponseEntity<>(response, HttpStatus.UNAUTHORIZED);
        }
    }
    ```
    - Interceptor에서 인증에 실패한 경우 CustomAuthenticationException가 발생하고, GlobalExceptionHandler에 handleCustomAuthenticationException 핸들러가 실행한다.

- 인증 설정 - 로그인
  - LoginController.class
    ```java
    @RestController
    @RequiredArgsConstructor
    public class LoginController {
        private final LoginService loginService;
        
        @PostMapping("/api/v1/login")
        public String login(HttpSession session, @RequestBody LoginRequestDTO loginRequestDTO) {
            MemberDTO member = loginService.login(loginRequestDTO.getEmail(), loginRequestDTO.getPassword())
                .orElseThrow(LoginFailedException::new);

            session.setAttribute(SecurityConstants.KEY_ROLE, member.getRole().name());
            session.setAttribute("email", member.getEmail());
            session.setMaxInactiveInterval(1800);

            return "ok";
        }
    }
    ```
- TODO. Test Code 작성
  - LoginControllerTest.class
    
    ```java
    @SpringBootTest(webEnvironment = WebEnvironment.MOCK)
    @AutoConfigureMockMvc
    public class LoginControllerTest {
	@Autowired MockMvc mockMvc;
	@Autowired MockHttpSession session;
	@Autowired ObjectMapper objectMapper;
	
	@Test
	void loginTest() throws Exception {
	    LoginRequestDTO loginRequestDTO = LoginRequestDTO.builder()
		.email("test@test.com").password("test1234").build();
		
	    MockHttpServletRequestBuilder builder = MockMvcRequestBuilders.post("/api/v1/login")
		.contentType(MediaType.APPLICATION_JSON_VALUE)
		.content(objectMapper.writeValueAsString(loginRequestDTO))
		.session(session);
		
	    mockMvc.perform(builder)
		.andDo(print())
		.andExpect(status().isOk());
		
	    assertEquals(session.getAttribute("email"), "test@test.com");
	}

    }
    ```
    
    - MockHttpSession: JUnit 환경에서 HttpSession을 Mock 객체로 만들어 사용할 수 있다.
    - MockServletRequestBuilder: 요청 헤더 객체를 만드는 데, MockHttpSession을 세션에 담겨서 보낼 수 있다. 요청에 대한 세션 유지를 위해 사용했다.

- 출처: https://brunch.co.kr/@springboot/491
