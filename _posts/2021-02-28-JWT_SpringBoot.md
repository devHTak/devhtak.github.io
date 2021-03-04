---
layout: post
title: JWT 기본과 Spring Boot 연동
summary: JWT 인증
author: devhtak
date: '2021-02-28 21:41:00 +0900'
category: Spring
---

#### JWT(JSON Web Token)

- JSON Web Token 이란
  - JSON 객체로 데이터를 안전하게 전송하기 위한 개방형 표준(RFC7519)이다.
  - 해당 정보는 디지털 서명이 되어 있어 신뢰할 수 있다.
    - 디지털 서명
  - HMAC 알고리즘 또는 RSA 또는 ECDSA를 사용하는 공개/개인키 쌍을 사용한다.
  
- JWT 사용 방식
  - Authorization
    - JWT를 사용하는 가장 일반적인 방식이다.
    - 클라이언트가 로그인 한 이후 JWT가 포함된 요청을 보내며, 토큰에 대한 허가를 바탕으로 서비스를 사용할 수 있게 한다.
    - 작은 overhead 장점으로 최근 SSO에서도 JWT를 많이 사용하고 있다.
  
  - Information Exchange
    - JWT는 두 개체 사이에서 안정성 있게 정보를 교환하기에 좋은 방법이다.
    - JWT는 개인/공개키를 사용한 인증을 사용하기 때문에 sender에 대한 검증이 가능하다.
    - header와 payload가 sign되어 있기 때문에 정보 조작에 대한 검증이 가능하다.

- JWT 구조
  - Header, Payload, Signature 구조로 이루어져 있으며 .으로 구분되어 있다.
    - XXXXX.YYYYY.ZZZZZ
  - Header
    - 헤더는 일반적으로 2가지로 구성되어 있다. 타입과 암호 알고리즘
    - 암호 알고리즘은 HMAC, SHA256, RSA 등을 사용한다.
      ```
      {
          "alg": "HS256",
          "typ": "JWT"
      }
      ```
  - Payload
    - 두 번째 부분은 payload 이며, claims를 포함하고 있다.
    - claims
      - Entity(일반적으론 User) 및 추가 데이터
      - (type)Registerd claims: 서비스에 필요한 정보들이 아닌, 토큰에 대한 정보를 담기 위해 이미 정해진 클레임, optional
      - (type)public claims: 충돌을 방지하기 위한 이름으로 URI 형식으로 짓습니다.
      - (type)private claims: 클라이언트와 서버간의 협의 하에 사용되는 클레임
    - 예제
      ```
      {
          "sub": "!234567890",
          "name": "John Doe",
          "admin": true
      }
      ```
  - Signature
    - Header의 인코딩 값과 Payload의 인코딩한 값을 주어진 비밀키로 해쉬하여 생성한다.
    - HMAC SHA256 알고리즘으로 생성한 Signature 예제
      ```
      HMACSHA256(
          base64UrlEncode(header) + "." +
          base64UrlEncode(payload), secret
      )
      ```
      
  - JWT는 Base64Url로 인코딩하여 형성한다.
- 출처: https://jwt.io/introduction/

#### Spring Boot에서 JWT 사용하기

- 소스 저장소 위치: https://github.com/devHTak/SpringAuth/tree/main/UsingJwtSpring

- 의존성 추가
  - jjwt-api
  - jjwt-impl
  - jjwt-jackson
  ```
  <!-- https://mvnrepository.com/artifact/io.jsonwebtoken/jjwt-api -->
  <dependency>
      <groupId>io.jsonwebtoken</groupId>
      <artifactId>jjwt-api</artifactId>
      <version>0.11.2</version>
  </dependency>
  <!-- https://mvnrepository.com/artifact/io.jsonwebtoken/jjwt-impl -->
  <dependency>
      <groupId>io.jsonwebtoken</groupId>
      <artifactId>jjwt-impl</artifactId>
      <version>0.11.2</version>
      <scope>runtime</scope>
  </dependency>
  <!-- https://mvnrepository.com/artifact/io.jsonwebtoken/jjwt-jackson -->
  <dependency>
      <groupId>io.jsonwebtoken</groupId>
      <artifactId>jjwt-jackson</artifactId>
      <version>0.11.2</version>
      <scope>runtime</scope>
  </dependency>
  ```

- 인증&인가를 위한 Interceptor
  - AuthInterceptor.java
    ```java
    @Slf4j
    @Component
    @RequiredArgsConstructor
    public class AuthInterceptor implements HandlerInterceptor {
        private final JwtAuthTokenProvider jwtAuthTokenProvider;
        private static final String AUTHORIZATION_HEADER = "x-auth-token";

        @Override
        public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)throws Exception {
            // TODO Auto-generated method stub

            log.info("AuthInterceptor preHandle");

            String token = resolveToken(request).orElseThrow(CustomAuthenticationException::new);
            JwtAuthToken jwtAuthToken = jwtAuthTokenProvider.convertAuthToken(token);
            if(jwtAuthToken.validate() && Role.USER.getCode().equals(jwtAuthToken.getData().get("role"))) {
                return true;
            }

            throw new CustomAuthenticationException();
        }

        private Optional<String> resolveToken(HttpServletRequest request) {
            String authToken = request.getHeader(AUTHORIZATION_HEADER);
            if(StringUtils.hasText(authToken)) {
                return Optional.of(authToken);
            }

            return Optional.empty();
        }
    }
    ```
    - 요청이 올 때 Request 헤더에 담겨 있는 x-auth-token 값을 가지고 온다.
    - JwtAuthToken은 인증을 진행하며, JwtAuthTokenProvider는 header에 담겨있는 값을 Token 객체로 변경하는 역할을 한다.
    - 유효 여부(토큰 값이 있고, role 확인) true를 리턴하여 요청을 진행시키고, 아닌 경우 CustomAuthenticationException을 발생시킨다.
      - 유효한 사용자인지(유효한 토큰인지) -> 인증
      - 리소스에 대한 권한이 있는지 검증 -> 인가

  - WebMvcConfig.java
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
    - AuthInterceptor를 등록시킨다.
    - Interceptor 예외 경로 등을 지정한다.

- 로그인 구현
  - 로그인 확인한 후에 token을 생성하여 response로 보내준다.
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
  - LoginService.java
    ```java
    @Service
    @RequiredArgsConstructor
    public class LoginService implements LoginUseCase {
        private final JwtAuthTokenProvider provider;
        private final static Long LOGIN_RETENTION_MINUTES = 30L;
        
        @Override
        public Optional<MemberDTO> login(String email, String password) {
            // TODO Auto-generated method stub		
            // TODO Login 연동		
            MemberDTO member = MemberDTO.builder()
                .username("test")
                .email(email)
                .role(Role.USER)
                .build();

            return Optional.of(member);
        }

        @Override
        public AuthToken createAuthToken(MemberDTO member) {
            // TODO Auto-generated method stub
            ZonedDateTime datePlusMinutes = LocalDateTime.now().plusMinutes(LOGIN_RETENTION_MINUTES).atZone(ZoneId.systemDefault());
            Date expiredDate = Date.from(datePlusMinutes.toInstant());

            return provider.createAuthToken(member.getId(), member.getRole().getCode(), expiredDate);
        }
    }
    ```

- 인증 & 인가 를 위한 JWT 구현
  - 사용자가 로그인을 성공하면 서버는 JWT 토큰을 생성한 후 생성된 토큰을 프론트엔드에 전달한다.
  - Front에서는 로그인이 성공한 후 받는 JWT 토큰을 잘 저장하여 필요한 리소스를 요청할 때 백엔드 API를 호출하면서 JWT 토큰을 HTTP 헤더에 함께 전송해야 한다.
  - 그러면 등록해 놓은 인터셉터가 해당 HTTP 헤더에 저장되어 있는 JWT 토큰을 가져와 확인한다.
  
  - JwtAuthToken.java
    ```java
    @Slf4j
    @Getter
    public class JwtAuthToken implements AuthToken<Claims>{
        private final String token;
        private final Key key;
        private static final String AUTHORITIES_KEY = "role";

        public JwtAuthToken(String token, Key key) {
            this.token = token;
            this.key = key;
        }

        public JwtAuthToken(String id, String role, Date expiredDate, Key key) {
            this.key = key;
            this.token = createJwtAuthToken(id, role, expiredDate).get();
        }

        @Override
        public boolean validate() {
            // TODO Auto-generated method stub
            return getData() != null;
        }

        @Override
        public Claims getData() {
            // TODO Auto-generated method stub
            try {
                return Jwts.parserBuilder().setSigningKey(key).build().parseClaimsJws(token).getBody();
            } catch(SecurityException e) {
                log.info("Invalid JWT signature");
                throw new CustomJwtRuntimeException();
            } catch(MalformedJwtException e) {
                log.info("Invalid JWT token");
                throw new CustomJwtRuntimeException();
            } catch(ExpiredJwtException e) {
                log.info("Expired JWT token");
                throw new CustomJwtRuntimeException();
            } catch(UnsupportedJwtException e) {
                log.info("Unsupported JWT token");
                throw new CustomJwtRuntimeException();
            } catch(IllegalArgumentException e) {
                log.info("Jwt token compact of handler are invalid.");
                throw new CustomJwtRuntimeException();
            }
        }

        private Optional<String> createJwtAuthToken(String id, String role, Date expiredDate) {
            String token = Jwts.builder()
                .setSubject(id)
                .claim(AUTHORITIES_KEY, role)
                .signWith(key, SignatureAlgorithm.HS256)
                .setExpiration(expiredDate)
                .compact();

            return Optional.of(token);
        }
    }
    ```
    - JwtAuthToken 객체는 id, role, expiredDate를 가지고 token 생성
      - setExpiration을 통해 토큰 만료시간을 지정해야 한다.
      - Secret Sign Key는 반드시 설정해야 하며, 해당 Key를 3자에게 절대 노출하면 안된다.
      
    - 인증, 인가 메서드를 제공
  - JwtConfiguration.java
    ```java
    @Configuration
    public class JwtConfiguration {
        @Value("${jwt.secret}")
        private String secret;

        @Bean
        public JwtAuthTokenProvider jwtProvider() {
            return new JwtAuthTokenProvider(secret);
        }
    }
    ```
    - JwtAuthTokenProvider 빈등록
    
  - JwtAuthTokenProvider.java
    ```java
    @Slf4j
    public class JwtAuthTokenProvider implements AuthTokenProvider<JwtAuthToken> {
      private final Key key;

      public JwtAuthTokenProvider(String secret) {
        this.key = Keys.hmacShaKeyFor(secret.getBytes());
      }

      @Override
      public JwtAuthToken createAuthToken(String id, String role, Date expiredDate) {
        // TODO Auto-generated method stub
        return new JwtAuthToken(id, role, expiredDate, this.key);
      }

      @Override
      public JwtAuthToken convertAuthToken(String token) {
        // TODO Auto-generated method stub
        return new JwtAuthToken(token, this.key);
      }
    }
    ```
    - JwtAuthToken 사이의 provider 객체로 token을 넘겨주고 JwtAuthToken을 가져오는 중간 역할을 한다.

- 테스트
  - LoginControllerTest.java
    ```java
    @SpringBootTest(webEnvironment = WebEnvironment.MOCK)
    @AutoConfigureMockMvc
    public class LoginControllerTest {
        @Autowired MockMvc mockMvc;
        @Autowired ObjectMapper objectMapper;

        @Test
        void loginTest() throws Exception {
            LoginRequestDTO login = LoginRequestDTO.builder()
                .email("test@test.com")
                .password("test1234")
                .build();

            mockMvc.perform(post("/api/v1/login")
                  .contentType(MediaType.APPLICATION_JSON_VALUE)
                  .content(objectMapper.writeValueAsString(login)))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(content().contentType(MediaType.APPLICATION_JSON_VALUE));
        }
    }
    ```
  - CoffeeControllerTest.java
    ```java
    @SpringBootTest(webEnvironment = WebEnvironment.MOCK)
    @AutoConfigureMockMvc
    public class CoffeeControllerTest {
        @Autowired MockMvc mockMvc;
        @Autowired LoginService loginService;

        private static final String AUTHORIZATION_HEADER = "x-auth-token";
        private String loginToken;

        @BeforeEach
        void beforeEach() throws Exception {
            MemberDTO member = loginService.login("test@test.com", "test1234")
                .orElseThrow(LoginFailedException::new);
            JwtAuthToken token = (JwtAuthToken)loginService.createAuthToken(member);
            this.loginToken = token.getToken();
        }

        @Test
        void coffeeTest() throws Exception {
            mockMvc.perform(get("/api/v1/coffees")
                .header(AUTHORIZATION_HEADER, loginToken))
              .andDo(print())
              .andExpect(status().isOk());
        }
    }
    ```
    - interceptor에서 확인하는 token을 만들기 위해 BeforeEach에서 구현

#### JWT 장점

- Session ID의 한계 해결
  - 여러 서버를 운영한다 하더라도 같은 Key값을 사용하고 있기 때문에 확인하는 토큰 값은 일정하게 유지된다.
  - 하지만 Secret Key의 노출이 문제가 될 수 있다.
  - 그래서, JWT Token expire 기간을 짧게 주는 것이 좋다.

- 출처: https://brunch.co.kr/@springboot/491
