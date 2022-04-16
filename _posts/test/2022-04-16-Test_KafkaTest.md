---
layout: post
title: Kafka Test
summary: Kafka Test
author: devhtak
date: '2022-04-16 21:41:00 +0900'
category: Test
---

#### 개요

- 실행 중인 외부 Kafka 서버에 의존하지 않는 안정적이고 독립적인 통합 테스트를 작성하는 방법
- Kafka 구성 및 producer, consumer를 작성하고 spring-kafka-test를 활용하여 테스트코드 작성

#### dependency

- starter.spring.io 에서 kafka를 선택하면 자동으로 의존성이 작성된다
- maven
  ```
  <dependency>
      <groupId>org.springframework.kafka</groupId>
      <artifactId>spring-kafka-test</artifactId>
      <scope>test</scope>
  </dependency>
  ```
- gradle kotlin
  ```
  testImplementation("org.springframework.kafka:spring-kafka-test")
  ```

#### kafka 구성

#### Kafka Test

- @EmbeddedKafka
  - kafka test를 위한 가장 간단한 방법으로 @EmbeddedKafka 애노테이션을 붙이면 EmbeddedKafkaBroker를 테스트 메서드, 세팅등에 사용할 수 있게 해준다.
  - @ExtendWith(SpringExtension.class) 로 Spring Context 구성
  - @Autowired로 EmbeddedKafkaBroker를 주입받을 수 있다.



  

#### 출처
- https://blog.mimacom.com/testing-apache-kafka-with-spring-boot-junit5/
- https://github.com/spring-projects/spring-kafka/blob/main/spring-kafka/src/test/kotlin/org/springframework/kafka/listener/EnableKafkaKotlinTests.kt
