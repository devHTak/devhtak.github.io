---
layout: post
title: NOSQL 특징 및 모델링 기법 그리고 MongoDB
summary: JUnit 사용법
author: devhtak
date: '2021-10-10 21:41:00 +0900'
category: No SQL
---

#### NOSQL

- NOSQL 특징
  - Not only SQL의 약자로 기존의 RDBMS 형태의 관계형 데이터베이스가 아닌 다른 형태의 데이터 저장 기술을 의미
  - RDBMS 제품군 등과 같이 공통된 형태의 데이터 저장 방식(테이블) 과 접근 방식(SQL)을 가즌 것이 아닌, RDBMS와 다른 형태의 저장 구조를 총칭
  - NOSQL은 데이터간의 관계를 정의하지 않는다.
  - 분산형 구조를 통해 데이터를 여러 대의 서버에 분산 저장, 분산 시에 데이터를 상호 복제해 특정 서버에 장애가 발생했을 때에도 데이터 유실이나 서비스 중지가 없는 형태
  - 분산형 구조를 띄고 있기 때문에 CAP 이론을 띄고 있다.
  
  - CAP 이론: 분산 컴퓨팅 환경은 Consistency, Availability, Partitioning 3가지 특징을 가지고 있으며, 이중 2가지만 만족한다는 이론
    - 일관성(Consistency): 모든 노드들은 같은 시간에 동일한 항목에 대하여 같은 내용의 데이터를 사용자에게 보여준다.
    - 가용성(Availability): 모든 사용자들이 읽기 및 쓰기가 가능해야 하며, 몇몇 노드의 장애 시에도 다른 노드에 영향을 미치면 안된다.
    - 분할내성(Partition tolerance): 메시지 전달이 실패하거나 시스템 일부가 망가져도 시스템이 계속 동작할 수 있어야 한다.
    - CP Category: There is a risk of some data becoming unavailable (MongoDB, HBase, Memcache, BigTable, Redis)
    - CA Category: Network problem migigt stop the system (RDBMS)
    - AP Category: Clients may read inconsistent data (Cassandra, RIAK, CouchDB)

- NOSQL 장단점
  - 특정 용도로 특화되어 있기 때문에 솔루션의 특징을 알 필요가 있다.
  - 데이터 분산에 용이하고 Join 연산은 대부분 불가능하다. 즉, 데이터 모델 자체가 독립적으로 설계되어 있어 데이터를 여러 서버에 분산시키는 것이 용이함
  - 데이터에 대한 캐시가 필요하거나 배열 형식의 데이터를 고속으로 처리할 필요가 있는 경우에 사용

- NOSQL 종류
  - Key/Value Store
    - 대부분의 NOSQL은 Key/Value 개념을 지원
    - Unique한 Key에 하나의 Value를 가지고 있는 형태
    - put(key, value), value := get(key) 형태 API 사용 (ex, redis)

  - Ordered Key/Value Store
    - Key 기준으로 sorting
    - Key 안에 column:value 조합으로 된 여러개의 필드를 가지는 구조
    - Hbase, cassandra

  - Document Key/Value Store
    - Key Value Store 의 확장된 형태
    - 저장되는 Value 의 데이터 타입으로 "Document"라는 구조화된 데이터 타입(JSON, XML, YAML 등)을 사용
    - 복잡한 계층 구조 표현 가능
    - 제품에 따라 추가 기능(Sorting, Join, Gouping) 지원

  - NoSQL System List
    - Key-Value Stores: Oracle Cohrerence, Redis, Kyoto Cabinet
    - BigTable-style Databases: Apache HBase, Apache Cassandra
    - Document Databases: MongoDB, CouchDB
    - Full Text Search Engines: Apache Lucene, Apache Solr
    - Graph Databases: neo4j, FlockDB

#### NOSQL 데이터 모델링

- Denormalization(비정규화)
  - 비정규화는 데이터 중복을 허용한다는 의미
  - 쿼리 프로세싱을 간단하게 하거나 최적화하기 위해서 또는, 사용자의 데이터를 특정한 데이터 모델에 맞추기 위해서 같은 데이터를 여러 도큐먼트나 테이블에 복제하여 중복하는 것을 허용
  - 비정규화로 인해 쿼리 수행 복잡도는 낮아지지만, 전체 데이터 사이즈는 커진다. 하지만 NOSQL은 분산환경, 대용량 데이터에 적합하다.

- Aggregates
  - 유연한 스키마 속성은 복잡하고 다양한 구조의 내부 요소(nested entities) 를 가지고 있는 데이터 클래스를 구성 가능하게 한다.
  - 1:N 관계를 최소화하여 결과적으로 Join 연산을 줄인다. (수행시간 단축)
  - 복잡하고 다양한 비즈니스 요소를 담을 수 있다. (추후 확장성 및 변동성에 대한 유연한 대응 가능)
  - 예제
    - MP3 File
      - RDBMS: musicId, title, artist
      - NoSQL: File { type: "mp3", 공통필드(fileID, size, data, name, directory), meta: { title: "music title", artist: "singer" }
    - Photo File
      - RDBMS: photoID, format
      - NoSQL: File { type: "photo", 공통필드(fileID, size, data, name, directory), meta: { format: "JPG" }
    - Movie File
      - RDBMS: movieID, format, title, length
      - NoSQL: File { type: "photo", 공통필드(fileID, size, data, name, directory), meta: { title: "Movie title", format: "mp4", length: "1:40" }

- Application Side Joins
  - 대용량 데이터에 대해 빠른 응답 성능과 확장성, 가용성을 최우선 목적으로 하기 때문에 쿼리 타임 조인을 최대한 피하여 데이터 모델을 구성하는 것을 가정함
  - Join 대상 데이터에 대해 비정규화, 어그리게이션을 수행할 때 문제가 발생하는 경우
    - JOIN 대상 데이터가 다대다 관계일 경우
    - JOIN 대상 데이터의 수시 변동
  - 대상 데이터가 수시로 변경되는 경우, 비정규화와 어그리게이션을 통해 많은 entity에 해당 데이터의 중복을 허용했기 때문에 해당 데이터 업데이트 시에 모두 찾아서 업데이트해야 하기 때문에 많은 비용 발생
  - 이런 경우 차라리 변경이 잦은 데이터만ㅇ르 추려내어 쿼리 타임 조인을 수행하는 것이 대안
    |Application|No SQL|
    |---|---|
    |V1=Get(key) from TABLE1|TABLE1|
    |V2=Get(TABLE.FK) from TABLE2|TABLE2|
    |Return V1, V2|-|
    - DB 상에서 Join하는 것이 아닌, Application에서 가져와 사용

- General Modeling Techniques
  - Composite Key
    - 하나 이상의 필드를 deliminator를 이용하여 구분 지어 사용하는 방법
    - Ordered KV Store 의 경우에는 이를 이용하여 order by와 같은 sorting 기능이나 grouping 구현 가능
    - NoSQL N개의 서버로 구성된 클러스터로 동작하고 데이터는 Key를 기준으로 N개의 서버에 나누어 저장
    - Key를 선정할 때는 전체 서버에 걸쳐서 부하가 골고루 분산될 수 있는 Key를 선정

  - Inverted Search Index
    - Value 의 내용을 Key로 하고, Key의 내용을 반대로 value로 하는 패턴
    - 검색엔진에서 많이 사용하는 방법
      - 검색엔진은 사이트의 모든 페이지를 검색 로봇이 검색해서 문서내의 단어들을 색인하여 URL 에 매핑해서 저장
      - 검색은 단어를 Key로 검색하기 때문에, value에 검색 키워드들이 들어가 있을 경우에는 효과적인 검색이 불가능함
      - 검색 키워드를 키로 해서 URL을 value로 하는 테이블을 다시 만든 다음, 검색 키워드로 검색을 하면 신속하게 검색 키워드를 가지고 있는 URL을 찾아낼 수 있음

- 계층 데이터 구조 모델링 패턴
  - NOSQL은 다양한 데이터 모델이 있지만, 기본적으로 row, column을 가지고 있는 테이블 구조의 저장 구조를 갖는다.
  - 어플리케이션 개발 중, 이런 테이블 구조뿐 아니라 Tree와 같은 계층형 데이터 구조를 저장해야 할 경우가 있는데, NOSQL은 이런 계층형 구조를 저장하는 것이 쉽지 않음
  - RDBMS의 경우에도 이런 계층형 구조를 저장하기 위해서 많은 고민을 해왔기 떄문에, 솔루션에서 기능적으로 자체 지원하기도 하고 데이터 모델링을 통해서 계층형 구조를 저장할 수 있음
  - NoSQL에서 계층형 구조를 저장하는 기법은 RDBMS에서 사용하는 기법들을 참고하여 구현함

  - Tree Aggregation
    - Tree 구조 자체를 하나의 Value에 저장하는 방식
    - JSON이나 XML 등을 이용하여 트리 구조를 정의하고, Value에 저장
    - Tree 자체가 크지 않고, 변경이 많이 없는 경우에 적합
    - Blog Post -> Content -> Comments1 -> Comments1-1...

  - Materialized Path
    - Tree 구조를 테이블에 저장할 때, root에서부터 현재 노드까지의 전체 경로를 key로 저장하는 방법
    - 구현에 드는 노력에 비해 매우 효율적인 저장 방식
    - Key에 대한 Search를 할 때 Regular Expression을 사용할 수 있으면, 특정 노드의 하위 트리등을 쿼리해 오는 기능 등 다양한 쿼리가 가능

- 데이터 모델링 예시
  - 도메인 모델 파악
    - 어떤 데이터 개체가 있는지, 개체간의 관계 분석
    - 저장할 데이터를 명확하게 이해하기 위해 필요(RDBMS와 동일)

  - 쿼리 결과(데이터 출력 형태) 디자인
    - 도메인 모델 기반으로 어플리케이션에서 쿼리가 수행되는 결과값을 먼저 정함
    - 출력 형식을 기반으로 필요한 쿼리 정의
    - 출력 데이터를 기반으로 테이블 추출
    - RDBMS와는 정반대의 순서를 가지고 있다.

  - 패턴을 이용한 데이터 모델링
    - NoSQL은 Sorting, Grouping, Join 등의 연산을 제공하지 않기 때문에 Get/Put으로만 데이터를 처리할 수 있는 형태로 데이터 모델 재정의
    - 예시
      |Table|Key||Value|
      |---|---|---|
      |Category|userId:categoryID|categoryName(value)|
      |Posting|userID:postID|categoryName(value), title(value), date(value), contents(value)|
      |Attechment|userID:postID:fileID|filename(value), date(value), location(value)|
      |comment|userID:postID:commentID|commentUser(value), date(value), comment(value)|
      
  - 기능 최적화
    - 첨부파일: 포스팅에 의존적이며 변경이 적고, 개수가 많지 않기 때문에 하나의 필드에 모아서 저장하는 것이 나음
    - 분류에 따른 포스팅 출력
      - 현재는 포스팅 순서대로만 출력 가능
      - 포스팅에 분류 필드를 별도로 넣고, 필드에 따라 where문으로 select 할 수 있어야 함(Secondary Index)
      - Posting: userID:postID : categoryName(value), title(value), date(value), contents(value), attach(value), categoryID(value)

  - NoSQL 선정 및 테스트
    - 모델링한 데이터 구조를 효과적으로 실행할 수 있는 NoSQL 검토
    - NoSQL 특성 분석 및 부하테스트, 안정성/확장성 테스트 수행
    - 경우에 따라서 여러 개의 NoSQL을 복잡하여 사용
    - 하나의 NoSQL만으로 모든 데이터를 처리하려고 하지 말고, RDBMS와 혼용하거나 다른 NoSQL을 사용하여 최적의 시스템 설계
  
  - 선정된 NoSQL에 최적화된 하드웨어 디자인까지 해야 한다.

#### MongoDB

- MongoDB 특징
  - 메모리맵 형태의 파일 엔진 DB이기 때문에 메모리에 의존적
    - 메모리 크기가 성능을 좌우
    - 메모리를 넘어서는 경우 성능이 급격히 저하
  - 쌓아놓고 삭제가 없는 경우 적합
    - 로그 데이터
    - 이벤트 참여 내역
    - 세션
  - 트랜잭션이 필요한 금융, 결제, 빌링, 회원 정보등에는 부적합
  - 도큐먼트 데이터 모델(데이터 설계를 종이문서 설계하듯이 설계해야 한다)
    - 속성의 이름과 값으로 이뤄진 쌍의 집합
    - 속성은 문자열이나 숫자, 날짜 가능
    - 배열 또는 다른 도큐먼트를 지정하는 것도 가능
    - 하나의 document에 필요한 정보를 모두 담아야 함
    - one query 로 모두 해결이 되게끔 collection model 설계를 해야 한다
    - join이 불가능하므로 미리 embedding 시켜야 함
  - JSON 형태로 되어 있다
  - 예제
    ```
    * Posts: id, title, body, published, created, updated, tags, comments: comment_id, author, email, body, created
    {
      "id": 1,
      "title": "Welcome to Mongo DB",
      "body": "Today, we...",
      "published": true,
      "created": "2021-10-10 13:00",
      "updated": "2021-10-10 13:00",
      "tags": ["databases", "MongoDB", "awesome"],
      "comments": [
        { comment_id: 2, "author": "Bob", "email": "bob@test.com", "body": "Good!", "created": "2021-10-10 13:05" }
      ]
    }
    ```

- 장점
  - Schema-less 구조
    - 다양한 형태의 데이터 저장 가능
    - 데이터 모델의 유연한 변화 기능(데이터 모델 변경, 필드 확장 용이)
  - Read/Write 성능 뛰어남
  - Scale out 구조
    - 많은 데이터 저장이 가능하며 장비 확장이 간단하다
  - Json 구조
  - 사용 방법이 쉽고, 개발이 편리하다. 

- 단점
  - 데이터 업데이트 중 장애 발생 시, 데이터 손실 가능
  - 많은 인덱스 사용 시, 충분한 메모리 확보 필요
  - 데이터 공간 소모가 RDBMS에 비해 많음 (비효율적인 Key 중복 입력)
  - 복잡한 JOIN 사용 시 성능 제약이 따름
  - Transactions 지원이 RDBMS 대비 미약함
  - 제공되는 MapReduce 작업이 Hadoop에 비해 성능이 떨어짐

- 빅데이터 처리 특화
  - Memory Mapped(데이터 쓰기 시에 OS의 가상 메모리에 데이터를 넣은 후 비동기로 디스크에 기록하는 방식)을 사용
  - 방대한 데이터 빠르게 처리 가능
  - OS의 메모리를 활용하기 때문에 메모리가 차면 하드디스크로 데이터 처리하여 속도가 급격히 느려짐
  - 하드웨어적인 측면에서 투자 필요

- MongoDB 불안전성
  - 데이터 양이 많을 경우 (중간 중간 일부 데이터를 유실할 수 있다)
    - 일부 데이터가 손실 가능성이 존재
    - 샤딩의 비정상적인 동작 가능성
    - 레플리카 프로세스의 비정상 동작 가능성

  - 하지만, 동일한 데이터를 가지고 CRUD를 할 때 RDBMS보다 빠르게 진행하며 Single Node와 Multi Node간에 성능 차이가 없다.
    - 쓰기: 100배, 읽기: 3배 정도 빠르다.

- 주요 기능
  - Query
    - Create: db.person.save({'name': 'John'});
    - Read: db.person.find();
    - Update: db.person.update({'name':' 'John'}, {'name': 'Cash', 'languages': ['English']});
    - Delete: db.person.remove({'name':' 'John'});

  - 인덱스
    - 다수 인덱스 설정 가능하며 복합 인덱스를 지원
    - 빠른 검색 지원
    - 도큐먼트에 저장된 데이터와 중복 저장 문제
    - 메모리가 부족한 시스템에서는 검색 속도 저하 문제

  - 복제
    - Master-Slave 구조 구성
    - 데이터 복사본을 Slave에 배치
    - Master 장애에 따른 데이터 손실 없이 Slave 데이터 사용 가능
    - ReplicaSet 사용 시 Master 장애가 발생했을 때, Slave에서 master를 선출 가능(중단없이 서비스 가능)
    - 데이터 손실을 최소화하기 위해 저널링 지원(MongoDB의 데이터 변화에 따른 모든 연산에 대한 로그 적재, 데이터 손실 발생 시 로그를 통한 복구 가능)

  - 샤딩
  
    ![image](https://user-images.githubusercontent.com/42403023/136687236-35f32889-a0ec-40e5-bc2c-d80871aed33b.png)
    
    - 대용량의 데이터를 저장하기 위한 방법
      - 소프트웨어 적으로 데이터베이스를 분산시켜 처리하는 구조
    - 샤딩 방식
      - 데이터베이스가 저장하고 있는 테이블을 테이블 단위로 분리하는 방법
      - 데이터베이스가 저장하고 있는 테이블 자체를 분할하는 방법
    - 분산 데이터베이스의 전통적인 분할 3계층 구조 지원
      - 응용 계층, 중개자 계층, 데이터 계층
      - 응용 계층은 데이터에 접근하기 위해 중개자를 통해 모든 데이터의 출력을 처리
      - 추상화된 한개의 데이터베이스가 존재하는 것처럼 운용
      
  - MapReduce

    ![image](https://user-images.githubusercontent.com/42403023/136687357-9d2b07d5-fd91-44ff-81a8-f213b34c1486.png)

    - 대용량의 데이터를 안전하고 빠르게 처리하기 위한 방법
      - 데이터를 분산하여 연산하고 다시 합치는 기술
      - 맵과 리듀스 단계로 나누어 처리, 사용자가 임의 코딩 가능
      - 입/출력 데이터는 Key-Value 형태로 구성
    - 한대 이상의 하드웨어를 활용하는 분산 프로그래밍 모델
      - 분산을 통해 분할된 조각으로 처리한 다음, 다시 모아서 훨씬 짧은 시간에 계산완료
    - 대용량 파일에 대한 로그 분석, 색인 구축 검색등에 활용
    - 일괄처리 방식으로 전체 데이터 셋을 분석할 필요가 있는 문제에 적합
    - input data를 split한 후, 각 서버에서 Map함수를 통해 각각 output을 만들고, shuffle/sort를 진행한 후 reduce하여 최종 output을 만들어 낸다.

#### 출처

- CAP 이론: https://itwiki.kr/w/CAP_%EC%9D%B4%EB%A1%A0
- shading 이미지 파일 출처: https://sudarlife.tistory.com/entry/mongodb-Sharding-%EB%AA%BD%EA%B3%A0%EB%94%94%EB%B9%84-%EC%83%A4%EB%94%A9-%EC%A0%81%EC%9A%A9%ED%95%98%EA%B8%B0-config-sever-replica-set
- map reduce 이미지 파일 출처: https://medium.com/@ekiny018/mongodb-and-mapreduce-64af7efd038
- T아카데미 유투브 강의, MongoDB 프로그래밍: https://www.youtube.com/playlist?list=PL9mhQYIlKEheyXIEL8RQts4zV_uMwdWFj
