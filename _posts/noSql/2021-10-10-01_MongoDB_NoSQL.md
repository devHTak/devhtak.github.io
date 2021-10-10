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
  - 장점
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

#### 출처

- CAP 이론: https://itwiki.kr/w/CAP_%EC%9D%B4%EB%A1%A0

