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

- 인증 & 인가 객체
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

- 출처: https://brunch.co.kr/@springboot/491
