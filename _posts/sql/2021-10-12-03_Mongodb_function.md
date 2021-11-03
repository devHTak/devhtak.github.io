---
layout: post
title: Mongodb - index, shading, replica
summary: NoSQL - MongoDB
author: devhtak
date: '2021-10-11 21:41:00 +0900'
category: No SQL
---

#### Mongodb index

- index
  - index는 디비의 검색을 빠르게 하기 위해 미리 데이터의 순서를 정리해두는 과정
  - Mongodb는 고정된 스키마는 없지만 원하는 데이터 필드를 인덱스로 지정하여 검색 결과를 빠르게 하는 것이 가능
  - NOSQL 에서도 index를 잘 설계해야 최대의 효율이 가능
  - Mongodb는 B-Tree 구조로 index를 구현
  - 고유 index, 희소 index, 다중 키 index, 복합 index, 단일 index 지원

- index 개념
  - index는 도큐먼트를 쿼리해오기 위한 작업량을 줄인다
    - 적당한 index가 없으면 질의 조건을 만족할 때까지 모든 도큐먼트를 순차적으로 스캔
  - 한 쿼리당 하나의 index만 유효
  - 두 개의 index가 필요하다면 복합 index를 사용
    - a, b 필드로 구성된 복합 index를 가지고 있다면 a에 대해 단일 index는 제거해도 됨
    - 복합 index에서 키의 순서는 매우 중요
  - _id는 기본적으로 생성되는 index로 도큐먼트를 가르키는 유일한 키값으로 사용
    - 도큐먼트에 빠르게 접근하기 위해서 각 _id는 index로 관리
    
- 효율
  - 어떤 데이터가 도큐먼트에 추가되거나 수정될 때마다 그 컬렉션에 대해 생성된 index도 그 새로운 도큐먼트를 포함시키도록 수정되어야 함
  - 최악의 경우에는 결국 데이터를 다시 정렬해야 하는 상황 발생
  - index는 읽기 위주의 어플리케이션에서 유용하기 때문에 읽기보다 쓰기 작업이 많다면 어느 정도 index를 포기하거나 index를 위한 컬렉션을 따로 운영해야 함
  - Mongodb는 기동시 모든 데이터 파일(모든 도큐먼트, 컬렉션, 인덱스)을 페이지(page)라고 부르는 4kb정도의 청크 단위로 메모리에 매핑함
  - 모든 데이터를 수용하지 못하면 페이지 폴트가 자주 발생하게 되고 운영체제가 디스크를 빈번하게 액세스하게 됨으로 인해 읽기/쓰기 연산 지연 발생
  - 최소한의 인덱스가 메모리에 위치 할 수 있도록 최소화될 필요가 있음
  - 복합인덱스는 더 많은 공간을 필요로 함을 고려해야 함
  
- B-Tree
  
  ![image](https://user-images.githubusercontent.com/42403023/136956370-797d993a-0986-4fd1-aace-265617a7e655.png)

  - 트리구조와 유사한 데이터 구조
  - 각 노드는 여러 개의 키를 갖는 것이 가능
  - Mongodb에서 사용하는 B-Tree는 새 노드에 대해 8192바이트를 할당함
    - 각 노드가 수백개의 키를 가질 수 있다.
  - 인덱스 키의 평균 크기에 따라 달라질 수도 있는데, 보통 키의 평균적인 크기는 30바이트 안팎임
  - 특징
    - 정확한 일치, 범위 조건, 정렬, 프리픽스 일치 등 다양한 쿼리를 용이하게 처리하도록 도와준다
    - 키가 추가되거나 삭제되더라도 밸런스 유지에 좋다
    
#### Mongodb index 활용

- index 확인
  ```
  > db.scores.getIndexes();
  [ { "v" : 2, "key" : { "_id" : 1 }, "name" : "_id_" } ]
  ```
  
- index 생성
  ```
  > db.scores.createIndex({"name": 1})
  {
    "numIndexesBefore" : 1,
    "numIndexesAfter" : 2,
    "createdCollectionAutomatically" : false,
    "ok" : 1
  }
  > db.scores.getIndexes();
  [
    {
      "v" : 2,
      "key" : {
        "_id" : 1
      },
      "name" : "_id_"
    },
    {
      "v" : 2,
      "key" : {
        "name" : 1
      },
      "name" : "name_1"
    }
  ]
  
  # db.scores.createIndex({"name": 1}, {unique: true})
  # db.scores.createIndex({"name": 1}, {unique: true, dropDups: true})
  ```
  - 오름차순이면 1, 내림차순이면 -1을 지정
  - unique를 사용하여 고유 인덱스를 생성할 수 있다.
    - unique 속성을 지정해서 중복 데이터가 저장되지 못하도록 하면, 데이터가 저장과 검색속도를 늘리는데 도움이 된다
  - dropDups(중복 데이터 삭제): 특정 필드를 Unique를 하게 했을 때 기존에 이미 중복된 데이터가 있을 경우에 대한 정책이 필요. dropDups를 하면 기존에 중복 데이터를 삭제하고 인덱스 저장이 가능

- index 삭제
  ```
  > db.scores.dropIndex({"name": 1})
  > db.scores.dropIndex()
  ```
  - 지정된 인덱스만 삭제가 가능하고, 모든 인덱스 삭제도 가능하다( \_id는 제외 ) 

#### 샤딩

- 목적
  - 데이터 분산 저장에 필요성
    - 단 한대의 서버에 빅데이터를 저장하는 것을 불가능
    - 서비스의 성능 저하 유발: 초당 발생하는 엄청난 양의 Insert 동작시 Write Scaling 문제 발생
    - 디스크를 사용하는 하드웨어 한계성

  - 백업과 복구 전략
    - 데이터 분산이라는 샤딩의 가장 대표적인 기능을 통해 얻는 효과
    - 시스템의 성능 향상
    - 데이터 유실 가능성으로부터 보호
    - 서버의 데이터가 유실된다면 그 데이터 양은 상상을 초월할 것이고 시스템 복구에 엄청난 시간과 비용 소요
    - 미리 데이터를 분산하여 저장해둔다면 리스크로부터 보호받고 효과적인 시스템 운영이 가능해진다.

  - 빠른 성능
    - 여러 대의 독립된 프로세스가 병렬로 작업을 동시에 수행하기 때문에 이상적으로 빠른 처리 성능을 보장받는다.

- 샤딩 시스템 구조

  ![image](https://user-images.githubusercontent.com/42403023/136961245-ac99aba9-1b8e-41bc-aec4-99760c7153d4.png)
  
  - 특징
    - 샤딩 시스템은 분산처리를 통한 효율 향상
      - 가능한 성능 보장을 위해 3대 이상의 서버를 샤드로 활용하는 것을 추천
      - 최소 2대만 있으면 샤드 서버 구축 가능
    - 기존에 한대의 서버보다 메모리를 20~30% 추가로 사용하게 된다
      - 샤드 시스템 구축 시 사용하는 라우팅 서버인 mongos, OpLog, Balancer 프로세스가 추가로 메모리를 사용
      - 기존에 싱글서버보다 20~30% 정도 추가 메모리 준비가 필요하다

- Config 서버 개요
  - Config 서버는 샤드 시스템에 대한 메타 데이터 저장/관리 역할
  - 샤드 서버의 인덱스 정보를 빠르게 검색 가능케 함
  - 샤드 서버와 별도의 서버에 구축이 기본
  - 장애 발생에 대비하여 최소 3대 이상 사용(최소 1대만으로 운영 가능)
  - 샤드 서버에 비해 저사양 서버 사용 가능

- Mongos 서버 특징
  - 하나 이상의 프로세스 사용
  - Config 서버의 Meta-data를 캐시한다.
  - 빅데이터를 샤드 서버로 분산해주는 프로세스
  ```
  - Config 서버는 각 샤드 서버에 어떤 데이터들일 어떤 식으로 분산 저장되어 있는지에 대한 Meta 데이터가 저장
  - mongos 서버를 통해 데이터를 쓰고 읽는 작업 가능
  - 또한 mongos 는 각 서버에서 어떤 일을 하는지 개발자가 모르게 해주는 역할을 한다.
  - 지금 샤딩 상태인지 리플리케이션 상태인지 개발자는 알 필요가 없다.
  ```
  
- Shading System layer
  - 중개자 계층: 샤딩 시스템의 가장 핵심적인 부분, 메타정보 저장 및 application과 data간에 적절한 질의 및 결과를 반환한다

- Shard Key
  
  ![image](https://user-images.githubusercontent.com/42403023/138533812-1facd66b-8520-4313-8fb5-095beffc3c09.png)
  
  - unique 한 단일 또는 복합 인덱스로 shard key 사용(default \_id)
  - 여러 개의 Shard 서버로 분할될 기준 필드를 가리키며, partition, load balancing에 기준이 된다.

#### Replication

- 복제
  - 고성능 DB에서 가장 핵심이 되는 기능
  - 같은 데이터를 갖는 여러개의 Mongodb 서버를 설계하는 과정
  - 고성능의 미러링 기능
  - 성능과 높은 가용성의 장점 제공
  - Master/Slave Replication
    - 자동 장애 조치 불가능(수동)
    - 13개 이상 노드까지 구성 가능

- 복제의 용도
  - 데이터 일관성
  - 읽기 분산
  - 운영 중 분산
  - 오프라인 일괄 적업용 데이터 소스

- Mongodb 복제 동작 원리

  ![image](https://user-images.githubusercontent.com/42403023/137127182-a4642f03-1705-42df-92c9-56095a84b7e0.png)

  - 몽고디비의 마스터는 쓰기 연산을 담당
  - 일반 마스터-슬레이브 방식과 동일하게 쓰기는 마스터에서만 이뤄짐, 슬레이브에서는 쓰기 명령을 사용할 수 없다.
  - 몽고디비에서 쓰기 연산이 실행되면 데이터 저장소와 Oplog 영역에 저장
  - Oplog에는 연산 수행과 관련된 명령어 자체를 타임스탬프(optime)와 함께 저장
  - 몽고디비의 슬레이브는 주기적으로 마스터에게 자신의 optime 보다 큰 oplog를 요청
  - 5초 안에 마스터에서 쓰기 연산이 발생하면 바로 데이터를 응답
  - 5초 안에 쓰기 연산이 발생하지 않으면, 데이터가 존재하지 않는다는 응답을 보내줌
  - 슬레이브는 요구한 Oplog 데이터가 존재하면 자신의 Oplog에 데이터를 저장한 다음 바로 마스터에 다시 Oplog 요청

- 시스템 구성
  - 서버 실행
    ```
    > mongod -dbpath 경로 -port 10000 -master # 마스터 서버 실행
    > mongod -dbpath 경로 -port 10001 -slave -source localhost:10000 
    > mongod -dbpath 경로 -port 10002 -slave -source localhost:10000 # 슬레이브 서버 실행
  
  - 데이터 저장
    ```
    # 마스터에 접속 및 입력
    > mongo localhost:10000
    > show dbs;
    > use test;
    > db.users.insert({name: 'Shin': phone: '010-1234-1234'});
    > db.users.find(); # shin 조회
    # slave에 접속 및 확인
    > mongo localhost:10001
    > user test;
    > db.users.find(); # shin replication 되어 조회 # 입력 등 write는 불가능
    ```

#### ReplicaSet

- ReplicaSet
  - primary server: ReplicaSet에서 첫번 째 입력을 담당하는 서버
  - secondary server: primary server를 제외한 나머지 서버
  - 만약 privary server에 문제가 생기면 자동으로 secondary server에서 데이터 입/출력을 담당
  
- 주요 특징
  - primary server는 secondary server를 2초 단위로 상태를 체크하여 데이터 동기화를 위한 heardbeat 를 확인
  - heartbeat의 수신 결과, secondary server를 사용할 수 없는 상황이 되더라도 데이터 복제만 중단 될 뿐 primary server는 데이터 수신/저장을 계속 담당
  - secondary server가 복구되면 그간의 밀린 데이터를 복구해주기 위해 primary server는 oplog를 저장하게 되는데, 이후 secondary가 복구되면 자동으로 동기화한다.
  - 만일 primary server가 장애 상황이 된다면, secondary server를 primary server로 만듦

- 동작 원리

  ![image](https://user-images.githubusercontent.com/42403023/137129896-fff506ac-12b3-4807-8722-775c0d166f3f.png)

  - 복제 집합은 한 개의 primary와 두 개의 secondary로 구성
  - 복제 집합으로 구성된 각각의 노드는 자신을 제외한 다른 노드들이 heartbeat를 이용하여 주기적으로 검사
  - 몽고디비의 heartbeat는 2초 단위로 수행되며, heartbeat를 받는 서버는 자신의 상태 코드를 heartbeat를 요청한 서버에 보내줌
  - primary server 의 heartbeat는 항상 복제 집합을 구성하고 있는 노드 개수의 과반수만큼을 유지하고 있어야 한다
  - 만약 primary server가 과반수의 heartbeat를 가지고 있지 않는다면, 해당 서버는 secondary 서버로 전환되고 전체 복제 집합은 primary server 부재에 따른 투표를 시행
  - primary가 될 수 있는 자격 조건으로는 priority(마스터가 될 수 있는 우선순위), votes(자신을 포함한 복제 집합의 노드 개수의 과반수 투표) 등을 가지고 있다.

- 한계
  - 슬레이브가 아무리 빨리 데이터를 동기화 한다고 해도, 마스터와의 통신 지연 시간만큼의 차이를 가질 수 있음
  - 부하를 견디지 못해서 마스터 서버가 죽었을 경우, Oplog에 동기화 되지 않은채 남아있는 데이터 연산을 잃어버리는 현상이 발생할 수 있음
  - 저널링 파일에 데이터를 저장하는 방법의 경우에도 group commits 주기(100ms) 안에 데이터가 존재하는데도 시스템이 다운되어 메모리에 저장된 데이터가 날아갔을 경우에는 복구가 불가능함

- replicaset 구성
  - 서버 실행
    ```
    mongod —replSet downSet -dbpath c:\mongodb\var -port 10000 # primary server 실행
    mongod —replSet downSet -dbpath c:\mongodb\var2 -port 10001 # Secondary 서버 실행 1
    mongod —replSet downSet -dbpath c:\mongodb\var3 -port 10002 # Secondary 서버 실행 2
    ```

  - 리플리카셋 환경 설정
    ```
    $ mongo localhost:10000 # Primary 서버에 접속
    > var config = {_id:'downSet', members:[{_id:0, host:'localhost:'10000'}, {_id:1, host:'localhost:'10001'}, {_id:2, host:'localhost:'10002'}]);
    # 리플리카셋 환경 설정
    > rs.initiate(config); # 리플리카셋 초기화
    ```

#### 출처

- B-Tree: https://ichi.pro/ko/mongodb-indegseu-simcheung-bunseog-indegseu-ihae-171312403020454
- shading: https://www.infoq.com/news/2010/08/MongoDB-1.6
- replication 동작 원리: https://givemesource.tistory.com/88
- replicaset 구성: https://eunsour.tistory.com/75
