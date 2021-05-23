---
layout: post
title: 마스터 스프링 클라우드 - 03. 스프링 클라우드 개요
summary: Spring Cloud
author: devhtak
date: '2021-05-23 14:41:00 +0900'
category: Spring Cloud
---

#### 기본부터 시작하기

- 스프링 클라우드는 스프링 부트와 달리 원격 서버에서 Configuration을 가지고 온다.
  - spring-cloud-starter-config

- Bootstrap Context
  - 스프링 클라우드는 부트스트랩 컨텍스트에 의해 작동한다.
  - 부트스트랩 컨텍스트는 메인 애플리케이션 컨텍스트의 부모 컨텍스트다.
  - 애플리케이션 내에 필요한 최소 설정하여 스프링 클라우드 프로젝트가 외부 소스에서 Configuration을 읽어올수 있는 역할을 한다.
  - application.~ 대신 bootstrap.~ 을 사용한다.
  - 설정
    - spring.cloud.bootstrap.enabled: 부트스트랩 컨텍스트 활성화/비활성화
    - spring.cloud.bootstrap.enabled: 설정 파일 위치 지정
    - bootstrap-{profile}.~ : 프로파일 지정

- Spring Cloud Commons
  - 스프링 클라우드 프로젝트에 부모 의존성으로 포함된 라이브러리
  - 서비스 디스커버리, 부하 분산, 서킷 브레이커 등의 메커니즘을 위한 공통의 추상 레이어 제공

#### Netflix OSS(Open Source SW)

- 넷플릭스는 MSA 오픈 소스를 공유하였다.
- Spring Cloud는 넷플릭스의 유명한 오픈 소스인 유레카(Eureka), 히스트릭스(Hystrix), 리본(Ribbon), 주울(Zuul) 등과 통합

- 유레카를 사용한 서비스 디스커버리
  - 클라이언트와 서버로 구분
  - 서비스 디스커버리 패턴이란?
    - MSA와 같은 분산 환경은 서비스 간의 원격 호출로 구성이 된다.
    - 클라우드 환경이 되면서 서비스가 동적으로 생성되거나, 컨테이너 기반의 배포로 인해 서비스의 IP가 동적으로 변경되는 일이 발생한다.
    - 서비스 디스커버리란 클라이언트가 서비스를 호출할 때 서비스의 위치(IP, Port)를 알아낼 수 있는 기능 제공
    - Service Registry에 제공할 Service에 IP를 저장하여 사용
    
      ![image](https://user-images.githubusercontent.com/42403023/119246284-5a1c1380-bbbb-11eb-8862-22fa9b819a2f.png)
      
      이미지 출처: https://bcho.tistory.com/1252

  - 클라이언트
    - spring-cloud-starter-eureka를 추가하여 사용
    - 클라이언트는 항상 애플리케이션의 일부로 원격의 디스커버리 서버에 연결하는 일을 담당한다.
    - 연결 후에 서비스 이름과 네트워크 위치 정보를 등록 메시지로 전송
    - 다른 마이크로서비스 API를 호출해야 할 경우 디스커버리 서버로부터 서비스 목록을 담은 최신 컨피규레이션을 수신한다.

  - 서버
    - spring-cloud-eureka-server를 추가하여 사용
    - 독립적인 스프링 부트 애플리케이션
    - 각 서버의 상태를 다른 서버에 복제해 가용성이 높다.

- 주울을 사용한 라우팅
  - spring-cloud-starter-zuul를 추가하여 사용
  - JVM 기반 라우터로, 부하 분산과 필터링을 수행
  - 다양한 곳에 적용 가능. 인증, 부하 평균 분배, 정적 응답 처리, 부하 테스트에 사용 가능
  - 유레카와 같이 독립적인 스프링 부트 애플리케이션
  - API 게이트웨이 역할을 수행
  - 각 서비스의 네트워크 위치 정보를 알아야 하므로 유레카(디스커버리) 서버와 통신하기 위해 디스커버리 클라이언트를 클래스 경로에 포함

- 리본을 사용한 부하 분산
  - spring-cloud-starter-ribbon를 추가하여 사용
  - 클라이언트(서비스) 측 부하 분산기
  - TCP, UDP, HTTP 등 유명한 프로토콜 지원하며 동기, 비동기, 리액티브 모델 지원한다.
  - HTTP와 TCP 클라이언트를 한 단계 더 추상화
  - 라운드 로빈, 가용성 필터링, 응답 시간에 가중치를 두어 부하 분산 등의 규칙을 즉시 제공, 쉽게 확장 가능
  - 이름 기반 부하 분산(네임드 클라이언트)에 기반

- 페인Feign을 활용하여 자바 HTTP 클라이언트 작성하기
  - spring-cloud-stater-feign를 추가하여 사용
  - 선언적인 REST 웹 서비스 클라이언트
  - 애노테이션 선언만으로 애플리케이션이 실행될 때 실제 구현이 실행됨
  - 리본 클라이언트와 통합돼 디스커버리 서비스와의 통신, 부하 분산 같은 리본의 기능을 기본으로 제공

- 히스트릭스(hystrix)를 사용해 대기 시간 및 장애 내성 다루기(서킷 브레이커)
  - spring-cloud-starter-hystrix를 추가하여 사용
  - 서킷 브레이커 패턴을 구현
    - ServiceA가 ServiceB를 호출하는 데, 어떤 문제로 인하여 ServiceB가 응답을 못한다.
    - ServiceA에서 ServiceB를 호출한 쓰레드는 응답을 받지 못하기 때문에, 계속 응답을 기다리는 상태로 잡혀있게 된다. 
    - 지속해서 Service A가 Service B를 호출을 하게 되면 앞과 같은 원리로 각 쓰레드들이 응답을 기다리는 상태로 변하게 되고 결과적으로는 남은 쓰레드가 없어서 다른 요청을 처리할 수 없는 상태가 된다.
    - 이를 해결하기 위한 패턴이 Circuit Breaker
    - Service B가 문제가 생겼음을 Circuit breaker가 감지한 경우에는 Service B로의 호출을 강제적으로 끊어서 Service A에서 쓰레드들이 더 이상 요청을 기다리지 않도록 해서 장애가 전파하는 것을 방지 한다. 
    - 강제적으로 호출을 끊으면 에러 메세지가 Service A에서 발생하기 때문에 장애 전파는 막을 수 있지만, Service A에서 이에 대한 장애 처리 로직이 별도로 필요하다.
    - 이를 조금 더 발전 시킨것이 Fall-back 메시징인데, Circuit breaker에서 Service B가 정상적인 응답을 할 수 없을 때, Circuit breaker가 룰에 따라서 다른 메세지를 리턴하게 하는 방법이다. 
  - 기본적으로 리본과 페인 클라이언트를 통합 가능
  - 서킷 브레이커 시간 만료(timeout) 발생 시 폴백 로직을 쉽게 설정 가능
  
- 아카이우스(Archaius)를 사용한 컨피규레이션 관리
  - 변경 전의 원본을 가져오거나 변경사항을 클라이언트에 전달하는 방법으로 컨피규레이션을 갱신
  
#### 디스커버리와 분산 컨피규레이션

- 서비스 디스커버리와 분산 컨피규레이션 관리는 유연한 키-값 저장소 형태로 특정 키와 값을 저장하는 방법으로 기술적으로 매우 비슷하다.
- 서버측과 클라이언트 측 지원으로 나누다.
  - 서버는 단 하나의 저장소로, 애플리케이션을 위한 모든 외부 속성을 관리
  - 컨피규레이션은 여러 버전과 프로파일로 동시에 유지하며 저장소로 깃을 사용하고 있다.

- 설정 파일은 파일 시스템, 서버 클래스 경로에 있을 수있다.
  -  백엔드를 볼트(Valut)로 사용 가능
  -  볼트는 토큰이나 패스워드, 자격 증명을 관리할 수 있는 오픈 소스 도구
  -  볼트를 통해 컨피규레이션 서버의 접근 레벨을 안전하게 관리할 수 있음

- 컨피그 서버
  - 독립형 스프링 부트 애플리케이션
  - HTTP와 리소스 기반 API를 통해 속성에 쉽게 접근
  - 기본 인증으로 보호되며, 개인 키/공개 인증을 사용한 SSL 연결도 설정 가능함
  - 서버 스타터 : spring-cloud-config-server, 클라이언트 스타터 : spring-cloud-config-starter
  - 컨피규레이션 서버를 속성 저장소로 사용하는 모든 마이크로서비스 클라이언트는 클라이언트가 시작되고 스프링 빈이 생성되기 전에 컨피규레이션 서버에 접속함

- 대안 1. 컨설(Consul)
  - spring-cloud-starter-consul-discovery를 추가하여 사용
  - 넷플릭스 디스커버리와 스프링 분산 컨피규레이션의 대안
  - 컨설 서버에 연결하기 위해서는 애플리케이션에 에이전트가 필요 (별도로 분리된 프로세스로 실행)
  - 기본적으로 http://localhost:8500 주소 제공
  - 서비스 등록, 서비스 목록 수집, 속성의 컨피규레이션을 수행할 수 있는 REST API 제공
  - 넷플릭스 리본 동적 라우터, 넥플릭스 주울 필터 지원
  - 컨설 아키텍처 참조 (https://www.consul.io/docs/internals/architecture.html)

- 아파치 주키퍼
  - 하나가 분산된 시스템간의 정보를 어떻게 공유할것이고, 클러스터에 있는 서버들의 상태를 체크할 필요가 있으며 또한, 분산된 서버들간에 동기화를 위한 락(lock)을 처리하는 것들이 문제로 부딪힌다. 해당 문제를 해결하는 시스템을 코디네이션 서비스 시스템이라고 하는데 Zookeeper가 대표적이다.
  - 컨피규레이션과 이름을 유지하는 중앙 서비스로의 분산 동기화, 그룹 서비스가 가능
  - 예시 : 공통 애노테이션을 통한 통합제공, 설정 파일을 통한 컨피규레이션, 리본/주울과 상호작용하기 위한 자동-컨피규레이션

- 스프링 클라우드 쿠버네티스(Kubernetes)
  - 배포, 확장, 애플리케이션 컨테이너의 관리를 자동화하는 시스템
  - 컨테이너 오케스트레이션(orchestration) 및 서비스 디스커버리, 컨피규레이션 관리, 부하 분산 등의 기능을 제공

- 스프링 클라우드 에티시디(Etcd)
  - 분산 컨피규레이션, 서비스 등록, 디스커버리 기능 제공
  - 쿠버네티스에서 서비스 디스커버리, 클러스터 탄생, 컨피규레이션 관리에 쓰이는 백엔드로 사용

#### 슬루스(Sleuth)를 사용한 분산 추적

- 분산 추적 기능 
  - 하나의 요청을 여러 마이크로서비스로 처리할 때 이어지는 요청을 연관 짓는 기능
- HTTP 헤더에 기반하여 추적, Slf4j, MDC로 개발됨
  - Slf4j는 특정 로깅 프레임워크의 추상화 퍼사드를 함
    - 퍼사드 패턴 : 단순화된 인터페이스를 통해서 서브 시스템을 더 쉽게 사용할 수 있도록 하는 패턴
  - MDC(mapped diagnostic context)는 다양한 소스의 로그 출력을 구분하고 부가 정보를 추가하는 솔루션
- 트레이스(trage) ID와 스팬(span) ID를 Slf4j, MDC에 추가해 관련된 로그를 추출 가능
- 샘플링 정책
 - 집킨으로 보낼 트래픽의 양을 결정하여 일부 데이터만 수집
- 집킨
  - MSA 내부의 지연 문제를 분석하기 위해 설계된 분산 추적 시스템
  - 시간 정보를 질의하고 시각화함
  
#### 메시징과 통합

- 일반적으로 스프링 클라우드는 동기/비동기 HTTP 통신과 메시지 브로커를 지원
- 스프링 클라우드 버스
  - spring-cloud-stater-bus-amqp 또는 spring-cloud-stater-bus-kafka
  - 컨피규레이션 속성 변경, 관리 명령 등의 상태 변경을 브로드캐스트로 애플리케이션에 알림
  - 공통 오퍼레이션을 위한 분산 메시지 기능 지원

- 스프링 클라우드 스트림
  - spring-cloud-stater-stream-kafka 또는 spring-cloud-stater-stream-rabbit 
  - 메시지 중심 마이크로서비스로 구성된 시스템에 적합
  - 스프링 인테그레이션에 기반함
    - 스프링 인테그레이션은 채널,애그리게이터,트랜스포머와 같은 엔터프라이즈 통합 패턴 프로그래밍 모델을 제공하는 프로젝트
  - 마이크로서비스 시스템 내의 애플리케이션 스프링 클라우드 스트림 입력 및 출력 채널을 통해 통신

#### 클라우드 플랫폼 지원

- 피보탈 클라우드 파운드리(PCF, Pivotal Cloud Foundry)
  - spring-cloud-services-stater-circuit-breaker 추가하여 사용
  - spring-cloud-services-stater-config-client 추가하여 사용
  - spring-cloud-services-stater-service-registry 추가하여 사용
  - 애플리케이션을 배포하고 관리하는 클라우드 네이티브 플랫폼
  - 스프링 부트의 실행 가능한 JAR, 컨피그 서버, 서비스 레지스트리, 서킷 브레이커 등 모든 스프링 클라우드 마이크로서비스 패턴을 지원

- 스프링 클라우드는 AWS를 지원하기 위해 유명한 웹 도구와 통합
  - Simple Queuing Service(SQS)
  - Simple Notification Service(SNS)
  - ElasticCache
  - Relation Database Service(RDS)
    - Aurora, MySQL, Oracle과 같은 엔진을 제공함

- 주요 모듈
  - Spring Cloud AWS Core : spring-cloud-stater-aws 스타터로 활성화. EC2 인스턴스로 직접 접근을 활성화하는 핵심 구소 요소 제공
  - Spring Cloud AWS Context : S3 저장소, 이메일 서비스, 캐싱 서비스로의 접근 제공
  - Spring Cloud AWS JDBC : spring-cloud-stater-aws-jdbc 스타터로 활성화. 스프링에서 지원하는 데이터 접근 기술을 사용할 수 있는 데이터 소스 조회 및 컨피규레이션 제공
  - Spring Cloud AWS Messaging : spring-cloudstater-aws-messaging 스타터로 활성화. 애플리케이션이  SQS(점대점 메시징) 또는 SNS(게시/구독 메시징)로 메시지를 보내고 받을 수 있게 함

- 스프링 클라우드 커넥터 프로젝트
  - 클라우드 플랫폼에 배포된 JVM 애플리케이션를 위한 추상화를 제공
  - SMTP, 래빗엠큐, 레디스(Redis), 전통 DB에 접속할 수 있는 클라우드 플랫폼
  - 히로쿠(Heroku)와 클라우드 파운드리를 지원

#### 다른 유용한 라이브러리

- 보안
  - 스프링 클라우드 시큐리티
    - spring-cloud-stater-security
    - OAuth2, JWT, 기본 인증 메키너즘 API 구현
    - 싱글 사인온(single sign-on)과 토큰 리플레이(token replay) 패턴을 지원

- 테스트 자동화
  - 스프링 클라우드 컨트랙트
    - 와이어목(WireMock)을 사용하여 트래픽을 기록
    - 메이븐 플러그인을 사용하여 스텁(stub) 생성

  - 스프링 클라우드 태스크
    - spring-cloud-stater-task
    - 한 번만 실행하고 종료하는 마이크로서비스를 개발하도록 지원
    - 보통 로컬 컴퓨터나 클라우드 환경에서 실행

- 클러스터 기능
  - 스프링 클라우드 클러스터
    - 주키퍼, 레디스, 해즐캐스트(Hazelcast), 컨설에 대한 추상화 및 구현을 사용해 리더 선출과 공통 상태유지 패턴을 위한 솔루션을 제공

- 프로젝트 개요
  
  |서비스|프로젝트 명|
  |---|---|
  |분산 컨피규레이션|Spring Cloud Config, Spring Cloud Zookeeper Config, Spring Cloud Consul Config, Spring Cloud Etcd Config, Spring Cloud Kubernetes Config|
  |서비스 디스커버리|Spring Cloud Eureka, Spring Cloud Zookeeper Discovery, Spring Cloud Consul Discovery, Spring Cloud Etcd Discovery, Spring Cloud Kubernetes Discovery|
  |상호 통신|Spring Cloud Hystrix, Spring Cloud Ribbon, Spring Cloud Feign, Spring Cloud Zuul|
  |추적|Spring Cloud Sleuth, Spring Cloud Sleuth Zipkin, Spring Cloud Sleuth Stream|
  |클라우드 플랫폼 지원|Spring Cloud Cloud Foundry, Spring Cloud AWS, Spring Cloud Function, Spring Cloud Connectors|
  |메시징 및 통합|Spring Cloud Stream, Spring Cloud Bus, Spring Cloud Stream Apps, Spring Cloud Data Flow|
  |기타|Spring Cloud Contract, Spring Cloud Security, Spring Cloud Task, Spring Cloud Cluster|

#### 릴리즈 트레인(release trains)

- 스프링 클라우드 내에 수많은 프로젝트가 있어 의존성 관리가 문제가 되어 모든 프로젝트 버전 간의 관계를 알아야 하는 문제가 발생
- 이를 위해 릴리즈 트레인 도입
- 릴리즈를 버전이 아닌 BOM에 기반한 이름으로 구분
  - BOM(Bill of materials)는 아티팩트 버전을 독립적으로 관리하는 표준 메이븐 개념
- 최신 릴리즈 목록 : https://project.spring.io/spring-cloud/
- 릴리즈 트레인 뒤에 붙은 표시
  - M\[X] : M은 마일스톤, X는 버전 번호
  - SR\[X] : 서비스 릴리즈로서 중요한 버그를 수정한 버전
  - 메이븐 또는 그래들로 의존성을 포함할 시 올바른 릴리즈 트레인 이름을 사용해야 함
  
#### 출처

- 서비스 디스커버리: https://bcho.tistory.com/1252
- 서킷 브레이커: https://bcho.tistory.com/1247
- Zookeeper: https://bcho.tistory.com/1016
- 마스터링 스프링 클라우드
