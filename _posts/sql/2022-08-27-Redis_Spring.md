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

#### Repository 활용하기 - Secondary Indexes

- Redis는 key-value 형태로 저장되기 때문에 key 인 @id 를 제외한 다른 필드로 쿼리 하기 어려운 점이 있다.
- 여러 필드로 쿼리를 하기 위해서는 다른 방법을 사용해주어야 한다
- @Indexed
  - entity 내에 @Indexed 애노테이션으로 인덱스 생성
  ```java
  @RedisHash(value="cart", timeToLive = 300)
  public class Cart {
      @Id
      private String id;
      @Indexed
      private Member member;
      private Item item;
      private int count;
      private int totalPrice;
  ```

- Query by Example
  - QBE 를 통해 다이나믹 쿼리를 생성할 수 있으며 filed 명 contain 등의 쿼리를 작성할 필요가 없어진다
    - ex) JPA : List\<Person> findByContainsFirstName(String firstName)
  - 사용법
    - Probe: 쿼리에 필요한 값들이 저장된 엔티티
    - ExampleMatcher: 특정 필드에 일치 방법을 나타내는 객체 
    - Example: Probe, ExampleMatcher로 생성되며, 쿼리를 만드는 데 사용된다.
  - 예제
    ```java
    Person person = new Person();
    person.setFirstName("Test");
    
    Example<Person> example1 = Example.of(person);
    
    ExampleMatcher matcher = ExampleMatcher.matching()
        .withIgnorePaths("lastname")
        .withIncludeNullValues()
        .withStringMatcherEnding();
       
    Example<Person> example2 = Example.of(person, matcher);
    ```
    - withIgnorePaths: 지정한 프로퍼티를 무시하도록 설정
    - withIncludeNullValues: 지정한 프로퍼티의 null 값도 포함하도록 설정
    - withStringMatcherEnding: 지정한 프로퍼티의 suffix 가 일치하도록 설정

- QueryByExampleExcutor
  - QBE 를 실행할 수 있는 interface 제공
  - Repository 에 구현할 수 있도록 한다.
    ```java
    public interface QueryByExampleExecutor<T> {
        <S extends T> Optional<S> findOne(Example<S> example);
        <S extends T> Iterable<S> findAll(Example<S> example);
        <S extends T> Iterable<S> findAll(Example<S> example, Sort sort);
        <S extends T> Page<S> findAll(Example<S> example, Pageable pageable);
        <S extends T> long count(Example<S> example);
        <S extends T> boolean exists(Example<S> example);
        <S extends T, R> R findBy(Example<S> example, Function<FluentQuery.FetchableFluentQuery<S>, R> queryFunction);
    }
    
    public interface CartRepository extends CrudRepository<Cart, String>, QueryByExampleExecutor<Cart> {} 
    ```

- 출처
  - https://docs.spring.io/spring-data/redis/docs/2.1.9.RELEASE/reference/html/#redis.repositories.indexes
  - https://redis.io/docs/reference/patterns/indexes/

#### Cache로 Redis 

- API Cache
  - 캐시란 한번 처리한 데이터를 임시로 저장소에 저장하는 것으로, 해당 임시 데이터를 동일하거나 유사 요청이 왔을 경우 저장소에서 바로 읽어와 응답하여 성능 및 응답속도 향상을 위한 기술
  - 요청이 왔을 경우 연산을 수행하거나 DB의 데이터를 불러와 response 하는 데, 캐시를 이용해 특정 요청을 저장소에 임시로 저장해두었다가 이후 동일한 응답을 해도 되는 요청이 왔을 경우 별도 연산없이 바로 저장소에서 데이터를 가지고 응답하여 성능/응답 시간을 개선할 수 있다

- dependency 구성
  - JPA / H2 Database / Redis / Spring web / Spring cache 로 구성

- 설정
  ```yml
  spring:
    application:
      name: redis-example
    cache:
      type: redis
    redis:
      host: localhost
      port: 6379
    h2:
      console:
        enabled: true
        path: /h2-console
      datasource:
        url: jdbc:h2:tcp://localhost/~/test
        username: user
        password:
        driver-class-name: org.h2.Driver
    jpa:
      hibernate:
        ddl-auto: create
      properties:
        hibernate:
          format_sql: true
  ```
  
- RedisConfig
  - @EnableCaching 애노테이션 추가
  - CacheManager 설정
    ```java
    @Bean
    public CacheManager redisCacheManager() {
        RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig()
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));

        RedisCacheManager redisCacheManager = RedisCacheManager.RedisCacheManagerBuilder.fromConnectionFactory(redisConnectionFactory()).cacheDefaults(redisCacheConfiguration).build();
        return redisCacheManager;
    }
    ```

- @Cacheable, @CachePut, @CacheEvict 사용하여 구성
  - 참고: https://devhtak.github.io/spring/2021/10/12/Spring_Cache.html
  - Entity
    ```java
    @Entity @Data
    public class Item {

        @Id
        @GeneratedValue
        private Long id;

        private String name;

        private int price;
    }
    ```
  - Service
    ```java
    @Service
    @Transactional
    public class ItemService {

        private final ItemJpaRepository itemRepository;

        @Autowired
        public ItemService(ItemJpaRepository itemRepository) {
            this.itemRepository = itemRepository;
        }


        public Item getById(Long itemId) {
            return itemRepository.findById(itemId).orElseThrow(IllegalArgumentException::new);
        }

        public Item save(ItemDto itemDto) {
            Item item = new Item();
            item.setName(itemDto.getName());
            item.setPrice(itemDto.getPrice());

            return itemRepository.save(item);
        }

        public Item update(Long itemId, ItemDto itemDto) {
            Item item = itemRepository.findById(itemId)
                    .orElseThrow(IllegalArgumentException::new);

            item.setName(itemDto.getName());
            item.setPrice(itemDto.getPrice());

            return item;
        }
    }
    ```
  - Controller
    ```java
    @RestController
    public class ItemController {

        private final ItemService itemService;

        @Autowired
        public ItemController(ItemService itemService) {
            this.itemService = itemService;
        }

        @GetMapping("/items/{itemId}")
        @Cacheable(value = "Items", key="#itemId")
        public ResponseEntity<Item> getItemById(@PathVariable Long itemId) {
            Item item = itemService.getById(itemId);

            return ResponseEntity.ok(item);
        }

        @PostMapping("/items")
        public ResponseEntity<Item> save(@RequestBody ItemDto itemDto) {
            Item item = itemService.save(itemDto);

            return ResponseEntity.status(HttpStatus.CREATED).body(item);
        }

        @PutMapping("/items/{itemId}")
        @CachePut(value = "Items", key="#itemId")
        public ResponseEntity<Item> updateItemById(@PathVariable Long itemId, @RequestBody ItemDto itemDto) {
            Item item = itemService.update(itemId, itemDto);

            return ResponseEntity.ok(item);
        }
    }
    ```
    - Controller 단에 응답을 저장하여, 결과값까지 캐시에 저장되도록 하였다
  - Redis 에서 확인하기
    ```
    $ get Items:1
    "{\"@class\":\"org.springframework.http.ResponseEntity\",\"headers\":{\"@class\":\"org.springframework.http.ReadOnlyHttpHeaders\"},\"body\":{\"@class\":\"com.example.item.Item\",\"id\":1,\"name\":\"TEST1\",\"price\":1000},\"statusCodeValue\":200,\"statusCode\":[\"org.springframework.http.HttpStatus\",\"OK\"]}"
    ```

#### 출처
- 블로그
  - https://bcp0109.tistory.com/328
  - https://jronin.tistory.com/m/126
  - https://pearlluck.tistory.com/m/727

- Spring Redis Document
  - https://docs.spring.io/spring-data/data-redis/docs/current/reference/html/#dependencies
