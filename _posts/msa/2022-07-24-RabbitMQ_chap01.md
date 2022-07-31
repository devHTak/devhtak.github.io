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
  - 대화를 시작하면 아래와 같이 프레임을 전달하여 대화 시작할 수 있는 채널을 형성한다
    - Client -> Server: Protocol Header 프레임 전달
    - Client <- Server: Connection.Start 프레임 전달
    - Client -> Server: Connection.StartOk 프레임 전달
- 채널
  - AMQP 스펙에는 Rabbit MQ와 통신하기 위한 채널 정의
  - 채널은 다른 채널의 대화로부터 전송을 격리하며, 여러 대화를 수행할 수 있다

- 프레임 구조
  - 명령형식
    - 클래스.메서드 형식(Connection.Start)
  - 프레임 컴포넌트
  
    |프|레|임|구|조|
    |---|---|---|---|---|
    |Frame 유형|채널번호|프레임크기| Payload |끝을 나타내는 바이트 표식|
  
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

- Basic.Get vs Basic.Consume
  - Basic.Get(동기)
    - Basic.Get 요청을 보내면 메시지가 있으면 Basic.GetOk, 없으면 Basic.GetEmpty 응답
    - Basic.Get 을 사용하면 메시지 수신을 확인해야 하며, 메시지가 있는지를 확인한다
  - Basic.Consume(비동기)
    - Basic.Consume 으로 구독하면 Basic.Cancel 을 전송하기 전까지 메시지를 수신할 수 있다.
    - 메시지가 처리됐음을 알리는 message.ack을 전달해야 한다

- 소비자 성능 조절
  - 무응답 모드로 빠르게 메시지 소비하기
    - no_ack: True
      - RabbitMQ가 소비자에게 메시지를 전달하는 가장 빠르고 안정적인 방법
      - TCP 소켓 연결을 통해 통신하는데 연결이 열려있으면 전달되었다고 판단
  - Consumer 프리페치 제어
    - 미리 지정된 수의 메시지를 수신하도록 처리할 수 있는 QoS 설정을 채널에 요청할 수 있다
    - Prefetch 값을 최적화 하여 사용해야 한다
    - RabbitMQ 는 기본적으로 RR 방식으로 메시지를 수신하는데 Prefetch 수가 성능에 영향을 미치는지 확인해야 한다
    - 네트워크 통신 최소화 + 메시지 처리량 향상
  - Consumer에서 트랜잭션 사용
    - Producer와 마찬가지로 트랜잭션을 사용하여 일괄 작업을 커밋하고 롤백할 수 있다.

- 메시지 거부하기
  - 메시지 처리 중 이슈가 발생했을 때, Basic.Nack, Basic.Reject 로 메시지 브로커에 다시 전달할 수 있다.
  - Basic.Reject
    - 컨슈머에서 Basic.Reject 를 사용하면 메시지를 삭제하거나 큐에 있는 메시지를 다시 삽입되도록 지시할 수 있다
    - 재삽입 플래그가 활성화되면 재처리할 수 있도록 큐에 메시지를 넣는다
  - Basic.Nack
    - Basic.Reject 는 단일 메시지만 처리 가능하고, Basic.Nack은 다중 메시지를 처리할 수 있다.
  - DLX
    - 메시지를 처리할 때 이슈가 발생하면 x-dead-letter-exchange 로 라우팅할 수 있다.
    - direct 방식으로 큐를 바인딩 하기 위해서는 x-dead-letter-routing-key 지정이 필요하다
  
- 큐제어하기
  - 큐 동작을 결정하는 설정이 있다 (자동삭제 큐, 큐독점 설정, 자동 메시지 만료, 대기 메시지 수 제한, 오래된 메시지 큐에서 제거 등)
  - 임시 큐
    - 자동삭제 큐
      - 큐에 Consumer가 없을 때 자신을 삭제
      - 채팅 application 에서 큐를 통해 사용자의 입력 버퍼로 사용하는 경우 연결이 끊어지면 자동으로 큐가 삭제되도록 설정
      - Queue.Declare 요청에서 auto-delete flag를 True 로 설정
    - 큐 독점 설정
      - 독점 설정하지 않으면 다수 소비자가 큐를 구독할 수 있다.
      - Basic.Declare 요청에 exclusive=True 인수 전달 
    - 자동 만료 큐
      - 일정 기간 사용하지 않은 큐 삭제 기능
      - 자동 만료 큐는 시간에 민감한 작업에 대해 RPC 응답을 무기한으로 대기하지 않을 경우 유용
      - Basic.Declare 에 x-expires 인수로 큐 선언
  - 영구적인 큐
    - 내구성 큐
      - 서버 재시작 후에도 계속 유지되어야 하는 경우 사용
      - durable = True 로 설정
    - 큐에서 메시지 자동 만료
      - 메시지를 소비하지 않을 때 자동으로 삭제하도록 설정
      - x-message-ttl 인수를 arguments로 설정
    - 제한된 수의 메시지 보관
      - 큐의 메시지 최대 크기 설정
      - x-max-length 인수를 arguments로 설정

#### 익스체인지 라우팅을 통한 메시지 패턴

- Exchange 
  - Direct Exchange
    - Routing key 가 동일한 문자열인지 검사
    - 매우 단순하며 RPC 패턴에 용이 (Application 의존성 분리, 독립적인 컴포넌트 구성, 확장성에 뛰어남)
    - Architecture
    
      - <img width="407" alt="image" src="https://user-images.githubusercontent.com/42403023/182009609-e2dd24ed-09b9-453b-be68-4c8811a86ef2.png">
      - MethodFrame: Basic.Publish
      - Contents Header Frame: reply-to(바인딩 될 큐 이름) / correlation-id (RPC 요청을 구분할 ID)
      - 본문 Frame 으로 구성
      
  - Fanout Exchange
    - Fanout Exchange 에 연결된 모든 Queue에 전달
    - Routing Key를 확인할 필요가 없다 -> 성능 이점 / 모든 Application이 메시지를 전달 받을 수 있다.
    
  - Topic Exchange
    - Routing Key 가 있는 모든 큐에 메시지를 라우팅
    - Direct Exchange 와의 차이점은 와일드 카드 기반의 패턴 매칭을 사용할 수 있다
    - 단일 목적의 Application이 단일 큐에 연결하는 아키텍처는 모놀리식 구조에 비해 유지보수 혹은 확장에 용이 (유연성)

  - Header Exchange
    - Message 속성 중 Headers 테이블을 사용해 라우팅 처리
    - Queue.Bind 메소드의 인수로 key/value 배열, x-match(any or all) 사용
  
  - Exchange 간에 Routing 하기
    - Rabbit MQ 는 AMQP 스펙에 없는 Exchange 조합으로 Message Routing 할 수 있는 유연한 방법 제공
    - Exchange.Bind RPC Method 를 통해 제공
    - 유연하지만 복잡성 + 추가적 오버헤드 발생
    
  - Consistent Hashing Exchange
    - Rabbit MQ 에 배포하는 플러그인으로 연결된 Queue 에 분배
    - 정수 기반 가중치를 사용하는 알고리즘으로 큐를 선택하고 메시지를 전달
    - Round Robin 하지 않고 Routing Key 의 Hash 값 or 메시지 속성 중 Header-Type의 값을 기반으로 메시지 전달

#### 출처
- RabbitMQ IN DEPTH
