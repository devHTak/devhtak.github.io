---
layout: post
title: Rabbit MQ 와 Application 아키텍처
summary: Rabbit MQ
author: devhtak
date: '2022-07-24 14:41:00 +0900'
category: msa
---

#### AMQP 와 Rabbit MQ

- AMQP
  - 프로토콜 수준에서 AMQP는 Client, Message Broker 간에 메시지를 주고 받을 수 있도록 협상(negotiation)과 같이 정보 중계 절차에 관한 의미를 정의
  - Rabbit MQ는 라이브러리를 통해 복잡한 AMQP 동작 방식 제공

- 대화 시작
  - 대화를 시작하면 아래와 같이 패킷을 전달하여 대화 시작할 수 있는 채널을 형성한다
    - Client -> Server: Protocol Header 패킷 전달
    - Client <- Server: Connection.Start 패킷 전달
    - Client -> Server: Connection.StartOk 패킷 전달
- 채널
  - AMQP 스펙에는 Rabbit MQ와 통신하기 위한 채널 정의
  - 채널은 다른 채널의 대화로부터 전송을 격리하며, 여러 대화를 수행할 수 있다

- 프레임 구조
  - 명령형식
    - 클래스.메서드 형식(Connection.Start)
  - 프레임 컴포넌트
    |---|---|---|---|---|
    |Frame 유형|채널번호|프레임크기| Payload | 끝을 나타내는 바이트 표식|
  - 프레임 유형
    - 헤더 프레임: Rabbit MQ 연결할 때 사용
    - Method 프레임: 서로 주고받는 요청/응답 전달
      - Basic.Publish + Exchange + Routing Key + Mendatory Flag
    - Contents Header 프레임: 메시지의 크기, 속성 전달
      - Basic.Properties
    - Body 프레임: 메시지 내용
      - JSON, XML 등 직렬화된 데이터
    - Heart-beat 프레임: Client, Server 상태 확인
  
#### 메시지 속성

- content-type
  - 메시지 본문의 데이터 형식

- content-encoding
  - 기본적으로 메시지를 압축하지 않으나, 발행 전 압축 발행 후 해제하여 사용함으로써 메시지 크기를 줄일 수 있다.
  - base64, gzip 등

- message-id
  - application 용도로 지정돼며 공식적인 정의는 없다.
  - 메시지를 고유하게 식별할 수 있도록 Header의 Data 전달

- correlation-id
  - application 용도로 지정돼며 공식적인 정의는 없다.
  - message와 관련된 참조하는 다른 데이터를 전달할 때 사용
  - 응답 Message의 ID로 Correlation-id 사용 / Transaction id로 사용 등

- timestamp
  - application 용도로 메시지 발행/생성 시점 기록
  
- expiration
  - rabbit mq 에서 소비하지 않는 메시지를 버려야할 때 파악(자동으로 메시지 만료)
  - 해당 기능은 큐를 정의할 때 x-message-ttl 속성을 통해서도 지정 가능
  
- delivery-mode
  - message를 memory or disk 저장 여부 결정
  - 안정성과 속도 조절(1: memory, 2: disk)
  
- app-id
  - 메시지를 발행시키는 어플리케이션 플랫폼, 버전등으로 설정

- user-id
  - rabbit mq가 메시지를 발행하는 사용자에 대해 user-id를 검사하고 일치하지 않으면 거부된다
  - 채팅, 인스턴트 메시징 서비스가 아닌 경우 권장하지 않는다.

- type
  - application 용도로 지정돼며 공식적인 정의는 없다.
  
- reply-to
  - application 용도로 지정돼며 공식적인 정의는 없다.
  - 메시지의 응답에 소비자가 사용해야 하는 라우팅 키를 전달하는 데 사용 / 동적인 작업 흐름
  
- headers
  - application 용도로 지정돼며 공식적인 정의는 없다.
  - header exchange 에 사용될 수 있음

- priority
  - message의 우선순위를 지정할 수 있으며 9인 메시지를 발행 후 0인 메시지를 발행하면 소비자는 0인 메시지를 먼저 받는다
  - 0 ~ 9 사이의 우선순위를 지정할 수 있으며, 0이 우선순위가 높다
  
- cluster-id, reserve: deprecated

#### 메시지 발행에서 성능/절충

- 메시지 브로커는 빠른 성능과 처리량도 중요하지만 신뢰할 수 있는 메시지 전달도 중요

- 발생 속도와 배달 보장의 균형 잡기
  - 고성능과 메시지 배달보장(delivery guarantee) 사이에 적절한 균형을 찾는 것이 중요

- 배달 보장을 사용하지 않는 경우
  - 미션 크리티컬한 데이터가 아닌 경우

- Mendatory Flag 설정한 Message를 Routing 할 수 없을 때
  - Mendatory flag는 Basic.Publish 명령과 함께 전달되는 인수
  - True로 전송하면 Message를 라우팅할 수 었을 때 Basic.Return을 통해 Publisher에게 메시지 전체를 전달
  - Publisher에서는 Basic.Return을 받으면 MessageReturnException을 발생한다.

- Transaction 보다 가벼운 발행자 확인
  - Publisher와 RabbitMQ 사이에 메시지가 잘 전달됐는 지 확인
  - Confirm.Select 요청을 전송하고 Confirm.SelectOK 응답 대기
  - Confirm.SelectOK 응답이 오면 Basic.Publish 로 메시지를 발행하고, Basic.Ack/Nack 로 메시지 전달에 대한 응답을 보낸다
  - Basic.Publish를 발행한 후 응답에 대한 Callback handler를 전달해야 하지만 블로킹되기 때문에 느리다

- 라우팅할 수 없는 메시시를 위한 익스체인지 사용
  - 발행자가 지정한 익스체인지에 라우팅할 수 없을 경우 지정한 대체 익스체인지로 발행되도록 설정할 수 있다.
  
- 트랜잭션으로 배치 처리
  - Publisher에 메시지를 발행하기 전에 Transaction을 생성할 수 있다.
  - Tx.Select 를 전송하면 Tx.SelectOk 로 응답이 오고 그 이후로 1개 이상에 메시지(Basic.Publish) 를 발행할 수 있다.
  - 정상적으로 메시지가 전달(Basic.Return)으로 오면 Tx.Commit 전송 / Tx.CommitOk 응답
  - 트랜잭션 예외가 발생 (NoActiveTransactionError) 하면 Tx.Rollback 전송 / Tx.RollbackOK 응답
  
- HA 큐를 사용해 노드 장애 대응
  - Rabbit MQ는 큐를 여러 서버에 중복해 복사본 저장
  - Queue에 대해 아래와 같이 설정할 수 있다
    - x-ha-policy: all
    - x-ha-nodes: ['nodes1', 'nodes2', ...]
  - Cluster에 특정 노드 메시지를 전달하면 Rabbit MQ 는 큐의 메시지를 동기화하고 서버에 저장
  
- dlivery-mode=2로 설정
  - Rabbit MQ 서버에 메시지를 디스크에 저장한다

#### 메시지를 받지 않고 소비하기

#### 익스체인지 라우팅을 통한 메시지 패턴

#### 출처
- RabbitMQ IN DEPTH
