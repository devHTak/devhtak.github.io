---
layout: post
title: Spring Cloud - Circuit Breaker
summary: Spring Cloud
author: devhtak
date: '2021-08-06 22:41:00 +0900'
category: msa
---

#### 마이크로서비스 통신 시 연쇄 오류

- 마이크로 서비스 환경에서는 여러 서비스 호출이 이뤄진다.
  - 만약 다른 서비스에서 생겨난 오류가 전파되어 요청한 서비스에서도 500 error와 같이 오류가 발생한 것으로 볼 수 있다.
  - 비록 다른 서비스에 대한 정보를 보여주지는 못하지만 200 OK를 보여주어야 한다.
  
- Circuit Breaker
  
  ![image](https://user-images.githubusercontent.com/42403023/128479637-01a12079-f983-4b1a-897a-60afddf65c58.png)

  * 이미지 출처: https://martinfowler.com/bliki/CircuitBreaker.html
  
  - 장애가 발생하는 서비스에 반복적인 호출이 되지 못하게 차단
    - circuit breaker close: 정상 호출
    - circuit breaker open: 호출받는 서비스에 장애로 정상적인 데이터를 받지 못하는 경우
  - 특정 서비스가 정상적으로 동작하지 않을 경우 다른 기능으로 대체 수행 -> 장애 회피

#### Spring Cloud Netflix Hystrix

- 사용
  - dependency
    ```
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
    </dependency>
    ```
    
  - application.java
    ```java
    @EnableCircuitBreaker
    public class Application {}
    ```
    
  - application.yml
    ```
    feign:
      hystrix:
        enabled: true
    ```

- Hystrix가 현재 지원을 중단하여 Resilience4j를 사용한다
  
  |---|---|
  |Current|Replacement|
  |Hystrix|Reslience4j|
  |Hystrix Dashboard/Turbine|Micronmeter + Motnitoring System|
  

#### Resilience4j

- 아래와 같은 서비스를 제공한다.
  - resilience4k-circuitbreaker: Circuit breaking
  - resilience4k-ratelimiter: Rate Limiting
  - resilience4k-bulkhead: Bulkheading
  - resilience4k-retry: Automatic retrying(sync and async)
  - resilience4k-timelimiter: Timeout handling
  - resilience4k-cache: Result caching

- Default Configuration
  - dependency
    ```
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
    </dependency>
    ```

  - CircuitBreakerFactory를 빈으로 받아 circuitBreaker 생성
    ```java
    @Autowired
    CircuitBreakerFactory circuitBreakerFactory;
    
    // ...
    CircuitBreaker circuitBreaker = circuitBreakerFactory.create("circuitbreaker");
    List<ResponseOrder> ordersList = circuitBreaker.run( () -> orderServiceClient.getOrders(userId), throwable -> new ArrayList<>());
    // ...
    ```
    - run() 메서드에서 첫번째 인자는 호출한 내용이고, 두번째 호출은 오류가 발생할 때 생성되는 것
    - 해당 예제는 빈 orderList를 리턴하도록 했다.
  
  - Customize CircuitBreakerFactory
    ```java
    @Configuration
    public class Reslience4JConfiguration {
      @Bean
      public CircuitBreaker<Reslience4JCircuitBreakerFactory> globalCustomConfiguration() {
        CircuitBreakerConfig circuitBreakerConfig = CircuitBreakerConfig.custom()
          .failureRateThreshold(4)
          .waitDurationInOpenState(Duration.ofMillis(1000))
          .slidingWindowType(CircuitBreakerConfig.SlidingWindowType.COUNT_BASED)
          .slidingWindowSize(2).build();
          
        TimeLimiterConfig timeLimiterConfig = TimeLimiterConfig.custom()
          .timeoutDuration(Duration.ofSeconds(4)).build();
          
        return factory -> factory.configureDefault(id -> new Resilience4JConfigBuilder(id)
          .timeLimiterConfig(timeLimiterConfig)
          .circuitBreakerConfig(circuitBreakerConfig).build());
      }
    }
    ```
    - failureRateThreadhold
      - CircuitBreaker를 열지 결정하는 failure rate threshold percentage(default: 50%)
    - waitDurationInOpenState
      - CircuitBreaker를 open한 상태를 유지하는 지속 기간 의미
      - 이 기간 이후에 half-open 상태
      - default 60seconds
    - slidingWindowType
      - CircuitBreaker가 닫힐 때 통화 결과를 기록하는 데 사용하는 슬라이딩 창의 유형 구성
      - 카운트 기반 또는 시간 기반
    - slidingWindowSize
      - CircuitBreaker가 닫힐 때 호출 결과를 기록하는 데 사용되는 슬라이딩 창의 크기 구성
      - default: 100
    - timeoutDuration
      - TimeLimiter는 feature supplier의 time limit을 정하는 API
      - default 1

#### 출처

- Spring cloud로 개발하는 마이크로서비스 애플리케이션(MSA)



