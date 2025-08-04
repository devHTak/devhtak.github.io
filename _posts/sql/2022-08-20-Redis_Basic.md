### Redis 특징
- 장점
  - 인메모리 데이터베이스는 뛰어난 성능과 간단하고 유연한 데이터베이스 기능, 간편한 설정이라는 장점
  - 속도가 빠르고, 자료형이 풍부하여 다양하게 활용할 수 있다.
- 단점
  - 레디스가 동작하는 메모리에는 SSD/HDD에 비해 가격이 높아 용량 확보가 어렵다
  - SQL 처럼 표현력이 뛰어난 수단이 없으며, 일부 트랜잭션 기능을 지원하지 않는다.
- Redis + RDBMS 같이 구성하여 사용하며 마스터 데이터는 RDBMS에, 처리된 결과 데이터를 캐시 데이터로 레디스에 저장하여 구성
- 속도가 빠르고 기능이 많은 인메모리 데이터 저장소
  - 인메모리 동작 기반으로 처리 속도가 빠르다
  - 자료형과 기능 및 관련 명령어가 다양
    - 다양한 자료형을 통히 ms 만에 응답하고 복잡한 데이터 구조를 저장하여 캐시 외에 다양한 용도로 활용 가능
  - 데이터 영속성 기능이 많다
    - 기본적으로 영속성 기능을 갖추고 있다.
    - 스냅샷 생성 기능, AOF 백업 기능으로 인메모리의 휘발성을 고려한 손실 위험 최소화
  - 레플리케이션/클러스터 기능을 통한 확장성 및 가용성이 높다.
    - 레플리케이션/클러스터 기능으로 확장성, 가용성 문제를 비교적 깔끔하게 해결할 수 있다.
  - 클라이언트/서버 모델 기반의 요청/응답 통신을 할 수 있다.
    - 클라이언트/서버 모델을 채택하여 요청/응답 방식으로 통신하며 다양한 환경에서 사용 가능
    - cli client로 redis-cli외에 telnet 등을 사용할 수 있다.
  - 루아(이페머럴 스크립트/레디스 함수)등 복잡한 로직을 구축하고 원자적으로 처리할 수 있다.
    - 루아 스크립트로 여러 개의 레디스 명령을 한번에 수행하여 원자적으로 수행 가능하다
    - 원자적이란 트랜잭션 내 작업이 모두 성공하거나 모두 실패한다는 것을 의미
  - 싱글 스레드 요청 이벤트를 주도적으로 처리할 수 있다.
    - 싱글 스레드 주도 처리 모델로 이벤트 루프를 형성하여 많은 요청을 처리할 수 있다.
    - 싱글스레드가 다른 지표에 비해 CPU 전체가 병목현상을 일으키는 경우는 비교적 드물다.

### 자료형과 기능
- 레디스의 주요 자료형은 String, List, Set, Sorted Set, Hash 형이 존재한다.
- 보조 자료형으론 비트맵, 지리적 공간 인덱스 등이 있으며 Pub/Sub 기능, HyperLogLog, redis stream 기능이 있다.
- 유틸리티성 명령어
  - KEYS : 키목록 확인
    - 실행 시간이 오래걸리기 때문에 운영 부하가 있다.
    - SCAN/SSCAN/HSCAN/ZSCAN 명령어인 SCAN 계열 명령어 사용 추천
  - EXISTS: 키 존재 여부 확인할 때 사용, 1(true)/0(false) 표시
  - TYPE: 자료형과 기능 확인 가능
  - DEL: 키 삭제
    - 운영 환경에서는 비동기적(background)인 형태로 삭제하기 위하여 unlink 사용
#### String
- String형에는 Bitmap 이라는 비트 단위 조작할 수 있는 보조자료형 존재
- 주요 특징
  - 키에 값을 일대일로 대응시키는 가장 간단한 자료형
  - 이진 안전 문자열
- 유스케이스
  - 캐시
    - 간단한 Key와 Value 쌍으로 대응되는 문자열
    - 세션 정보
    - 이미지 데이터 등 이진 데이터
  - 카운터
    - 방문자 수 등 접근 수 카운트
  - 실시간 매트릭스
    - 각 항목의 수치를 파악할 수 있는 지표
- 주요 명령어
  ```
  $ SET key value // 값 저장
  $ GET key // 조회
  
  $ MSET key1 value1 key2 value2 // bulk 저장
  $ MGET key1 key2 key3 key4 // bulk 조회

  $ SET key1 100
  $ TYPE key1
  string
  $ INCR key1 # 수 증가, 문자열인 경우 Error 발생
  $ DECR key1 # 수 감소
  $ INCRBY key1 increment # 수 increment 만큼 증가
  $ DECRBY key decrement
  $ INCRBYFLOAT key increment # 값을 지정한 부동소수점만큼 증가

  $ APPEND key value // 키가 존재하는 경우 키값 끝에 인수내용 추가
  $ STRLEN key // 키값의 문자열 길이 반환 O(1)
  $ GETRANGE key start end // 범위를 지정하여 값 가져오기
  $ SETRANGE key offset vlaue // 범위를 지정하여 값 저장

  $ SETEX key ttl value // ttl 과 함께 저장
  $ TTL key // key ttl 조회
  ```
#### List 형
- 특징
  - 문자열 컬렉션으로 삽입 순서를 유지
- 유스케이스
  - 스택, 큐, SNS 최신 게시물, 로그 등에 사용
- 주요 명령어
  ```
  $ LPOP key [count] // 왼쪽부타 값을 가져오고 삭제
  $ LPUSH key elemnet [element...] // 왼쪽부터 값 삽입
  $ RPOP key [count] // 오른쪽부터 값을 가져오고 삭제
  $ RPUSH key element [element..] // 오른쪽부터 값 삽입
  $ LMPOP numkeys key [key...] <left/right> [count] // 왼쪽 혹은 오른쪽부터 여러개의 값을 가져오고 삭제
  $ BLMPOP timeout numkeys key [key...] <left/right> [count] // 블록 기능을 갖춘 LMPOP
  $ LINDEX key index // 리스트에서 지정한 인덱스에 값을 조회
  $ LINSERT key before|after pivot element // 리스트에서 지정한 인덱스에 값을 삽입
  $ LLEN key // 리스트의 길이 가져오기 O(1)
  $ LRANGE key start stop // 리스트에 지정한 범위의 인덱스에 있는 값 가져오기
  $ LREM key count element // 리스트에서 지정한 요소를 지정한 수만큼 삭제
  $ LSET key index element // 리스트에서 지정한 인덱스에 있는 값을 지정한 값으로 저장
  $ LTRIM key start stop // 지정한 범위 인덱스에 포함된 요소로 리스트 갱신
  $ LPOS key eleemnt [rank] [count num-matches] [maxlen] // 리스트 중 지정한 인덱스에 있는 값 가져오기
  ```

#### Hash 형
- 특징
  - 필드와 값의 쌍 집합
  - 필드와 연결된 값으로 구성된 맵, 필드와 값 모두 문자열
  - 하나의 Hash에 저장할 수 있는 빌드 수는 2^32 - 1개로 메모리에 여유가 있다면 제한이 없다고 봐도 무방
  - Hash형 키에 값을 계속 추가하면, 볼륨이 거대해지기 때문에 명령어를 실행할 때 처리 시간에 문제가 생길 수 있다.
- 유스케이스
  - 객체 표현(ex. 각 사용자를 객체로 간주하여 사용자별로 이름, 나이 등 정보를 저장하는 경우)
- 주요 명령어
  ```
  $ HDEL key field // 해시에 지정한 필드 삭제
  $ HEXISTS key field // 패시에 지정한 필드가 존재하는 지 확인
  $ HGET key field // 해시에 지정한 필드값 가져오기 O(1)
  $ HGETALL key // 해시에 모든 필드 및 지정된 값 쌍 가져오기 O(N)
  $ HKEYS key // 해시에 지정한 모든 필드 목록 반환 O(N)
  $ HLEN key // 키로 지정한 해시에 포함된 필드 수를 반환 O(1)
  $ HMSET key field value [field vlaue...] // key 해시에 field / value 쌍 bulk 저장
  $ HSET key field value // key 해시에 field / value 쌍 저장
  $ HVALS key // 해시의 모든 필드값 가져오기
  $ HSCAN key cursor [MATCH pattern] [COUNT num] // 반복 처리하여 해시의 필드와 그에 연결된 목록 가져오기
  
  $ HINCRBY key field increment // 해시에 지정한 필드 값을 지정한 수만큼 증가
  $ HINCREBYFLOAT key field increment // 해시에 지정한 필드 값을 지정한 부동소수점 수만큼 증가
  ```
- 성능 발휘를 위한 주의사항
  
### 고급 기능

#### 파이프라인

#### 루아

#### 트랜잭션

#### 모듈

#### 키공간 알림
#### 클라이언트 측 캐싱
