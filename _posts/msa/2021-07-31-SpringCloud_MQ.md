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

- 예제
  - mariaDB 설치(docker 활용)
    - mariadb 설치
      ```
      $ docker pull mariadb
      $ docker run --name mariadb -p 3306:3306 -e MYSQL_ROOT_PASSWORD=test1357 mariadb
      ```
      - port 는 3306, password는 test1357로 세팅했다


 
 #### 출처
 
 - Spring Cloud로 개발하는 마이크로서비스 애플리케이션(MSA) 강의
