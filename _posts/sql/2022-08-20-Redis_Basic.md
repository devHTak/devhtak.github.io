---
layout: post
title: Redis Basic
summary: Redis Basic
author: devhtak
date: '2022-08-20 21:41:00 +0900'
category: No SQL
---

#### Redis 특징

- 고성능 키-값 저장소로서 문자열, 리스트, 해시, 셋, 정렬된 셋 형식의 데이터를 지원하는 NoSQL
  - 단순 캐시 서버로 인식해서는 안된다.
    - Memcached 와 동일한 기능을 제공하지만 영속성, 다양한 데이터 구조와 같은 부가적인 기능을 지원하기 때문이다.
  - 모든 데이터를 메모리에 저장하고 조회(인메모리 데이터베이스 솔루션)
    - 영속성을 지원하는 인메모리 데이터 저장소
  - 읽기 성능 증대를 위해 서버 측 복제(replica) 를 지원
  - 쓰기 성능 증대를 위해 클라이언트 측 샤딩 지원
  - 키-값 저장소로 문자열, 리스트, 해시, 셋, 정렬된 셋과 같은 다양한 데이터형 지원

- NO SQL 특징
  - https://devhtak.github.io/no%20sql/2021/10/10/01_MongoDB_NoSQL.html 참고

#### Redis 시작

- 레디스와 데이터 구조
  - 레디스는 거대한 키-값 저장소로 거대한 맵(Map) 저장소이다
  - 키 하나에 데이터 형 하나가 저장되어 있는 단순한 구조로 이것이 레디스의 큰 장점이자 단점이다
    - 장점은 직관적이지만 단점은 저장된 데이터를 가공하는 방법에 제한이 있다는 것
- 레디스 데이터 구조와 명령어
  - 문자열 데이터
    - 저장가능한 최대 문자열 길이는 512MB 로, 크기가 넘어가는 경우 Error 발생
    - 레디스가 문자열 데이터를 저장할 때는 인코딩된 문자열과 몇가지 부가정보가 포함된 구조체로 변환하여 저장
    - 문자열 데이터의 입력과 조회
      - mset/mget: multi set or get (O(N) - N 개는 저장/조회 키 수)
      - getset: 키에 대한 값이 있으면 조회/ 없으면 저장(O(1))
      - incrby/decrby: 숫자의 증가와 감소 (O(1))
      - setbit/getbit: 비트연산(O(1))
      - strlen/bitcount: 문자열/비트 길이 조회(O(1))
  - 해시 데이터
    - 해시 데이터는 문자열 필드와 값으로 이루어진 맵 구조
    - 해시 데이터에는 2^32 - 1 개(42억개)의 필드와 값을 저장할 수 있다
    - 그룹데이터 저장
      - hmset: 주어진 필드와 값의 쌍을 해시 데이터에 저장 (O(1)
      - hsetnx: 주어진 필드가 존재하지 않을 때 저장, 존재한다면 값을 저장하지 않는다 (O(1))
      - hmget: 주어진 필드 목록을 주어진 키에 조회 (O(1))
      - hlen: 주어진 키에 저장된 필드의 개수 조회(존재하지 않으면 0, O(1))
      - hdel: 주어진 키에 저장된 필드를 제거(O(1))
    - 숫자의 증감(해시에 저장된 데이터의 필드값이 숫자일 때)
      - hincrby: 주어진 키에 저장된 필드에 숫자 증감처리(O(1))
      - hincrbyfloat:: 주어진 키에 저장된 필드에 숫자 증감처리(float, O(1))
    - 해시 데이터의 키 목록 조회
      - hkeys: 주어진 키에 저장된 모든 필드 목록 조회(O(N))
      - hvals: 주어진 키에 저장된 모든 값에서 필드 이름을 제외한 값의 목록 조회(O(N))
  - 셋 데이터
    - 중복을 허용하지 않는 집합 형태의 자료구조로 정렬되어 있지 않다.
    - 셋 데이터에는 2^32 - 1 개의 값을 저장할 수 있다
    - 집합 연산
      - sinter: 주어진 키에 저장된 요소들의 교집합을 돌려준다(O(N) - N은 가장 적은 수의 요소를 가진 셋에 저장된 요소 수)
      - scard: 주어진 키에 저장된 요소들의 개수를 돌려준다(O(1))
      - srem: 주어진 키에 저장된 요소를 제거하고 제거된 요소의 개수를 돌려준다(O(1))
      - spop: 주어진 키에 저장된 요소 중에 임의의 요소를 제거하고 제거된 요소를 돌러준다. 키가 없으면 nil을 돌려준다(O(1))
      - sismember: 입력된 요소가 주어진 키에 저장되어 있는 지 검사. 존재하지 않으면 0을 돌려준다(O(1))
      - smove: 원본키에 저장된 요소를 대상키로 이동하고 이동 결과를 돌려준다 (O(1))
  - 정렬된 셋 데이터
    - 셋 데이터와 동일한 특징을 가지면서 요소 정렬이라는 부가적인 특징을 갖는다
      - zadd key 가중치 value: key에 대하여 정렬된 셋 데이터를 저장하며 가중치를 정할 수 있다
      - zrevrange: 주어진 키에 저장된 요소들의 내림차순 정렬 순서 범위에 해당하는 요소 조회(O(log(N) + M))
    - 가중치 변경
      - 아이템의 가중치를 새로 지정하는 방법 외에 증가/감소 시킬 수 있다
      - zincrby: 주어진 키에 저장된 셋 데이터 중 지정된 요소의 가중치를 입력된 값만큼 증가(O(log(N)))
        - zincrby key 가중치 value
      - zrank: 주어진 키에 저장된 셋 데이터 중 지정된 요소의 순위 조회(O(logN))
        - zrank key value : 가중치 조회
        - zrevrank: 주어진 키에 저장된 셋 데이터 중 지정된 요소의 순위 조회(O(log(N))
        - zreverange key startIndex endIndex: value 들 조회
      - zrange:: 정렬된 셋의 범위 데이터를 오름차순으로 조회
        - zrangebyscore: 정렬된 셋의 점수 범위에 해당하는 데이터 조회
        - zremrangebyrank: 정렬된 셋에서 순위에 해당하는 범위의 데이터 제거
        - zremrangebyscore: 정렬된 셋에서 가중치에 해당하는 범위의 데이터 제거
        - zrevrange: 정렬된 셋의 범위 데이터를 내림차순으로 조회
    - 순위 처리와 조회
      - zscore: 주어진 키에 저장된 셋 데이터 중 지정된 요소의 가중치 조회 (O(1))
      - zrevrangebyscore: 정렬된 셋에 저장된 데이터의 가중치 범위에 해당하는 요소의 목록 조회 (O(logN))
    - 정렬된 셋 명령의 특별한 표현식
      - zcount: 가중치 범위의 값에 해당하는 요소의 개수를 조회
      - zrangebyscore: 가중치 범위의 값에 해당하는 요소의 목록을 오름차순으로 조회
      - zremrangebyscore: 가중치 범위의 값에 해당하는 요소 제거
      - zrevrangebyscore: 가중치 범위의 값에 해당하는 요소의 목록을 내림차순으로 조회
  - 리스트 데이터
    - 저장 순서를 기억하는 데이터 구조로 중복 허용
    - 2^32 - 1 개 저장 가능
    - 내부적으로 덱 구조를 구현하고 있으며 스택과 큐의 특징을 하나로 모은 자료구조
    - 스택 연산
      - lindex 키: 키 조회 인덱스 (O(N))
      - rpop 키: 키에 저장된 요소 중 맨 오른쪽 요소 조회 (O(1))
      - blpop 키 만료시간: 리스트에 저장된 요소 중 맨 왼쪽 요소 조회(O(1))
      - brpop 키 만료시간: 저장된 요소 맨 오른쪽의 요소 조회
    - 큐 연산
      - lpush key value / rpush key value
      - lpop key value / lpush key value
      - lrange key startIndex endIndex / rrange key startIndex endIndex
      - lindex key index
    - 블로킹 조회
      - 어떤 연산이 특정 조건이 될 때까지 대기하는 것
      - blpop brpop brpoplpush

- 키 관리
  - del key : 저장된 키와 값 삭제 (O(1))
  - rename 변경할key 변경될key:  (O(1))
  - expire key 만료시간: 지정된 키에 만료시간을 초 단위로 설정  (O(1))
  - ttl key: 지정된 키의 남은 만료시간을 초단위로 조회 (O(1))
  - exists key: 키가 존재하는 지 확인  (O(1))
  - expireat key ‘unix timestamp’: 만료시간 설정 (O(1))
  - persist key: 지정된 키의 만료시간을 없앤다
  - keys: 키 목록 조회

- 레디스 키 설계
  - NoSQL에 단순성을 해치지 않으면서 정보를 효율적으로 저정할 수 있는 방법이 키 설계이다.
  - 키에 의미를 부여하기 위해 콜론 기호를 사용
  - 키를 설계하는 방법
    - 관계형 데이터베이스의 스키마를 기본으로 하여 레디스의 저장구조로 바꾸는 방법
      - User 테이블(사용자번호,이름) / Posting 테이블(사용자번호, 작성글번호, 제목, 내용,작성일자)이 1:N 구조로 RDBMS 스키마가 정의되어 있다면
      - 레디스에서는 작성글:사용자번호:작성글번호 를 키값으로 제목/내용/날짜가 저장된 해쉬데이터 구조로 저장할 수 있다.
        - key: posting(작성글 테이블):11452(사용자번호):151(작성글번호)
      - 화면에 출력될 기준으로 하여 키를 설계하는 방법
      - 두 방법 모두 키에 부가적인 정보를 포함한다

#### 레디스 클라이언트

- 레디스는 http rest와 같은 범용 프로코콜을 사용하지 않고 전용 프로토콜을 사용하여 서버와 통신한다
- 레디스 클라이언트 라이브러리를 사용하면 된다
- 프로토콜 구조
  - 전송프로토콜
    ```
    * 인자 개수 CRLF
    $ 첫번째 인자의 개수 CRLF
    인자 CRLF
    ...
    $ N번째 인자의 개수 CRLF
    인자 CRLF
    ```
  - [ ] 수신 프로토콜
    - 첫번째 바이트가 응답 데이터 종류를 의미
    - 응답 데이터 종류에 따라서 응답 프로토콜의 구조가 달라진다
    ```
    + 상태응답
    - 에러응답
    : 숫자응답
    $ 단일벌크응답
    * 멀티벌크응답
    * ```
- 대량의 데이터 입력
  - 파이프라인
  
#### 출처
- 이것이 레디스다 초고속 읽기 쓰기를 제공하는 인메모리 기반 NoSQL, Redis: 한빛미디어, 정경석 저자