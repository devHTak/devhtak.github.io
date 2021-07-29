---
layout: post
title: 마스터 스프링 클라우드 - 06. 마이크로서비스 간의 커뮤니케이션
summary: Spring Cloud
author: devhtak
date: '2021-05-27 22:41:00 +0900'
category: msa
---

#### 들어가기

- 서비스 디스커버리를 사용하거나 사용하지 않고 서비스 간 커뮤니케이션에 스프링 RestTemplate 사용하기
- 리본 클라이언트 사용자 정의하기
- 리본 클라이언트에서 제공하는 주요 기능에 대한 설명
  - 예를 들면, 리본 클라이언트, 서비스 디스커버리, 상속, zone 지원과 통합

### 다양한 커뮤니케이션 스타일

- 동기식/비동기식 커뮤니케이션 프로토콜
  - 비동기식 커뮤니케이션의 핵심은 응답을 기다리는 동안 클라이언트의 스레드가 멈추지 않아도 된다.
    - AMQP 프로토콜을 많이 사용한다.
  - 하지만 여전히 동기방식의 HTTP 프로토콜을 많이 사용한다.

- 단일 메시지 수신기 또는 다수의 수신기 여부에 따라 다양한 커뮤니케이션 타입을 나눌 수 있다.
  - 일대일 커뮤니케이션에서는 각 요청이 정확히 하나의 서비스 인스턴스에 의해 처리
  - 일대다 커뮤니케이션에서는 각 요청이 다수의 다른 서비스에 의해 처리될 수 있다.

### 스프링 클라우드를 사용한 동기식 통신

- 스프링 클라우드는 마이크로서비스 간의 커뮤니케이션 구현을 도와주는 구성 요소 집합 제공

- RestTemplate
  - 클라이언트가 RESTful 웹 서비스를 사용할 때 항상 사용된다.
  - @LoadBalanced 한정자를 사용한다

- Netflix Ribbon
  - IP 주소 대신 서비스 이름을 사용해 서비스 디스커버리를 활용할 수 있게 된다.
  - Ribbon은 클라이언트 측 부하 분산기로서 HTTP, TCP 클라이언트의 행동을 제어하는 간단한 인터페이스 제공
  - 개발자에게 완전 투명(Transparent) 하다.
    - 추상화 계층이 하부의 복잡한 것을 감추는 덕분에 추상화 계층의 스펙에 따른 사용자의 설정에 따라 시스템이 일관되게 동작한다.

- Feign
  - Netflix OSS 스택에서 온 선언적인 REST Client이다.
  - 페인은 부하 분산 및 서비스 디스커버리에서 데이터를 가져오기 위해 리본을 사용한다.
  - @FeignClient를 사용하여 인터페이스에 선언할 수 있다.

### 리본을 사용한 부하 분산

- 리본의 주요 개념은 이름 기반 서비스 호출 클라이언트(named client)라고 말할 수 있다.
- 서비스 디스커버리에 접속할 필요 없이 호스트 이름과 포트를 사용한 전체 주소 대신에 이름을 사용해 다른 서비스 호출이 가능하다.
- 주소 목록이 application.yml 파일의 리본 컨피규레이션 설정에 제공 되어야 한다.

#### (리본)리본 클라이언트를 사용해 마이크로 서비스 간 커뮤니케이션하기

- 예제로 재품 구매하는 간단한 주문 시스템 개발
- 해당 예제는 서비스 디스커버리 사용없이 진행한다.
- 주문 서비스 -> 제품 서비스, 고객서비스, 계정 서비스
- 고객 서비스 -> 계정 서비스

#### (리본)정적 부하 분산 컨피규레이션

- ribbon과 web 의존성 추가
  ```
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>

  <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-ribbon -->
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-ribbon</artifactId>
      <version>1.4.2.RELEASE</version>
  </dependency>
  ```

- OrderService의 application.yml
  - OrderService는 모든 서비스와 커뮤니케이션을 해야 한다.
  - 그래서 ribbon.listOfServers 속성을 사용해 세개의 다른 리본 클라이언트에 네트워크 주소를 설정해야 한다.
  ```
  server:
    port: 8090

  AccountService:
    ribbon:
      eureka:
        enabled: false
      listOfServers: localhost:8091
  CustomerService:
    ribbon:
      eureka:
        enabled: false
      listOfServers: localhost:8092
  ProductService:
    ribbon:
      eureka:
        enabled: false
      listOfServers: localhost:8093
  ```

- OrderApplication.java
  - 메인클래스나 다른 스프링 컨피규레이션 클래스에 @RibbonClients를 사용한다.
  - RestTemplate 빈을 등록하고 @LoadBalanced를 사용해 스프링 클라우드 구성요소와 상호작용이 가능하도록 해야 한다.
  ```java
  @SpringBootApplication
  @RibbonClients({
    @RibbonClient(name = "account-service"),
    @RibbonClient(name = "customer-service"),
    @RibbonClient(name = "product-service")
  })
  public class OrderApplication {

    public static void main(String[] args) {
      SpringApplication.run(OrderApplication.class, args);
    }

    @LoadBalanced
    @Bean
    RestTemplate restTemplate() {
      return new RestTemplate();
    }

    @Bean
    OrderRepository repository() {
      return new OrderRepository();
    }

  }
  ``

#### (리본)다른 서비스 호출하기
- 컨트롤러는 다른 HTTP 종단점을 호출할 수 있도록 RestTemplate을 사용한다.
- IP address, Port 는 application.yml을 사용하며, 등록한 이름으로 호출할 수 있다.
- OrderController
  ```java
  @RestController
  public class OrderController {

    @Autowired
    OrderRepository repository;

    @Autowired
    RestTemplate template;

    @PostMapping("/orders")
    public Order prepare(@RequestBody Order order) {
      int price = 0;
      Product[] products = template.postForObject("http://ProductService/ids", order.getProductIds(), Product[].class);
      Customer customer = template.getForObject("http://CustomerService//customers/withAccounts/{id}", Customer.class, order.getCustomerId());
      for (Product product : products) 
        price += product.getPrice();
      final int priceDiscounted = priceDiscount(price, customer);
      Optional<Account> account = customer.getAccounts().stream().filter(a -> (a.getBalance() > priceDiscounted)).findFirst();
      if (account.isPresent()) {
        order.setAccountId(account.get().getId());
        order.setStatus(OrderStatus.ACCEPTED);
        order.setPrice(priceDiscounted);
      } else {
        order.setStatus(OrderStatus.REJECTED);
      }
      return repository.add(order);
    }

    @PutMapping("/orders/{id}")
    public Order accept(@PathVariable Long id) {
      final Order order = repository.findById(id);
      template.put("http://AccountService/accounts/{id}/withdraw/{amount}", null, order.getAccountId(), order.getPrice());
      order.setStatus(OrderStatus.DONE);
      repository.update(order);
      return order;
    }

    private int priceDiscount(int price, Customer customer) {
      double discount = 0;
      switch (customer.getType()) {
      case REGULAR:
        discount += 0.05;
        break;
      case VIP:
        discount += 0.1;
        break;

      default:
        break;
      }
      int ordersNum = repository.countByCustomerId(customer.getId());
      discount += (ordersNum*0.01);
      return (int) (price - (price * discount));
    }

  }
  ```
  
- mvn cliean install로 빌드
- java -jar로 모든 마이크로 서비스를 실행한다.
- 호출하여 테스트할 수 있다.

### 서비스 디스커버리와 함께 RestTemplate 개발하기

- 서비스 디스커버리와의 통합은 리본 클라이언트의 기본 행동이다.
- 서비스 디스커버리의 존재는 예제에서 서비스 간의 커뮤니케이션 중에 스프링 클라우드 구성 요소의 컨피규레이션을 간단하게 만든다.

#### 예제 애플리케이션 개발

- 새로운 discover-service 모듈
- 리본 클라이언트와 관련된 모든 컨피규레이션과 애노테이션을 제거한 후, 제거된 리본 클라이언트는 @EnableDiscoveryClient를 사용해 유레카 디스커버리 클라이언트를 사용하는 것으로 대체하고 유레카 서버 주소를 application.yml에 제공
  - OrderService.java
    ```
    @SpringBootApplication
    @EnableDiscoveryClient
    public class OrderApplication {
      @LoadBalanced
      @Bean
      RestTemplate restTemplate() {
        return new RestTemplate();
      }
      
      publi static void main(String[] args) {
        SpringApplication.run(DiscoveryServiceApplication.class, args);
      }
    }
    ```
  - application.yml
    ```
    spring:
      application:
        name: OrderService
    server:
      port: ${PORT:8090}
    eureka:
      client:
        serviceUrl:
          defaultZone: ${EUREKA_URL:http://localhost:8761/eureka/}      
    ```
  - 다른 서비스를 실행하면 Eureka 서버에서 확인이 가능하다.

- 기본으로 리본 클라이언트는 등록된 모든 마이크로 서비스의 인스턴스 간에 균등하게 트래픽을 분배한다.
  - 해당 알고리즘은 라운드 로빈(round robbin) 이라 한다.
  - 클라이언트가 마지막 요청을 보낸 곳을 기억하고 이번 요청을 다음 순서의 서비스로 전달하는 것

- 서비스 디스커버리가 아닌 다른 방식으로 재정의할 수 있다.
  - 서비스 디스커버리 없이 ribbon.listOfServers에 ,로 분리된 서비스 주소 목록을 설정해 구헝할 수 있다.

### Feign 클라이언트 사용하기

- RestTemplate 은 스프링 클라우드와 마이크로 서비스의 상호작용을 위해 도입된 스프링 구성 요소
- Netflix에서 REST 커뮤니케이션을 위해 독립적으로 개발한 웹 서비스 클라이언트가 Feign
  - @LoadBalanced을 사용하는 RestTemplate으로 동일하지만, 애노테이션을 템플릿화된 요청으로 처리해 동작하는 HTTP 클라이언트 바인더이다.
  - 페인 클라이언트는 서비스 디스커버리에서 모든 네트워크 주소를 가져오는 부하 분산 HTTP 클라이언트를 제공하기 위해 리본 및 유레카와 통합한다.

#### 여러 존의 지원

- 여러 존을 구성하기 위해 서비스 디스커버리에서 유레카 사용
- zone 내부에서 여러 서비스가 커뮤니케이션 하는 구조

#### 애플리케이션에서 페인 사용하기

- 프로젝트에 페인을 포함하기 위해서는 spring-cloud-starter-feign 아티팩트 또는 스프링 클라우드 넷플릭스를 위한 spring-cloud-starter-openfeign 의존성을 추가 해야 한다.
  ```
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-feign</artifactId>
  </dependency>
  ```

- Main에 @EnableFeignClients 애노테이션을 추가해 페인 사용
  ```java
  @SpringBootApplicatioin
  @EnableDiscoveryClient
  @EnableFeignClients
  public class OrderApplication {
    public static void main(Strings[] args) {
      // ...
    }
  }
  ```
- @FeignClient Interface 생성
  ```java
  @FeignClient(name="order-service")
  public interface OrderServiceClient {
    @GetMapping("/order-service/{userId}/orders")
    List<ResponseOrder> getOrders(@PathVariable String userId);
  }
  ```
  - @FeignClient에 등록된 name은 Eureka에 등록된 서비스 네임을 사용

- 서비스에서 @FeignClient Interface를 가져와서 사용

- FeignClient 에서 로그 사용
  - application.yml 에 로그 레벨 설정
    ```
    logging:
      level:
        com.example.userservice.client: DEBUG
    ```
  - Feign Logger 빈 등록
    ```java
    @Bean
    public Logger.Level feignLoggerLevel() {
      return Logger.Level.FULL;
    }
    ```
    
- Feign Client 예외 처리
  - FeignException 처리
    - 해당 방식으로 try-catch를 사용하면 orders에 대한 정보를 보여주지 않는다.
      ```
      try {
        ordersList = orderServiceClient.getOrders(userId);
      } catch(FeignException e) {
        log.error(e.getMessage());
      }
      ```

#### 출처

- Mastering Spring Cloud 책 발췌
