---
layout: post
title: 도커를 활용한 MongoDB 설치 및 쉘 사용
summary: JUnit 사용법
author: devhtak
date: '2021-10-11 21:41:00 +0900'
category: No SQL
---

#### Docker로 MongoDB 설치

- MongoDB 이미지 다운로드
  ```
  $ docker pull mongo
  ```
  - docker hub에서 이미지 다운르드 
  - :version 을 명시하지 않으면 최신 버전의 이미지를 받는다

- MongoDB 컨테이너 생성 및 실행
  ```
  $ docker run --name mongodb-container -v ~data:data/db --p 27017:27017 -d mongo
  ```
  - mongo 이미지 컨테이너 생성 및 백그라운드 실행(-d)
  - 27017(host):27017(container) 포트 포워딩(-p)
  - 컨테이너 이름 mongodb-container(--name)
  - 호스트(컨테이너를 구동하는 로컬 컴퓨터)의 ~/data 디렉터리와 컨테이너의 /data/db 디렉터리를 마운트(-v)

- MongoDB 컨테이너 접속
  ```
  $ docker exec -it mongodb-container /bin/bash
  root@871373604e88:/# mongo
  MongoDB shell version v5.0.3
  connecting to: mongodb://127.0.0.1:27017/?compressors=disabled&gssapiServiceName=mongodb
  Implicit session: session { "id" : UUID("030e5e3b-80e5-4e0a-8b94-ce7890cd435d") }
  MongoDB server version: 5.0.3
  ================
  Warning: the "mongo" shell has been superseded by "mongosh",
  which delivers improved usability and compatibility.The "mongo" shell has been deprecated and will be removed in
  an upcoming release.
  We recommend you begin using "mongosh".
  For installation instructions, see
  https://docs.mongodb.com/mongodb-shell/install/
  ================
  ---
  The server generated these startup warnings when booting: 
          2021-10-11T03:31:23.952+00:00: Access control is not enabled for the database. Read and write access to data and configuration is unrestricted
  ---
  ---
          Enable MongoDB's free cloud-based monitoring service, which will then receive and display
          metrics about your deployment (disk utilization, CPU, operation statistics, etc).

          The monitoring data will be available on a MongoDB website with a unique URL accessible to you
          and anyone you share the URL with. MongoDB may use this information to make product
          improvements and to suggest MongoDB products and deployment options to you.

          To enable free monitoring, run the following command: db.enableFreeMonitoring()
          To permanently disable this reminder, run the following command: db.disableFreeMonitoring()
  ---
  > show dbs;
  admin   0.000GB
  config  0.000GB
  local   0.000GB
  testDB  0.000GB
  > show collections;
  > use admin; # admin 계정 사용
  > db.shutdownServer(); # 서버 종료
  ```

- MongoDB 시작, 중지, 재시작
  ```
  # MongoDB Docker 컨테이너 중지
  $ docker stop mongodb-container
  
  # MongoDB Docker 컨테이너 시작
  $ docker start mongodb-container
  
  # MongoDB Docker 컨테이너 재시작
  $ docker restart mongodb-container
  ```
  
#### Mongodb 구성

- Mongodb 구성
  - 수평적 확장이라는 특징을 가지기 때문에 한 대 이상의 서버로 구성하는 것이 일반적이다
  - 메모리 사용 가능량에 대비하여 성능이 좌우되기 때문에 독립된 서버에서 실행 권장
  - mongos 서버를 통해 마치 한대의 데이터베이스 서버처럼 사용 가능
  - 보통 3개의 Replica 단위로 구성하여 데이터 복제
  - mongod라는 실행파일을 단위로 실행

- architecture
  
  ![image](https://user-images.githubusercontent.com/42403023/136729841-53e46beb-e12c-4d83-819c-28f354465eb7.png)

#### Shell 실행

- shell 특징
  - mongodb 데이터베이스 조회
    ```
    > show dbs;
    admin   0.000GB
    > use test;
    switched to db test
    > show dbs;
    admin   0.000GB
    > db.test.save({a:1});
    WriteResult({ "nInserted" : 1 })
    > show dbs;
    admin   0.000GB
    test    0.000GB
    ```
    - db.test.save() 는 데이터 저장을 의미하는 데, 실제 데이터를 넣어주어야 show dbs를 통해 조회가 가능하다

  - mongodb 컬렉션 조회
    ```
    $ show collections;
    ```
    - RDBMS에서 테이블과 동일하다.
    - 테이블과의 차이점은 생성하기 전에 스키마를 정의할 필요가 없다
    - 컬렉션은 정해진 스키마와 상관없이 들어오는 데이터를 저장할 수 있다.(schemaless)

  - javascript 명령 사용 가능
    ```
    > a = 5;
    > a * 10;
    > for(i = 0; i < 10; i++) { print('hello'); }
    ```
    - Json 형태 데이터를 저장
      ```
      var a = { 'age': 25 };
      var n = { 'name': 'Ed', 'languages':['c', 'java', 'javascript'] }
      ```

- 저장
  - save 명령
    ```
    > db.scores.save({a: 99});
    WriteResult({ "nInserted" : 1 })
    > for(i = 0; i < 10; i++) { db.scores.save({a:i, scores: i * 10}); }
    WriteResult({ "nInserted" : 1 })
    > db.scores.find();
    { "_id" : ObjectId("6163b796637b5ee01a43f8a5"), "a" : 99 }
    { "_id" : ObjectId("6163b922637b5ee01a43f8a6"), "a" : 0, "scores" : 0 }
    { "_id" : ObjectId("6163b922637b5ee01a43f8a7"), "a" : 1, "scores" : 10 }
    { "_id" : ObjectId("6163b922637b5ee01a43f8a8"), "a" : 2, "scores" : 20 }
    { "_id" : ObjectId("6163b922637b5ee01a43f8a9"), "a" : 3, "scores" : 30 }
    { "_id" : ObjectId("6163b922637b5ee01a43f8aa"), "a" : 4, "scores" : 40 }
    { "_id" : ObjectId("6163b922637b5ee01a43f8ab"), "a" : 5, "scores" : 50 }
    { "_id" : ObjectId("6163b922637b5ee01a43f8ac"), "a" : 6, "scores" : 60 }
    { "_id" : ObjectId("6163b922637b5ee01a43f8ad"), "a" : 7, "scores" : 70 }
    { "_id" : ObjectId("6163b922637b5ee01a43f8ae"), "a" : 8, "scores" : 80 }
    { "_id" : ObjectId("6163b922637b5ee01a43f8af"), "a" : 9, "scores" : 90 }
    ```
    
- 조회(find)
  - 조건 입력
    ```
    > db.scores.find({a:2});
    { "_id" : ObjectId("6163b922637b5ee01a43f8a8"), "a" : 2, "scores" : 20 }
    > db.scores.find({a: {'$gt': 15}});
    { "_id" : ObjectId("6163b796637b5ee01a43f8a5"), "a" : 99 }
    > db.scores.find({a: {'$gt': 2, '$lt': 4}});
    { "_id" : ObjectId("6163b922637b5ee01a43f8a9"), "a" : 3, "scores" : 30 }
    > db.scores.find({a: {'$in': [2, 3]}});
    { "_id" : ObjectId("6163b922637b5ee01a43f8a8"), "a" : 2, "scores" : 20 }
    { "_id" : ObjectId("6163b922637b5ee01a43f8a9"), "a" : 3, "scores" : 30 }
    > db.scores.find({ $or: [{a: {'$lt': 1}}, {a: {'$gt': 15}}]});
    { "_id" : ObjectId("6163b796637b5ee01a43f8a5"), "a" : 99 }
    { "_id" : ObjectId("6163b922637b5ee01a43f8a6"), "a" : 0, "scores" : 0 }
    > db.scores.find({scores: {'$exist':false}});
    { "_id" : ObjectId("6163b796637b5ee01a43f8a5"), "a" : 99 }
    ```
    - AND 연산(,) 가능, or 연산({'$or': [{}, {}]})
    - 비교 연산자: gt: >, lt: <, gte: >=, lte: <=, ne: !=, in: 'is in array', nin: '!in array'
    - 필드 존재 유무 확인(exists)
  - 반환 필드 선택
    ```
    > db.scores.find({}, {a: 1, scores: 1, _id:0};
    { "a" : 99 }
    { "a" : 0, "scores" : 0 }
    { "a" : 1, "scores" : 10 }
    { "a" : 2, "scores" : 20 }
    { "a" : 3, "scores" : 30 }
    { "a" : 4, "scores" : 40 }
    { "a" : 5, "scores" : 50 }
    { "a" : 6, "scores" : 60 }
    { "a" : 7, "scores" : 70 }
    { "a" : 8, "scores" : 80 }
    { "a" : 9, "scores" : 90 }
    ```
    - 1: 출력, 0: 미출력

- 수정(update)
  - 미리 데이터 생성
    ```
    > db.users.save({name: 'Johnny', languages: ['ruby', 'c']});
    WriteResult({ "nInserted" : 1 })
    > db.users.save({name: 'Sue', languages: ['scala', 'lisp']});
    WriteResult({ "nInserted" : 1 })
    ```
  - update 명령 (조건, 수정 내용)
    - 전체 내용 변경
      ```
      > db.users.update({name: 'Johnny'}, {name: 'Cash', languages: ['english']});
      > db.users.find({name: 'Cash'});
      { "_id" : ObjectId("6163bdd7637b5ee01a43f8b0"), "languages" : [ "english"] }
      ```
      - 두번째 인자에는 전체 데이터를 입력해주어야 한다.
    - 일부 수정
      ```
      > db.users.update({name: 'Cash'}, {'$set': {'age': 50}});
      ```
      - $set 사용
      - 해당 값이 있으면 수정하고, 없는 경우 추가한다.
    - 필드 내용 삭제
      ```
      > db.users.update({name: 'Cash'}, {'$unset': {'age': 50}});
      ```
      - $unset 사용
    - 배열 값 추가 및 삭제
      ```
      > db.users.update({name: 'Sue'}, {'$push': {'languages': 'java'}}); # 추가
      > db.users.update({name: 'Sue'}, {'$pull': {'languages': 'scala'}}); # 삭제
      ```
      - $push: 추가, $pull: 삭제
      
- 삭제(remove)
  ```
  > db.users.remove({name: 'Sue'});
  > db.users.remove({});
  ```
  - {}를 사용하면 전체 삭제

- 컬렉션 삭제(drop)
  ```
  > db.users.drop();
  true
  > show collections;
  scores
  test
  ```
  
- 예제
  - test db 사용
  - users collection 사용
  - 100명에 대한 Document insert
    - {name: "name0", pos: 0}, {name: "name1", pos: 1}, ..., {name: "name99", pos: 99}
  - 조회: 6 <= pos <= 27 or 77 < pos <= 90, 출력 value: id, name
  - pos가 10인 데이터에 대하여 job: software engineer 추가
  - 90 <= pos < 100 삭제
  - 데이터 전체 삭제 후 users collection 삭제
  ```
  > for(i = 0; i < 100; i++) { db.users.save({name: 'name' + i, pos: i}); }
  > db.users.find({'$or': [{pos: {'$gte': 6, '$lte': 27}}, {pos: {'$gt': 77, '$lte': '90'}}]}, {pos: 0});
  > db.users.update({pos: 10}, {'$set': {job: 'software engineer'}});
  > db.users.find({pos: {'$lte': 10});
  > db.users.remove({pos: {'$gte': 90, '$lt': 100'}});
  > db.users.remove({});
  > db.users.drop();
  ```
  
#### 출처
- Docker 를 활용한 Mongodb 설치 및 실행: https://poiemaweb.com/docker-mongodb
- 
