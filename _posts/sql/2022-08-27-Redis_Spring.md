---
layout: post
title: Redis with Spring
summary: Redis + Spring
author: devhtak
date: '2022-08-27 21:41:00 +0900'
category: No SQL
---

#### Redis Client - Java(Spring Boot)

- spring-data-redis 라이브러리를 활용하여 Redis 를 사용할 수 있다.
- 크게 Lettuce, Jedis가 존재한다
  - Jedis에 경우 멀티쓰레드 불안정, Pool 한계 등 여러 가지 단점이 존재
  - Lettuce 는 Netty 기반의 비동기 지원 가능한 장점 존재로 현재는 Lettuce 를 많이 쓰는 추세
  - Spring Boot 2.0 부터는 Lettuce 사용

- Spring 설정
  - 의존성 추가
    ```java
    implementation 'org.springframework.boot:spring-boot-starter-data-redis'
    ```
  - application.yml 에 접속 정보 설정
    ```yml
    spring:
      redis:
        host: localhost
        port: 6379
    ```
    - 기본 정보를 가져와 RedisConnectionFactory, RedisTemplate 으로 설정해줄 수 있다
      ```java
      @Configuration
      @EnableRedisRepositories
      public class RedisConfig {
          @Value("${spring.redis.host}")
          private String host;

          @Value("${spring.redis.port}")
          private int port;

          @Bean
          public RedisConnectionFactory redisConnectionFactory() {
              return new LettuceConnectionFactory(host, port);
          }

          /**
           * RedisTemplate 세팅
           * RedisTemplate 을 사용하면, Redis CLI 명령어를 사용하여 데이터 셋에 맞게 구현할 수 있다
           * @return
           */
          @Bean
          public RedisTemplate<String, Object> redisTemplate() {
              RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
              redisTemplate.setConnectionFactory(redisConnectionFactory());
              // Key: String
              redisTemplate.setKeySerializer(new StringRedisSerializer());
              // Value: 직렬화에 사용할 Object 사용하기
              redisTemplate.setValueSerializer(new Jackson2JsonRedisSerializer<>(String.class));
              return redisTemplate;
          }
      }
      ```

- RedisTemplate
  - 종류
    ```java
    RedisTemplate redisTemplate;
    StringRedisTemplate stringRedisTemplate;
    ReactiveRedisTemplate reactiveRedisTemplate;
    StringReactiveRedisTemplate stringReactiveRedisTemplate;
    ```
    - RedisTemplate 과 StringRedisTemplate의 차이는 직렬화 방식
      - JdkSerializationRedisSerializer, StringRedisSerializer
      - redisTemplate.setKeySerializer / redisTemplate.setValueSerializer 을 활용하여 key/value 직렬화를 커스텀할 수 있다      
    - ReactiveRedisTemplate과 RedisTemplate의 차이는 blocking vs non-blocking 차이이다.
      - Reactive를 사용하기 위해서는 org.springframework.boot:spring-boot-starter-data-redis-reactive 로 의존성 추가 필요
    
#### Redis Repository 사용 방법과 RedisTemplate 사용 방법

- Spring Data Redis 라이브러리는 2가지 접근방식을 제공한다
  - Redis Repository 를 이용한 방식
  - RedisTemplate 을 이용한 방식

- Redis Repository
  - Config: @EnableRedisRepositories annotation 및 RedisConnectionFactory / RedisTemplate 빈 정의
  - @RedisHash annotation 을 통해 entity 정의
    ```java
    @RedisHash("people")
    public class Person {
        @Id String id;
        String firstname;
        String lastname;
        Address address;
    }
    ```
    - @RedisHash annotation 을 통하여 해당 value를 hash 타입으로 redis server 에 저장/조회할 수 있도록 한다
      - value(keyspace 값) / timeToLive(만료시간 seconds 단위) 제공
    - @Id
      - org.springframework.data.annotation.Id
      - keyspace:id 로 key 정의됨
    - @Index
      - findBy not working: https://stackoverflow.com/questions/53121627/unable-to-get-result-from-the-redis-using-crud-repository-in-spring-boot
  - spring data에서 제공하는 CrudRepository 를 통한 Repository 정의
    ```java
    public interface PersonRepository extends CrudRepository<Person, String> {
        
    }
    ```
    - CrudRepository 는 Redis 가 아닌 spring-data 에서 제공
    - findAll() 에 경우 Iterator을 반환하기 때문에 List 로 override 해주는 것이 편하다
    
- RedisTemplate
  - RedisTemplate 은 redis command를 사용하기 위해 value data type에 따른 추상화 객체를 제공한다
    ```java
    ValueOperations<String, String> valueOperations = redisTemplate.opsForValue(); // String type으로 serializer/deserializer 해주는 인터페이스
    ListOperations<String, String> listOperations = redisTemplate.opsForList(); // List type으로 serializer/deserializer 해주는 인터페이스
    SetOperations<String, String> setOperations = redisTemplate.opsForSet(); // Set type으로 serializer/deserializer 해주는 인터페이스
    ZSetOperations<String, String> zSetOperations = redisTemplate.opsForZSet(); // Sorted Set type으로 serializer/deserializer 해주는 인터페이스
    HashOperations<String, Object, Object> hashOperations = redisTemplate.opsForHash(); // Hash type으로 serializer/deserializer 해주는 인터페이스
    ```
    
- 예제
  - https://github.com/devHTak/redis-spring-example 참고

#### Cache로 Redis 

#### 출처
- 블로그
  - https://bcp0109.tistory.com/328
  - https://jronin.tistory.com/m/126
  - https://pearlluck.tistory.com/m/727

- Spring Redis Document
  - https://docs.spring.io/spring-data/data-redis/docs/current/reference/html/#dependencies
