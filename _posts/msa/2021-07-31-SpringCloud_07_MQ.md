---
layout: post
title: Spring Cloud - MSA 데이터 동기화를 위한 Kafka 활용
summary: Spring Cloud
author: devhtak
date: '2021-07-31 22:41:00 +0900'
category: msa
---

#### Apache Kafka

- Apache Software Foundation의 scalar 언어로 된 오픈 소스 메시지 브로커 프로젝트
- 실시간 데이터 피드를 관리하기 위해 통일된 높은 처리량, 낮은 지연 시간을 지닌 플랫폼 제공

- Kafka 이전에 시스템 아키텍처
  - End-to-End 연결 방식의 아키텍처
  - 데이터 연동의 복잡성 증가(HW, 운영체제, 장애 등)
  - 서로 다른 데이터 Pipeline 연결 구조
  - 확장이 어려운 구조

- Kafka 특징
  - Kafka를 통해 모든 시스템으로 데이터를 실시간으로 전송하여 처리할 수 있는 시스템
  - 데이터가 많아지더라도 확장이 용이한 시스템이 되었다.
  - Producer/Consumer 분리
  - 메시지를 여러 Consumer에게 허용
  - 높은 처리량을 위한 메시지
  - Scale-out 가능, Eco-system

- Kafka Broker (Server)
  
  ![image](https://user-images.githubusercontent.com/42403023/127726404-68d4f420-1c61-4a1d-96d5-cb9a64fea948.png)
  
  - 이미지 출처: https://data-flair.training/blogs/kafka-broker/
  
  - 실행된 Kafka 애플리케이션 서버
  - 3대 이상의 Broker Cluster 구성을 권장
  - Zookeeper 연동
    - 역할: 메타데이터(Broker ID, Controller ID 등) 저장
    - Controller 정보 저장
  - n개 Broker 중 1대는 Controller 기능 수행
    - Controller 역할
      - 각 Broker에게 담당 파티션 할당 수행
      - Broker 정상 동작 모니터링 관리

#### Kafka 설치

- https://kafka.apache.org/downloads 에서 원하는 버전의 Binary 다운로드
- 압축 해제 
  ```
  $ tar xvf kafka_2.13-2.7.0.tgz
  ```
  
- 폴더 확인
  - config
    - properties로 설정 파일들이 저장되어 있다.
    - zookeper.properties, server.properties 등 zookeeper, server, producer, consumer 등에 대한 설정을 제공한다.

  - bin
    - linux, max, window(sh, batch) 등에서 사용할 수 있는 실행 파일을 제공한다
 
 #### Producer와 Consumer
 
 - Kafka client
   - Kafka 와 데이터를 주고받기 위해 사용하는 Java Library     
   - Producer, Consumer, Admin, Stream 등 Kafak 관련 API 제공
   - 다양한 3rd party library 종재
     - 참고: https://cwiki.apache.org/confluence/display/KAFKA/Clients
 
 
- 서버 기동
  - Zookeeper 및 Kafka 서버 기동
    - $KAFKA_HOME/bin/zookeeper-server-start.sh $KAFKA_HOME/config/zookeeper.properties
    - $KAFKA_HOME/bin/kafka-server-start.sh $KAFKA_HOME/config/server.properties
    
  - Topic 생성
    - $KAFKA_HOME/bin/kafka-topics.sh --create --topic quickstart-events --bootstrap-server localhost:9092 --partitions 1
    - Producer에서 메세지를 보내면 Topic에 메세지가 저장이 되며 Consumer가 Topic에 있는 메세지를 읽어올 수 있다.
    - --create --topic으로 토픽을 생성할 수 있다(quickstart-events는 토픽이름) 
    
  - Topic 목록 확인
    - $KAFKA_HOME/bin/kafka-topics.sh --bootstrap-server localhost:9092 --list

  - Topic 정보 확인
    - $KAFKA_HOME/bin/kafka-topics.sh --describe --topic quickstart-events --bootstrap-server localhost:9092

  - kafka-console-producer, kafka-console-consumer 을 통해서 콘솔상에서 테스트가 가능하다.
    - $KAFKA_HOME/bin/kafka-console-producer.sh --broker-list localhost:9092 --topic quickstart-events

      ![image](https://user-images.githubusercontent.com/42403023/127727761-0bf522ce-6505-4bb3-aa7d-0677a7bbaa86.png)

    - $KAFKA_HOME/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic quickstart-events --from-begining

      ![image](https://user-images.githubusercontent.com/42403023/127727768-ae3b11b5-4341-45fe-9456-cea1d238b7e0.png)

    - producer에서 작성하면 consumer에게 전달된다
    - from-begining 은 topic에 저장되어 있는 처음 메세지부터 다 받을 수 있다.
  
 #### Kafka Connect
 
 - Kafka Connect를 통해 Data를 Import/Export 가능
 - 코드 없이 Configuration으로 데이터를 이동
 - Standalone mode, Distribution mode 지원
   - RESTful API 지원
   - stream 또는 batch 형태로 데이터 전송 가능
   - 커스텀 Connector를 통해 다양한 Plubin 제공(File, AWS S3, DB etc ..)

- Source System (Hive, jdbc ..) -> Kafka Connect Source -> Kafka Cluster -> Kafka Connect Sink -> Target System(AWS S3, ...)
- Kafka Connector 설치
  ```
  $ curl -O http://packages.confluent.io/archive/5.5/confluent-community-5.5.2-2.12.tar.gz
  $ curl -O http://packages.confluent.io/archive/6.1/confluent-community-6.1.0.tar.gz
  $ tar xvf confluent-community-6.1.0.tar.gz
  $ cd  $KAFKA_CONNECT_HOME
  ```
- Kafka Connect 설정(기본으로 사용)
  ```
  $ KAFKA_HOME/config/connect-distriuted.properties
  ```
- Kafka Connect 실행
  ```
  ./bin/windows/connect-distributed.bat ./etc/kafka/connect-distributed.properties
  ```
- Kafka Connect 실행 전에 Topic 목록 확인
  ```
  $ ./bin/windows/kafka-topics.bat --bootstrap-server localhost:9092 --list
  __consumer_offsets
  connect-configs
  connect-offsets
  connect-status
  ```

- JDBC Connector 설치
  - https://docs.confluent.io/5.5.1/connect/kafka-connect-jdbc/index.html
    - download and extract the ZIP file -> confluentinc-kafka-connect-jdbc-10.2.1.zip 다운로드
  - etc/kafka/connect-distributed.properties 파일 마지막에 아래 plugin 정보 추가
    - plugin.path=[confluentinc-kafka-connect-jdbc-10.2.1 폴더]
      ```
      plubin.path=\D:\\Work\\Excutions\\confluentinc-kafka-connect-jdbc-10.2.1\\lib
      ```
      - window에 경우 \을 사용하기 위해 \를 하나 더 붙인다
  - JdbcSourceConnector에서 MariaDB 사용하기 위해 mariadb 드라이버 복사
    - ./share/java/kafka/ 폴더에 mariadb-java-client-2.7.3.jar  파일 복사
    - ${USER.HOMW}\.m2\repository\org\mariadb\jdbc\mariadb-java-client\2.7.3 폴더에서 maraidb-java-client-2.7.3.jar 파일이 있다

- Kafka Source Connect 사용
  - Kafka Connect API 활용하여 Source Connector 등록
    ```
    POST http://localhost:8083/connectors
    Body:  {
            "name" : "my-source-connect",
            "config" : {
                "connector.class" : "io.confluent.connect.jdbc.JdbcSourceConnector",
                "connection.url":"jdbc:mysql://localhost:3306/mydb",
                "connection.user":"root",
                "connection.password":"test1357",
                "mode": "incrementing",
                "incrementing.column.name" : "id",
                "table.whitelist":"users",
                "topic.prefix" : "my_topic_",
                "tasks.max" : "1"}}
    ```
    - mode: incrementing -> 데이터가 등록될 때 자동으로 증가시키는 모드로 설정
    - incrementing.column.name: id -> 자동으로 증가하는 컬럼은 id
    - table.whitelist: "users" -> whitelist는 MariaDB에 변경 사항이 생기면 topic에 저장하게 되는 데, 해당 테이블을 설정
    - topic.prefix: 'my_topic_' -> 저장할 topic은 my_topic_으로 시작한다.
    
  - Kafka Connect 목록 확인
    ```
    GET 192.168.56.2:8083/connectors
    [
      "my-source-connect"
    ]
    ```
    - vm에 kafka를 설치해였기 때문에 8083 방화벽을 오픈하여 연결하였다.
    - my-source-connect가 생성된 것을 확인할 수 있다.
  - Kafka Connect 확인
    ```
    GET http://localhost:8083/connectors/my-source-connect/status
    {
      "name": "my-source-connect",
      "connector": {
        "state": "RUNNING",
        "worker_id": "127.0.1.1:8083"
      },
      "tasks": [
        {
            "id": 0,
            "state": "RUNNING",
            "worker_id": "127.0.1.1:8083"
        }
      ],
      "type": "source"
    }
    ```
  
- Kafka Sink Connect 사용
  ```
    POST http://localhost:8083/connectors
    Body:  {
            "name" : "my-sink-connect",
            "config" : {
                "connector.class" : "io.confluent.connect.jdbc.JdbcSinkConnector",
                "connection.url":"jdbc:mysql://localhost:3306/mydb",
                "connection.user":"root",
                "connection.password":"test1357",
                "auto.create":"true",
                "auto.evolve": "true",
		"delete.enabled": "false",
		"tasks.max": "1",
		"topics": "my_topic_users"}}
    response-body: 
      {
      "name": "my-sink-connect",
      "config": {
          "connector.class": "io.confluent.connect.jdbc.JdbcSinkConnector",
          "connection.url": "jdbc:mysql://localhost:3306/mydb",
          "connection.user": "root",
          "connection.password": "test1357",
          "auto.create": "true",
          "auto.evolve": "true",
          "delete.enabled": "false",
          "tasks.max": "1",
          "topics": "my_topic_users",
          "name": "my-sink-connect"
      },
      "tasks": [],
      "type": "sink"
      }
    ```
    - connector를 사용하여 Sink 지정
    - "auto.create": "true" -> 토픽과 같은 테이블을 생성한다는 의미로 테이블 구조는 토픽이 갖고 있는 구조 그대로 사용하여 생성한다.
    - "topics": "my-topic_users" -> 구독하고자 하는 
  - Kafka Connect 목록 확인
    ```
    GET 192.168.56.2:8083/connectors
    Response Body:
    [
      "my-sink-connect",
      "my-source-connect"
    ]
    ```
    
  - Kafka Connect 확인
    ```
    GET 192.168.56.2:8083/connectors/my-sink-connect/status
    Response body: 
    {
    "name": "my-sink-connect",
    "connector": {
        "state": "RUNNING",
        "worker_id": "127.0.1.1:8083"
    },
    "tasks": [
        {
            "id": 0,
            "state": "RUNNING",
            "worker_id": "127.0.1.1:8083"
        }
    ],
    "type": "sink"
}
    ```
- 예제
  - mariaDB 설치(docker 활용)
    - mariadb 설치
      ```
      $ docker pull mariadb
      $ docker run --name mariadb -p 3306:3306 -e MYSQL_ROOT_PASSWORD=test1357 --cap-add=NET_ADMIN mariadb 
      ```
      - port 는 3306, password는 test1357로 세팅했다
      - --cap-add=NET_ADMIN을 통해 외부 클라이언트에서 docker로 접속할 수 있도록 했다 (3306 포트 방화벽 오픈)
    - mariadb 실행
      ```
      $ docker exec -i -t mariadb bash
      root@....:/# mysql -uroot -ptest1357
      MariaDB [(none)]> show databases;
      MariaDB [(none)]> create database mydb;
      MariaDB [(none)]> use mydb;
      ```
      - show databases; 를 통해 사용가능한 Database를 확인할 수 있다.
      - create database mydb: mydb란 신규 데이터베이스 생성
      - use mydb: mydb 사용
    
  - pom.xml에 mariadb dependency 추가
    ```java
    <dependency>
      <groupId>org.mariadb.jdbc</groupId>
      <artifactId>mariadb-java-client</artifactId>
    </dependency>
    ```
  - h2-console에 들어가 테이블 생성
    - url: org.mariadb.jdbc.Driver
    - driverClassName: jdbc:mariadb://192.168.56.2:3306/mydb
    - username: root
    - password: test1357
    
    ![image](https://user-images.githubusercontent.com/42403023/127762157-a1481a65-ddda-414c-b8aa-0a76c088f47f.png)
      
      - vm에 docker로 mariadb를 설치했기 때문에 docker ip인 192.168.56.2로 설정하였다.
      - 이미지출처: https://empty-cloud.tistory.com/84
      
    ```
    create table users( 
      id int auto_increment primary key,
      name varchar(20),
      pwd varchar(20),
      created_at datetime default NOW()
    );
    ````
    
  - Kafka Source Connect 추가(MariaDB)
    - 직접 명령어를 주어 추가할 수 있다.
    - 또는 POST /connectors로 
    ```
    $ echo '{
      > "name" : "my-source-connect",
      > "config" : {
      >    "connector.class" : "io.confluent.connect.jdbc.JdbcSourceConnector",
      >    "connection.url":"jdbc:mysql://localhost:3306/mydb",
      >    "connection.user":"root",
      >    "connection.password":"test1357",
      >    "mode": "incrementing",
      >    "incrementing.column.name" : "id",
      >    "table.whitelist":"users",
      >    "topic.prefix" : "my_topic_",
      >    "tasks.max" : "1"
      >    }
      >}' | curl -X POST -d @- http://localhost:8083/connectors --header "content-Type:application/json"
    {"name":"my-source-connect",
    "config":{
        "connector.class":"io.confluent.connect.jdbc.JdbcSourceConnector",
        "connection.url":"jdbc:mysql://localhost:3306/mydb",
        "connection.user":"root",
        "connection.password":"test1357",
        "mode":"incrementing",
        "incrementing.column.name":"id",
        "table.whitelist":"users",
        "topic.prefix":"my_topic_",
        "tasks.max":"1",
        "name":"my-source-connect"},
    "tasks":[],"type":"source"}
    ```
    
  - MariaDB에 데이터 추가
    ```
    MariaDB [mydb]> insert into users(name, pwd) values('test', 'test1234');
    Query OK, 1 row affected (0.010 sec)
    MariaDB [mydb]> select * from users;
    +----+------+----------+---------------------+
    | id | name | pwd      | created_at          |
    +----+------+----------+---------------------+
    |  1 | test | test1234 | 2021-08-01 11:04:17 |
    +----+------+----------+---------------------+
    1 row in set (0.002 sec)
    ```
  - my_topic_users에서 확인
    - 토픽 확인
      ```
      $ ./bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
      __consumer_offsets
      connect-configs
      connect-offsets
      connect-status
      my_topic_users
      ```
      - my_topic_users가 생성되었다.
    - kafka-console-consumer에서 입력 확인
      ```
      $ ./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic my_topic_users --from-beginning
      {"schema":
        {"type":"struct",
        "fields":[
	    {"type":"int32","optional":false,"field":"id"},
	    {"type":"string","optional":true,"field":"name"},
	    {"type":"string","optional":true,"field":"pwd"}, 
	    {"type":"int64","optional":true,"name":"org.apache.kafka.connect.data.Timestamp","version":1,field":"created_at"}],
	  "optional":false,
	  "name":"users"
        },
        "payload":{"id":1,"name":"test","pwd":"test1234","created_at":1627815857000}}
      ```

  - Kafka sink connect 생성
    ```
    $ echo '{
      > "name" : "my-sink-connect",
      > "config" : {
      >    "connector.class" : "io.confluent.connect.jdbc.JdbcSinkConnector",
      >    "connection.url":"jdbc:mysql://localhost:3306/mydb",
      >    "connection.user":"root",
      >    "connection.password":"test1357",
      >    "auto.create": "true",
      >    "auto.evolve": "true",
      >    "delete.enabled": "false",
      >    "tasks.max": "1",
      >    "topics": "my_topic_users"
      >    }
      >}' | curl -X POST -d @- http://localhost:8083/connectors --header "content-Type:application/json"
    ```
    - my_topic_users란 테이블이 생성된 것을 확인할 수 있으며 users에 저장한 데이터가 입력된 것을 확인할 수 있다.
  
  - Kafka Producer를 이용하여 Kafka Topic에 데이터 직접 전송
    - Kafka-console-producer에서 데이터 전송 -> topic에 추가 -> mariadb에 추가
      ```
      $ ./bin/kafka-console-producer.sh --broker-list localhost:9092 --topic my_topic_users
      > {"schema":
        {"type":"struct",
        "fields":[
	    {"type":"int32","optional":false,"field":"id"},
	    {"type":"string","optional":true,"field":"name"},
	    {"type":"string","optional":true,"field":"pwd"}],
	  "optional":false,
	  "name":"users"
        },
        "payload":{"id":2,"name":"test2","pwd":"test2234"}}
      ```
      - schema: 전달하고자 하는 데이터 구조
      - payload: 전달하는 데이터

#### Service에 적용하기

- 예제: Orders -> Catalogs
  - Orders Service에 요청된 주문의 수량 정보를 Catalogs Service에 반영
  - Orders Service에서 Kafka Topic으로 메시지 전송 -> Producer
  - Catalogs Service에서 Kafka Topic에 전송된 메시지 취득 -> Consumer

  - Catalog Service (Consumer)
    - dependency 추가
      ```
      <dependency>
        <groupId>org.springframework.kafka</groupId>
        <artifactId>spring-kafka</artifactId>
      </dependency>      
      ```
      
    - KafkaConsumerConfig.java 
      - Topic 구독 후 변경사항을 이벤트 리스너로 사용
        ```java
        @EnableKafka
        @Configuration
        public class KafkaConsumerConfig {
          /* 카프카 접속 정보 저장 빈*/
          @Bean
          public ConsumerFactory<String, String> consumerFactory() {
            Map<String, Object> properties = new HashMap<>();
            properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "192.168.56.2:9092");
            properties.put(ConsumerConfig.GROUP_ID_CONFIG, "consumerGroupId"); // 여러 토픽을 그룹으로 지정하여 사용할 수 있다.
            properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
            properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);		
            return new DefaultKafkaConsumerFactory<>(properties);
          }	
          /* 리스너 */
          @Bean
          public ConcurrentKafkaListenerContainerFactory<String, String> kafkaListenerContainerFactory() {
            ConcurrentKafkaListenerContainerFactory<String, String> factory = 
                new ConcurrentKafkaListenerContainerFactory<>();
            factory.setConsumerFactory(consumerFactory());
            return factory;
          }
        }
        ```

    - kafkaConsumer.java
      ```java      
      @Service
      @Slf4j
      @Transactional
      public class KafkaConsumer {
        private final CatalogRepository catalogRepository;
        @Autowired 
        public KafkaConsumer(CatalogRepository catalogRepository) {
          this.catalogRepository = catalogRepository;
        }	
        @KafkaListener(topics = "example-catalog-topic")
        public void upateQty(String kafkaMessage) {
          log.info("Kafka message: " + kafkaMessage);	
          ObjectMapper mapper = new ObjectMapper();
          Map<Object, Object> map = new HashMap<>();	
          try {
            map = mapper.readValue(kafkaMessage, new TypeReference<Map<Object, Object>>() {});
          } catch (JsonMappingException e) {
            e.printStackTrace();
          } catch (JsonProcessingException e) {
            e.printStackTrace();
          }
		
          Catalog catalog = catalogRepository.findById((Long)map.get("productId")).orElseThrow(IllegalArgumentException::new);
          catalog.setQty(catalog.getQty() - (Integer)map.get("qty"));
        }
      }
    
  - Order Service (Producer)
    - dependency 추가
      ```
      <dependency>
        <groupId>org.springframework.kafka</groupId>
        <artifactId>spring-kafka</artifactId>
      </dependency>
      ```
    - KafkaProducerConfig.java
      ```java
      @EnableKafka
      @Configuration
      public class KafkaProducerConfig {
        /*카프카 접속 정보 저장 빈*/
        @Bean
        public ProducerFactory<String, String> producerFactory() {
          Map<String, Object> properties = new HashMap<>();
          properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "192.168.56.2:9092");
          properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringDeserializer.class);
          properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringDeserializer.class);			
          return new DefaultKafkaProducerFactory<>(properties);
        }
        /* Kafka Template -> 데이터 publish할 때 사용 */
        @Bean
        public KafkaTemplate<String, String> kafkaTemplate() {
          return new KafkaTemplate<>(producerFactory());
         }	
      }
      ```
    - KafkaProducer.java
      ```java
      @Service
      @Slf4j
      @Transactional
      public class KafkaProducer {
        private final KafkaTemplate<String, String> kafkaTemplate;
        @Autowired 
        public KafkaProducer(KafkaTemplate<String, String> kafkaTemplate) {
          this.kafkaTemplate = kafkaTemplate;
        }	
        public Orders send(String topic, Orders order) {
          ObjectMapper mapper = new ObjectMapper();
          String jsonString = "";	
          try {
            jsonString = mapper.writeValueAsString(order);
          }catch(JsonProcessingException e) {
            e.printStackTrace();
          }		
          kafkaTemplate.send(topic, jsonString);	
          log.info("Kafka Producer send data from the order service: " + jsonString);				
          return order;
        }
      }
      ```
    - 주문 생성 시 KafakProducer를 통해 토픽에 데이터 전송
      ```java
      public ResponseOrder createOrder(Long userId, RequestOrder requestOrder) {
        Orders order = new Orders();
        order.setProductId(requestOrder.getProductId());
        order.setQty(requestOrder.getQty());
        order.setUnitPrice(requestOrder.getUnitPrice());
        order.setTotalPrice(requestOrder.getQty() * requestOrder.getUnitPrice());
        order.setUserId(userId);
        Orders returnOrder = orderRepository.save(order);	
        /* send this order to kafka */
        kafkaProducer.send("example-category-topic", returnOrder);
        return new ResponseOrder(returnOrder.getId(), returnOrder.getUserId(), returnOrder.getQty(), returnOrder.getUnitPrice(), 
            returnOrder.getTotalPrice(), returnOrder.getCreatedAt());
      }
      ```
      
#### Multi Service에서 데이터 동기화 문제

- 만약 Order Service를 확장하여 여러개를 구동한다면, DB도 분산되기 떄문에 따로 저장되어 문제가 발생한다. (동기화 문제)
- 해결 방법
  - Orders Service에 요청된 주문 정보를 DB가 아니라 Kafka Topic으로 전송
  - Kafka Topic에 설정된 Kafka Sink Connect를 사용해 단일 DB에 저장 -> 데이터 동기화
  
#### 출처
 
 - Spring Cloud로 개발하는 마이크로서비스 애플리케이션(MSA) 강의
