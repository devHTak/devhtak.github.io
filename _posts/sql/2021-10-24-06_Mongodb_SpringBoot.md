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
    - reactive mongo db logging
      ```
      logging.level.org.springframework.data.mongodb.core.ReactiveMongoTemplate=DEBUG
      ```
      
#### MongoDB 의 \_id

- MongoDB의 \_id 속성
  - MongoDB는 Collection별로 document를 저장한다. (db의 table의 1 row를 저장하는 것과 비슷)
  - document별로 \_id 속성이 있는데 이는 Collection에 저장된 여러 document에 대해 유일함을 식별하기 위한 기본 key이다.
  - 별다른 선언이 없으면 ObjectId type을 사용한다.

- ObjectId
  - ObjectId는 BSON type으로 12 byte로 이루어져 있다.
  - ObjectId layout
    
    ![image](https://user-images.githubusercontent.com/42403023/141745338-4577c785-3a0a-488d-93d6-645897677396.png)
    
    - MongoDB는 _id를 client에서 만든다. ( 분산 database 환경 )
    - 동일 time에 동일 machine에서 동일 process id에서 만드는 경우 0 ~ 8까지 동일한 값이 나오지만 마지막 3 byte inc가 중복이 되는 것을 막는다. 

- @Id 애노테이션
  - @Id로 지정된 field가 있으면 MongoDB의 \_id로 맵핑된다.
  - 만약 @Id로 지정된 field가 없다면 id란 이름의 field가 MongoDB의 _id로 맵핑된다.

#### Spring POJO 객체에 Shard Key 지정

- 아래 버전 이상
  - MongoDB Version 4.2.x
  - MongoDB Driver 4.2.3
  - Spring FW 5.3.7
  - Spring Boot 2.5.0
  - Spring Data MongoDB 3.2.1

- Spring Data POJO Class
  - @Sharded Annotation 옵션에 shardKey 지정 필요
    ```java
    @Data
    @Sharded(shardKey={"name"})
    public class Product {
        @Id private ObjectId _id;
    }
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

#### MongoRepository 사용 시 SELECT 에 다양한 방법

- 메소드 명으로 정의하기
  ```java
  // db.city.find({'name': '서울', 'year': 2021})
  List<City> findByNameAndYear(String name, int year); 
  
  // db.city.find({'createdAt' : {'$gte': '2021-12-01 00:00:00'}, 'city': '서울'})
  List<CovidStatus> findByCreatedAtGreaterThanAndCity(LocalDateTime createdAt, String city);
  ```
  - 해당 방식과 같이 메소드 명을 컬럼명, 조건, AND/OR 조건등으로 연결할 수 있다.

- @Query Annotation
  ```
  @Query("{'name': :#{#name}, 'year': :#{#year} }")
  List<City> findCityByNameAndYear(@Param("name") String name, @Param("year") int year);
  
  @Query(values="{'name': :#{#name}, 'year': :#{#year} }", fields= {"name", "year", "number"}
  List<City> findFieldsByNameAndYear(@Param("name") String name, @Param("year") int year);
  ```
  - @Query 애노테이션을 활용하여 필요한 쿼리, 조회하고자 하는 필드 등을 지저할 수 있다.
  - 이외
    - count: 쿼리의 개수를 리턴하도록 설정(true/false)
    - exists: 쿼리에 대한 데이터가 존재하는 지 설정(true/false)
    - sort: sorting 기준 설정 (sort = "{title: 1}") or (sort= "{title: -1}")
    - delete: 조회한 데이터를 삭제할지 설정. true로 설정할 시 쿼리 일치 데이터를 삭제하고 삭제된 행 수를 반환합니다.

- 페이징 처리
  ```java
  @Query("{'author' : ?0}")
  Page<Book> findBy(String author, Pageable pageable);
  ```
  - Pageable 을 넘겨주면 paging 처리가 가능하다

#### 관계 정의

- Mongodb는 @OneToMany와 같은 형태의 선언이 필요 없다.
- RDB table <-> java object 간 데이터 맵핑을 위한 설정이 필요 없는 이유는 하위 관계까지 java object 형태 그대로 Collection에 저장할 수 있기 때문이다.
- 다만 Collection에 저장할지 혹은 개별 Collection을 참조할지에 대해 선언을 해주는 @DBRef annotation을 지원한다.

- @DBRef
  - DBRef는 참조할 도큐먼트의 \_id 필드의 값과 옵션으로서의 데이터베이스 이름을 이용하여 어느 하나의 도큐먼트가 다른 도큐먼트를 참조하는 것이다.
  - 옵션
    - $ref: 참조할 도큐먼트가 존재하는 컬렉션의 이름.
    - $id: 참조된 도큐먼트 내 _id 필드의 값.
    - $db: 참조할 도큐먼트가 존재하는 데이터베이스의 이름.
    - 예시
      ```
      > db.users.insert({"_id" : "mike", "display_name" : "Mike D"})
      > db.users.insert({"_id" : "kristina", "display_name" : "Kristina C"})
      
      > db.notes.insert({"_id" : 5, "author" : "mike", "text" : "MongoDB is fun!"})
      > db.notes.insert({"_id" : 20, "author" : "kristina", "text" : "... and DBRefs are easy, too", "references": [{"$ref" : "users", "$id" : "mike"}, {"$ref" : "notes", "$id" : 5}]})

      > db.users.find().pretty()
      { "_id" : "mike", "display_name" : "Mike D" }
      { "_id" : "kristina", "display_name" : "Kristina C" }

      > db.notes.find().pretty()
      { "_id" : 5, "author" : "mike", "text" : "MongoDB is fun!" }
      {
          "_id" : 20,
          "author" : "kristina",
          "text" : "... and DBRefs are easy, too",
          "references" : [
              DBRef("users", "mike"),
              DBRef("notes", 5)
          ]
      }
      ```

#### 출처

- https://data-make.tistory.com/679
- https://jsonobject.tistory.com/559
- https://gofnrk.tistory.com/38
- https://luvstudy.tistory.com/62
