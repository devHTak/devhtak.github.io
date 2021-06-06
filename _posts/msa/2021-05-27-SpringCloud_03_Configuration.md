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

#### 네이티브 프로파일 지원

- 파일 시스템 백엔드를 기반으로 시작
  - 기본적으로 스프링 클라우드 컨피그 서버는 Git 저장소에서 컨피규레이션 데이터를 가져오려고 한다.
  - 네이티브 프로파일을 활성화하기 위해서는 spring.profiles.active=native로 설정 해야 한다.
  - classpath:/, classpath:/config, file:./, file:./config에서 컨피규레이션 파일을 찾는다.
    - 속성 또는 YAML 파일이 JAR 파일 안에 위치할 수 있다는 뜻이다.

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

#### Vault Backend

- 사용할 일이 없을 듯 하여 현재는 넘어가도록 하겠다..

#### 출처

- 마스터링 스프링 클라우드 

