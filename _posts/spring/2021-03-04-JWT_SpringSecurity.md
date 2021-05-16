---
layout: post
title: JWT, Spring Security 연동
summary: JWT 인증
author: devhtak
date: '2021-03-04 21:41:00 +0900'
category: Spring
---

#### 이전 글

- Spring Session을 이용한 로그인
  https://devhtak.github.io/spring/2021/02/23/JWT_SpringSession.html
- JWT와 Spring boot를 이용한 로그인
  https://devhtak.github.io/spring/2021/02/28/JWT_SpringBoot.html

#### JWT와 Spring Security

- JWT와 Spring Security를 연동하여 로그인 구현을 진행하려고 한다.
- 이전 글에서는 하드코딩된 User와 Coffee를 조회하였는 데, JPA를 활용하여 DB 연동까지 진행하려고 한다.

- 회원 엔티티
  - Member.java
  ```java
  @Getter @Entity
  public class Member {
  
      @Id @GeneratedValue 
      @JsonIgnore
      private Long id;

      @NotNull @Size(min = 4, max = 50)
      private String username;

      @NotNull @Size(min = 4, max = 100)
      private String password;

      @NotNull @Size(min = 4, max = 50)
      private String email;

      @NotNull @Size(min = 4, max = 50)
      private String role;
  }
  ```
  - //TODO password에 경우 encoding된 것을 저장해야 하기 떄문에 범위가 달라질 수 있다.
  - //TODO role과 enum 타입의 Role의 차이가 발생했다. @Enumerated(EnumType.STRING)로 타입이 저장될 수 있도록 수정이 필요하다.
    - EnumType.ORDINAL과 EnumType.STRING
    - Anti pattern: ordinal에 경우 숫자가 저장되어 type safety에 위배된다. 만약 enum타입 중간에 추가되거나 순서가 변경되는 경우, 저장된 데이터가 완전 꼬이게 된다.

- 인증 & 인가 구현
  - 기존 Http Session을 활용한 예제와 JWT + Spring Boot를 활용한 예제에서는 Interceptor를 구현했지만 여기에서는 Filter를 사용하여 Security Config에 설정한다.
    - Interceptor: prevHandle, postHandle, afterCompletion으로 핸들러 앞에서 요청에 대한 처리를 해줄 수 있다.
    - Filter: DispatcherServlet 앞단에서 동작하여 요청에 대한 처리를 해줄 수 있다.
    - 둘의 가장 큰 차이는 Context(실행 영역)에 있다. Filter에 경우 Web Application 내에서 동작하기 때문에 Spring Context에 접근하기 어렴지만, Interceptor에 경우 Spring 영역 내에 있기 때문에 Spring Context에 접근하기 용이하다.
  - JwtFilter.java
    ```java
    public class JWTFilter extends GenericFilterBean {

        private static final String AUTHORIZATION_HEADER = "x-auth-token";
        private JwtAuthTokenProvider jwtAuthTokenProvider;

        JWTFilter(JwtAuthTokenProvider jwtAuthTokenProvider) {
            this.jwtAuthTokenProvider = jwtAuthTokenProvider;
        }

        @Override
        public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain)
            throws IOException, ServletException  {
            HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;
            Optional<String> token = resolveToken(httpServletRequest);

            if (token.isPresent()) {
                JwtAuthToken jwtAuthToken = jwtAuthTokenProvider.convertAuthToken(token.get());
                if(jwtAuthToken.validate()) {
                    Authentication authentication = jwtAuthTokenProvider.getAuthentication(jwtAuthToken);
                    SecurityContextHolder.getContext().setAuthentication(authentication);
                }
            }
            filterChain.doFilter(servletRequest, servletResponse);
       }

       private Optional<String> resolveToken(HttpServletRequest request) {
            String authToken = request.getHeader(AUTHORIZATION_HEADER);
            if (StringUtils.hasText(authToken)) {
                return Optional.of(authToken);
            } else {
                return Optional.empty();
            }
        }
    }
    ```
    
    - GenericFilterBean을 활용하여 Filter 생성
      - Spring은 spring config 설정 정보를 쉽게 처리하기 위해 GenericFilterBean을 제공한다.
      - Filter 구현과 동일하며 getFilterConfig()나 getEnvironment()를 제공한다.
    - 인증 과정은 동일하다.
    - Spring Security에 경우 SecurityContext에 인증 객체를 설정해야 한다!!
      ```java
      SecurityContextHolder.getContext().setAuthentication(authentication);
      ```
    - 인증 객체는 Authentication 의 구현체만 가능하며, UsernamePasswordAuthenticationToken을 활용하였다.
  
  - JwtFilter를 등록한 후 WebSecurityConfigurerAdapter에 적용해준다.
    - SecurityConfigureAdapter를 상속하는 JWTConfigurer 클래스에 필터를 설정한 후
    - WebSecurityConfig 설정에 필터를 최종 설정하였다.
    
    - JwtConfigure.java
      ```java
      public class JWTConfigurer extends SecurityConfigurerAdapter<DefaultSecurityFilterChain, HttpSecurity> {
          private JwtAuthTokenProvider jwtAuthTokenProvider;

          public JWTConfigurer(JwtAuthTokenProvider jwtAuthTokenProvider) {
              this.jwtAuthTokenProvider = jwtAuthTokenProvider;
          }

          @Override
          public void configure(HttpSecurity http) {
              JWTFilter customFilter = new JWTFilter(jwtAuthTokenProvider);
              http.addFilterBefore(customFilter, UsernamePasswordAuthenticationFilter.class);
          }
      }
      ```
      
    - SecurityConfig.java
      ```java
      @Configuration
      @EnableWebSecurity
      @RequiredArgsConstructor
      public class SecurityConfig extends WebSecurityConfigurerAdapter{
          private final JwtAuthTokenProvider jwtAuthTokenProvider;
          private final JwtAuthenticationEntryPoint authenticationErrorHandler;
          private final JwtAccessDeniedHandler jwtAccessDeniedHandler;

          @Bean
          public PasswordEncoder passwordEncoder() {
              return new BCryptPasswordEncoder();
          }

          @Override
          protected void configure(HttpSecurity httpSecurity) throws Exception {
              httpSecurity
                  .csrf().disable()

                  .exceptionHandling()
                  .authenticationEntryPoint(authenticationErrorHandler)
                  .accessDeniedHandler(jwtAccessDeniedHandler)

                  .and()
                  .headers()
                  .frameOptions()
                  .sameOrigin()

                  .and()
                  .sessionManagement()
                  .sessionCreationPolicy(SessionCreationPolicy.STATELESS)

                  .and()
                  .authorizeRequests()
                  .antMatchers("/api/v1/login/**").permitAll()

                  .antMatchers("/api/v1/coffees/**").hasAnyAuthority(Role.USER.getCode())
                  .anyRequest().authenticated()

                  .and()
                  .apply(securityConfigurerAdapter());
          }

          @Override
          public void configure(WebSecurity web) {
              web.ignoring()
                  .antMatchers(HttpMethod.OPTIONS, "/**")

                  // allow anonymous resource requests
                  .antMatchers(
                      "/",
                      "/h2-console/**"
                  );
          }

          private JWTConfigurer securityConfigurerAdapter() {
              return new JWTConfigurer(jwtAuthTokenProvider);
          }
      }
      ```
      - exceptionHandling() 
        - 인증 또는 인가에 실패한 경우 exception 처리
      - sessionManagement() 
        - 세션 기능을 사용하지 않는다.
      - antMatchers("/api/v1/login").permitAll() 
        - 인증 없이 /api/v1/login으로 접근 가능하다.
      - antMatchers("/api/v1/coffess").hasAnyAuthority(Role.USER.getCode())
        - Role.USER 권한이 있는 경우 api/v1/coffess 접근이 가능하다.
      - headers().frameOptions().sameOrigin()
        - HTTP 응답 헤더 중 X-Frame-Options가 있다. frame 또는 iframe 태그로 접근할 수 있는 설정
        - deny: 어떤 사이트에서도 frame 상에서 보여질 수 없다.
        - sameOrigin: 동일한 사이트의 frame만 보여진다.
        - allow-from uri: 지정된 특정 URI의 frame만 보여진다.

- 로그인 구현 및 인증 통과

  - LoginController.java
    ```java
    @RestController
    @RequiredArgsConstructor
    public class LoginController {
        private final LoginService loginService;

        @PostMapping("/api/v1/login")
        public CommonResponse login(@RequestBody LoginRequestDTO loginRequestDTO) {

            MemberDTO optionalMemberDTO = loginService.login(loginRequestDTO.getEmail(), loginRequestDTO.getPassword())
                .orElseThrow(LoginFailedException::new);

            JwtAuthToken jwtAuthToken = (JwtAuthToken) loginService.createAuthToken(optionalMemberDTO);

            return CommonResponse.builder()
                    .code("LOGIN_SUCCESS")
                    .status(200)
                    .message(jwtAuthToken.getToken())
                    .build();

        }
    }
    ```
    - login한 객체를 가져와, token을 생성하고, response message에 넣어 보내준다.
  - LoginService.java
    ```java
    @Service
    @RequiredArgsConstructor
    public class LoginService implements LoginUseCase {

        private final AuthenticationManagerBuilder authenticationManagerBuilder;
        private final JwtAuthTokenProvider jwtAuthTokenProvider;
        private final static long LOGIN_RETENTION_MINUTES = 30;


        @Override
        public Optional<MemberDTO> login(String email, String password) {
            // TODO Auto-generated method stub
            UsernamePasswordAuthenticationToken authenticationToken = new UsernamePasswordAuthenticationToken(email, password);
            Authentication authentication = authenticationManagerBuilder.getObject().authenticate(authenticationToken);
            SecurityContextHolder.getContext().setAuthentication(authentication);

            Role role = authentication.getAuthorities().stream()
                        .map(GrantedAuthority::getAuthority)
                        .findFirst()
                        .map(Role::of)
                        .orElse(Role.UNKNOWN);

            MemberDTO memberDTO = MemberDTO.builder()
                    .username("eddy")
                    .email(email)
                    .role(role)
                    .build();

            return Optional.ofNullable(memberDTO);
        }

        @Override
        public AuthToken createAuthToken(MemberDTO memberDTO) {
            // TODO Auto-generated method stub
            Date expiredDate = Date.from(LocalDateTime.now().plusMinutes(LOGIN_RETENTION_MINUTES).atZone(ZoneId.systemDefault()).toInstant());
            return jwtAuthTokenProvider.createAuthToken(memberDTO.getEmail(), memberDTO.getRole().getCode(), expiredDate);
        }
    }
    ```
    - login 메서드를 확인해보면 UsernamePasswordAuthenticationToken 인증 객체를 만든다.
    - 패스워드 비교 등에 로직은 구현하지 않았다.
    - UserDetailsService에서 구현한 loadUserByUsername 메서드에서 실행한다.
  - CustomUserDetailsService.java
    ```java
    @Service
    @RequiredArgsConstructor
    public class CustomUserDetailsService implements UserDetailsService{
        private final MemberRepository memberRepository;

        @Override
        public UserDetails loadUserByUsername(String email) throws UsernameNotFoundException {
            // TODO Auto-generated method stub
            return memberRepository.findByEmail(email)
                .map(this::createSpringSecurityUser)
                .orElseThrow(RuntimeException::new);
        }

        private User createSpringSecurityUser(Member member) {
            List<GrantedAuthority> authority = Collections.singletonList(new SimpleGrantedAuthority(member.getRole()));

            return new User(member.getEmail(), member.getPassword(), authority);
        }
    }
    ```
    - loadByUsername에서 회원 객체를 가져와야 한다.
    - UserDetails 인터페이스를 리턴해야 하는데, 여기에서는 Spring에서 제공하는 User를 리턴했다.
    - User 객체 생성할 때에는 username, password, Collection<? extends GrantedAuthority> authorities가 파라미터로 입력되는데, authorities는 ROLE을 정의하는 리스트로 만들어준다.

- Spring Security ExceptionHandler
  - Login 실패 시 BadCredentialsException 예외가 발생한다.
    ```java
    @ExceptionHandler(BadCredentialsException.class)
    protected ResponseEntity<CommonResponse> handleBadCredentialsException(BadCredentialsException e) {

        log.info("handleBadCredentialsException", e);

        CommonResponse response = CommonResponse.builder()
                .code(ErrorCode.Login_FAILED.getCode())
                .message(e.getMessage())
                .status(ErrorCode.Login_FAILED.getStatus())
                .build();

        return new ResponseEntity<>(response, HttpStatus.UNAUTHORIZED);
    }
    ```
  - 인증 실패
    ```java
    private final JwtAuthenticationEntryPoint authenticationErrorHandler;
    private final JwtAccessDeniedHandler jwtAccessDeniedHandler;
    
    // ...
    .exceptionHandling()
    .authenticationEntryPoint(authenticationErrorHandler)
    .accessDeniedHandler(jwtAccessDeniedHandler)
    // ...
    ```
    - 인증이 되지 않았을 경우 AuthenticationEntryPoint 부분에서 AuthenticationException이 발생한다.
    - 인증은 되었으나 해당 요청에 대한 권한이 없는 경우 AccessDeniedHandler 부분에서 AccessDeniedException이 발생한다.
    - 그렇기 떄문에 인증 실패에 대한 재정의는 AuthenticationEntryPoint, AccessDeniedHandler를 재정의 해서 사용한다.
    - JwtAccessDeniedHandler
      ```java
      @Component
      @RequiredArgsConstructor
      public class JwtAuthenticationEntryPoint implements AuthenticationEntryPoint {
          private final HandlerExceptionResolver handlerExceptionResolver;

          @Override
          public void commence(HttpServletRequest request,
                               HttpServletResponse response,
                               AuthenticationException authException) throws IOException {

              //response.sendError(HttpServletResponse.SC_UNAUTHORIZED, authException.getMessage());
              handlerExceptionResolver.resolveException(request, response, null, authException);

          }
      }
      ```
   - AuthenticationExceptionHandler
     ```java
     @ExceptionHandler(InsufficientAuthenticationException.class)
     protected ResponseEntity<CommonResponse> handleInsufficientAuthenticationException(InsufficientAuthenticationException e) {

         log.info("handleInsufficientAuthenticationException", e);

         CommonResponse response = CommonResponse.builder()
             .code(ErrorCode.AUTHENTICATION_FAILED.getCode())
             .message(ErrorCode.AUTHENTICATION_FAILED.getMessage())
             .status(ErrorCode.AUTHENTICATION_FAILED.getStatus())
             .build();

         return new ResponseEntity<>(response, HttpStatus.UNAUTHORIZED);
     }
     ```
    - JwtAuthenticationEntryPoint
      ```java
      @Component
      @RequiredArgsConstructor
      public class JwtAccessDeniedHandler implements AccessDeniedHandler {

          private final HandlerExceptionResolver handlerExceptionResolver;

          @Override
          public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException accessDeniedException) throws IOException {
            handlerExceptionResolver.resolveException(request, response, null, accessDeniedException);
          }
      }
      ```
    - AccessDeniedExceptionHandler
      ```java
      @ExceptionHandler(AccessDeniedException.class)
      protected ResponseEntity<CommonResponse> handleAccessDeniedException(AccessDeniedException e) {

          log.info("handleAccessDeniedException", e);

          CommonResponse response = CommonResponse.builder()
                .code(ErrorCode.ACCESS_DENIED.getCode())
                .message(ErrorCode.ACCESS_DENIED.getMessage())
                .status(ErrorCode.ACCESS_DENIED.getStatus())
                .build();

          return new ResponseEntity<>(response, HttpStatus.UNAUTHORIZED);
      }
      ```

- 출처: https://brunch.co.kr/@springboot/491
