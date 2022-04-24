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

- Kafka Config
  - application.yml 로 세팅하기
    ```yml
    spring:
      kafka:
        consumer:
          bootstrap-servers: localhost:9092
          group-id: foo
          auto-offset-reset: earliest
          key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
          value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
        producer:
          bootstrap-servers: localhost:9092
          key-serializer: org.apache.kafka.common.serialization.StringSerializer
          value-serializer: org.apache.kafka.common.serialization.StringSerializer
    ```
    - spring.kafka.consumer
      - bootstrap-servers
        - Kafka 클러스터에 대한 초기 연결에 사용할 호스트:포트쌍의 쉼표로 구분된 목록
        - 글로벌 설정이 있어도, consumer.bootstrap-servers가 존재하면 consuemer 전용으로 오버라이딩
      - group-id
        - Consumer는 Consumer Group이 존재하기 때문에, 유일하게 식별 가능한 Consumer Group을 작성
      - auto-offset-reset
        - Kafka 서버에 초기 offset이 없거나, 서버에 현재 offset이 더 이상 없는 경우 수행할 작업을 작성
        - Consumer Group의 Consumer는 메시지를 소비할 때 Topic내에 Partition에서 다음에 소비할 offset이 어디인지 공유를 하고 있다.
        - 오류 등으로 인해. 이러한 offset 정보가 없어졌을 때 어떻게 offeset을 reset 할 것인지를 명시한다고 보시면 됩니다.
          - latest : 가장 최근에 생산된 메시지로 offeset reset
          - earliest : 가장 오래된 메시지로 offeset reset
          - none : offset 정보가 없으면 Exception 발생. 직접 Kafka Server에 접근하여 offset을 reset할 수 있지만, Spring에서 제공해주는 방식은 위와 같다.
      - key-deserializer / value-deserializer
        - Kafka에서 데이터를 받아올 때, key / value를 역직렬화
        - 여기서 key와 value는 뒤에서 살펴볼 KafkaTemplate의 key, value를 의미
        - 메시지가 문자열 데이터이므로 StringDeserializer를 사용, JSON 데이터를 넘겨줄 것이라면 JsonDeserializer도 가능
    - spring.kafka.producer
      - bootstrap-servers
        - consumer.bootstrap-servers와 동일한 내용이며, producer 전용으로 오버라이딩 하려면 작성
      - key-serializer / value-serializer
        - Kafka에 데이터를 보낼 때, key / value를 직렬화
        - consumer에서 살펴본 key-deserializer, value-deserializer와 동일

  - 코드로 세팅하기
    - Producer
      ```kotlin
      @Component
      class KafkaProducerConfig {

          @Bean
          fun kafkaProducer(): ProducerFactory<String, String> {
              var properties = mapOf(
                  ProducerConfig.BOOTSTRAP_SERVERS_CONFIG to "127.0.0.1:9092",
                  ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG to StringSerializer::class.java,
                  ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG to StringSerializer::class.java
              )

              return DefaultKafkaProducerFactory(properties)
          }

          @Bean
          fun kafkaTemplate(): KafkaTemplate<String, String> {
              return KafkaTemplate(kafkaProducer())
          }
      }
      ```
    - Consumer
      ```kotlin
      @Component
      class KafkaConsumerConfig {

          @Bean
          fun consumerFactory(): ConsumerFactory<String, String> {
              val properties = mapOf(
                  ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG to "127.0.0.1:9092",
                  ConsumerConfig.GROUP_ID_CONFIG to "consumer_group",
                  ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG to StringDeserializer::class.java,
                  ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG to StringDeserializer::class.java
              )

              return DefaultKafkaConsumerFactory(properties)
          }

          @Bean
          fun concurrentKafkaListenerContainerFactory(): ConcurrentKafkaListenerContainerFactory<String, String> {
              val factory = ConcurrentKafkaListenerContainerFactory<String, String>()
              factory.consumerFactory = consumerFactory()
              return factory
          }
      }
      ```

- Kafka Producer
  ```kotlin
  @Service
  class BookProducer(
      private val objectMapper: ObjectMapper,
      private val kafkaTemplate: KafkaTemplate<String, String>
  ) {

      @Value("topic.catalog")
      private val TOPIC_CATALOG = "TOPIC_CATALOG";

      fun sendBookEvent(bookChanged: BookChanged) {
          val message = objectMapper.writeValueAsString(bookChanged)
          kafkaTemplate.send(ProducerRecord(TOPIC_CATALOG, message)).get()
      }
  }
  ```

- Kafka Consumer
  ```kotlin
  @Service
  class BookConsumer(
      private val objectMapper: ObjectMapper,
      private val bookService: BookService
  ) {

      @KafkaListener(topics = arrayOf("TOPIC_BOOK"), groupId = "consumer_group")
      fun consume(kafkaMessage: String) {
          val bookStockEventDto = objectMapper.readValue(kafkaMessage, BookStockEventDto::class.java)
          bookService.processChangeBookState(bookStockEventDto.bookId, bookStockEventDto.bookStock)
      }
  }
  ```

#### Kafka Test

- @EmbeddedKafka
  - kafka test를 위한 가장 간단한 방법으로 @EmbeddedKafka 애노테이션을 붙이면 EmbeddedKafkaBroker를 테스트 메서드, 세팅등에 사용할 수 있게 해준다.
  - @ExtendWith(SpringExtension.class) 로 Spring Context 구성
  - @Autowired로 EmbeddedKafkaBroker를 주입받을 수 있다.



  

#### 출처
- https://blog.knoldus.com/testing-spring-embedded-kafka-consumer-and-producer/
- https://github.com/spring-projects/spring-kafka/blob/main/spring-kafka/src/test/kotlin/org/springframework/kafka/listener/EnableKafkaKotlinTests.kt
