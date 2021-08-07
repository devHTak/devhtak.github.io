---
layout: post
title: Spring Cloud - 분산 추적
summary: Spring Cloud
author: devhtak
date: '2021-08-06 22:41:00 +0900'
category: msa
---

#### 분산 추적

- 분산추적이 필요한 이유는 분산된 마이크로서비스간의 트래픽을 추적하여 문제를 사전에 방지하거나 해결을 위해서 이다.
- Sleuth + Zipkin 사용
  - 동일한 트랙잰션에 해당하는 트래픽들에 동일한 TraceID를 부여
  - 위 주문 트랜잭션에서 모든 트래픽이 동일한 Trace ID를 갖고 있다면 쉽게 추적할 수 있다
  - Sleuth를 적용한 후 Log4j, Logback, SLF4J(Simple Logging Facade for Java)등을 사용하여 로깅하면, 자동으로 로그에 Service명, Trace ID, Span ID가 삽입됩니다.
    
    ![image](https://user-images.githubusercontent.com/42403023/128593518-e13d321f-230c-48eb-a2e1-31eed8916e37.png)
  
    - 이미지 출처: https://docs.spring.io/spring-cloud-sleuth/docs/2.2.x-SNAPSHOT/reference/html/

#### Zipkin

- https://zipkin.io/
- Twitter에서 사용하는 분산 환경의 Timing 데이터 수집, 추적 시스템(오픈 소스)
- Google Drapper에서 발전하였으며, 분산환경에서의 시스템 병목 현상 파악
- Collector, Query Service, Databasem WebUI로 구성
- span
  - 하나의 요청에 사용되는 작업의 단위
  - 64 bit unique ID
- trace
  - 트리 구조로 이뤄진 Span 셋
  - 하나의 요청에 대한 같은 Trace ID 발급

- 설치(도커)
  ```
  $ docker run -d -p 9411:9411 openzipkin/zipkin
  ```
  - localhost:9411/zipkin 접속 확인
  

#### Spring Cloud Sleuth

- 스프링부트 애플리케이션을 Zipkin과 연동
- 요청 값에 따른 Trace ID, Span ID 부여
- Trace와 Span Ids를 로그에 추가 가능
  - server filter
  - rest template
  - scheduled actions
  - message channels
  - feign client


#### 출처

- Spring Cloud로 개발하는 마이크로서비스 애플리케이션(MSA)
