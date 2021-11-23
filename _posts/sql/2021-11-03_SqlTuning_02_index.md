---
layout: post
title: SQL Tuning. index 기본
summary: SQL Tuning
author: devhtak
date: '2021-11-03 21:41:00 +0900'
category: SQL
---

#### 인덱스 구조 및 탐색

- 미리보는 인덱스 튜닝

  - 데이터베이스 테이블에서 데이터를 찾는 방법 2가지
    - 테이블 전체 스캔
    - 인덱스 이용

  - Scan 의 종류
    - index scan(Sequential 액세스)
      - 레코드간 논리적 또는 물리적인 순서를 따라 차례대로 읽어 나가는 방식
      - 인덱스 리프 블록에 위치한 모든 레코드는 포인터를 따라 논리적으로 연결돼 있고, 이 포인터를 따라 스캔하는 것은 Sequential 액세스 방식
    - Table random access(Random access)
      - Random 액세스는 레코드간 논리적, 물리적인 순서를 따르지 않고, 한 건을 읽기 위해 한 블록씩 접근하는 방식
      
  - 인덱스 튜닝의 핵심요소
    - 인덱스 스캔 효율화 튜닝
      - 인덱스 스캔과정에서 발생하는 비효율을 줄이는 것
    - 테이블 랜덤 액세스 최소화 튜닝
      - 테이블 액세스 횟수를 줄이는 방법
      - 인덱스 스캔 후 테이블 레코드를 액세스할 때 랜덤 I/O 방식을 사용

    ![image](https://user-images.githubusercontent.com/42403023/140036433-ca7cf8ec-0d6d-4f98-a6af-f9349a60c5d5.png)

    - 인덱스 스캔 효율화보다 테이블 랜덤 액세스 최소화 튜닝이 성능 향상에 더 좋다.
    - SQL 튜닝은 랜덤 I/O를 줄이는 것이 중요하다

- 인덱스 구조
  - B* Tree
    
    ![image](https://user-images.githubusercontent.com/42403023/140037570-a8c21e4b-6bf1-4ee5-8640-b50bd1b1a654.png)

    - 블록에 있는 레코드가 가리키는 하위 블록에는 해당 값에 대한 범위로 값을 갖는 블록이 존재한다.
    - 각 레코드는 키값 순으로 정렬돼 있을 뿐만 아니라 테이블 레코드를 가리키는 주소값(rowid)을 갖는다
      - rowid: 데이터 블록 주소 + row 번호
      - 데이터 블록 주소: 데이터 파일 번호 + 블록 번호
      - 블록 번호: 데이터 파일 내에서 부여한 상대적 순번
      - 로우 번호: 블록 내 순번

    - 인덱스 탐색 과정
      - 수직적 탐색: 인덱스 스캔 시작지점을 찾는 과정
        - 정렬된 인덱스 레코드 중 조건을 만족하는 첫 번째 레코드를 찾는 과정으로 인덱스 스캔 시작지점을 찾는 과정
        - 루트 블록에서 시작하여 하위 블록에 대한 주소값과 범위에 대한 기준으로 리프 블록까지 수직적 탐색을 하는 과정
        - 범위와 같은 기준을 통해 조건을 만족하는 첫번째 레코드를 만날 수 있다.

      - 수평적 탐색: 데이터를 찾는 과정
        - 데이터를 찾는 과정
        - 트리의 구성은 양방향 연결리스트이기 때문에 앞뒤 블록에 대한 주소값을 갖기 때문에 수평적 탐색이 가능하다
        - 수평적 탐색 이유
          - 조건절을 만족하는 데이터를 모두 찾기 위해서
          - 테이블 스캔을 위한 rowid를 얻기 위해서

#### 인덱스 기본 사용법

- 인덱스를 사용한다는 것
  - index 기본 사용법은 range scan을 의미하며 index 확장 기능은 range scan 외에 다양한 스캔 방식을 말한다.
  - index range scan 방식
    - 스캔 시작점을 찾아 시작점부터 스캔을 하며 중간에 멈추는 것을 말한다.
    - 예시) 사전에서 글로벌이란 단어를 찾는다면 ㄱ 이라는 시작 지점으로 부터 찾을 것이다.
  - index full scan 방식
    - 인덱스를 사용할 수 있지만, 스캔 시작점을 찾을 수 없고, 멈출 수도 없이 리프 블록 전체를 스캔해야 되는 것
    - 예시) 사전에서 '인덱스'가 포함된 단어를 찾는다면 시작점을 찾을 수 없고, 멈출 수 없고 색인 전체를 스캔해야 한다.
  - index range scan 을 사용하지 못하는 경우
    - 인덱스 컬럼을 가공하는 경우 range scan을 하지 못한다.
    
    ```
    - SUBSTR(BIRTHDAY, 2, 5) = '05' // 생년월일(YYYYMMDD)이 index이지만 생년 월일이 5월인 학생을 찾을 때
    - NVL(ORD_COUNT, 0) < 100 // NULL값이 0으로 치환한 값
    - NAME LIKE '%현%' // LIKE 연산으로 중간값을 찾는 경우
    - PHONE_NUMBER = :phoneNumber OR CUSTOMER_NAME = :customerName // OR 연산에 경우 어느 한 시작점을 찾을 수 없다
     // OR 연산, IN 연산에 경우 Index Full Scan 이 발생하기 때문에 UNION ALL로 해결할 수 있다
    ```
    - 결합 인덱스
      ```
      -- index: [TEAM + MEMBER_NAME + MEMBER_AGE]
      SELECT MEMBER_ID, TEAM, MEMBER_AGE, JOIN_DATE, PHONE_NUMBER
      FROM MEMBER
      WHERE MEMBER_NAME = '홍길동';
      ```
      - 인덱스 구성에 의해 TEAM 순으로 정렬 -> MEMBER_NAME 순으로 정렬 -> MEMBER_AGE 순으로 정렬되었다
      - MEMBER_NAME이 아닌 TEAM이 기준이기 때문에 하나의 시작점을 찾을 수 없다. 즉 전체 리프 블록을 모두 스캔해야 한다

  - index scan이 가능한 경우
    - 인덱스 선두 컬럼이 가공되지 않은 상태로 조건절에 있으면 index range scan이 가능하다

- 인덱스를 활용한 소트 연산 생략
  - 인덱스를 통해 데이터가 기본적으로 sorting이 되어 있다.
  - 그렇기 때문에 옵티마이저는 인덱스의 속성을 활용하여 ORDER BY가 있어도 정렬 연산을 따로 수행하지 않는다.
  - ASC(오름차순)에 경우 좌측으로 수직적 탐색을 한 후 우측으로 수평적 탐색을 한다
  - DESC(내림차순)에 경우 가장 큰 값을 찾기 위해 우측으로 수직적 탐색을 한 후 좌측으로 수평적 탐색을 한다.
  
- ORDER BY 에서 컬럼 가공
  - 조건절이 아닌 order by 또는 select list 에서 컬럼을 가공함으로 인해 인덱스를 정상적으로 사용할 수 없는 경우가 발생한다.
    - select list란? select from 사이에 원하는 컬럼만 조회하는 방식
  - 예시 index: \[장비번호 + 변경일자 + 변경순번]
    - 장비번호가 C인 데이터를 변경일자, 변경순번으로 정렬하여 출력
      ```
      SELECT *
      FROM 상태변경이력
      WHERE 장비번호 = 'C'
      ORDER BY 변경일자, 변경순번
      ```
      - 수직적 탐색을 통해 장비번호가 'C'인 데이터를 찾아 인덱스 리프블록을 스캔하면 변경일자 + 변경순번으로 정렬된 데이터를 얻기 때문에 정렬 연산을 생략할 수 있다.
    - 만약 order by 절을 가공한다면 인덱스는 가공하지 않은 상태로 저장했기 때문에 정렬 연산이 필요하다
      ```
      SELECT *
      FROM 상태변경 이력
      WHERE 장비번호 = 'C'
      ORDER BY 변경일자 || 변경순번
      ```
    - 아니면 select list로 가공한 컬럼을 통해 정렬한다면 
      ```
      SELECT *
      FROM (
          SELECT TO_CHAR(A.주문번호, 'FM000000') AS 주문번호 /* 0 여섯개로 시작하는 문자 값 변환 */
            , A.업체번호, A.주문금액
          FROM 주문 A
          WHERE A.주문일자 = :searchDate
            AND A.주문번호 > NVL(:nextOerderNo, 0)
          ORDER BY 주문번호
      )
      WHERE ROWNUM <= 30
      -----------------------------------------------
      |ID|OPERATION                        |NAME    |
      |0 | SELECT STATEMENT                |        |
      |1 |  COUNT STOPKEY                  |        |
      |2 |   VIEW                          |        |
      |3 |    SORT ORDER BY STOPKEY        |        |
      |4 |     TABLE ACCESS BY INDEX ROWID | 주문   |
      |5 |      INDEX RANGE SCAN           | 주문PK |
      -----------------------------------------------
      ```
      - 이런 경우 ORDER BY A.주문번호로 하면 SORT ORDER BY STOPKEY 연산이 이뤄지지 않는다
        
- SELECT LIST 에서 컬럼 가공
  - 인덱스로 구성된 컬럼의 MIN, MAX를 구한다면 정렬 sort을 따로 수행하지 않는다.
    ```
    /* index 장비번호 + 변경일자 + 변경순번 */
    SELECT MIN(변경순번)
    FROM 상태변경이력
    WHERE 장비번호 = 'C'
      AND 변경일자 = '20180316'
    ------------------------------------------------------------------------------
    ROWS | ROW Source Operation
    0    | Statement
    1    |  SORT AGGREGATE (cr=6, pr=0, pw=0, time=81us)
    1    |   FIRST ROW (cr=6, pr=0, pw=0, time=59 us)
    1    |    INDEX RANGE SCAN (MIN/MAX) 상태변경이력_PK (cr=6, pr=0, pw = 0 ...)
    // 인덱스 리프블록의 왼쪽/오른쪽에서 레코드 하나(FIRST ROW)만 읽고 멈춘다.
    ```
    
    - 참고
      - rows: 각 수행단계에 출력된 row수
      - cr: consistent 모드 블록 읽기 (Query가 시작된 시점을 기준으로 Commit된 데이터를 읽는 )
      - pr(r): 디스크 블록 읽기
      - pw(w): 디스크 블록 쓰기
      - time: 소요시간
      - cost: Cost Based Optimizer(CBO)에서 예측한 비용
      - size: 리턴한 데이터 size(bytes)
      - card: cardinality, Cost Based Optimizer(CBO)에서 예측한 row수

    - 인덱스 컬럼의 MIN, MAX여도 가공을 한다면 sort 연산이 진행된다.
      ```
      /* index 장비번호 + 변경일자 + 변경순번 */
      SELECT NVL(MAX(TO_NUMBER(변경순번)), 0)
      FROM 상태변경이력
      WHERE 장비번호 = 'C'
        AND 변경일자 = '20180316'
      ------------------------------------------------------------------------------
      ROWS | ROW Source Operation
      0    | Statement
      1    |  SORT AGGREGATE (cr=4, pr=0, pw=0, time=81us)
      1    |   FIRST ROW (cr=4, pr=0, pw=0, time=59 us)
      1    |    INDEX RANGE SCAN (MIN/MAX) 상태변경이력_PK (cr=4, pr=0, pw = 0 ...)
      ```
    - min, max를 구한 후 가공하면 된다
      ```
      /* index 장비번호 + 변경일자 + 변경순번 */
      SELECT NVL(TO_NUMBER(MAX(변경순번)), 0)
      FROM 상태변경이력
      WHERE 장비번호 = 'C'
        AND 변경일자 = '20180316'
      ------------------------------------------------------------------------------
      ROWS   | ROW Source Operation
      0      | Statement
      1      |  SORT AGGREGATE (cr=1670 pr=0 pw=0 time=101326 us)
      131577 |   INDEX RANGE SCAN 상태변경이력_PK (cr=1670, pr=0, pw = 0 ...)
      ```
     
  - SELECT LIST 내에 scalar subquery 사용하면 MIN/MAX, FIRST ROW 방식으로 정렬연산없이 실행하게 된다.
    ```
    SELECT 장비번호, 장비명, 상태코드
      , ( SELECT MAX(변경일자)
          FROM 상태변경이력
          WHERE 장비번호 = P.장비번호) 최종변경일자
    FROM 장비 P
    WHERE P.장비구분코드 = 'P00001'
    ------------------------------------------------------------------------------
    ROWS   | ROW Source Operation
    10     | SORT AGGREGATE (cr=22 pr=0 pw=0 time=0us)
    10     |  FIRST ROW (cr=22 pr=0 pw=0 time=0us cost=3 size=12 card=1)
    10     |   INDEX RANGE SCAN (MIN/MAX) 상태변경이력_PK (cr=22 pr=0 pw=0 time=0us)
    10     |  TABLE ACCESS BY INDEX ROWID 장비 (cr=4 pr=0 pw=0 time=0us)
    10     |   INDEX RANGE SCAN 장비_N1 (cr=2 pr=0 pw=0 time=153 us)
    ```
    - 만약 여러개의 데이터를 scalar subquery로 조회하게 된다면동일한 테이블을 여러번 읽어야 하므로 비효율적이다.
      ```
      SELECT 장비번호, 장비명, 상태코드
        , ( SELECT MAX(변경일자)
            FROM 상태변경이력
            WHERE 장비번호 = P.장비번호) 최종변경일자
        , ( SELECT MAX(변경순번)
            FROM 상태변경이력
            WHERE 장비번호 = P.장비번호
              AND 변경일자 = (SELECT MAX(변경일자) FROM 상태변경이력 WHERE 장비번호 = P.장비번호)) 최종변경순번
      FROM 장비 P
      WHERE P.장비구분코드 = 'A00001'
      ```
    - 만약 데이터를 가공하여 하나의 scalar subquery로 하여도 가공하기 때문에 데이터가 많은 경우 성능이 떨어진다.
      ```
      SELECT 장비번호, 장비명, 상태코드
        , ( SELECT MAX(변경일자 || 변경순번)
            FROM 상태변경이력
            WHERE 장비번호 = P.장비번호) 최종변경일자
      FROM 장비 P
      WHERE P.장비구분코드 = 'P00001'
      ```
    - Top N 알고리즘을 통해 해결 가능 

- 자동 형변환
  ```
  SELECT *
  FROM 고객
  WHERE 생년월일 = 19900216
  ------------------------------------------------------------------------------
  ROWS   | ROW Source Operation
  0      | SELECT STATEMENT Optimizer=ALL_ROWS(Cost = 3 Card = 1 Bytes = 38)
  1      |  TABLE ACCESS (FULL) OF '고객' (TABLE) (Cost = 3 Card = 1 Bytes = 38)
  ------------------------------------------------------------------------------
  -- Optimizer가 조건절을 변환하여 결과적으로 인덱스 컬럼이 가공된다.
  -- TO_NUMBER(생년월일) = 19900216
  ```
  
  - 날짜형과 문자형
    - 오라클 기준으로 날짜형과 문자형이 만나면 날짜형이 이간다
  - 숫자형과 문자형
    - 오라클 기준으로 숫자형과 문자형이 만나면 숫자형이 이간다
    - 다만 LIKE 연산에서는 다르다

  - Oracle의 성능은 연산횟수 보다 블록 I/O를 줄이는 것에 판단되기 때문에 자동형변환을 통한 성능 이슈보다 TO_NUMBER, TO_CHAR, TO_DATE 등에 형변환으로 의도적으로 사용해야 한다.

#### 인덱스 확장기능 사용법

- Index Range Scan
  
  ![image](https://user-images.githubusercontent.com/42403023/140049598-2610ec30-8aec-4cc8-b5a7-5925ee56aad9.png)
  
  ```
  SELECT * FROM EMP WHERE DEPT_NO = 20;

  Execution Plan
  0   SELECT STATEMENT Optimizer=ALL_ROWS
  1 0 TABLE ACCESS (BY INDEX ROWID) OF 'EMP' (TABLE)
  2 1 INDEX (RANGE SCAN) OF 'EMP_DEPTNO_IDX' (INDEX)
  ```
  
  - 리프 블록까지 수직적으로 탐색한 후 필요한 범위만 스캔
  
- Index Full Scan
  
  ![image](https://user-images.githubusercontent.com/42403023/140050107-bc7028ce-db43-4106-95f9-e926a7fa2326.png)
  
  
  ```
  CREATE INDEX EMP_ENAME_SAL_IDX ON EMP(ename, sal);
  SELECT * FROM EMP WHERE SAL > 2000 ORDER BY ENAME;

  Execution Plan
  0   SELECT STATEMENT Optimizer=ALL_ROWS
  1 0 TABLE ACCESS (BY INDEX ROWID) OF 'EMP' (TABLE)
  2 1 INDEX (FULL SCAN) OF 'EMP_ENAME_SAL_IDX' (INDEX)
  ```
  
  - 인덱스 스캔없이 인덱스 리프 블록을 처음부터 끝까지 수평 탐색하는 방식
  - 데이터 검색을 위한 최적의 인덱스가 없을 때 차선으로 선택
  - index full scan의 효용성
    - 예제처럼 ENAME이 조건절에 없으면 옵티마이저는 먼저 TABLE FULL SCAN을 고려하지만 데이터가 많은 경우 부담이 크다.
    - 이런 경우 index를 full scan을 통해 필터링하는 효과를 볼 수 있다.
    - index range scan 보다는 효과가 작지만 table full scan보다 큰 효과를 볼 수 있다.
  - 인덱스를 이용한 소트 전략 생략
    - index full scan을 하면 결과 집합이 인덱스 컬럼 순으로 소팅되기 때문에 sort order by 연산이 생략된다.

- Index Unique Scan
  
  ![image](https://user-images.githubusercontent.com/42403023/140052140-a679386f-5fed-45b8-b774-3b47dd81700c.png)
  
  ```
  create unique index pk_emp on emp(empno);
  select empno, ename from emp where empno = 7788;
  
  Execution Plan
  0   SELECT STATEMENT Optimizer=ALL_ROWS
  1 0 TABLE ACCESS (BY INDEX ROWID) OF 'EMP' (TABLE)
  2 1 INDEX (UNIQUE SCAN) OF 'PK_EMP' (INDEX)
  ```
  
  - Unique index 를 '=' 조건으로 탐색하는 경우에 동작
  - Unique index라 해도 범위 조건을 활용하면 index range scan이 활용된다.

- Index Skip Scan
  
  ![image](https://user-images.githubusercontent.com/42403023/140053654-4e91dfa2-52eb-4680-82f0-5336c6c7b4cc.png)

  - Oracle 9i 버전부터 생성
  - 인덱스 선두 컬럼의 Distinct Value 개수가 적고, 후행 컬럼의 Distinct Value 개수가 많을 때 유용하다
  - 예시
  
    ![image](https://user-images.githubusercontent.com/42403023/140052310-1ddb11d0-da54-4130-93e6-9e419ac2874c.png)

    ```
    SELECT /*+ index_ss(사원 사원_IDX) */
    FROM 사원
    WHERE 연봉 BETWEEN 2000 AND 4000;

    Execution Plan
    0   SELECT STATEMENT Optimizer=ALL_ROWS
    1 0 TABLE ACCESS (BY INDEX ROWID) OF 'EMP' (TABLE)
    2 1 INDEX (UNIQUE SCAN) OF 'PK_EMP' (INDEX)
    ```
    - 1 ~ 2 번 블록에 경우 가능성이 없기 때문에 SKIP하고, 3번 블록은 가능성이 있기 때문에 스캔한다.
    - 4 ~ 5 번 블록에 경우 가능성이 없기 때문에 SKIP하고, 6번 블록은 가능성이 있기 때문에 스캔한다.
      - 조건은 남&10000 이상이지만 여성&3000미만이 있을 수 있기 때문에 스캔해야 한다.
    - 7 ~ 9 번 블록에 경우 가능성이 없기 때문에 SKIP하고, 10번 블록은 가능성이 있기 때문에 스캔한다.
      - 비록 여&10000 이상이지만 여성을 제외한 다른 조건이 있을 수 있기 때문에 스캔해야 한다.

  - Index Skip Scan 조건
    - Distinct Value 개수가 적은 선두 컬럼이 조건절에 없고, 후행 컬럼의 Distinct Value 개수가 많을 때 효과적
    - hint에 index_ss 작성(ss: skip scan)
  
- Index Fast Full Scan

  ![image](https://user-images.githubusercontent.com/42403023/140054758-f416c878-089a-4f22-bf34-7529991bfb0f.png)
  
  - 논리적인 인덱스 트리 구조를 무시하고 인덱스 세그먼트 전체를 Multiblock I/O 방식으로 스캔
  - 특징
    - Multiblock I/O 방식 사용(디스크로부터 대량의 인덱스 블록을 읽어야 할 때 큰 효과 발휘)
      - Single Block I/O : 한번의 I/O Call에 하나의 데이터 블록만 읽어 메모리에 적재하는 방법. 인덱스를 통한 Table Acess일 경우 인덱스와 테이블 모두 이방식을 사용
      - Multiblock I/O : Call이 필요한 시점에 인접한 블록들을 같이 읽어 메모리에 적재하는 방법. Extent 범위 단위에서 읽는다.db_file_muliblock_read_count 파라미터에 의해 결정된다.
    - 인덱스 키 순서대로 정렬되지 않음(속도는 빠르나 인덱스 리프 노드가 갖는 연결 리스트 구조를 무시한 채 데이터를 읽기 때문에 결과집합이 인덱스 키 순서대로 정렬되지 않는다)
    - 쿼리에 사용한 컬럼이 모두 인덱스에 포함돼 있을 때 사용 가능
    - 인덱스가 파티션 돼 있지 않더라도 병렬 쿼리가 가능(Index Range Scan 또는 Index Full Scan과 달리)
    - 병렬 쿼리 시에는 Direct I/O방식 사용하기 때문에 I/O 속도가 더 빨라짐
  - Index Full Scan vs Index Fast Full Scan
    - Index Full	Scan
      - I/O 방식:	Single Block I/O
      - 정렬: 정렬 보장
      - 속도: 느림
      - 병렬읽기: 지원안됨
    - Index Fast Full Scan
      - I/O 방식: Multi Block I/O
      - 정렬: 정렬 안됨
      - 속도: 빠름
      - 병렬읽기: 지원됨

- Index Range Scan Descending
  ```
  SELECT * FROM EMP WHERE EMPNO > 0 ORDER BY EMPNO DESC;

  Execution Plan
  0   SELECT STATEMENT Optimizer=ALL_ROWS
  1 0 TABLE ACCESS (BY INDEX ROWID) OF 'EMP' (TABLE)
  2 1 INDEX (RANGE SCAN DESCENDING) OF 'PK_EMP' (INDEX (UNIQUE))
  ```
  - index range scan과 동일한 방식이지만 내림차순으로 정렬된 결과집합을 얻고자 할 때 사용
  ```
  SELECT DEPTNO, DNAME, LOC
      , (SELECT MAX(SAL) FROM EMP WHERE DEPTNO = A.DEPTNO)
  FROM DEPT D;

  Execution Plan
  0   SELECT STATEMENT Optimizer=ALL_ROWS
  1 0   SORT (AGGREGATE)
  2 1     FIRST ROW
  3 2       INDEX (RANGE SCAN (MIN/MAX) OF 'EMP_02' (INDEX)
  4 0 TABLE ACCESS (FULL) OF 'EMP' (TABLE)
  ```
  - MAX 값을 구할 때에도 해당 컬럼에 인덱스가 있는 경우 descending이 사용된다

#### 참고

- 조시형 저자의 개발자를 위한 SQL 튜닝 입문서 친절한 SQL 튜닝
  - https://url.kr/fjm9l2
- sequential & randomaccess: https://m.cafe.daum.net/Oracle-/DHl0/94?listURI=%2FOracle-%2F_rec
