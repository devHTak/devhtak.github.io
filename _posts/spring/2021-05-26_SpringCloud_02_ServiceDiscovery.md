---
layout: post
title: 마스터 스프링 클라우드 - 04. Service Discovery
summary: Spring Cloud
author: devhtak
date: '2021-05-26 14:41:00 +0900'
category: Spring Cloud
---

#### 들어가기

- Eureca
  - 서비스 디스커버리로 넷플릭스의 Eureka를 많이 사용한다.
  - Eureka는 서버와 클라이언트로 라이브러리가 나눠져 있다.
  - 서버
    - 서버 API는 등록된 서비스의 목록을 수집하기 위한 API와 새로운 서비스를 네트워크 위치 주소와 함께 등록하기 위한 API로 구성
    - 서버는 각 서버의 상태를 다른 서버로 복제해 설정하고 배포함으로써 가용성을 높일 수 있다.
  - 클라이언트
    - 클라이언트는 마이크로서비스 애플리케이션에 의존성을 포함해 사용
    - 클라이언트는 애플리케이션 시작 후 등록과 종료 전 등록 해제를 담당하고 유레카 서버로부터 주기적으로 최신 서비스 목록을 받아온다.

- 유레카를 사용한 서비스 디스커버리
  - 클라이언트와 서버로 구분
  - 서비스 디스커버리 패턴이란?
    - MSA와 같은 분산 환경은 서비스 간의 원격 호출로 구성이 된다.
    - 클라우드 환경이 되면서 서비스가 동적으로 생성되거나, 컨테이너 기반의 배포로 인해 서비스의 IP가 동적으로 변경되는 일이 발생한다.
    - 서비스 디스커버리란 클라이언트가 서비스를 호출할 때 서비스의 위치(IP, Port)를 알아낼 수 있는 기능 제공
    - Service Registry에 제공할 Service에 IP를 저장하여 사용
    
      ![image](https://user-images.githubusercontent.com/42403023/119246284-5a1c1380-bbbb-11eb-8862-22fa9b819a2f.png)
      
      ** 이미지 출처: https://bcho.tistory.com/1252

  - 클라이언트
    - spring-cloud-starter-eureka를 추가하여 사용
    - 클라이언트는 항상 애플리케이션의 일부로 원격의 디스커버리 서버에 연결하는 일을 담당한다.
    - 연결 후에 서비스 이름과 네트워크 위치 정보를 등록 메시지로 전송
    - 다른 마이크로서비스 API를 호출해야 할 경우 디스커버리 서버로부터 서비스 목록을 담은 최신 컨피규레이션을 수신한다.

  - 서버
    - spring-cloud-eureka-server를 추가하여 사용
    - 독립적인 스프링 부트 애플리케이션
    - 각 서버의 상태를 다른 서버에 복제해 가용성이 높다.

- 배울 내용
  - 유레카 서버를 내장한 애플리케이션 배포하기
  - 클라이언트 측 애플리케이션에서 유레카 서버 연결하기
  - 고급 디스커버리 클라이언트 설정
  - 클라이언트와 서버 사이의 보안 통신
  - 가용성을 높이기 위한 설정 및 동료 간 복제 메커니즘
  - 다른 가용 존에 클라이언트 측 애플리케이션의 인스턴스 등록

#### 서버 측에서 유레카 서버 실행하기

- 의존성 추가
  ```
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
  </dependency>
  ```

- Main Application
  ```java
  @SpringBootApplication
  @EnableEurekaServer
  public class EurekaServerApplication {
    public static void main(String[] args) {
      SpringApplication.run(EurekaServerApplication.class, args);
    }
  }
  ```

- application.yaml 설정
  - 서버 스타터에 클라이언트의 의존성도 포함되어 있다.
  - 디스커버리 인스턴스를 고가용성 모드로 동작할 경우 디스커버리 인스턴스 사이의 peer-to-peer 통신에만 유용하다.
  - 단일 인스턴스로 실행할 경우, 시작시에 에러 로그만 찍힐 뿐 유용하지 않다.
  - 그래서 spring-cloud-netflix-eureka-client 의존성을 제외하거나 컨피규레이션 속성의 디스커버리 클라이언트를 비활성화 한다.
    ```
    server:
      port: ${PORT:8761}
    eureka:
      client:
        registerWithEureka: false
        fetchRegistry: false 
    ```

- 서버 실행, 로그 확인
  ```
  com.example.EurekaServerApplication: Started EurekaServerApplication in 13.213 seconds (JVM running for 14.375)
  ```
  - 서버가 실행되면 간단한 UI 대시보드가 http://localhost:8761에서 서비스 된다.
  - /eureka/* 경로로 HTTP API 메서드를 호출할 수 있다.

#### 클라이언트 측에서 유레카 활성화하기

- 의존성 추가
  ```
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
  </dependency>
  ```

- @EnableDiscoveryClient 애노테이션
  - 해당 예제는 서버와 통신하는 것이 전부다.
    - 자신을 등록하고 호스트, 포트, 상태 정보, URL, 홈페이지 URL을 보낸다.
    - 유레카 서버는 서비스의 각 인스턴스로부터 생존신호 메시지를 받는다.
  - @EnableDiscoverClient 활성화
    - 서버로부터 데이터를 가져와 캐싱하고 주기적으로 변경사항을 점검
    - spring-cloud-commons에 존재하여 컨설, 유레카, 주키퍼 등 다수의 클라이언트 구현체에서 사용
    - @EnableEurecaClient는 spring-cloud-netflix만 존재한다.
  - 소스
    ```java
    @SpringBootApplication
    @EnableDiscoveryClient
    public class EurekaClient1Application {
      public static void main(String[] args) {
        SpringApplication.run(EurekaClient1Application.class, args);
      }
    }
    ```
- application.yaml 설정
  ```
  spring:
    application:
      name: client-service
  server:
    port: ${PORT:8081}
  eureka:
    client:
      serviceUrl:
        defaultZone: ${EUREKA_URL:http://localhost:8761/eureka/}
  ```

- localhost:8761/eureka/apps 에서 등록 확인

#### 종료 시 등록 해제

- 중단된 이벤트를 가로채거나 이벤트를 서버에 보내기 위해 우아하게 멈춰야 한다.
- 가장 좋은 방법은 spring actuator(spring-boot-starter-actuator)를 사용하는 것
  - pom.xml에 actuator를 추가하자
    ```
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    ```
  - actuator 세팅 및 user/password 보안을 해제한다.
    ```
    management:
      endpoint:
        shutdown:
          enabled: true # shutdown endpoint 활성화
      endpoints:
        web:
          exposure:
            include: shutdown,info
    ```
  - /shutdown API 메서드를 POST로 호출한다.
    ```
    request: Post http:localhost:8081/actuator/shutdown
    response: {"message": "Shutting down, bye....}   
    ```

- 우아한 종료가 최선의 방법이지만 언제나 그렇듯이 항상 우아한 결과를 얻는 것은 아니다.
  - 예를 들면, 서버 머신이 재시작하거나 애플리케이션 장애, 서버와 클라이언트 간의 네트워크 인터페이스 문제 등에 문제를 겪을 수 있다.
  - 제대로 종료되지 않는 이유?
    - 서버에서 클라이언트에게 주기적으로 heartbeat을 보내고, 클라이언트에서 받지 못하는 경우 제거한다.
    - 하지만 네트워크 장애 등의 문제로 등록된 모든 서비스가 해제되는 것을 방지하기 위해 등록만료를 중단한다.
    - 즉, 정해진 시간동안 제거하지 않고, 이를 Self-preservation mode(자기 보호 모드)라고 한다.
    - 이를 해결하기 위해서는 아래와 같은 설정을 추가하면 된다.
      ```
      eureka:
        server:
          enableSelfPreservation: false
      ```

#### 프로그램 방식으로 디스커버리 클라이언트 사용하기

- 클라이언트 애플리케이션이 시작된 후 유레카 서버로부터 등록된 서비스 목록을 가져온다.
- API를 사용하여 가져오는 두가지 방식이 있다.
  - com.netflix.discovery.EurekaClient
    - 유레카 서버가 노출하는 모든 HTTP API를 구현한다.
    - 유레카 API 영역에 설명돼 있다.
  - org.springframework.cloud.client.discovery.DiscoveryClient
    - 넷플릭스를 대체하는 Spring Cloud의 구현이다.
    - 이것은 모든 Discovery Client 용으로 사용하는 간단한 범용 API이다.
    - getServices와 getInstances의 두가지 메서드가 있다.

- 예제
  ```java
  @RestController
  public class ClientController {

    private static final Logger LOGGER = LoggerFactory.getLogger(ClientController.class);

    @Autowired
    private DiscoveryClient discoveryClient;

    @GetMapping("/ping")
    public List<ServiceInstance> getServiceInstances() {
      List<ServiceInstance> instances = discoveryClient.getInstances("CLIENT_SERVICE");
      LOGGER.info("INSTANCES: count={}", instances.size());

      instances.stream().forEach(instance -> {
        LOGGER.info("INSTANCE: id={}, port={}", instance.getServiceId(), instance.getPort());
      });
      return instances;
    }

  }
  ```
  
  ```
  request: GET /ping
  response: []
  ```
  - 아직 등록된 인스턴스가 없다.

#### 고급 컨피규레이션 설정

- 크게 3가지로 나뉜다.
  - 서버
    - 서버의 행동을 정의한다.
    - eureka.server.* 를 접두어로 사용하는 모든 속성
    - EurekaServerConfigBean 클래스를 참조한다.
    - 참고 URL: https://github.com/spring-cloud/spring-cloud-netflix/blob/main/spring-cloud-netflix-eureka-server/src/main/java/org/springframework/cloud/netflix/eureka/server/EurekaServerConfigBean.java 
  - 클라이언트
    - 유레카 클라이언트에서 사용할 수 있는 두가지 속성 중 하나
    - 클라이언트가 레지스트리에서 다른 서비스의 정보를 얻기 위해 질의하는 방법의 컨피규레이션을 담당
    - eureka.client* 를 접두어로 사용하는 모든 속상
    - 전체 속성 목록은 EurekaClientConfigBean 클래스를 참조한다.
    - 참고 URL: https://github.com/spring-cloud/spring-cloud-netflix/blob/main/spring-cloud-netflix-eureka-client/src/main/java/org/springframework/cloud/netflix/eureka/EurekaClientConfigBean.java
  - 인스턴스
    - 포트나 이름 등의 현재 유레카 클라이언트의 행동을 재정의한다.
    - eureka.instance.* 를 접두어로 사용하는 모든 속성을 포함한다.
    - 전체 속성 목록은 EurekaInstanceConfigBean 클래스를 참조한다.
    - 참고 URL: https://github.com/spring-cloud/spring-cloud-netflix/blob/main/spring-cloud-netflix-eureka-client/src/main/java/org/springframework/cloud/netflix/eureka/EurekaInstanceConfigBean.java

#### 레지스트리 갱신하기

- self-preservation mode를 비활성화 하여도 여전히 서버가 임대를 취소하는 것은 오래 걸린다.
  - 첫번째 이유. 모든 클라이언트 서비스가 30초(default)마다 서버로 하트비트를 보낸다.
    - eureka.instance.leaseRenewalIntervalInSeconds 속성
    - 서버가 하트비트를 받지 못하면 레지스트리에서 인스턴스를 제거하기 전에 90초를 기다린다.
  - 두번째 이유. 등록을 해제해서 인스턴스로 더 이상 트래픽이 가지 못하도록 차단할 수 있기 때문이다.
    - eureka.instance.leaseExpirationDurationInSeconds 속성
  - 해당 설정은 client에서 설정하며, 초단위이다.
  - 클라이언트 Configuration
    ```
    eureka:
      instance:
        lease-renewal-interval-in-seconds: 1
        lease-expiration-duration-in-seconds: 2
    ```
  - 서버 Configuration
    - 서버 측에서도 변경을 해줘야 한다. 
    - 이유는 Evict(퇴거) 이라는 백그라운드 태스크 때문이다.
      - 이것이 하는 일은 클라이언트로부터 하트비트가 계속 수신 되는지 점검하는 일이다.
      - 기본값으러 60초마다 실행되기 때문에 클라이언트에서 설정했던 위에 두 값을 작은 값으로 설정해도 서비스 인스턴스를 제거하는 데 최악의 경우 60초가 걸린다.
      - eureka.server.evictionIntervalTimerInMs 속성으로 설정 가능하며 millisecond 단위다.
      
        ```
        eureka:
          server:
            enable-self-preservation: false
            eviction-interval-timer-in-ms: 3000
	```
	
- 이런 속성을 조작하여 임대 만료 제거 절차에 대한 유지 관리를 사용자가 정의할 수 있다.
- 그러나 정의된 컨피규레이션이 시스템의 성능을 부족하게 만들지 않는 것도 중요하다.
- 부하 분산, 게이트웨이, 서킷 브레이커 등이 해당 컨피규레이션 변경에 민감한 요소가 된다.

#### 인스턴스 식별자 변경하기

- Eureka에 등록된 인스턴스는 이름으로 묶여있다. 그러나 서버가 인식할 수 있는 유일한 ID를 보내야 한다.
  ```
  ${spring.cloud.client.hostname}:${spring.application.name}:${spring.application.instance_id:${server.port}}
  ```
- 예제
  - VM 인자로 -DSEQUENCE_NO=[n]을 입력받아 Port와 InstanceId를 동적으로 설정하도록 한다.
    ```
    server:
      port: 808${SEQUENCE_NO}
    eureka:
      instance:
        instanceId: ${spring.application.name}-${SEQUENCE_NO}
    ```
    
#### IP 주소 우선하기

- 기본적으로 모든 인스턴스는 호스트명으로 등록된다.
  - DNS가 있으면 매우 편리하지만, 마이크로 서비스 환경을 구성할 때 DNS가 없는 것이 일방적이다.
  - 이 경우 모든 머신의 /etc/hosts 파일에 호스트명과 IP 주소를 추가하는 것 외에 방법이 없다.

- 유레카 클라이언트에 eureka.isntance.preferIpAddress: true로 설정하자.
  - 레지스트리의 모든 서비스 인스턴스는 유레카 대시보드에 호스트명을 담은 instanceId를 사용한다.
  - 하지만 링크를 클릭하면 IP 주소 기반으로 리다이렉션된다.

- 하지만 ip address를 우선하여도 문제가 발생한다.
  - 바로 한대의 서버에 하나 이상의 네트워크 인터페이스가 있는 경우.
  - 방법 1. application.yml 에 무시할 패턴의 목록을 정의 하면 된다.
    ```
    spring:
      cloud:
        inetutils:
          ignored-interfaces: 
          - eth1*
    ```
  - 방법 2. application.yml 에 원하는 네트워크 주소를 정의하는 방법도 있다.
    ```
    spring:
      cloud:
        inetutils:
          preferred-networks:
          - 192.168
    ```
    
#### 응답 캐시

- 유레카 서버
  - 기본적으로 응답을 캐시하며 30초마다 캐시 데이터를 지운다.
    - /eureka/apps API를 통해 확인할 수 있다.
    - 클라이언트를 등록한 다음에는 바로 조회되지 않는다.
    - 30초 후에야 조회된다.
  - 캐시의 타임아웃은 reponseCacheUpdateIntervalMs 속성으로 재정의 가능하다.
    ```
    eureka:
      server:
        response-cache-update-interval-ms: 3000
    ```
  - 유레카 대시보드에 등록된 인스턴스가 표시될 때에는 캐시가 없다. REST API와 반대로 응답 캐시를 사용하지 않는다.

- 유레카 클라이언트
  - 클라이언트 측에서도 유레카 레지스트리를 캐싱을 한다.
  - 서버에서 변경해도 클라이언트에서 갱신되는 데에는 시간이 걸린다.
    - 서버의 레지스트리는 30초마다 실행되는 백그라운드 태스크에 의해 비동기로 갱신이 되기 때문이다.
    - registryFetchIntervalSeconds 속성으로 변경 가능하다.
    - shouldDisableDelta 속성을 통해 마지막으로 시도한 값에서 변경된 내용만 가져오도록 할 수 있다.
    - shouldDisableDelta을 false로 지정할 수 있지만 이것은 대역폭 낭비다.
      ```
      eureka:
        client:
          registryFetchIntervalSeconds: 3
          shouldDisableDelta: true
      ```

#### 클라이언트와 서버간의 보안 통신하기

- 유레카 서버는 모든 클라이언트의 연결을 인증하지 않는다.
- 하지만 운영에서는 보안이 부족하면 문제가 될 수 있기 때문에 spring security를 활용하여 기본 인증을 사용하여 최소한의 보안을 적용하자

- Eureka Server
  - spring-security 의존성 추가
    ```
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    ```
  
  - application.yml에 컨피규레이션 설정
    ```
    spring:
      security:
        user:
          name: admin
          password: admin123
    ```
    - 기본 username과 password 생성
  
  - SecurityConfig.java 생성
    ```
    @Configuration
    @EnableWebSecurity
    public class SecurityConfig extends WebSecurityConfigurerAdapter {
      @Autowired
      private PasswordEncoder passwordEncoder;
      
      @Override
      protected void configure(HttpSecurity http) throws Exception {
        // TODO Auto-generated method stub
        http.authorizeRequests().antMatchers("/login", "/logout", "favicon.ico").permitAll()
			.and().authorizeRequests().anyRequest().authenticated()
			.and().csrf().disable()
			.and().formLogin()
			.and().httpBasic();
      }
	
      @Override
      protected void configure(AuthenticationManagerBuilder auth) throws Exception {
	// TODO Auto-generated method stub
	auth.inMemoryAuthentication()
	  .withUser("admin")
	  .password(passwordEncoder.encode("admin123"))
	  .roles("SYSTEM");
      }
    }
    ```
    - client에서 연결할 때 username:password@url:port 방식으로 연결한다.
    - formLogin을 해주었기 때문에 csrf를 disable 해주어야 가능하다.
  
  - Server로 접근하면 login form으로 이동한다.
    - http://localhost:8761 -> redirect -> http://localhost:8761/login
    
- Eureka Client
  - application.yml 파일에 다음 컨피규레이션 설정과 같이 URL 연결 주소에 자격증명을 제공하자
    ```
    eureka:
      client:
        fetch-registry: true
        register-with-eureka: true
        serviceUrl:
          defaultZone: http://admin:admin123@localhost:8761/eureka/
    ```
    - 디스커버리 클라이언트와 서버 간에 인증서를 사용한 안전한 SSL 연결을 맺는 등 더 진보된 사용을 위해서는 DiscoveryClientOptinalArgs를 맞춤형으로 구현해야한다.

#### 안전한 서비스 등록하기

- 스프링 부트 애플리케이션에 SSL을 활성화하려면 사설 인증서를 생성해야 한다. JRE 폴더 아래 bin 폴더 안에 있는 keytool로 생성
  ```
  $ keytool -genkey -alias client -storetype PKCS12 -keyalg RSA -
  $ keytool 2048 -keystore keystore.p12 -validity 3650
  ```
- 필요한 데이터를 입력하고 생성된 keystore 파일 keystore.p12를 애플리케이션의 src/main/resources 폴더에 복사한다. 다음으로 application.yml 파일의 컨피규레이션 속성을 사용해 스프링 부트의 HTTPS를 활성화 한다.
  ```
  server:
    port: ${PORT:8081}
    ssl:
      key-store: classpath:keystore.p12
      key-store-password: 123456
      keyStoreType: PKCS12
      keyAlias: client
  ```
- 애플리케이션을 시작한 후 안전한 https://localhost:8761/info에 접근할 수 있다. 또한 유레카 클라이언트 인스턴스의 컨피규레이션을 변경해야 한다.
  ```
  eureka:
    instance:
      securePortEnabled: true
      nonSecurePortEnabled: false
      statusPageUrl: https://${eureka.hostname}:${server.port}/info
      healthCheckUrl: https://${eureka.hostname}:${server.port}/health
      homePageUrl: https://${eureka.hostname}:${server.port}/
  ```
  
#### 유레카 API

- spring cloud netflix는 개발자가 유레카 API를 다룰 필요 없게 하는 자바로 작성된 클라이언트를 제공한다.
- 스프링이 아닌 다른 프레임워크를 사용하는 경우 API를 직접 호출해야 하는 경우가 있다.
- 난.. spring을 사용하기 때문에 pass??

#### 복제와 고가용성

- 서비스 장애를 대비하여 운영환경에서는 2개 이상의 디스커리 서버를 구성해야 한다.
- 유레카는 리더쉽 선출이나 클러스터에 자동 참여와 같은 표준 클러스터링 메커니즘을 제공하지 않는다.
  - 대신 동료간(peer-to-peer) 복제 모델에 기반한다.
  - 이는 모든 서버가 현재 서버 노드에 구성된 모든 동료에게 데이터를 복제하고 하트비트를 보낸다는 것이다.
  - 데이터를 저장한다는 목적에는 간단하면서도 효과적이지만 확장성 면에서는 모든 노드가 서버에 저장하는 모든 부하를 견뎌야 하는 단점이 있다.
- 유레카 서버 2.0은 복제 메커니즘을 제공하기 위해 노력하고 있다.
  - 복제 최적화와 더불어 등록된 목록의 모든 변경을 서버에서 클라이언트로 보내는 푸시 모델, 자동 확장된 서버, 대시보드와 같은 것이다.


    
#### 출처

- 마스터링 스프링 클라우드 서적
- 소스 코드: 
