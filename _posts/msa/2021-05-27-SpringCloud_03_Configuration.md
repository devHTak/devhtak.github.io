---
layout: post
title: 마스터 스프링 클라우드 - 05. 분산 컨피규레이션
summary: Spring Cloud
author: devhtak
date: '2021-05-27 22:41:00 +0900'
category: msa
---

### 들어가기

- 분산 컨피규레이션에 필요성
  - JAR 파일안에 설정 파일 구성
    - 수정이 필요하면 다시 배포해야 하는 단점이 있다.
  - 외부의 파일 시스템에 저장된 명시적 컨피규레이션 사용
    - spring.config.location 속성으로 구성
    - 다시 배포할 필요는 없지만, 마이크로 서비스가 많을 경우 파일 시스템에 저장된 명시적인 파일을 기반으로 컨피규레이션을 관리하기 어렵다
  
- 분산 컨피규레이션 사용
  - 클라우드 네이티브 환경에서 많이 사용한다.
  - 해당 솔루션을 사용하면 전체 환경에 걸쳐 애플리케이션을 위한 외부 속성을 관리하는 중앙의 단일 장소가 생긴다.
  - 서버는 HTTP와 자원 기반 API 인터페이스를 노출하며 반환된 속성 값에 대해 복호화와 암호화를 수행
  - 클라이언트는 서버로부터 컨피규레이션 설정을 가져온 후 복호화 한다.

- 컨피규레이션 저장소
  - Git(EnvironmentRepository의 기본 구현)
  - SVN
  - 파일 시스템, 볼트 등에서도 사용이 가능하다.

- 다룰 주제
  - 스프링 클라우드 컨피그 서버에 의해 노출되는 HTTP API
  - 서버 측의 다른 타입의 저장소 백엔드
  - 서비스 디스커버리와 통합
  - 스프링 클라우드 버스와 메시지 브로커를 사용하여 컨피규레이션을 자동으로 로딩

### HTTP API 자원의 소개

- 컨피그 서버는 다양한 방법으로 호출할 수 있는 HTTP API를 제공

  |종단점|설명|
  |---|---|
  |/{application}/{profile}\[/{label}]|JSON 형태로 데이터를 반환, label 입력값은 선택이다.|
  |/{application}-{profile}.yml|YAML 형태를 반환|
  |/{label}/{application-profile}.yml|위 종단점의 변형으로 label 입력값을 선택적으로 입력할 수 있다.|
  |/{application}-{profile}.properties|프로퍼티 파일에서 사용하는 간단한 키/값 형태로 반환|
  |/{label}/{application}-{profile}.properties|위 종단점의 변형으로 label 입력값을 선택적으로 입력할 수 있다.|
  
- 클라이언트의 관점에서 application 입력값은 애플리케이션의 이름으로 spring.application.name 또는 spring.config.name 속성에서 가져온다.
- profile은 활성화된 프로파일 또는 콤마로 분리된 활성화된 프로파일의 목록이다.
- label은 선택적 속성인데, 깃에 경우 브랜치 이름으로 설정한다. 기본 값은 master이다.
- localhost:8888/config_filename/profiles 를 통해 관리되는 설정정보를 알수있다.

#### 네이티브 프로파일 지원

- 파일 시스템 백엔드를 기반으로 시작
  - 기본적으로 스프링 클라우드 컨피그 서버는 Git 저장소에서 컨피규레이션 데이터를 가져오려고 한다.
  - 네이티브 프로파일을 활성화하기 위해서는 spring.profiles.active=native로 설정 해야 한다.
  - classpath:/, classpath:/config, file:./, file:./config에서 컨피규레이션 파일을 찾는다.
    - 속성 또는 YAML 파일이 JAR 파일 안에 위치할 수 있다.

- 예제
  - 테스트를 위해 src/main/resources 아래 config 폴더를 생성했다.
  - 서비스 디스커버리에서 사용했던 클라이언트 configuration을 configuration service로 옮기자
  - client-service-zone1.yml, client-service-zone2.yml, client-service-zone3.yml 로 나누어 옮겼다.
    - 한 파일에 같이 사용할 수도 있다.
    ```
    spring:
      profiles: zone1
    eureka:
      instance:
        metadata-map:
          zone: zone1
        client:
          service-url:
            default-zone: http://localhost:8761/eureka
    server:
      port: ${PORT:8081}
    ```

#### 서버측 애플리케이션 개발하기

- 디스커버리 서버와 동일하게 컨피그 서버는 스프링 부트 애플리케이션으로 실행된다.
- 의존성 포함
  ```
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
  </dependency>
  ```
  
- 메인 애플리케이션에서 컨피그 서버 활성화
  ```java
  @SpringBootApplication
  @EnableConfigServer
  public class SampleConfigServiceApplication {
	  public static void main(String[] args) {
		  SpringApplication.run(SampleConfigServiceApplication.class, args);
	  }
  }
  ```
  
- application.yml 속성
  ```
  server:  
    port: ${PORT:8888}    
  spring:
    application:
      name: config-server
  ```
    - 서버 포트를 8888로 변경
      - 클라이언트 측의 spring.cloud.config.uri 기본 속성이기 때문이다.
    - spring.config.name=configserver
      - 내장된 spring-cloud-config-server 라이브러리에 포함된 configserver.yml을 사용하게 한다.

#### 클라이언트 측 애플리케이션 개발하기

- 의존성 추가
  ```
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
  </dependency>
  ```

- application.yml 대신 bootstrap.yml 파일로 수정
  ```
  spring:  
    application:
      name: client-service
    cloud:
      config:
        uri: http://localhost:8889
  ```
  - spring.cloud.config.uri는 config server를 8888 기본 포트를 사용하면 설정하지 않아도 된다.

- 애플리케이션을 실행할 때 --spring.profiles.active=zone1 이라는 인자로 시작하면 자동으로 컨피규레이션 서버의 zone1 프로파일에서 설정을 가져온다.
  
#### 유레카 서버 추가하기

- 클라이언트 속성에 디스커버리 서비스의 네트워크 위치 주소가 있다.
  - 그래서 클라이언트를 실행하기 전에 eureka server가 먼저 실행중이어야 한다.
- configuration server에 discovery-service-zone1.yml, discovery-service-zone2.yml, discovery-service-zone3.yml 를 추가하자
  ```
  eureka:
    instance:
      hostname: localhost
      metadataMap:
        zone: zone1
    client:
      serviceUrl:
        defaultZone: http://localhost:8762/eureka/,http://localhost:8763/eureka/
  server:  
    port: ${PORT:8761}
  ```
  
- 의존성 추가
  ```
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
  </dependency>
  ```
  
- application.yml 대신 bootstrap.yml 파일로 수정
  ```
  spring:  
    application:
      name: discovery-service
    cloud:
      config:
        uri: http://localhost:8889
  ```

- 유레카 서버 실행 시 --spring.profiles.active 속성을 프로파일 이름으로 세팅한다.
  - discovery-seervice-zone1.yml ...

- 이렇게, config-service, client, eureka-server 를 세팅하면, client, eureka-server가 기동할 때 config-service에 저장 한 config 파일을 읽어서 사용하게 된다.

### 클라이언트 측에 부트스트랩 접근 방식 사용

- 컨피그 우선 부트스트랩 (Config First Bootstrap)
  - spring-cloud-config-client 아티팩트를 사용하는 애플리케이션의 기본 행동을 스프링 클라우드의 명명 규칙에 따라 행동하는 것
  - 컨피그 클라이언트가 시작할 때 컨피그 서버에 연결해 원격 속성을 사용하여 컨텍스트를 초기화 한다.
  - 즉, spring-cloud-config-client를 갖는 client, discovery는 config server에 대한 네트워크 위치를 알아야 한다.
- 디스커버리 우선 부트스트랩(Discovery First Bootstrap)
  - config server가 서비스 디스커버리 서비스에 등록되고 모든 애플리케이션이 Discovery Client 를 사용해 컨피그 서버를 찾는 것

#### 컨피그 서버 디스커버리

- sample discovery service 모듈
  - 예제 소스
    - https://github.com/piomin/sample-spring-cloud-netflix/tree/config_with_discovery
  - spring-cloud-starter-config 의존성은 필요 없다.
  - bootstrap.yml
    ```
    spring:
      application:
        name: discovery-service
    server: 
      port: ${PORT:8761}
    eureka:
     client:
       register-with-eureka: false
       fetch-registry: false
    ```

- config server
  - spring-cloud-starter-eureka 의존성 포함
  - 메인 application에 @EnableDiscoverClient 선언
  - application.yml 파일에 eureka.client.service-uri.default-zone 속성을 http://localhost:8761/eureka/ 로 설정
  
- client application
  - 더 이상 컨피규레이션 서버의 주소를 가지고 있을 필요가 없다.
  - config server가 다를 경우 service id만 설정하면 된다.
    ```
    spring:
      application:
        name: client-service
      config:
        discovery:
          enabled: true
          service-id: config-server
    ```

- 해당 방법으로 eureka 대쉬보드를 확인하면 config service와 client application 3개가 등록되었다.
- 모든 스프링 부트 클라이언트 인스턴스는 --spring-profiles-active=zone\[n] 인자로 실행한다.

### 백엔드 저장소 타입

- 현재, spring.profiles.active=native를 사용하여 파일시스템인 클래스파일을 활용하였다.
- 그러나, 운영용으로 사용하기 위해서는 다른 옵션을 고려하는 것이 좋다.
  - git, svn, vault, HashiCorp 등의 방법이 있다.
  - 여기에서는 git만 알아볼 예정이다.

#### 파일시스템

- 클래스 경로에 속성 소스를 저장하여 사용
- 해당 기본 위치는 spring.cloud.config.server.native.searchLocations 속성으로 재정의할 수 있다.
- JAR 파일을 다시 컴파일해야 하기 때문에 운영환경에는 적합하지 않다.

#### Git Backend

- Git 사용 장점
  - Git에서 제공하는 Commit, Revert, Branching과 같은 VSC 메커니즘을 사용하면 중요한 운영작업을 쉽게 사용할 수 있다.
  - config server code와 configuration 파일을 강제로 분리하여 관리할 수 있다.
  - git을 사용하면 컨피규레이션 변경을 실행중인 모든 인스턴스와 매우 쉽게 공유할 수 있다.

- 프로토콜
  - 깃 저장소 위치 설정
    - application.yml 파일에 spring.cloud.config.server.git.uri 속성 사용

  - cloning에서 file, http/https, ssh 프로토콜을 모두 사용할 수 있다.
    - 예시) spring.cloud.config.server.git.uri=file:/home/git/config-repo

  - 고가용 모드를 사용하기 위해서는 SSH 또는 HTTPS 프로토콜 사용해야 한다.
    - 스프링 클라우드 컨피그가 원격 저장소를 복제하고 그것을 로컬 작업 경로에 캐싱한다.

- URI에 플레이스홀더 사용
  - application, profile, label 인 모든 플레이스홀더를 지원한다.
    - application: 애플리케이션마다 단일 저장소를 생성할 수 있다.
    - profile: 프로파일마다 저장소를 생성할 수 있다.
    - label: HTTP의 label 입력값을 커밋 ID, branch, tag 에 대응하는 git의 label과 매핑할 수 있다.
  
- 서버 애플리케이션(config service) 개발
  - bootstrap.yml 수정
    ```
    spring:
      application:
        name: config-server
      cloud:
        config:
          server:
            git:
              uri: ${github.uri}
              username: ${github.username}
              password: ${github.password}
              clone-on-start: true
    ```
    - spring.cloud.config.server.git.clone-on-start: true
      - 애플리케이션 시작 직후 저장소 복제를 강제로 시작

  - 종단점 호출이 가능하다
    - http://localhost:8889/client-service/zone1 or http://localhost:8889/client-service-zone1.yml
    - git을 사용하기 때문에 호출하는 uri에 branch, commit id 등을 사용할 수 있다.
      - ex) http://localhost:8889/discovery/zone1  -- branch를 discover로 생성한 경우 
      - ex) http://localhost:8889/e546dd6/zone1 -- commit id가 e546dd6인 경우

- 클라이언트 측 개발
  - client application의 bootstrap.yml 안에 profile 속성을 설정하는 대신 spring.profiles.active 인자를 전달 할 수 있다.
  - 해당 컨피규레이션을 통해 클라이언트가 discovery 브랜치로부터 속성을 가져온다.
    ```
    spring:
      application:
        name: client-service
      cloud:
        config:
          uri: http://localhost:8889
          profile: zone1
          label: discovery
    ```

- 다중 저장소
  - 단일 컨피그 서버에 연러 저장소를 구성할 필요가 있을 수 있다.
  - 예를 들면, 기술적 컨피규레이션과 비즈니스 컨피규레이션을 구분해야 하는 상황
    ```
    spring:
      cloud:
        config:
          server:
            git:
              uri: ${github.uri}
              repos:
                simple: https://github.com/simple/config-repo
                special:
                  pattern: special*/dev*, *special*/dev*
                  uri: https://github.com/special/config-repo
                local:
                  pattern: local*
                  uri: file:/home/config/config-repo
    ```
    
#### 서비스 중단없이 설정 파일 적용하기

- 3가지 방법이 있다.
  - 서버 재기동
  - Actuator refresh
    - Application 상태, 모티너링
    - Metric 수집을 위한 Http Endpoint 제공
    - 여러개 service가 구성되어 있는 경우 모두 refresh하는 것은 번거러울 수 있다.
  - Spring cloud bus 사용
    - 분산 시스템의 노드를 경량 메시지 브로커와 연결
    - 상태 및 구성에 대한 변경 사항을 연결된 노드에게 전달(Broadcast)
  
- actuator의 refresh 사용
  - actuator 등록
    ```
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    ```
  - actuator의 endpoint 등록
    ```
    # auctuator 설정 - 사용하고자 하는 endpoint 
    management:
      endpoints:
        web:
	  exposure:
	    include: refresh, health, beans
    ```
  - http://localhost:${server.port}/actuator/refresh (POST method)으로 설정파일 변경을 바로 적용할 수 있다.

### 추가 기능

- 시작 시 실패와 재시도
  - 컨피그 서버가 사용 불가능한 경우 클라이언트에서 예외를 발생시켜 멈추게 해야 한다.
    - spring.cloud.config.failFast=true 설정 필요
  
  - 하지만 이런 방식보다는 접속 시도를 여러번 주어 시도해 볼 수 있다.
    - spring.cloud.config.failFast=true 설정 유지
    - spring-retry 라이브러리와 spring-boot-starter-aop 를 application class path에 추가해야 한다.
    - 기본 행동은 초기 1000ms의 백오프 간격으로 6번 재시도한다.
    - spring.cloud.config.retry.* 컨피규레이션 속성을 사용해 설정을 재정의할 수 있다.

- 클라이언트 안전하게 하기
  - 서비스 디스커버리 서버와 마찬가지로 컨피그 서버도 기본 인증을 사용해 안전하게 할 수 있다.
  - 컨피그 서버에 spring security 적용
  - client의 bootstrap.yml에 사용자 이름과 비밀번호 설정해야 한다.
    ```
    spring:
      cloud:
        config:
	  uri: https://localhost:8889
	  username: user
	  password: secret
    ```

### 자동으로 컨피규레이션 다시 읽기

- 파일 시스템, Git, Valut 중 어떤 것을 선택하더라도 새로운 컨피규레이션을 가져오기 위해서는 클라이언트 애플리케이션을 다시 시작해야 한다.
- 하지만 MSA 환경에서 최적화된 솔루션은 아니다.
- 컨피그 서버에 컨피그 내용을 동적으로 클라이언트에 적용하는 방법
  - 첫 번째, 컨피그 서버에 웹훅을 호출하게 될 특별한 종단점을 활성화한다.
  - 두 번째, 클라이언트에 @RefreshRemote 빈 설정 및 컨피그 서버와 클라이언트에 MQ를 설치하여 컨피그 서버에 컨피그 변경 이벤트가 발생하면 클라이언트에 전달하도록 한다. 
  - 세 번째, spring-cloud-config-monitor의 /monitor 종단점으로 변경 사항 적용
  - 네 번째, 지속적인 통합 도구인 Gitlab을 활용하여 이벤트 

- 솔루션 아키텍처
  - 서비스 디스커버리에서 서버와 연결을 끊기 위해 /shutdown 종단점을 제공한 것과 같은 종단점을 활용하여 동적으로 컨피규레이션을 가져올 수 있다.
  - 소스 코드 저장소 제공자는 웹훅(WebHook) 메커니즘을 사용해 저장소의 변경을 알릴 수 있다.
    - 웹훅은 서비스 제공자가 제공하는 웹 대시보드에 URL과 이벤트 타입의 목록을 선택해 설정할 수 있다.
    - 서비스 제공자는 웹훅에 정의된 경로에 POST 요청을 호출해 커밋의 목록을 전송한다.
  - 컨피그 서버
    - 종단점을 활성화하기 위해서 프로젝트에 스프링 클라우드 버스 의존성을 포함해야 한다.
      - 스프링 클라우드 버스: Rabbit MQ, Kafka
    - 웹훅이 호출되면 컨피그 서버는 마지막 커밋에 의해 변경된 소스 속성의 목록을 이벤트로 보낼 준비를 하며, 이 이벤트는 MQ를 통해 전송된다.
  - 클라이언트
    - 데이터를 받기 위해서 스프링 클라우드 버스 의존성을 추가하자.
    - @RefreshScope을 설정하여 동적 리프레시 메커니즘을 활성화한다.

- @RefreshScope를 사용해 컨피규레이션 다시 읽기
  - 클라이언트 컨피규레이션
    ```
    eureka:
      instance:
        metadata-map: 
	  zone: zone1
      client:
        service-url:
	  default-zone: http://localhost:8761/eureka/
    server:
      port: ${PORT:8081}
    management:
      security:
        enabled: false
    sample:
      string:
        property: Client App
      int:
        property: 1
    ```
    - management.security.enabled: false
      - 스프링 부트 액추에이터의 종단점 보안을 비활성화
      - 종단점을 비밀번호 없이 호출 가능
    - sample.string.property, sample.int.property
      - 예제에서 값에 기반한 빈을 생성하기 위해 추가
    - /refresh
      - 스프링 클라우드는 스프링 부트 액추에이터를 위한 몇가지 추가적인 HTTP 관리 종단점을 제공하는 것 중 하나
      - http://localhost:8081/refresh 종단점을 HTTP POST 메서드로 호출하면 @RefreshScope을 사용한 빈을 갱신
      - 이 경우, 디스커버리와 컨피그 서버가 실행중 이어야 한다.
    - 클라이언트 애플리케이션은 --spring.profiles.active=zone1 인자로 실행
  
  - @RefreshScope 빈
    ```java
    @Component
    @RefreshScope
    public class ClientConfiguration {
      @Value("${sample.string.property}")
      private String sampleStringProperty;
      
      @Value("${sample.int.property}")
      private int sampleIntProperty;
      
      public String showProperties() {
      	return String.format("Hello from %s %d", sampleStringProperty, sampleIntProperty);
      }
    }
    ```
    
  - 빈은 ClientController 클래스로 주입되고 http://localhost:8081/ping에 노출되는 내부의 ping 메서드를 호출한다.
    ```java
    @RestController
    public class ClientController {
      @Autowired private ClientConfiguration conf;
      
      @GetMapping("/ping")
      public String ping() { return confi.showProperties(); }
    }
    ```
  
  - config server의 client-service-zone1.yml 파일 수정
  - /client-service/zone1 HTTP 종단점을 호출하면 최신값이 반환된다.
  - 하지만, /ping 메서드를 호출하면 이전 값이 그대로 보인다.

  - 변경은 감지했지만, 외부 작응 없이(MQ event) 자동으로 갱신할 수 없기 떄문에 재시작하거나 /refresh 메서드를 호출해 컨피규레이션을 강제로 읽어주어야 한다.
      
  - rabbit mq 실행
    - container 실행
      ```
      $ docker run -d --name rabbit -p 5672:5672 -p 15672:15672 rabbitmq:management
      ```
      - 5672: 클라이언트 연결 용, 15672: 대시보드 연결 용
    - 클라이언트 pom.xml에 rabbitmq 의존성 추가
      ```
      <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-bus-amqp</artifactId>
      </dependency>
      ```
      - 해당 라이브러리는 자동으로 설정을 지원하지만, 윈도우에서 도커릴 실행했기 때문에 기본 속성 재정의
      
        ```
        spring:
          rabbitmq:
          host: 192.168.99.100
          port: 5672
          username: guest
          password: guest
        ```
      - http:192.168.99.100:15672 로 접속하면 대쉬보드에 접속할 수 있다.

- 컨피그 서버에서 저장소 변경 모니터링
  - 클라이언트에 spring-cloud-config-monitor 의존성 추가
    ```
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-config-monitor</artifactId>
    </dependency>
    ```
    - /monitor 종단점을 노출하도록 한다.
  - application.yml 에 컨피규레이션 모니터링 활성화
    - 스프링 클라우드에는 저장소 제공자마다 다르게 구현되기 때문에 선택 필요
    - 현재는 Github
    ```
    spring:
      application:
        name: config-server
    cloud:
      config:
        server:
	  monitor:
	    github:
	      enabled: true
    ```
  - 변경 감지 메커니즘 사용자 정의
    ```java
    @Bean
    public GithubPropertyPathNotificationExtractor githubPropertyPathNotificationExtractor {
      return new GithubPropertyPathNotificationExtractor();
    }
    ```
    - 기본으로 애플리케이션의 이름과 매칭되는 파일의 변경을 감지한다.
    - PropertyPathNotificationExtractor를 구현해 제공
      - Github으로 부터의 알림을 이용하기 위해 GithubPropertyPathNotificationExtractor
    
- 변경 이벤트를 수동으로 흉내내기
  - 수동으로 /monitor 종단점을 POST로 호출하면 웹훅을 쉽게 흉내낼 수 있다.
  - 깃허브 명령은 요처에 X-Github-Event 헤더를 포함해야 한다.
    ```
    $ curl -H "X-Github-Event: push" -H "Content-Type: application/json" -X POST -d '{"commits":[{"modified": ["client-service-zone1.yml"]}]}' http//localhost:8889/monitor
    ```
    - 만얀 sample.int.property를 수정한다면, 클라이언트 애플리케이션 로그에 Received remote refresh requests. key refreshed [sample.int.property]라고 표시된다.
    - /ping 종단점을 호출하면 변경된 속성의 최신값이 반환된다.

- 깃랩 인스턴스를 사용해 로컬 호스트에서 테스트하기
  - 지속적인 통합 도구인 깃랩 컨테이너 실행
    ```
    $ docker run -d --nae gitlab -p 10443:443 -p 10080:80 -p 10022:22 gitlab/gitlab-ce:latest
    ```
  - 웹 대시보드(포트: 10080) 접속
    - admin 계정 생성하고 로그인
  - 프로젝트 생성
    - 프로젝트 명: sample-spring-cloud-config-repo
    - http://192.168.99.100:10080/root/sample-spring-cloud-config-repo 에서 복제
  - 깃허브에서 컨피규레이션 파일을 가져와 커밋
  - 컨피그 서버의 /monitor 존단점에서 푸시 알림을 주는 웹 훅을 정의
    - Settings | Integration 섹션, URL 필드에 서버 주소 입력 (localhost 대신 hostname을 이용)
    - Push Events 체크박스 선택
  - 컨피그 서버에 설정 추가(application.yml)
    ```
    spring:
      application:
        name: config-server
    cloud:
      config:
        server:
	  monitor:
	    github:
	      enabled: true
	git:
	  uri: http://192.168.99.100:10080/root/sample-spring-cloud-config-repo.git
	  username: root
	  password: root123
	  clone-on-start: true
    ```
  - 컨피그 서버에 빈 설정
    ```java
    @Bean
    public GitlabPropertyPathNotificationExtractor gitlabPropertyPathNotificationExtractor() {
      return new GitlabPropertyPathNotificationExtractor();
    }
    ```
  
  - 컨피규레이션 파일을 변경한 후 커밋하면 클라이언트 애플리케이션의 컨피규레이션도 갱신되는 것을 확인할 수 있다.


#### 메시지 브로커로 부터 이벤트 받기(Spring Cloud Bus)
- Spring CLoud Config Server + Spring Cloud Bus
  - AMQP(Advanced Message Queuing Protocol)을 활용하여 Cloud Config Server가 여러 서비스에게 이벤트를 Push할 수 있다.
  - AMQP
    - 메시지 지향 미들웨어를 위한 개방형 표준 응용 계층 프로토콜
    - 메시지 지향, 큐잉, 라우팅(P2P, Publish-Subscriber), 신뢰성, 보안
    - Erlang, RabbitMQ에서 사용
  - Kafka 프로젝트
    - Apache Software Foundation이 Scalar 언어로 개발한 오픈 소스 메시지 브로커 프로젝트
    - 분산형 스트리밍 플랫폼
    - 대용량의 데이터를 처리 가능한 메시징 시스템
- Actuator bus-refresh Endpoint
  - 분산 시스템의 노드를 경량 메시지 브로커와 연결
  - 상태 및 구성에 대한 변경 사항을 연결된 노드에게 전달(Broadcast)
  - 흐름
    - client -> HTTP Post /busrefresh 호출(호출되는 서비스는 상관 없음)
    - Spring Cloud Bus가 변경된 이벤트를 감지
    - 연결된 Service로 변경된 사안을 업데이트한다.

- AMQP 사용
  - Config Server 및 Service에 dependency 추가
  - rabbit mq 설정
    ```
    spring:
      rabbitmq:
        host: 127.0.0.1
	port: 5672
	username: guest
	password: guest
    ```
  - actuator busrefresh endpoint 추가
    ```
    management:
      endpoints:
        web:
	  exposure:
	    include: refresh, health, beans, httptrace, busrefresh
    ```
  - 기동 순서
    - rabbitmq 실행 -> config server 실행하면 rabbitmq 관련되 정보가 정상 연결된 것을 확인할 수 있다.

#### 암호화하여 사용하기

- config server에 bootstrap 등록
  - spring-cloud-starter-bootstrap dependency

- 대칭키를 이용한 암호화
  - bootstrap.yml에 encrypt 키 등록
    ```
    encrypt:
      key: ...
    ```
    - bootstrap.yml은 자동으로 읽어오지 않는다.
    - spring-cloud-starter-bootstrap dependency를 등록해주어야 한다.

  - POST /encrypt 에 body를 작성하여 요청하면 암호화된 값을 받을 수 있다.

    ![image](https://user-images.githubusercontent.com/42403023/127247936-bbb32a0a-7627-4e68-9c6d-62315c4b6dce.png)  
    
  - yml파일에 사용하고자 하는 값을 encrypt하여 사용한다.
    ```
    spring:
      datasource:
        url: jdbc:h2:~/test;
        driverClassName: org.h2.Driver
        username: sa
        password: '{cipher}...'
    ```
  
  - 비대칭키
    - Public, Private key 생성 -> JDK keytool 이용
      ```
      $ mkdir ${user.home}/Desktop/Work/keystore
      $ keytool -genkeypair -alias apiEncryptionKey -keyalg RSA\
        -dname "CN=test, OU=API Development, O=test.co.kr, L=Seoul, C=KR"\
        -keypass "..." -keystore apiEncryptionKey.jks -storepass "..."
      $ keytool -export -alias apiEncryptionKey -keystore apiEncryptionKey.jks -rfc -file trustServer.csr
      $ keytool -import -alias trustServer -file trustServer.cer -keystore publicKey.jks
      ```
      - alias로 지정한 값을 통해 호출할 수 있다.
      - keypass, storepass는 임의로 지정한 키이다
      - export 를 사용하여 인증서를 만들 수 있다.
      - import 를 사용하여 public jks 파일을 만들 수 있다
    - key-store 을 config server 속성에 작성한다.
      ```
      encrypt:
        key-store:
	  location: file://${user.home}/.../keystore
	  password: ...
	  alias: apiEncryptionKey
      ```
    - POST /encrypt, /decrypt 를 호출하면 암호화/복호화된 값을 볼 수 있다
    - yml파일에 사용하고자 하는 값을 encrypt하여 사용한다.
      ```
      spring:
        datasource:
          url: jdbc:h2:~/test;
          driverClassName: org.h2.Driver
          username: sa
          password: '{cipher}...'
      ```

#### 출처

- 마스터링 스프링 클라우드
- Spring Cloud로 개발하는 마이크로서비스애플리케이션
