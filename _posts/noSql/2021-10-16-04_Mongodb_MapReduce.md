---
layout: post
title: Mongodb Map Reduce
summary: NoSql - Mongodb
author: devhtak
date: '2021-10-16 21:41:00 +0900'
category: No SQL
---

#### Map Reduce

- 개념
  - 대용량의 데이터를 안전하고 빠르게 처리하기 위한 방법
  - 한 대 이상의 하드웨워를 활용하는 분산 프로그래밍 모델
  - 2006년 Hadoop이라는 오픈소스 프로젝트 진행
    - Hadoop은 HDFS(Hadoop File System)이라는 대규모 분산 파일시스템을 구축하여 탁월한 성능과 안전성을 보여준다.
    - MapReduce는 대용량 파일에 대한 로그 분석, 색인 구축, 검색에 탁월한 능력을 보여준다.

- 동작 원리

  ![image](https://user-images.githubusercontent.com/42403023/137570498-84446cce-49f3-4431-b4ff-4ccf3db6c046.png)
  
  - 대용량의 input 데이터를 일정 단위 블록으로 나눈다.
  - Map함수를 통해 split된 데이터를 처리한다
  - 중간 결과물을 key 단위로 합친 후 reduce 함수를 실행하여 output을 만든다.

- 특징
  - 맵 리듀스는 데이터를 분산하고 연산하고 다시 합치는 기술
  - 맵과 리듀스 단계로 나누고 맵 단계는 입력과 출력으로 key-value 형태를 가지고 있음
  - 데이터를 섞어서 병합하고 리듀스 함수를 통해 최종적으로 결과 제공
  - 맵과 리듀스는 사용자가 임의로 코딩이 가능한 형태로 제공
  - 분산을 통해 분할된 조각으로 처리한 뒤 다시 모아 훨씬 짧은 시간에 계산을 완료
  - 분할된 조각이 작을수록 부하 분산에 더 좋은 효과를 낸다.
  - 너무 과하게 데이터를 분할할 경우 맵을 생성하기 위한 태스크의 오버헤드가 커지기때문에 역효과가 발생
  
- 장점
  - 분산모델을 감추어 대용량 처리를 단순하게 만듦
  - 특정 데이터 모델이나 스키마에 의존적이지 않은 유연성
  - 저장구조의 독립성
  - 높은 확장성
  
- 단점
  - 기존 RDBMS보다 불편한 스키마와 질의
  - DBMS와 비교하여 낮은 성능
  - 단순한 데이터 처리
  
#### Hadoop의 HDFS, Map Reduce

- HDFS: Hadoop Filesystem
- 구성 요소
  - 클라이언트: 데이터 송수신 요청
  - 네임노드: HDFS 전체 시스템 제어
  - 데이터 노드: 수천대의 서버로 구성, 데이터 저장 역할
  
- 데이터 처리 원칙
  - Block size: 파일을 저장하는 단위(64MB or 128MB)
  - Replication: 모든 블록은 여러 노드에 분산되어 저장(기본 3개)

- 파일 전송 과정
  
  ![image](https://user-images.githubusercontent.com/42403023/137571125-dd501e62-bfe0-4b42-8a9c-8c10d0e7b208.png)
  
  - 클라이언트는 저장하고자 하는 파일을 여러개의 블록으로 분리
  - 첫번째 블록을 저장하기 위해 네임 노드에 저장 작업 요청
  - 네임 노드는 해당 블록을 저장하기 위한 3개의 데이터 노드를 선정하여 클라이언트에게 목록 전달
  - 클라이언트는 전달받은 첫번째 데이터 노드에 데이터 전송
  - 첫번째 데이터 노드는 데이터를 받으면서 받은 데이터를 다음 데이터 노드에게 순차적으로 전달
  - 모든 데이터 노드들이 데이터 블록을 저장하고 나서 네임노드에 DONE 신호를 전송
  - 남은 블록들도 같은 방법으로 반복 처리, 마지막에 파일을 닫음
  - 네임노드는 해당 파일에 대한 메타데이터 저장

- 파일 수신 과정
  - 클라이언트가 받고자 하는 파일에 대한 정보를 네임노드에 요청
  - 네임노드는 해당 파일에 대한 모든 블록의 목록과 각 블록의 데이터 노드 목록을 클라이언트에게 회신
  - 클라이언트는 네임 노드로부터 받은 정보를 바탕으로 네트워크상에 가장 가까운 데이터 노드로부터 첫번째 블록을 다운 받음
  - 모든 블록을 데이터 노드로 부터 받고 나면 하나의 파일을 저장하고 수신 완료

- 오류 발생에 대한 장애 대응
  - 네임 노드 오류 발생: 모든 클러스터가 죽는 문제 발생
  - 데이터 노드 오류 처리
    - 데이터 노드는 네임 노드에 3초 단위로 heart beat를 전송
    - 네임 노드가 특정 데이터 노드의 heart beat를 10분 동안 받지 못하면 해당 데이터 노드가 죽었다고 판단
    
  - 데이터 송수신 시 오류 처리
    - 클라이언트가 데이터 노드에 데이터를 전송할 때마다 데이터 노드는 ACK 응답
    - ACK 응답이 오지 않으면 여러번 시도해보고 노드가 죽었거나 네트워크 오류로 판단

  - 데이터 체크섬 확인
    - 데이터 전송시 해당 데이터에 대한 체크섬을 같이 보냄
    - 데이터 노드에서 데이터를 하드디스크에 저장할 때 체크섬도 같이 저장
    - 데이터 노드는 주기적으로 네임 노드에 블록 리포트를 전송
    - 데이터 노드가 블록 리포트를 보내기 전에 체크섬이 맞는지 확인하고 손상된 블록은 제외하고 블록 목록을 작헝하여 보냄
    - 네임 노드는 블록 리포트를 통해 문제가 발생한 데이터 블록을 알아내고 조치할 수 있음

- 불완전 복제 방지
  - 네임 노드는 블록 목록과 데이터 노드 위치 목록을 관리
  - 2개의 목록을 지속적으로 업데이트하며 수시로 모니터링 진행
  - 장애가 발생한 노드를 찾으면 블록 목록과 데이터 노드 목록을 업데이트하고 오류가 발생한 데이터 노드와 블록 삭제
  - 오류 발생으로 복제 개수가 완전하지 않은 블록을 불완전 복제라고 함
  - 불완전 복제 블록을 제거하기 위해 데이터 노드에 새로운 복제소로 복사할 것을 요청하여 복제 개수를 맞춤

- 복제 위치 선정 전략
  - 클러스터는 여러 개의 데이터 노드를 가지는 랙으로 분리됨
  - 첫번째 복제소는 클라이언트 데이터가 같은 렉의 노드에 있으면 첫번째 복제소로 선정, 그렇지 않으면 랜덤으로 복제 노드를 선택
  - 첫번째 노드와는 다른 렉에서 2개의 다른 데이터 노드를 선택
  - HDFS는 적어도 1개의 복제소에 대해 확실한 보장을 하기 위해 최적의 복제소 선정

- 맵리듀스 과정
  
  ![image](https://user-images.githubusercontent.com/42403023/137571617-98f8bd3d-683c-4050-99d3-28be666d9076.png)
  
  - 원본데이터(파일, DB Record)는 map 함수에 의해서 <key, value> 쌍으로 전환
  - map() 함수: 입력을 출력 key와 관련되는 1~N개의 <key, value> 를 생성
    - map() 함수들은 병렬로 작동하며 여러 입력 자료셋으로 부터 여러 중간 value들을 생성
  - Map 단계 다음에서 출력 key 의 중간 value 들은 하나의 리스트로 합쳐짐
  - reduce() 함수: 같은 출력 key를 가지는 final value로 중간 value들을 통합
    - reduce() 함수 병렬로 작동하며 출력 key를 기준으로 각각 작업 수행
    - map 단계가 끝나지 않으면 reduce는 시작할 수 없다

#### Mongodb의 MapReduce

- Mongodb는 관계형 데이터베이스에서 제공하는 데이터 그룹함수들을 지원하고 있지 않다
- MapReduce 를 통해서 집계 함수 구현 가능

#### MapReduce 활용 1. WordCounting

- 입력파일의 텍스트 내용에 포함된 단어의 수를 세는 프로그램
- 두가지 함수의 사용자 인터페이스 구현
  - map(in_key, in_value) -> (inter_key, inter_value) list
  - reduce(inter_key, inter_value) -> (out_key, out_value) list

- 구현
  - 데이터 입력
    ```javascript
    db.words.save({'text': 'read a book'})
    db.words.save({'text': 'write a book'})
    ```
    
  - split mapper
    - 데이터 셋을 key, value의 리스트로 변경하는 map() 함수
    ```
    map = function() {
      var res = this.text.split(" ");
        for(var i in res) {
            key = {word: res[i]};
            value= {count: 1}
            emit(key, value);
        }
    }
    // ('text', 'read a book') -> ('read', 1), ('a', 1), ('book', 1)
    ```
  
  - sum reducer
    ```javascript
    reduce = function(key, values) {
        var totalCount = 0;
        for(var i in values) {
            totalCount += values[i].count;
        }
        
        return {count: totalCount};
    }
    // ('A', [42, 100, 312]) -> ('A', 454)
    // ('B' [12, 6, -2]) -> ('B', 16)
    ```
  - 맵리듀스 실행
    ```
    db.words.mapReduce(map, reduce, "wordcount");
    ```
  - 실행 결과 확인
    ```
    db.wordcount.find();
    { "_id" : { "word" : "a" }, "value" : { "count" : 2 } }
    { "_id" : { "word" : "read" }, "value" : { "count" : 1 } }
    { "_id" : { "word" : "book" }, "value" : { "count" : 2 } }
    { "_id" : { "word" : "write" }, "value" : { "count" : 1 } }
    ```

#### MapReduce 활용 2. Inverted Search Index

- value의 내용을 키로 하고, key의 내용을 반대로 value로 하는 패턴으로 inverting한 collection을 새로 만든다.
- 만약크롤링을 통해 URL에 대한 키워드를 저장한다면 key 는 URL, value 는 keyword가 된다.
  - 검색할 때에는 keyword로 검색하기 때문에 key는 keyword, value는 URL로 inverting 한 컬렉션을 다시 생성하는 것

- 두가지 함수의 사용자 인터페이스 구현
  - Inverted Mapper
    - 데이터셋을 key, value의 리스트로 변경하는 map() 함수
  - Combine Reducer

- 구현
  - 데이터 입력
   ```
   db.actors.save({'actor': "Ricard Gere", 'movies': ["Pretty woman", "Runaway bride", "Chicago"]});
   db.actors.save({actor: "Julia Roberts", movies: ["Pretty woman", "Runaway bride", "Erin brockovich"]});
   ```
  
  - map
    ```javascript
    map = function() {
        for(var i in this.movies) {
            key = {movie: this.movies[i]};
            value = {actors: [this.actor]};
            emit(key, value);
        }
    }
    ```

  - reduce
    ```javascript
    reduce = function(key, values) {
        actor_list = { actors: [] };
        for(var i in values) {
            actor_list.actors = values[i].actors.concat(actor_list.actors);
        }
        return actor_list;
    }
    ```

  - MapReduce 실행
    ```
    db.actors.mapReduce(map, reduce, 'pivot');
    ```
    
  - 결과 확인
    ```
    db.pivot.find();
    { "_id" : { "movie" : "Runaway bride" }, "value" : { "actors": [ "Ricard Gere", "Julia Roberts" ] } }
    { "_id" : { "movie" : "Pretty woman" }, "value" : { "actors": [ "Ricard Gere", "Julia Roberts"] } }
    { "_id" : { "movie" : "Erin brockovich" }, "value" : { "actors": [ "Julia Roberts" ] } }
    { "_id" : { "movie" : "Chicago" }, "value" : { "actors": [ "Ricard Gere" ] } }
    ```

#### 출처

- MapReduce 동작 원리: https://mrsence.tistory.com/16
- HDFS파일전달 원리: https://sites.google.com/site/medialoghadoop/01-hadub-gicho/03-hadub-bunsan-pail-siseutem
