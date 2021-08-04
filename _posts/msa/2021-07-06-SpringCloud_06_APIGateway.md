---
layout: post
title: 마스터 스프링 클라우드 - 07. API Gateway
summary: Spring Cloud
author: devhtak
date: '2021-07-06 22:41:00 +0900'
category: msa
---

#### API Gateway란?

![image](https://user-images.githubusercontent.com/42403023/124608140-704f0c00-dea9-11eb-9e30-e16c9927ccab.png)

** 이미지 출처: https://m.blog.naver.com/dktmrorl/222129517689

- API Gateway는 서비스 앞단에서 모든 API 요청을 단일화해주는 역할을 한다.
- 즉, 클라이언트 측에서는 마이크로서비스에 IP 변경등에 영향을 받지 않고 오로지 API Gateway에게 요청을 보내며 해당 요청을 API Gateway가 서비스에 보내주는 역할을 한다.

- API Gateway 기능 
  - 인증 및 권한 부여
  - 서비스 검색 통합
  - 응답 캐싱
  - 정책, 회로 차단기 및 QoS 다시 시도
  - 속도 제한
  - 부하 분산
  - 로깅, 추적, 상관관계
  - 헤더, 쿼리 문자열 및 청구 변환
  - IP 허용 목록에 추가(방화벽)

- Spring Cloud에서의 MSA 간 통신
  - RestTemplate
    ```java
    RestTemplate restTemplate = new RestTemplate();
    restTemplate.getForObject("http://localhost:8080/", User.class, 200);
    ```
  - Feign Client
    ```java
    @FeignClient("stores")
    public interface StoreClient {
      @GetMapping(value="/stores")
      List<Store> getStores();
    }
    ```
    
- Ribbon: Client side LoadBalancer
  - 서비스 이름으로 호출 (MSA Service Name)
  - Health Check
  - Reactive(RxJava)와 호환이 안되어 최근에는 많이 사용하지 않는다.
  - Spring Cloud Ribbon은 Spring Boot 2.4에서 maintenance 상태

- Netflix Zuul
  - Routing, API Gateway 역할을 한다.
  - client는 service를 직접 호출하는 것이 아닌 Zuul에 요청을 하면 Zuul이 service에게 요청을 보낸다.    
  - Spring Cloud Zuul은 Spring Boot 2.4에서 mainenance 상태

#### Netflix Zuul 구현

- Spring Boot 2.3.8을 사용해야 한다 (2.4는 지원하지 않는다.
- First Service와 Second Service 구현
  - dependency: lombok, web, eureka client
  - RestController: GET /welcome 요청 처리
    ```java
    @RestController
    public void FirstController {
      @GetMapping("/welcome")
      public String welcome() {
        return "Welcome First Service";
      }
    }
    ```
  - application.yml
    ```
    server:
      port: 8081
    spring:
      application:
        name: first-service
    eureka:
      client:
        register-with-eureka: false
        fetchRegistry: false
    ```
    ```
    server:
      port: 8082
    spring:
      application:
        name: second-service
    eureka:
      client:
        register-with-eureka: false
        fetchRegistry: false
    ```
    
- Zuul Service 구현
  - Spring Boot: 2.3.8
  - dependency: lombok, web, zuul
  - 메인 함수
    ```java
    @SpringBootApplication
    @EnableZuulProxy
    public class ZuulServiceApplication {
      public static void main(String[] args) {
        SpringApplication.run(ZuulServiceApplication.class, args);
      }
    }
    ```
    
  - application.yml 설정
    ```
    server:
      port: 8000

    spring:
      application:
        name: zuul-service

    zuul:
      routes:
        first-service:
          path: /first-service/**
          url: http://localhost:8081
        second-service:
          path: /second-service/**
          url: http://localhost:8082
    ```
  - 테스트
    - 요청1: http://localhost:8080/first-service/welcome
    - 요청2: http://localhost:8080/second-service/welcome

- ZuulFilter
  
  - ZuulFilter를 상속받아 Filter를 구현할 수 있다.
  - filterType(): 요청 사전, 사후 필터를 작성할 수 있으며, filterType()이 pre면 사전, post는 사후이다.
  - filterOrder(): 필터의 순서를 지정할 수 있다.
  - shouldFilter(): true/false : 필터 사용여부
  - run(): 필터 실행 메소드
    ```java
    @Slf4j
    @Component
    public class ZuulLoggingFilter extends ZuulFilter {

      @Override
      public Object run() throws ZuulException {
        // TODO Auto-generated method stub
        log.info("====print log");
        HttpServletRequest request = RequestContext.getCurrentContext().getRequest();
        log.info("request: " + request.getRequestURI());
        return null;
      }
      @Override
      public boolean shouldFilter() {
        // TODO Auto-generated method stub
        return true;
      }
        @Override
      public String filterType() {
        // TODO Auto-generated method stub
        return "pre";
      }
      @Override
      public int filterOrder() {
        // TODO Auto-generated method stub
        // filter 순서
        return 0;
      }
    }
    ```
  - 로그를 확인할 수 있다.
    ```
    2021-07-06 23:42:05.017  INFO 10252 --- [nio-8000-exec-2] c.example.demo.filter.ZuulLoggingFilter  : ====print log
    2021-07-06 23:42:05.018  INFO 10252 --- [nio-8000-exec-2] c.example.demo.filter.ZuulLoggingFilter  : request: /second-service/welcome
    2021-07-06 23:42:15.155  INFO 10252 --- [nio-8000-exec-3] c.example.demo.filter.ZuulLoggingFilter  : ====print log
    2021-07-06 23:42:15.156  INFO 10252 --- [nio-8000-exec-3] c.example.demo.filter.ZuulLoggingFilter  : request: /first-service/welcome
    ```

#### Spring Cloud Gateway

- Spring Boot 2.4 버전 이상에서 사용할 수 있는 API Gateway
- 비동기 처리(WebFlux)가 가능하다

- 프로젝트 생성
  - spring boot version: 2.4.9
  - dependency: lombok, gateway, eureka client
  - application.yml로 라우팅하기
    ```
    server:
      port: 8000

    eureka:
      client:
        register-with-eureka: false
        fetch-registry: false
        service-url:
        
          default-zone: http://localhost:8761/eureka

    spring:
      application:
        name: apigateway-service
      cloud:
        gateway:
          routes:
          - id: first-service
            uri: http://localhost:8081/
            predicates:
            - Path= /first-service/**
          - id: second-service
            uri: http://localhost:8081/
            predicates:
            - Path= /second-service/**
    ```
    - Path로 지정한 uri로 api gateway의 URI를 구분할 수 있다.
    - 다만 Path가 그대로 service에 전달하게 된다.
    - 즉, localhost:8000/first-service/welcome -> localhost:8081/first-service/welcome
    - 그래서 first-service, second-service의 url을 변경해주어야 한다.
  - java code를 활용하여 filter 추가하여 라우팅하기
    ```java
    @Configuration
    public class FilterConfig {
      @Bean
      public RouteLocator gatewayRoutes(RouteLocatorBuilder builder) {
        return builder.routes()
            .route(r -> r.path("/first-service/**")
                .filters(f -> f.addRequestHeader("first-request", "first-request-header")
                    .addResponseHeader("first-response", "first-response-header"))
                .uri("http://localhost:8081"))
            .route(r -> r.path("/second-service/**")
                .filters(f -> f.addRequestHeader("second-request", "second-request-header"))
                .uri("http://localhost:8082"))
            .build();
      }
    }
    ```
    - application.yml 파일에 작성해두었던 spring.cloud.gateway.routes 밑에 부분은 주석처리한다.
    - filters 메소드를 통해, request, response header를 설정할 수 있다.
    
- Filter
  
  ![image](https://user-images.githubusercontent.com/42403023/124709325-4e4f9b00-df36-11eb-94b3-53fdf60b40dd.png)

  ** 이미지 출처: https://www.baeldung.com/spring-cloud-gateway-webfilter-factories
  
  - property를 통한 filter 추가
    ```
    spring:
      application:
        name: apigateway-service
      cloud:
        gateway:
          routes:
          - id: first-service
            uri: http://localhost:8081/
            predicates:
            - Path= /first-service/**
            filters:
            - AddRequestHeader=first-request, first-request-header2
            - AddResponseHeader=first-response, first-response-header2
          - id: second-service
            uri: http://localhost:8081/
            predicates:
            - Path= /second-service/**
            filters:
            - AddRequestHeader=second-request, second-request-header2
            - AddResponseHeader=second-response, second-response-header2
    ```
  - custom filter
    ```java
    @Component
    @Slf4j
    public class CustomFilter extends AbstractGatewayFilterFactory<CustomFilter.Config>{

      public static class Config {
        // put the configuration properties
      }

      public CustomFilter() {
        super(Config.class);
      }

      @Override
      public GatewayFilter apply(Config config) {
        // TODO Auto-generated method stub

        return (exchange, chain) -> {
          // Custom Pre Filter
          ServerHttpRequest request = exchange.getRequest();
          ServerHttpResponse response = exchange.getResponse();

          log.info("Custom Pre filter: {}", request.getId());

          // chaining한 후 Post Filter
          return chain.filter(exchange).then(Mono.fromRunnable(()-> {
            log.info("Custom Post filter.response status code: {}", response.getStatusCode());
          }));
        };
      }
    }
    ```
    - java code를 활용한 custom filter
    - AbstractGatewayFilterFactory fmf tkdthrqkedk tkdydgksek.
    - ServletHttpRequest, ServletHttpResponse 가 아닌 ServerHttpRequest, ServerHttpResponse를 사용
    
    ```
    spring:
      application:
        name: apigateway-service
      cloud:
        gateway:
          routes:
          - id: first-service
            uri: http://localhost:8081/
            predicates:
            - Path= /first-service/**
            filters:
            #- AddRequestHeader=first-request, first-request-header2
            #- AddResponseHeader=first-response, first-response-header2
            - CustomFilter
          - id: second-service
            uri: http://localhost:8081/
            predicates:
            - Path= /second-service/**
            filters:
            #- AddRequestHeader=second-request, second-request-header2
            #- AddResponseHeader=second-response, second-response-header2
            - CustomFilter
    ```
    - application.yml 에 등록
    ```
    2021-07-08 10:44:59.837  INFO 7088 --- [ctor-http-nio-2] com.example.demo.CustomFilter            : Custom Pre filter: 288bb018-1
    2021-07-08 10:45:00.283  INFO 7088 --- [ctor-http-nio-4] com.example.demo.CustomFilter            : Custom Post filter.response status code: 200 OK
    ```
    - 요청 시 확인

  - Global Filter
    - Global Filter도 만드는 방법은 이전과 비슷하다.
    - Custom Filter는 Custom Filter에 경우 application.yml에 route 정보에 따른 filter를 등록했다.
    - Global Filter는 모든 route 정보에 적용된다.
    
    ```java
    @Component
    @Slf4j
    public class GlobalFilter extends AbstractGatewayFilterFactory<GlobalFilter.Config>{

      @Data
      public static class Config {
        private String baseMessage;
        private boolean pre;
        private boolean post;

      }

      public GlobalFilter() {
        super(Config.class);
      }

      @Override
      public GatewayFilter apply(Config config) {
        // TODO Auto-generated method stub
        return (exchange, chain) -> {
          ServerHttpRequest request = exchange.getRequest();
          ServerHttpResponse response = exchange.getResponse();

          if(config.isPre()) { // Config에 있는 getter 사용
            log.info("Global Pre Filter -> Request ID: {}", request.getId());
            log.info(config.getBaseMessage()); 
          }

          return chain.filter(exchange).then(Mono.fromRunnable(()->{
            if(config.isPost()) {
              log.info("Global Post Filter -> Response Status Code: {}", response.getStatusCode());
            }
          }));
        };
      }
    }
    ```
    - java code 작성
    - CustomFilter와 작성 방식은 같다.
    - CustomFilter 또한 GlobalFilter.Config 에 데이터를 놓고 getter, setter로 접근 가능하며 application.yml에 있는 값을 가져온다.
    - AbstractGatewayFilterFactory를 상속받는다.
        ```java
        GatewayFilter filter = new OrderedGatewayFilter((exchange, chain)-> {
          // pre 정의
          return chain.filter(exchange).then(Mono.fromRunnable(()-> {
            // post 정의
          }));
        }, Ordered.HIGHEST_PRECEDENCE);

        return filter;
        ```
      - public GatewayFilter apply 를 구현해주어야 한다.
      - 리턴 객체는 GatewayFilter는 OrderedGatewayFilter 구현체를 사용하며 filter 메소드를 재정의해야 한다.
      - OrderedGatewayFilter 구현을 해주었기 때문에 Filter 순위를 적용할 수 있다.
    
    ```
    spring:
      application:
        name: apigateway-service
      cloud:
        gateway:
          default-filters:
          - name: GlobalFilter
            args:
              baseMessage: Spring Cloud Gateway Log Filter 
              pre: True
              post: True
          routes:
          - id: first-service
            uri: http://localhost:8081/
            predicates:
            - Path= /first-service/**
            filters:
            - CustomFilter
          - id: second-service
            uri: http://localhost:8081/
            predicates:
            - Path= /second-service/**
            filters:
            - CustomFilter
    ```
    - application.yml에 default_filtes로 GlobalFilter를 등록할 수 있다.
    - args로 Config 값을 설정하였다.
    ```
    2021-07-08 11:09:41.275  INFO 21260 --- [ctor-http-nio-3] com.example.demo.GlobalFilter            : Global Pre Filter -> Request ID: 54a9517e-1
    2021-07-08 11:09:41.276  INFO 21260 --- [ctor-http-nio-3] com.example.demo.CustomFilter            : Custom Pre filter: 54a9517e-1
    2021-07-08 11:09:41.695  INFO 21260 --- [ctor-http-nio-1] com.example.demo.CustomFilter            : Custom Post filter.response status code: 200 OK
    2021-07-08 11:09:41.695  INFO 21260 --- [ctor-http-nio-1] com.example.demo.GlobalFilter            : Global Post Filter -> Response Status Code: 200 OK
    ```
    - 로그를 확인할 수 있다.

- Spring Cloud Gateway 와 Eureka 연동
  - client가 localhost:8000/first-service/welcome을 요청
  - Spring Cloud Gateway
    - API Gateway는 Eureka에 요청정보를 전달하여 Service의 위치를 전달 받는다.
  - Eureka Server: Service Discovery, Registration
    - Eureka Server는 API Gateway에 요청된 정보를 토대로 First Service 또는 Second Service의 위치 정보를 전달한다.
  - First Service, Second Service
    - API Gateway에게 응답을 전달한다.
    - localhost:8081/first-service/welcome
    - localhost:8082/second-service/welcome
  - API Gateway는 Service에게 전달받은 응답을 Client에게 전달한다.

  - 개발 순서
    - Step1) Eureka Client 추가 - pom.xml
      ```
      <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
      </dependency>
      ```
    - Step2) Eureka Client 설정 -  application.yml
      - Spring Cloud Gateway
        - 기존 사용했던 URI 정보를 lb://eureka-등록-service-name 으로 등록이 가능하다.
        ```
        eureka:
          client:
            register-with-eureka: true
            fetch-registry: true
            service-url:
              default-zone: http://localhost:8761/eureka
        spring:
          application:
            name: apigateway-service
          cloud:
            gateway:
              default-filters:
              - name: GlobalFilter
                args:
                  baseMessage: Spring Cloud Gateway Log Filter 
                  pre: true
                  post: true
              routes:
              - id: first-service
                uri: lb://FIRST-SERVICE
                predicates:
                - Path= /first-service/**
                filters:
                # - AddRequestHeader=first-request, first-request-header2
                # - AddResponseHeader=first-response, first-response-header2
                - CustomFilter
              - id: second-service
                uri: lb://SECOND-SERVICE
                predicates:
                - Path= /second-service/**
                filters:
                #- AddRequestHeader=second-request, second-request-header2
                #- AddResponseHeader=second-response, second-response-header2
                - CustomFilter
        ```
        
      - First Service, Second Service에 등록
        ```
        eureka:
        client:
          register-with-eureka: true
          fetchRegistry: true
          service-url:
            defaultZone: http://localhost:871/eureka
        ```
        
    - Step3) Eureka Server에 등록 확인
      - localhost:8761

      ![image](https://user-images.githubusercontent.com/42403023/125160281-068b7680-e1b7-11eb-962a-429d68e0775b.png)

  
    
#### 출처

- Sprint Cloud로 개발하는 마이크로서비스 애플리케이션, 인프런 강의
