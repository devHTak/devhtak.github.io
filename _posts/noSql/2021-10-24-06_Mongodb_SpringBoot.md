---
layout: post
title: Spring Boot 와 MonboDB 연동
summary: NoSql - Mongodb
author: devhtak
date: '2021-10-24 21:41:00 +0900'
category: No SQL
---

#### Spring Boot에 MonboDB 연동

- dependency 추가
  - Maven pom.xml
    ```
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-mongodb</artifactId>
    </dependency>
    ```
  - Gradle build.gradle
    ```
    implementation 'org.springframework.boot:spring-boot-starter-data-mongodb'
    ```

- 설정
  - mongodb 접속 정보 application.yml
    ```
    spring:
      data:
        mongodb:
          host: localhost
          port: 27017
          database: tutorial
          # url: mongodb://localhost:27017/tutorial
    ```
  - mongodb 접속 정보 Configuration 등록
    ```java
    @Configuration
    public class MongoConfig extends AbstractMongoClientConfiguration {

        @Override
        public MongoClient mongoClient() {
            // 연결정보 세팅
            MongoClientSettings mongoClientSettings = MongoClientSettings.builder()
                    .applyConnectionString(new ConnectionString("mongodb://localhost:27017/test"))
                    .build();

            return MongoClients.create(mongoClientSettings);
        }

        @Override
        protected String getDatabaseName() {
            // 사용할 db name
            return "test";
        }

        @Bean
        public MongoTransactionManager transactionManager(MongoDatabaseFactory mongoDatabaseFactory) {
            // transaction manager
            return new MongoTransactionManager(mongoDatabaseFactory);
        }
    }
    ```
    - @Transactional 사용에 요구되는 MongoTransactionManager 빈 설정
    - mongoClient(): ConnectionString 을 통해 접속 정보 설정
    - getDatabaseName(): 연결할 데이터베이스 이름 입력
    
  - query logging
    ```
    logging:
      level:
        org:
          springframework:
            data:
              mongodb:
                core:
                  MongoTemplate: DEBUG
    ```
    
#### Spring Boot 에서 MongoDB 접근 API

- 예제 Collection 생성
  ```java
  @Document("events")
  @Getter @Setter
  public class Event {

      @Id
      private String id;

      private String title;

      private String image;
  }
  ```
  - @Document 가 MongoDB에서 Collection이 된다
  
- CASE1) MongoTemplate
  - MongoTemplate 을 빈으로 입력받아 사용
    ```java
    @Service
    public class EventTemplateService {

        @Autowired
        private MongoTemplate mongoTemplate;

        public Event getEvent(String id) {
            Event event = mongoTemplate.findById(id, Event.class);
            return Optional.ofNullable(event).orElseThrow(IllegalArgumentException::new);
        }

        public Page<Event> getEventsByTitle(String title, Pageable pageable) {
            Query query = new Query().addCriteria(Criteria.where("title").is(title));
            if(ObjectUtils.isEmpty(pageable)) {
                query.with(pageable);
            }
            query.with(Sort.by(Sort.Direction.DESC, "id");

            List<Event> events = mongoTemplate.find(query, Event.class);
            return PageableExecutionUtils.getPage(events, pageable, () -> {
                return mongoTemplate.count(Query.of(query).limit(-1).skip(-1), Event.class);            
            });
        }

        public Event saveEvent(Event event) {
            return mongoTemplate.insert(event);
        }
    }
    ```
    - insert vs save
      - insert: \_id가 없으면 저장하고 \_id가 있으면 오류가 발생
      - save: \_id가 없으면 insert를 진행하고 \_id가 없으면 그 document를 수정
    - update vs save
      - update: 특정 필드만 수정
      - save: 필드 단위로 수정하지 않고 데이터를 덮어쓰므로 이전 데이터는 사라진다

- CASE2) MongoRepository
  - JPA 와 유사하게 MongoRepository 를 구현하여 사용할 수 있다.
    ```java
    @Service
    public class EventRepoService {

        @Autowired
        private EventRepository eventRepository;

        public Event findEventById(String id) {
            return eventRepository.findById(id).orElseThrow(IllegalArgumentException::new);
        }

        public Page<Event> findEventListByTitle(String title, Pageable pageable) {
            return eventRepository.findByTitle(title, pageable);
        }

        public Event insertEvent(Event event) {
            return eventRepository.insert(event);
        }
    }
    ```
    
#### 출처

- https://data-make.tistory.com/679
- https://jsonobject.tistory.com/559
- https://gofnrk.tistory.com/38
