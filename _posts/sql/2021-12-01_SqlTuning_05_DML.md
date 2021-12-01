---
layout: post
title: SQL Tuning. DML
summary: SQL Tuning
author: devhtak
date: '2021-12-01 21:41:00 +0900'
category: SQL
---

#### 기본 DML 튜닝

- DML 성능에 영향을 미치는 요소
  - DML 성능에 영향을 미치는 요소
	  - 인덱스
		- 무결성 제약
		- 조건절
		- 서브쿼리
		- Redo 로깅
		- Undo 로깅
		- Lock
		- Commit
		
  - 인덱스와 DML 성능
    - 테이블 레코드를 입력하거나 삭제할 때에는 인덱스에도 입력하거나 삭제해야 한다.
		- 테이블은 Freelist(입력 가능한 블록 모음)통해 할당받지만, 인덱스는 정렬된 자료구조이기 때문에 수직적 탐색을 통해 입력할 블록을 찾아야 한다.
		- 레코드를 사게할 때에도 관련 인덱스 레코드를 모두 찾아 삭제해주어야 한다
		- 수정을 할때에는 인덱스 레코드를 삭제 후 삽입하는 방식으로 처리한다.
  - 무결설 제약과 DML 성능
    - 데이터 무결성 규칙
      - 개체 무결성
      - 참조 무결성
      - 도메인 무결성
      - 사용자 정의 무결성(또는 업무 제약 조건)
    - Check, Not Null 같은 제약은 조건을 확인하면 되지만 PK, FK 는 실제 데이터를 조회하기 때문에 성능에 더 큰 영향을 미친다.
  - 조건절과 DML 성능
    - UPDATE, DELETE 할 때 거는 조건절은 SELECT와 똑같은 실행계획을 갖는다.
  - 서브쿼리와 DML 성능
    - UPDATE, DELETE 할 때 거는 서브쿼리는 SELECT와 똑같은 실행계획을 갖는다.
  - Redo 로깅과 DML 성능
    - 오라클은 데이터파일과 컨트롤 파일에 가해지는 모든 변경사항을 Redo 로그에 기록한다.
    - Redo 로깅 목적
      - Database Recovery
      - Cache Recovery
      - Fast Commit
    - DML을 수행할 때마다 Redo 로그를 생성해야 하므로 Redo 로깅은 DML 성능에 영향을 미친다.
      - Insert 작업에 경우 Redo 로깅 생략 기능을 제공한다
  - Undo 로깅과 DML 성능
    - Undo 로깅은 Rollback에 대한 정보를 저장한다.
    - Redo에는 트랜잭션을 재현하는데 필요한 정보를 로깅하고, Undo는 변경된 블록을 이전 상태로 되돌리는 데 필요한 정보를 로깅한다.
    - Undo 로깅 목적
      - Transaction Rollback
      - Transaction Recovery(Instance Recovery시 rollback 단계)
      - Read Consistency
    - DML을 수행할 때마다 Undo 로그를 생성해야 하므로 Undo 로깅은 DML 성능에 영향을 미친다.
  - Lock과 DML 성능
    - Lock을 필요 이상으로 자주, 길게 사용하거나 isolation level을 높게 잡을 수록 DML 성능은 느려지며, 그렇다고 낮게 잡으면 데이터 품질이 낮아진다.
  - 커밋과 DML 성능
    - 커밋은 DML과 별개로 실행하지만, DML을 끝내려면 커밋까지 완료해야 하므로 서로 밀접한 관련이 있으며 
    - 특히 DML이 Lock에 의해 블로킹(Blocking)된 경우, 커밋은 DML 성능과 직결된다. DML을 완료할 수 있게 Lock을 푸는 열쇠가 바로 커밋이기 때문이다.
    - 모든 DBMS가 Fast Commit을 구현하고 있다. 구현방식은 서로 다르지만, 갱신한 데이터가 아무리 많아도 커밋만큼은 빠르게 처리한다는 점은 같다. Fast Commit의 도움으로 커밋을 순간적으로 처리하긴 하지만, 커밋은 결코 가벼운 작업이 아니다.
    - DB 버퍼캐시
      - DB에 접속한 사용자를 대신해 모든 일을 처리하는 서버 프로세스는 버퍼캐시를 통해 데이터를 읽고 쓴다. 버퍼캐시에서 변경된 블록(Dirty 블록)을 모아 주기적으로 데이터파일에 일괄 기록하는 작업은 DBWR(Database Writer) 프로세스가 맡는다.
    - Redo 로그버퍼
      - 버퍼캐시는 휘발성이므로 DBWR 프로세스가 Dirty 블록들을 데이터파일에 반영할 때까지 불안한 상태라고 생각할 수 있다. 하지만, 버퍼캐시에 가한 변경사항을 Redo 로그에도 기록해 두었으므로 안심해도 된다. 버퍼캐시 데이터가 유실되더라도 Redo 로그를 이용해 언제든 복구할 수 있기 때문이다.
      - 그런데 Redo 로그도 파일이다. Append 방식으로 기록하더라도 디스크 I/O는 느리다. Redo 로깅 성능 문제를 해결하기 위해 오라클은 로그버퍼를 이용한다. Redo 로그 파일에 기록하기 전에 먼저 로그버퍼에 기록하는 방식이다. 로그버퍼에 기록한 내용은 나중에 LGWR(Log Writer) 프로세스가 Redo 로그 파일에 일괄(Batch) 기록한다.
    - 트랜잭션 데이터 저장 과정
      - DML 문을 실행하면 Redo 로그버퍼에 변경사항을 기록한다.
      - 버퍼블록에서 데이터를 변경(레코드 추가/수정/삭제)한다. 물론, 버퍼캐시에서 블록을 찾지 못하면, 데이터파일에서 읽는 작업부터 한다.
      - 커밋한다.
      - LGWR 프로세스가 Redo 로그버퍼 내용을 로그파일에 일괄 저장한다.
      - DBWR 프로세스가 변경된 버퍼블록들은 데이터파일에 일괄 저장한다.
    - ‘커밋 = 저장 버튼’
      - 데이터베이스 트랜잭션을 문서 작업에 비유하면, 커밋은 문서 작업 도중에 ‘저장’ 버튼을 누르는 것과 같다. 서버 프로세스가 그때까지 했던 작업을 디스크에 기록하라는 명령어인 셈이다. 저장을 완료할 때까지 서버 프로세스는 다음 작업을 진행할 수 없다. Redo 로그버퍼에 기록된 내용을 디스크에 기록하도록 LGWR 프로세스에 신호를 보낸 후 작업을 완료했다는 신호를 받아야 다음 작업을 진행할 수 있다. Sync 방식이다. LGWR 프로세스가 Redo 로그를 기록하는 작업은 디스크 I/O 작업이다. 커밋은 그래서 생각보다 느리다.

- 데이터베이스 call과 성능
  - 데이터 베이스 Call
    - SQL 실행 단계
      - Parse Call:  SQL 파싱과 최적화를 수행하는 단계
      - Execute Call: SQL 을 실행하는 단계
      - Fetch Call: 데이터를 읽어 사용자에게 결과 집합을 전송하는 과정으로 SELECT 문에서만 나타난다.
    - Call이 어디서 발생하느냐에 따라 2가지로 나뉜다
      - User Call: 네트워크를 경유해 DBMS 외부로부터 인입되는 Call
      - Recursive Call: DBMS 내부에서 발생하는 Call
  - 절차적 루프 처리
    - PL/SQL 을 통한 데이터(100만개) insert
      - 루프를 돌며 데이터를 insert 한 후, commit
        - 한번에 commit하면 성능이 좋지만 Undo 로그 공간 부족 발생
        - 1건마다 commit하면 성능이 떨어지며 트랜잭션 원자성에도 문제가 생길 수 있다
      - 네트워크를 경유하지 않는 Recursive Call이 발생하기 때문에 빠르게 수행한다.
    - JAVA 소스를 통한 데이터(100만개) insert
      - 네트워크를 경유하는 User Call이 발생하기 때문에 성능이 떨어진다
  - One SQL 중요성
    - 단 한번의 Call 로 처리하기 때문에 빠르게 처리할 수 있다.
    - 종류
      - Insert Into Select
      - 수정가능 조인 뷰
      - Merge 문
- Array processing 활용
  - 한건을 insert 하여 건마다 User Call을 호출하는 것보다 한번에 insert하여 UserCall을 한번 발생하는 것이 성능에 좋다
  
- 수정가능 조인 뷰
  - 전통적인 방식의 UPDATE
  - 수정가능 조인 뷰
    - ‘조인 뷰’는 FROM 절에 두 개 이상 테이블을 가진 뷰를 가리키며, ‘수정가능 조인 뷰(Updatable/Modifiable Join View)’는 말 그대로 입력, 수정, 삭제가 허용되는 조인 뷰를 말한다. 
    - 단, 1쪽 집합과 조인하는 M쪽 집합에만 입력, 수정, 삭제가 허용된다.
  - 키 보존 테이블이란?
    - ‘키 보존 테이블’이란’, 조인된 결과집합을 통해서도 중복 값 없이 Unique 하게 식별이 가능한 테이블을 말한다. Unique한 1쪽 집합과 조인되는 테이블이어야 조인된 결과집합을 통한 식별이 가능하다.
    - 단적으로 말해 ‘키 보존 테이블’이란, 뷰에 rowid를 제공하는 테이블을 말한다.
  - ORA-01779 오류 회피
 
- MERGE 문 활용
  - DW에서 가장 흔히 발생하는 오퍼레이션은 기간계 시스템에서 가져온 신규 트랜잭션 데이터를 반영함으로써 두 시스템 간 데이터를 동기화하는 작업이다.
  - 예시
    - 고객(customer) 테이블에 발생한 변경분 데이터를 DW에 반영하는 프로세스는 다음과 같다. 이 중에서 3번 데이터 적재 작업을 효과적으로 지원하기 위해 오라클 9i에서 MERGE 문이 도입됐다.
    - 전일 발생한 변경 데이터를 기간계 시스템으로부터 추출(Extraction)
      ```
      CREATE 
      TABLE CUSTOMER_DELTA
      AS
      SELECT *
      FROM CUSTOMER
      WHERE MOD_DT >= TRUNC(SYSDATE) - 1
      AND MOD_DT < TRUNC(SYSDATE);
      ```
      - CUSTOMER_DELTA 테이블을 DW 시스템으로 전송(Transportaion)
      - DW 시스템으로 적재(Loading)
      ```
      MERGE INTO CUSOMER T USING CUSTOMER_DELTA S 
        ON (T.CUST_ID = S.CUST_ID)
      WHEN MATCHED THEN
        UPDATE SET T.CUST_NM = S.CUST_NM
      -- …
      WHEN NOT MATCHED THEN 
        INSERT (cust_id, cust_nm … ) VALUES( s.cust_id, s.cust_nm …)
        ```
  - MERGE 문은 Source 테이블 기준으로 Target 테이블과 Left Outer 방식으로 조인해서 성공하면 UPDATE, 실패하면 INSERT 한다. MERGE 문을 UPSERT(=UPDATE + INSERT)라고도 부르는 이유다. 위 MERGE 문에서 Source는 Customer_Delta 테이블이고, Target은 Customer 테이블이다.
		
#### Direct Path I/O 활용
	
- Direct Path I/O
  - 일반적으로 블록 I/O는 DB 버퍼캐시를 경유하여 읽고자 하는 블록을 먼저 버퍼캐시에서 찾아보고, 찾지 못할 때에만 디스크에서 읽는다.
  - 데이터를 변경할 때에도 먼저 버퍼캐시에서 찾은 후 변경한다. 이 후 DBWR 프로세스가 변경된 블록들을 주기적으로 찾아 데이터 파일에 반영
  - 하지만 대량 데이터를 읽고 쓸 때 건건이 버퍼캐시를 탐색한다면 찾을 가능성이 적기 때문에 성능이 안좋아진다.
  - 이럴 때 버퍼캐시를 통하지 않고 바로 디스크에서 스캔하는 것이 Direct Path I/O
  - Direct Path I/O 기능이 작동하는 경우
    - 병렬 쿼리로 Full Scan을 수행할 때
    - 병렬 DML 을 수행할 때
    - Direct Path Insert 를 수행할 때
    - Temp 세그먼트 블록들을 읽고 쓸 때
    - Direct 옵션을 지정하고 export를 수행할 때
    - Nocache 옵션을 지정한 LOB 컬럼을 읽을 때

  - 병렬쿼리
    ```
    SELECT /*+ full(t) parallel(t 4) */ * FROM big_table t;
    SELECT /*+ index_ffs(t big_table_x1) parallel_index(t big_table_x1 4) */ count(*) from big_table t;
    ```
    - Parallel or Parallel_index 힌트를 사용하면, 지정한 병렬도(parallel degree) 만큼 병렬프로세스가 떠서 동시에 작업을 진행한다.
    - Direct Path I/O 를 사용하여 빨라진다. 버퍼 캐시를 탐색하지 않고, 디스크로부터 버퍼캐시에 적재하는 부담도 없으니 빠른 것이다.
- Direct Path Insert
  - Freelist 란
    - 테이블 HWM(High-Water Mark) 아래쪽에 있는 블록 중 데이터 입력이 가능한(여유 공간) 블록을 목록으로 관리하는 것을 Freelist라 한다.
  - INSERT 가 느린 이유
    - 데이터를 입력할 수 있는 블록을 Freelist에서 찾는다.
    - Freelist 에서 할당받은 블록을 버퍼캐시에서 찾는다.
    - 버퍼캐시에 없으면, 데이터 파일에서 읽어 버퍼캐시에 적재한다.
    - INSERT 내용을 Undo 세그먼트에 기록한다.
    - INSERT 내용을 Redo 세그먼트에 기록한다.
  - Direct Path Insert 방식으로 입력하는 방법
    - INSERT … SELECT 문에 append 힌트 사용
    - Parallel 힌트를 이용해 병렬모드로 INSERT
    - Direct 옵션을 지정하고 SQL*Loader(sqlldr)로 데이터 적재
    - CTAS(create table … as select ) 문 수행
  - Direct Path Insert 빠른 이유
    - Freelist 를 참조하지 않고 HWM 바깥 영역에 데이터를 순차적 입력
    - 블록을 버퍼캐시에서 탐색하지 않는다
    - 버퍼캐시에 적재하지 않고, 데이터파일에 직접 기록
    - Undo 로깅을 하지 않는다.
    - Redo 로깅을 안하게 할 수 있다. 테이블 아래와 같이 nologging 모드로 전환한 상태에서 Direct Path Insert를 하면 된다(alter table t NOLOGGING;)
  - 주의점
    - Exclusive Mode TM Lock이 걸려 commit하기 전까지 해당 테이블에 DML을 수행하지 못한다.
    - Freelist를 조회햊 않기 때문에 테이블에 여유 공간이 있어도 재활용하지 않는다.

- 병렬 DML
  - UPDATE, DELETE 는 병렬 DML 처리를 통해 Direct Path Write 방식을 사용할 수 있다.
    ```
    Alter session enable parallel dml;
    ```
    - 병렬 DML 활성화
      - 힌트를 사용하여 Direct Path Write 방식을 사용할 수 있다.
        ```
        UPDATE /*+ full( c ) parallel (c 4) */ FROM 고객 c
        WHERE 최종거래일시 < '20211111';
        ```
    - 병렬 DML이 잘 작동하는지 확인하기 위해서는 실행계획에서 PX COORDINATOR 밑에 UPDATE가 있어야 한다.

#### 파티션을 활용한 DML 튜닝

- 테이블 파티션
  - 파티셔닝은 테이블 또는 인덱스 데이터를 특정 컬럼 값에 따라 별도 세그먼트에 나누어 저장
  - 시계열에 따라 Range 방식으로 분할하여 빠른 조회 및 관리가 쉽다.
  - 파티션이 필요한 이유
    - 관리적 측면: 파티션 단위 백업, 추가, 삭제, 변경 -> 가용성 향상
    - 성능적 측면: 파티션 단위 조회 및 DML, 경합 또는 부하 분산
  - Range Partition
    - 가장 기초적인 방식으로 주로 날짜 컬럼을 기준으로 파티셔닝한다.
    - 파티션 테이블에 값을 입력하면 각 레코드를 파티션 키 값에 따라 분할 저장
    - 조회 시 검색 조건에 만족하는 파티션만 골라 읽을 수 있어 이력성 데이터를 Full Scan 방식으로 조회할 때보다 성능이 향상된다.
  - Hash Partition
    - 파티션 키 값을 해시 함수에 입력해서 반환받은 값이 같은 데이터를 같은 세그먼트에 저장하는 방식
    - 조건절 비교값에 똑같은 해시 함수를 적용하여 읽을 파티션을 결정한다.
    - 해시 알고리즘 특성 상 등치, In-List 조건으로 검색할 때에만 파티션 Pruning이 동작한다.
  - List Partition
    - 사용자가 정의한 그룹핑 기준에 따라 데이터를 분할 저장하는 방식
    - 업무 친화도에 따라 그룹핑 기준을 정화되, 될 수 있으면 각 파티션에 값이 고르게 분산되도록 해야 한다.
  
- 인덱스 파티션
  - 테이블 파티션 구분
    - 비파티션 테이블
    - 파티션 테이블
  - 인덱스 파티션 구분
    - 로컬 파티션 인덱스
    - 글로벌 파티션 인덱스
    - 비파티션 인덱스
    - 로컬 파티션 인덱스는 각 테이블 파티션과 인덱스 파티션이 서로 1:1 대응 관계가 되도록 오라클이 자동으로 관리하는 파티션 인덱스
      - 로컬 파티션 인덱스가 아닌 모든 파티션 인덱스는 글로벌 파티션 인덱스라고 한다
  - 로컬 파티션 인덱스
    - 각 인덱스 파티션은 테이블 파티션의 속성을 그대로 상속받는다. 따라서 테이블 파티션 키가 주문일자면 인덱스 파티션 키도 주문일자가 된다.
    - 로컬 파티션 인덱스는 테이블과 정확히 1:1 대응 관계를 갖도록 오라클이 파티션을 자동으로 관리해 준다.
  - 글로벌 파티션 인덱스
    - 파티션을 테이블과 다르게 구성한 인덱스
      - 파티션 유형 or 파티션 키 or 파티션 기준값 정의
    - 테이블 파티션 구성을 변경하는 순간 Unusable 상태로 바뀌므로 곧바로 인덱스를 재생성해야 하며 그 동안 해당 테이블을 사용하는 서비스를 중단해야 한다.
  - 비파티션 인덱스
    - 하나의 인데스 구조가 여러 파티션을 가리킨다.
    - 테이블 파티션 구성을 변경하는 순간 Unusable 상태로 바뀌므로 곧바로 인덱스를 재생성해야 하며 그 동안 해당 테이블을 사용하는 서비스를 중단해야 한다.
  - Prefixed vs Nonprefixed
    - 인덱스 파티션 키 컬럼이 인덱스 구성상 왼쪽 선두 컬럼에 위치하는지에 따른 구분이 이뤄진다.
    - Prefixed : 인덱스 파티션 키 컬럼이 인덱스 키 컬럼 왼쪽 선두에 위치
    - Nonprefixed: 인덱스 파티션 키 컬럼이 인덱스 키 컬럼 왼쪽 선두에 위치하지 않고, 파티션 키가 인덱스 컬럼에 아예 속하지 않을 때도 여기에 속한다.
  - 중요한 인덱스 파티션 제약
    - Unique 인덱스를 파티셔닝하려면, 파티션 키가 모두 인덱스 구성 컬럼이어야 한다.



#### 참고

- 조시형 저자의 개발자를 위한 SQL 튜닝 입문서 친절한 SQL 튜닝
  - https://url.kr/fjm9l2
