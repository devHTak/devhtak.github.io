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
