---
layout: post
title: Cookie, Session, Filter, Interceptor
summary: Spring MVC Basic
author: devhtak
date: '2021-09-06 21:41:00 +0900'
category: Spring
---

#### Cookie, Session

- stateless한 HTTP 프로토콜의 특징과 대비하게 Stateful한 상태를 위해 사용
- 쿠키와 세션의 차이점
  |-|쿠키|세션|
  |--|--|--|
  |저장 위치|클라이언트(=접속자 PC)|웹 서버|
  |저장 형식|text (key-value)|Object|
  |만료 시점|쿠키 저장시 설정(만료 시간 기준)|브라우저 종료시 삭제(기간 지정 가능)|
  |사용하는 자원(리소스)|클라이언트 리소스|웹 서버 리소스|
  |용량 제한|총 300개, 하나의 도메인 당 20개, 하나의 쿠키 당 4KB(=4096byte)|서버가 허용하는 한 용량제한 없음.|
  |속도|세션보다 빠름|쿠키보다 느림|
  |보안|세션보다 안좋음|쿠키보다 좋음|

#### Cookie 를 통한 로그인

- Server에서 로그인이 성공하면 HTTP 응답에 쿠키를 담아 브라우저에 전달
- 브라우저는 앞으로 해당 쿠키를 지속해서 보내준다.
- 쿠키에는 영속 쿠키와 세션 쿠키가 있다.
  - 영속 쿠키: 만료 날짜를 입력하면 해당 날짜까지 유지
  - 세션 쿠키: 만료 날짜를 생략하면 브라우저 종료시 까지만 유지
  - 브라우저 종료 시 로그아웃이 되야 하기 때문에 세션 쿠키이다.
  
- 예시
  ```java
  Cookie idCookie = new Cookie("memberId", String.valueOf(loginMember.getId());
  response.addCookie(idCookie);
  ```
  - 로그인 성공 시 쿠키를 생성하고 HttpServletResponse에 담는다.
  - 쿠키 이름은 memberId이고, 회원의 ID를 담아둔다.
  - 웹 브라우저는 종료 전까지 회원의 ID를 서버에 계속 보내줄 것이다.
  
  - @CookieValue를 통한 쿠키 조회
    ```java
    @GetMapping("/")
    public String homeLogin(@CookieValue(name="memberId", required="false", Long memberId, Model model) {
        if(memberId == null) return "home";
        Member loginMember = memberRepository.findById(memberId);
        if(loginMember == null) return "home";
        
        model.addAttribute("member", loginMember);
        return "loginHome";
    }
    ```
    - @CookieValue 를 사용하면 편리하게 쿠키를 조회할 수 있다.
    - 로그인 하지 않은 사용자도 홈에 접근할 수 있기 때문에 required = false 를 사용한다
    
  - 로그아웃
    ```java
    @PostMapping("/logout")
    public String logout(HttpServletResponse response) {
        expireCookie(response, "memberId");
        return "redirect:/";
    }
    private void expireCookie(HttpServletResponse response, String cookieName) {
        Cookie cookie = new Cookie(cookieName, null);
        cookie.setMaxAge(0);
        response.addCookie(cookie);
    }
    ```
    - 로그아웃도 응답 쿠키를 생성하는데 Max-Age=0 를 확인할 수 있다. 해당 쿠키는 즉시 종료된다.

- 쿠키와 보안 문제
  - 보안 문제
    - 쿠키 값은 임의로 변경할 수 있다.
      - 클라이언트가 쿠키를 강제로 변경하면 다른 사용자가 된다.
      - 실제 웹브라우저 개발자모드 Application Cookie 변경으로 확인
      - Cookie: memberId=1 Cookie: memberId=2 (다른 사용자의 이름이 보임)
    - 쿠키에 보관된 정보는 훔쳐갈 수 있다.
      - 만약 쿠키에 개인정보나, 신용카드 정보가 있다면?
      - 이 정보가 웹 브라우저에도 보관되고, 네트워크 요청마다 계속 클라이언트에서 서버로 전달된다.
      - 쿠키의 정보가 나의 로컬 PC가 털릴 수도 있고, 네트워크 전송 구간에서 털릴 수도 있다.
    - 해커가 쿠키를 한번 훔쳐가면 평생 사용할 수 있다.
      - 해커가 쿠키를 훔쳐가서 그 쿠키로 악의적인 요청을 계속 시도할 수 있다.
  - 대안
    - 쿠키에 중요한 값을 노출하지 않고, 사용자 별로 예측 불가능한 임의의 토큰(랜덤 값)을 노출하고, 서버에서 토큰과 사용자 id를 매핑해서 인식한다. 그리고 서버에서 토큰을 관리한다.
    - 토큰은 해커가 임의의 값을 넣어도 찾을 수 없도록 예상 불가능 해야 한다.
    - 해커가 토큰을 털어가도 시간이 지나면 사용할 수 없도록 서버에서 해당 토큰의 만료시간을 짧게(예: 30분) 유지한다. 또는 해킹이 의심되는 경우 서버에서 해당 토큰을 강제로 제거하면 된다.

#### Session 을 통한 로그인

- Cookie에 저장할 때 생기는 보안문제가 발생하기 때문에 중요한 정보는 서버에 저장해야 한다.
- 클라이언트와 서버는 추정 불가능한 임의의 식별자 값으로 연결해야 한다.
- 서버에 중요한 정보를 보관하고 연결을 유지하는 방법이 세션

- 세션 로그인 동작 방식
  - 로그인
    - 사용자가 loginId, password 정보를 전달하면 서버에서 해당 사용자가 맞는지 확인
  - 세션 생성
    - 세션 ID를 생성하는 데, 추정 불가능해야 한다.
    - UUID는 추정이 불가능하다.
    - 생성된 세션 ID와 세션에 보관할 값(memberA)을 서버의 세션 저장소에 보관한다.
  - 세션 id를 응답 쿠키로 전달
  - 클라이언트와 서버는 결국 쿠키로 연결이 되어야 한다.
    - 서버는 클라이언트에 mySessionId 라는 이름으로 세션ID 만 쿠키에 담아서 전달한다.
    - 클라이언트는 쿠키 저장소에 mySessionId 쿠키를 보관한다.
    - 여기서 중요한 포인트는 회원과 관련된 정보는 전혀 클라이언트에 전달하지 않는다는 것이다.
    - 오직 추정 불가능한 세션 ID만 쿠키를 통해 클라이언트에 전달한다.
  - 클라이언트의 세션id 쿠키 전달
    - 클라이언트는 요청시 항상 mySessionId 쿠키를 전달한다.
    - 서버에서는 클라이언트가 전달한 mySessionId 쿠키 정보로 세션 저장소를 조회해서 로그인시 보관한 세션 정보를 사용한다.

- 보안 이슈 해결
  - 쿠키 값을 변조 가능, 예상 불가능한 복잡한 세션Id를 사용한다.
  - 쿠키에 보관하는 정보는 클라이언트 해킹시 털릴 가능성이 있다. 세션Id가 털려도 여기에는 중요한 정보가 없다.
  - 쿠키 탈취 후 사용 해커가 토큰을 털어가도 시간이 지나면 사용할 수 없도록 서버에서 세션의 만료시간을 짧게(예: 30분) 유지한다.
  - 해킹이 의심되는 경우 서버에서 해당 세션을 강제로 제거하면 된다.

- HttpSession을 활용한 로그인
  - Servlet이 제공하는 HttpSession
  - 쿠키 이름이 JSESSIONID이고 value는 랜덤값으로 생성된다.
  - 로그인 기능
    ```java
    @PostMapping("/login")
    public String login(@Valid @ModelAttribute LoginForm form, BindingResult result, HttpServletRequest request) {
        if(result.hasErrors()) return "login/loginForm";
        
        Member member = loginService.login(form.getLoginId(), form.getPassword());
        if(member == null) {
            result.reject("loginFail", "아이디 또는 비밀번호가 맞지 않습니다.");
            return "login/loginForm";
        }
        // 로그인 성공 처리
        // 세션이 있으면 있는 세션 반환, 없으면 신규 세션 생성
        HttpSession session = request.getSession();
        // 세션에 로그인 회원 정보 보관
        session.setAttribute("loginMember", loginMember);
        
        return "redirect:/";
    }    
    ```
  - home(redirect:/) 처리
    ```java
    @GetMapping("/")
    public String home(HttpServletRequest request, Model model) {
        HttpSession session = request.getSession(false); // 기존에 생성되어 있지 않으면 null
        if(sessino == null) return "home";
        
        Member member = (Member)session.getAttribute("loginMember"); // 로그인 객체가 없는 경우
        if(member == null) return "home";
        
        model.addAttribute("member", member); // 로그인 완료
        return "loginHome"; 
    }
    ```
    - getSession에서 boolean create를 파라미터로 받는다.
      - true: 세션이 있으면 기존 세션을 반환하고 없으면 새로운 세션을 생성해서 반환한다.
      - false: 세션이 없으면 새로운 세션을 생성하지 않고, null을 반환
  - 로그아웃
    ```java
    @PostMapping("/logout")
    public String logoutV3(HttpServletRequest request) {
        // 세션을 삭제한다.
        HttpSession session = request.getSession(false);
        if (session != null) {
            session.invalidate();
        }
        return "redirect:/";
    }
    ```

- @SessionAttribute 으로 Member를 꺼내올 수 있다.
  ```java
  public String home(@SessionAttribute(name = "loginMember", required=false) Member loginMember, Model model) {
      // ...
  }
  ```
  - 해당 애노테이션을 통해 attribute를 가져올 수 있다.
  - 생성하지는 않기 때문에 찾아올 때에 사용한다.

- TrackingModes
  - 로그인을 처음 시도하면 URL이 다음과 같이 jsessionid 를 포함하고 있는 것을 확인할 수 있다.
    ```
    http://localhost:8080/;jsessionid=F59911518B921DF62D09F0DF8F83F872
    ```
  - 웹 브라우저가 쿠키를 지원하지 않을 때 쿠키 대신 URL을 통해서 세션을 유지하는 방법으로  URL에 이 값을 계속 포함해서 전달해야 한다.
  - 타임리프 같은 템플릿은 엔진을 통해서링크를 걸면 jsessionid 를 URL에 자동으로 포함해준다. 
  - Tracking Mode를 URL이 아닌 Cookie를 통해 진행하기 위해서는 application.properties 에 설정
    ```
    server.servlet.session.tracking-modes=cookie
    ```

- Session 정보와 Timeout
  - session 정보
    ```java
    @GetMapping("/session-info")
    public String sessionInfo(HttpServletRequest request) {
        HttpSession session = request.getSession(false);
        if (session == null) {
            return "세션이 없습니다.";
        }
        //세션 데이터 출력
        session.getAttributeNames().asIterator()
            .forEachRemaining(name -> log.info("session name={}, value={}", name, session.getAttribute(name)));
        log.info("sessionId={}", session.getId());
        log.info("maxInactiveInterval={}", session.getMaxInactiveInterval());
        log.info("creationTime={}", new Date(session.getCreationTime()));
        log.info("lastAccessedTime={}", new Date(session.getLastAccessedTime()));
        log.info("isNew={}", session.isNew());
        return "세션 출력";
    }
    ```
    - sessionId : 세션Id, JSESSIONID 의 값이다. 예) 34B14F008AA3527C9F8ED620EFD7A4E1
    - maxInactiveInterval : 세션의 유효 시간, 예) 1800초, (30분)
    - creationTime : 세션 생성일시
    - lastAccessedTime : 세션과 연결된 사용자가 최근에 서버에 접근한 시간, 클라이언트에서 서버로
    - sessionId ( JSESSIONID )를 요청한 경우에 갱신된다.
    - isNew : 새로 생성된 세션인지, 아니면 이미 과거에 만들어졌고, 클라이언트에서 서버로 sessionId ( JSESSIONID )를 요청해서 조회된 세션인지 여부
  
  - session timeout 설정
    - 세션과 관련된 쿠키( JSESSIONID )를 탈취 당했을 경우 오랜 시간이 지나도 해당 쿠키로 악의적인 요청을 할 수 있다.
    - 세션은 기본적으로 메모리에 생성된다. 메모리의 크기가 무한하지 않기 때문에 꼭 필요한 경우만 생성해서 사용해야 한다. 10만명의 사용자가 로그인하면 10만게의 세션이 생성되는 것
    - Session 종료 시점은 세션 생성 시점이 아닌 가장 최근 요청 시간을 기준으로 session timeout 시간을 주는 것
    - application.properties에 설정을 줄 수 있다.
      ```
      server.servlet.session.timeout=60 // 초 단위
      ```
    - session.getLastAccessedTime(): 최근 세션 접근 시간
    - timeout 시간이 지나면 WAS가 내부에서 해당 세션 제거

#### Servlet Filter

- 공통 관심 사항(cross-cutting concern)
  - AOP로 해결할 수 있지만, 웹과 관련된 공통 관심사는 Servlet Filter와 Spring Intereptor를 사용하는 것이 좋다.
  - HttpServletRequest를 제공하기 때문에 Header, Body 등에 정보를 알 수 있다.

- Servlet Filter 특징
  - 필터 흐름
    - HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 컨트롤러
    - 필터를 적용하면 필터가 호출 된 다음에 서블릿이 호출된다. 
    - 스프링을 사용하는 경우 여기서 말하는 서블릿은 스프링의 디스패처 서블릿으로 생각하면 된다.
  - 필터 제한
     - HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 컨트롤러 //로그인 사용자
     - HTTP 요청 -> WAS -> 필터(적절하지 않은 요청이라 판단, 서블릿 호출X) //비 로그인 사용자
  - 필터 체인
    - HTTP 요청 -> WAS -> 필터1 -> 필터2 -> 필터3 -> 서블릿 -> 컨트롤러
    - 여러 필터를 chaining 할 수 있다.
  - chain.doFilter(request, response); 호출
    - 다음 필터 또는 서블릿을 호출할 때 request, response 를 다른 객체로 바꿀 수 있다.
    - ServletRequest , ServletResponse 를 구현한 다른 객체를 만들어서 넘기면 해당 객체가 다음 필터 또는 서블릿에서 사용된다.
  - 필터 인터페이스
    ```java
    public interface Filter {
        public default void init(FilterConfig filterConfig) throws ServletException {}
        public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException;
        public default void destroy() {}
    }    
    ```
    - 필터 인터페이스를 구현하고 등록하면 서블릿 컨테이너가 필터를 싱글톤 객체로 생성하고, 관리한다.
    - init(): 필터 초기화 메서드, 서블릿 컨테이너가 생성될 때 호출된다.
    - doFilter(): 고객의 요청이 올 때 마다 해당 메서드가 호출된다. 필터의 로직을 구현하면 된다.
    - destroy(): 필터 종료 메서드, 서블릿 컨테이너가 종료될 때 호출된다.

- 예시 1) Servlet Filter - 요청 로그
  - Filter 생성
  ```java
  @Slf4j
  public class LogFilter implements Filter {
      @Override
      public void init(FilterConfig filterConfig) throws ServletException {
          log.info("log filter init");
      }

      @Override
      public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException{
          HttpServletRequest request = (HttpServletRequest)request;
          String requestURI = httpRequest.getRequestURI();
          String uuid = UUID.randomUUID().toString();

          try {
              log.info("REQUEST: {}, {}", uuid, requestURI);
              chain.doFilter(request, response);
          } catch(Exception e) {
              throw e;
          } finally {
              log.info("RESPONSE: {}, {}", uuid, requestURI);
          }
      }

      @Override
      public void destroy() {
          log.info("log filter destroy");
      }
  }
  ```
- Filter 빈 설정
  ```java
  @Configuration
  public class WebConfig {
      @Bean
      public FilterRegistrationBean logFilter() {
          FilterRegistrationBean<Filter> filterRegistrationBean = new FilterRegistrationBean<>();
          filterRegistrationBean.setFilter(new LogFilter());
          filterRegistrationBean.setOrder(1);
          filterRegistrationBean.addUrlPatterns("/*");
          return filterRegistrationBean;
      }
  }
  ```
  - setFilter(new LogFilter()) : 등록할 필터를 지정한다.
  - setOrder(1) : 필터는 체인으로 동작한다. 따라서 순서가 필요하다. 낮을 수록 먼저 동작한다.
  - addUrlPatterns("/*") : 필터를 적용할 URL 패턴을 지정한다. 한번에 여러 패턴을 지정할 수 있다

- 예시 2) ServletFilter - 인증 체크
  - Filter 생성
    ```java
    @Slf4j
    public class LoginCheckFilter implements Filter {
        private static final String[] whitelist = {"/", "/members/add", "/login", "/logout","/css/*"};
        @Override
        public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
            HttpServletRequest httpRequest = (HttpServletRequest) request;
            String requestURI = httpRequest.getRequestURI();
            HttpServletResponse httpResponse = (HttpServletResponse) response;
            try {
                log.info("인증 체크 필터 시작 {}", requestURI);
                if (isLoginCheckPath(requestURI)) {
                    log.info("인증 체크 로직 실행 {}", requestURI);
                    HttpSession session = httpRequest.getSession(false);
                    if (session == null || session.getAttribute(SessionConst.LOGIN_MEMBER) == null) {
                        log.info("미인증 사용자 요청 {}", requestURI);
                        //로그인으로 redirect
                        httpResponse.sendRedirect("/login?redirectURL=" + requestURI);
                        return; //여기가 중요, 미인증 사용자는 다음으로 진행하지 않고 끝!
                    }
                }
                chain.doFilter(request, response);
            } catch (Exception e) {
                throw e; //예외 로깅 가능 하지만, 톰캣까지 예외를 보내주어야 함
            } finally {
                log.info("인증 체크 필터 종료 {}", requestURI);
            }
        }
        /**
        * 화이트 리스트의 경우 인증 체크X
        */
        private boolean isLoginCheckPath(String requestURI) {
            return !PatternMatchUtils.simpleMatch(whitelist, requestURI);
        }
    }
    ```
    - init, destroy는 Filter 인터페이스에 default로 생성되어 오버라이딩을 안해도 된다.
    - 로그인 필요 시 redirectURL을 넘겼기 때문에 로그인 시 해당 경로로 넘어가도록 수정
      ```java
      @PostMapping("/login")
      public String loginV4(@Valid @ModelAttribute LoginForm form, BindingResult bindingResult, @RequestParam(defaultValue = "/") String redirectURL, HttpServletRequest request) {
          if (bindingResult.hasErrors()) {
              return "login/loginForm";
          }
          Member loginMember = loginService.login(form.getLoginId(), form.getPassword());
          log.info("login? {}", loginMember);
          
          if (loginMember == null) {
              bindingResult.reject("loginFail", "아이디 또는 비밀번호가 맞지 않습니다.");
              return "login/loginForm";
          }
          HttpSession session = request.getSession();
          session.setAttribute(SessionConst.LOGIN_MEMBER, loginMember);
          return "redirect:" + redirectURL;
      }
      ```
  - 빈 등록
    ```java
    @Bean
    public FilterRegistrationBean loginCheckFilter() {
        FilterRegistrationBean<Filter> filterRegistrationBean = new FilterRegistrationBean<>();
        filterRegistrationBean.setFilter(new LoginCheckFilter());
        filterRegistrationBean.setOrder(2);
        filterRegistrationBean.addUrlPatterns("/*");
        return filterRegistrationBean;
    }
    ```
    - 모든 URL에 필터가 호출되도록 하고 filter에서 URL을 확인하도록 했다.
    - 필터에 대한 성능 저하는 크지 않다.

#### Spring의 Interceptor

- 스프링 인터셉터도 서블릿 필터와 같이 웹과 관련된 공통 관심 사항을 효과적으로 해결할 수 있는 기술로 스프링 MVC가 제공한다.

- 특징
  - 흐름
    ```
    HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 스프링 인터셉터 -> 컨트롤러
    ```
    - 스프링 인터셉터는 디스패처 서블릿과 컨트롤러 사이에서 컨트롤러 호출 직전에 호출 된다.

  - 제한
    ```
    HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 스프링 인터셉터 -> 컨트롤러 //로그인 사용자
    HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 스프링 인터셉터(적절하지 않은 요청이라 판단, 컨트롤러 호출 X) // 비 로그인 사용자
    ```
    - 인터셉터에서 적절하지 않은 요청이라고 판단하면 거기에서 끝을 낼 수도 있다. 그래 로그인 여부를 체크하기에 딱 좋다.
        
  - 체인
    ```
    HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 인터셉터1 -> 인터셉터2 -> 컨트롤러
    ```
  
- 인터페이스
  ```java
  public interface HandlerInterceptor {
      default boolean preHandle(HttpServletRequest request, HttpServletResponse response,Object handler) throws Exception {}
      default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable ModelAndView modelAndView) throws Exception {}
      default void afterCompletion(HttpServletRequest request, HttpServletResponse response,Object handler, @Nullable Exception ex) throws Exception {}
  }
  ```
  - 인터셉터는 컨트롤러 호출 전( preHandle ), 호출 후( postHandle ), 요청 완료 이후( afterCompletion )와 같이 단계적으로 잘 세분화 되어 있다.
  - 서블릿 필터의 경우 단순히 request , response 만 제공했지만, 인터셉터는 어떤 컨트롤러( handler )가 호출되는지 호출 정보도 받을 수 있다. 그리고 어떤 modelAndView 가 반환되는지 응답 정보도 받을 수 있다
  
  - 정상 흐름
    - preHandle : 컨트롤러 호출 전에 호출된다. (더 정확히는 핸들러 어댑터 호출 전에 호출된다.)
      - preHandle 의 응답값이 true 이면 다음으로 진행하고, false 이면 더는 진행하지 않는다. 
      - false 인 경우 나머지 인터셉터는 물론이고, 핸들러 어댑터도 호출되지 않는다.
    - postHandle : 컨트롤러 호출 후에 호출된다. (더 정확히는 핸들러 어댑터 호출 후에 호출된다.)
    - afterCompletion : 뷰가 렌더링 된 이후에 호출된다.
  
  - Controller에서 예외 발생 시
    - preHandle : 컨트롤러 호출 전에 호출된다.
    - postHandle : 컨트롤러에서 예외가 발생하면 postHandle 은 호출되지 않는다.
    - afterCompletion : afterCompletion 은 항상 호출된다. 이 경우 예외( ex )를 파라미터로 받아서 어떤 예외가 발생했는지 로그로 출력할 수 있다.
  
- 예시 1) 요청 로그 인터셉터
  - 신규 인터셉터 생성
    ```java
    @Slf4j
    public class LogInterceptor implements HandlerInterceptor {
        public static final String LOG_ID = "logId";
        @Override
        public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
            String requestURI = request.getRequestURI();
            String uuid = UUID.randomUUID().toString();
            request.setAttribute(LOG_ID, uuid);

            //@RequestMapping: HandlerMethod
            //정적 리소스: ResourceHttpRequestHandler
            if (handler instanceof HandlerMethod) {
                HandlerMethod hm = (HandlerMethod) handler; //호출할 컨트롤러 메서드의 모든 정보가 포함되어 있다.
            }
            log.info("REQUEST [{}][{}][{}]", uuid, requestURI, handler);
            return true; //false 진행X
        }
        @Override
        public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
            log.info("postHandle [{}]", modelAndView);
        }
        @Override
        public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
            String requestURI = request.getRequestURI();
            String logId = (String)request.getAttribute(LOG_ID);

            log.info("RESPONSE [{}][{}]", logId, requestURI);

            if (ex != null) {
                log.error("afterCompletion error!!", ex);
            }
        }
    }
    ```
    - request.setAttribute(LOG_ID, uuid);
      - preHandle, postHandle, afterCompletion 이 나누어져 있기 때문에 request 객체에 담아서 사용
      
    - HandlerMethod
      - 핸들러 정보는 어떤 핸들러 매핑을 사용하는가에 따라 달라진다. 스프링을 사용하면 일반적으로 @Controller , @RequestMapping 을 활용한 핸들러 매핑을 사용하는데, 이 경우 핸들러 정보로 HandlerMethod 가 넘어온다

    - ResourceHttpRequestHandler
      - @Controller 가 아니라 /resources/static 와 같은 정적 리소스가 호출 되는 경우 ResourceHttpRequestHandler 가 핸들러 정보로 넘어오기 때문에 타입에 따라서 처리가 필요하다.

    - postHandle, afterCompletion
      - 종료 로그를 postHandle 이 아니라 afterCompletion 에서 실행한 이유는, 예외가 발생한 경우 postHandle 가 호출되지 않기 때문이다. afterCompletion 은 예외가 발생해도 호출 되는 것을 보장한다.
  
  - 인터셉터 등록
    ```java
    @Configuration
    public class WebConfig implements WebMvcConfigurer {
        @Override
        public void addInterceptors(InterceptorRegistry registry) {
            registry.addInterceptor(new LogInterceptor())
            .order(1)
            .addPathPatterns("/**")
            .excludePathPatterns("/css/**", "/*.ico", "/error");
        }
    }
    ```
    
- 예시 2) 인증 인터셉터
  - 새로운 인터셉터 생성
    ```java
    @Slf4j
    public class LoginCheckInterceptor implements HandlerInterceptor {
        @Override
        public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
            String requestURI = request.getRequestURI();
            log.info("인증 체크 인터셉터 실행 {}", requestURI);
            HttpSession session = request.getSession(false);
            
            if (session == null || session.getAttribute(SessionConst.LOGIN_MEMBER) == null) {
                log.info("미인증 사용자 요청");
                //로그인으로 redirect
                response.sendRedirect("/login?redirectURL=" + requestURI);
                return false;
            }
            
            return true;
        }
    }
    ```
    - 인증은 preHandle 만 구현하면 된다.
    
  - 인터셉터 등록
    ```java
    @Configuration
    public class WebConfig implements WebMvcConfigurer {
        @Override
        public void addInterceptors(InterceptorRegistry registry) {
            registry.addInterceptor(new LogInterceptor())
                .order(1)
                .addPathPatterns("/**")
                .excludePathPatterns("/css/**", "/*.ico", "/error");
            
            registry.addInterceptor(new LoginCheckInterceptor())
                .order(2)
                .addPathPatterns("/**")
                .excludePathPatterns("/", "/members/add", "/login", "/logout", "/css/**", "/*.ico", "/error");
        }
    }
    ```

#### 출처

- 인프런 강의: 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술 (김영한님 강의)
- https://velog.io/@fortice/Spring-%EC%84%B8%EC%85%98-%EC%9D%B8%ED%84%B0%EC%85%89%ED%84%B0-%EC%BF%A0%ED%82%A4
