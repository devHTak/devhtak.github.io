---
layout: post
title: SQL Tuning. Sort
summary: SQL Tuning
author: devhtak
date: '2021-11-16 21:41:00 +0900'
category: SQL
---

#### Sort 연산에 대한 이해

- Sort 수행 과정
  
  ![image](https://user-images.githubusercontent.com/42403023/142968918-34a1db1e-4c77-490c-b2b9-8f6ffc23b161.png)
  
  - Sort 연산은 PGA에 할당한 Sort Area에서 이뤄지며, Sort Area 공간이 부족한 경우 Temp tablespace를 활용한다.
  - Sort 두가지 유형
    - In-Memory Sort: 전체 데이터 정렬 작업을 메모리 내에서 완료하는 것(Internal Sort)
    - To-Disk Sort: 할당받은 Sort Area 내에서 정렬을 완료하지 못해 디스크 공간까지 사용하는 경우(External Sort)
  - Tablespace 사용
    - Sort Run: Sort Area가 찰때마다 Temp 영역에 저장해 둔 중간 단계의 집합
    - Run Merge: PGA에 있는 데이터가 이미 sorting 된 결과값이기 때문에 Merge 과정은 어렵지 않다
  - Disk IO 까지 발생하는 Disk Sort가 발생하는 경우 부분범위 처리를 불가능하기 때문에 성능 저하의 주요원인이 되기도 한다

- Sort Operation 종류
  - Sort Aggregate
    ```
    ID  OPERATION            NAME    ROWS    BYTES    COST    TIME
    0   SELECT STATEMENT               1       4       3      00:00:01
    1    SORT AGGREGATE                1       4          
    2     TABLE ACCESS FULL   EMP      14      56      3      00:00:01
    ```
    - Sort 라는 표현을 사용하지만 실제로 데이터를 정렬하지 않고, 전체 로우를 대상으로 집계를 수행하며 Sort Area를 사용한다는 의미
    - 정렬하지 않은 체 MIN, MAX, SUM, COUNT, AVG 등을 사용할 때 진행
      - 데이터를 확인하는 과정에서 MIN, MAX, SUM, COUNT, AVG 등의 값들을 갱신
      
  - Sort Order By
    ```
    ID  OPERATION            NAME    ROWS    BYTES    COST    TIME
    0   SELECT STATEMENT              14      518      4      00:00:01
    1    SORT ORDER BY                14      518      4      00:00:01
    2     TABLE ACCESS FULL   EMP     14      518      4      00:00:01
    ```
    - 데이터를 정렬할 때 나타난다.
    
  - Sort Group By
    ```
    ID  OPERATION            NAME    ROWS    BYTES    COST    TIME
    0   SELECT STATEMENT              11      165      4      00:00:01
    1    SORT GRUOP BY                11      165      4      00:00:01
    2     TABLE ACCESS FULL   EMP     14      210      4      00:00:01
    ```
    - 소팅 알고리즘을 사용해 그룹별 집계를 수행할 때 사용
    - 만약 부서별 급여에 대한 정보(최대, 평균 ..)를 조회한다면
      - Group By의 기준인 부서의 수만큼 공간을 준비하고, 해당 부서에 대한 데이터를 해당 공간에 기록하며 Sort Aggregate한다.
      - 즉, 부서의 수만큼만 Sort Area를 사용하기 때문에 Tablespace를 사용할 필요가 없다.
    - 주의할 점은 GROUP BY에 대한 결과가 sort되지는 않기 때문에 정렬된 결과를 얻고자 한다면 order by를 작성해주어야 한다.
    
  - Sort Unique
    - Subquery Unnesting
      ```
      SELECT /*+ ordered use_nl(dept) */ *
      FROM dept d
      WHERE d.dept_no in ( SELECT /*+ unnest */ deptno from emp where job='CLERK' )
      
      ID  OPERATION                      NAME           ROWS    BYTES    COST    TIME
      0   SELECT STATEMENT                               3      87       4      00:00:01
      1    NESTED LOOPS                                  3      87       4      00:00:01
      2     SORT UNIQUE                                  3      33       2      00:00:01
      3      TABLE ACCESS BY INDEX ROWID  EMP            3      33       2      00:00:01
      4       INDEX RANGE SCAN            EMP_JOB_INDEX  3               1      00:00:01
      5      TABLE ACCESS BY INDEX ROWID  DEPT           1      18       1      00:00:01
      6       INDEX RANGE SCAN            DEPT_PK        1               0      00:00:01
      ```
      - 옵티마이저가 서브쿼리를 풀어 일반 조인문으로 변환하는 것
      - 이 때 조인 컬럼에 Unique 인덱스가 없으면, 메인 쿼리와 조인하기 전 중복 레코드를 삭제하기 위해 Sort Unique 오퍼레이션이 발생한다.
    - Union, Minus, Intersect 같은 집합 연산자 사용 시 Sort Unique 사용
    - Ditinct 연산자 사용 시 Sort Unique 사용
    
  - Sort Join
    ```
    SELECT /*+ ordered use_merge(e) */ *
    FROM dept d, emp e
    WHERE d.dept_no = e.dept_no
    
    ID  OPERATION            NAME    ROWS    BYTES    COST    TIME
    0   SELECT STATEMENT              14      770      8      00:00:01
    1    MERGE JOIN                   14      770      8      00:00:01
    2     SORT JOIN                   4       72       4      00:00:01
    3      TABLE ACCESS FULL DEPT     4       72       3      00:00:01
    4     SORT JOIN                   14      518      4      00:00:01
    5      TABLE ACCESS FULL EMP      14      518      3      00:00:01
    ```
    - merge join 할 때 수행한다.
    
  - Window Sort
    ```
    SELECT emp_no, ename, job, mgr, sal, 
        avg(sal) over (partition by deptno)
    FROM emp
    
    ID  OPERATION            NAME    ROWS    BYTES    COST    TIME
    0   SELECT STATEMENT              14      406      4      00:00:01
    2     WINDOW JOIN                 14      406      4      00:00:01
    3      TABLE ACCESS FULL EMP      14      406      3      00:00:01
    ```
    - window 함수를 수행할 때 나타난다.

#### Sort 가 발생하지 않도록 SQL 작성

- Union, Minus, Distinct 연산은 중복 레코드를 제거하기 위해 소트 연산을 발생시키므로 필요한 경우에 사용해야 한다.

- Union VS Union ALL
  - Union vs Union ALL
    - Union은 중복 데이터를 삭제하기 위해 sort 연산을 진행하며 Union ALL은 sort 연산 없이 데이터를 합친다.
  - 즉, Union은 데이터 중복이 있을 경우에만 사용해야 하며, 중복이 없는 경우에는 성능을 위해서 Union ALL을 사용해야 한다.
    
- Exists 활용
  - distinct, minus 연산은 전체 데이터를 읽어서 작업해야 하는데, 대부분 Exists 서브쿼리로 변환이 가능하다.
  - Exists를 사용하면 전체 데이터를 읽지 않고 존재 여부만 확인하면 되기 때문에 성능 향상이 가능하다
  - Distinct 튜닝
    ```
    SELECT DISTINCT p.상품번호, p.상품명, ...
    FROM 상품 p, 계약 c
    WHERE p.상품유형코드 = :pclscd
    AND   c.상품번호 = p.상품번호
    AND   c.계약일자 between :dt1 and :dt2
    AND   c.계약구분코드 = :cptcd
    
    SELECT p.상품번호, p.상품명, ...
    FROM 상품 p
    WHERE p.상품유형코드 = :pclscd
    AND   EXISTS ( SELECT 'x' FROM 계약 c
                   WHERE c.상품번호 = p.상품번호
                   AND   c.계약일자 between :dt1 and :dt2
                   AND   c.계약구분코드 = :cptcd)
    ```
    - 계약 테이블에 인덱스를 \[상품번호 + 계약일자] 일 때, DISTINCT를 사용하면 상품번호, 계약일자에 대한 모든 계약데이터를 읽어야 한다.
    - Exists 서브쿼리를 사용하면 계약 테이블에 상품번호, 계약일자에 대한 데이터가 있는지만 확인하면 된다.
  - MINUS 튜닝
    - 상품 테이블에 데이터 중 계약 테이블에 해당하는 계약일자, 계약 코드 등을 제외한 데이터를 구한다
    - 기존 (상품 데이터) MINUS (계약일자, 계약코드에 해당하는 상품 데이터) 쿼리에서 (상품 데이터 NOT EXISTS (계약절차, 계약코드에 해당하는 상품 데이터)) 로 변경할 수 있다.

- 조인 방식 변경
  - Index를 활용하여 sort가 필요없는 경우에도 hash join에 경우에는 sort 연산이 발생한다.
  - 이런 경우, nl 조인이나 sort merge 조인을 사용하면 sort연산(sort order by) 없이 결과를 가져올 수 있다.

#### Index를 이용한 Sort 연산 생략

- sort order by 생략
  ```
  SELECT 거래일시, 체결건수, 체결수량, 거래대금
  FROM   종목거래
  WHERE  종목코드 = 'KR123456'
  ORDER BY 거래일시
  ```
  - 인덱스를 \[종목코드 + 거래일시]로 구성하지 않은 경우
    ```
    ID  OPERATION            NAME    ROWS    BYTES    COST
    0   SELECT STATEMENT             4000    315K     1372
    1    SORT ORDER BY               4000    315K     1372
    2     TABLE ACCESS FULL  종목    4000    315K     1372
    3      INDEX RANGE SCAN  종목_n1 4000             258
    ```
    - Sort Order By가 필요하며 종목코드에 해당하는 레코드를 인덱스에서 모두 읽고, 그만큼 테이블 랜덤 액세스가 발생하며, 거래일시에 대한 sorting 작업또한 발생한다. 
  - 인덱스를 \[종목코드 + 거래일시]로 구성한 경우
    ```
    ID  OPERATION            NAME    ROWS    BYTES    COST
    0   SELECT STATEMENT             4000    315K     1372
    1    TABLE ACCESS FULL   종목    4000    315K     1372
    2      INDEX RANGE SCAN  종목_pk 4000             258
    ```
    - Sort Order By가 생략된다.
    - 종목코드에 해당하는 전체 레코드를 읽지 않고도 바로 결과 집합을 출력할 수 있기 때문에 부분범위 처리가 가능한 상태가 되었다.

- Top N 쿼리
  - Top N 쿼리는 전체 결과집합 중 상위 N개 레코드만 선택하는 쿼리이며, Oracle에 경우 rownum 으로 처리한다.
  - Top N Stopkey 알고리즘
    - 실행계획에서 COUNT(STOPKEY) 로 표현되며 조건절에 부합하는 레코드가 많아도, 그 중 rownum에 해당하는 건수만큼 결과 레코드를 얻으면 멈춘다는 뜻이다.
  - 페이징 처리
    - 대량의 데이터를 조회할 때 사용하며, 뒤쪽 페이지로 이동할수록 읽는 데이터량이 많아지지만, 보통 앞부분만 읽기 때문에 많이 사용된다.
    - 부분범위 처리 가능하도록 SQL을 작성해야 한다.
      - 인덱스를 사용 가능하도록 조건절을 구사하고, 조인은 NL 조인 위주로 처리, Order By 절이 있어도 소트 연산을 생략할 수 있도록 인덱스를 구성해주는 것을 의미한다.   
  - 페이징 처리 ANTI Pattern
    - 잘못된 페이징 처리 ROWNUM 쿼리를 하게 되면, 데이터는 N개의 데이터를 내보내지만, 전체 데이터를 스캔하는 경우가 발생한다.
    - Execution plan을 돌려 확인해보자
      - 기존 Top N Stopkey Execution plan: COUNT(STOPKEY)
      - paging 처리가 적용되지 않은 Execution plan: COUNT /* NO SORT + NO STOP */
  - 최소값/최대값 구하기
    - 최소값/최대값을 구하기 위해 Sort aggregation을 사용하여 전체 데이터를 정렬하지는 않지만, 비교하여 값을 읽는다.
    - 인덱스는 정렬돼 있으므로 인덱스를 이용하면 전체 데이터를 읽지 않고도 쉽게 찾을 수 있다.
    - 인덱스를 이용해 최소/최대값 구하기 위한 조건
      - 인덱스를 이용해 최소/최대값을 구하려면, 조건절 컬럼과 MIN/MAX 함수 인자 컬럼이 모두 인덱스에 포함돼 잇어야 한다.
        ```
        SELECT MAX(SAL)
        FROM EMP
        WHERE DEPT_NO='30'
        AND    MGR = '7698';
        ```
      - 인덱스를 \[DEPT_NO, MGR, SAL] 구성 또는 \[DEPT_NO, SAL, MGR 구성]
        - 조건을 만족하는 레코드 하나를 찾았을 때 바로 멈추는 것을 의미 (FIRST ROW STOPKEY 알고리즘)
        - INDEX RANGE SCAN으로 탐색한다
      - 인덱스 \[SAL, DEPT_NO, MGR] 구성
        - 조건절 컬럼이 모두 인덱스 선두 컬럼이 아니므로 INDEX FULL SCAN이 발생한다
        - FIRST ROW STOPKEY 알고리즘은 적용된다.
      - 인덱스 \[DEPT_NO, SAL] 구성
        - MGR 컬럼이 인덱스로 구성되어 있지 않기 때문에 MGR에 대한 조건은 테이블 필터를 거처야 한다
        - 즉, FIRST ROW STOPKEY 알고리즘은 적용되지 않는다.
				
  - Top N 쿼리를 이용해 최소/최대값 구하기
    ```
    SELECT *
    FROM (
        SELECT SAL
        FROM EMP
        WHERE DEPT_NO='30' AND MGR = '7968'
        ORDER BY SAL DESC
    ) 
    WHERE ROWNUM <= 1;
    ```
    - 인덱스 [DEPT_NO, SAL] 구성
      - TOP N STOPKEY 알고리즘을 사용하여 모든 컬럼이 인덱스에 포함되어 있지 않아도 잘 작동한다.
      - 테이블 액세스할 때 MGR= '7968'을 만족하는 값을 찾으면 바로 멈춘다.
      - 성능적인 면에서 MIN/MAX보다 낫다.
- 이력 조회
  - 예시
    - 장비 테이블과 해당 이력 관리 테이블인 상태변경이력 테이블 2개를 조회
      - 상태변경이력 테이블에 인덱스를 [장비번호 + 변경일자 + 변경순번] 로 구성
    - 가장 단순한 이력 조회
      ```
      SELECT 장비번호, 장비명, 상태코드
          , ( SELECT MAX(변경일자) FROM 상태변경이력 WHERE 장비번호 = P.장비번호) 최종변경일자
      FROM 장비 P
      WHERE 장비구분코드 = 'A001'
      ```
      - 인데스에 장비번호, 변경일자가 있기 때문에 FIRST ROW STOPKEY 알고리즘 적용
    - 점점 복잡해지는 이력 조회
      ```
      SELECT 장비번호, 장비명, 상태코드
          , SUBSTR(최종이력, 1, 8) 최종변경일자
          , TO_NUMBER(SUTSTR(최종이력, 9, 4)) 최종변경순번
      FROM (SELECT 장비번호, 장비명, 상태코드
                , ( SELECT MAX(H.변경일자 || LPAD(H.변경순번, 4)) FROM 상태변경이력 H WHERE 장비번호 = P.장비번호) 최종이력
            FROM 장비 P
            WHERE 장비구분코드 = 'A001'
      )
      ```
      - 해당 방식으로는 인덱스 컬럼을 가공했기 때문에 FIRST ROW STOPKEY 가 적용되지 않는다.
      ```
      SELECT 장비번호, 장비명, 상태코드
           , ( SELECT MAX(H.변경일자) 
               FROM 상태변경이력 H WHERE 장비번호 = P.장비번호) 최종변경일자
           , ( SELECT MAX(H.변경순번) 
              FROM 상태변경이력 H 
              WHERE 장비번호 = P.장비번호
              AND 변겨일자 =  ( SELECT MAX(H.변경일자) FROM 상태변경이력 H WHERE 장비번호 = P.장비번호) ) 최종변경순번
      FROM 장비 P
      WHERE 장비구분코드 = 'A001'
      ```
      - 상태변경이력을 3번 조회하는 비효율이 있지만, FIRST ROW STOPKEY 알고리즘이 잘 작동하므로 성능을 비교적 좋다
    - INDEX_DESC 힌트 사용
      - 인덱스 컬럼을 가공하여 사용한다면 TOP N 알고리즘을 사용하고 INDEX_DESC 를 사용하면 좋은 성능을 가져올 수 있다
      - 인덱스 컬럼의 구성이 바뀌면 결과집합에 문제가 생길 수 있다.
			
  - Sort Group By 생략
    ```
    SELECT REGION, AVG(AGE), COUNT(*)
    FROM CUSTOMER
    GROUP BY REGION
    ```
    - GROUP BY 연산에도 인덱스를 사용하면 Sort Group BY 연산을 생략할 수 있다.
    - 실행계획에 SORT GROUP BY NOSORT라고 표시된다.

#### Sort Area 를 적게 사용하도록 SQL 작성

- 소트 데이터 줄이기
  ```
  SELECT *
  FROM 예수금원장
  ORDER BY 총예수금 DESC

  SELECT 계좌번호, 총예수금
  FROM 예수금원장
  ORDER BY 총예수금 DESC
  ```
  - SELECT - LIST 가 Sort Area에 영향을 준다.
  - 위 쿼리보다 아래 쿼리가 더 데이터 영역을 조금 사용한다.
		
- TOP N 쿼리의 소트 부하 경감 원리
  - TOP N SORT 알고리즘
    - 최대값 10개를 구한다고 했을 때, 정렬되지 않은 상태에서 10명을 가져오고 순차적으로 비교하면서 가장 큰 10개의 값을 유지하도록 하는 알고리즘
  - 페이징 쿼리에서 인덱스를 통한 소트 연산 생략을 진행할 수 없을 때 Table Full Scan을 실행하지만 이 때 TOP N SORT 알고리즘을 진행한다.
    - 실행계획에 SORT ORDER BY STOPKEY 로 표현된다
  - TOP N SORT 알고리즘을 사용하면 메모리를 페이징 개수만큼 사용하기 때문에 physical area(Tablespace)를 사용하지 않아도 된다.

- TOP N 쿼리가 아닐 때 발생하는 소트 부하
  - ROWNUM 조건을 생략하면 TOP N STOP 알고리즘이 작동하지 않기 때문에 디스크를 읽는 경우가 발생한다(Tablespace)

- 분석함수에서의 TOP N 소트
  - Rank나 row_number 함수는 TOP N 알고리즘이 적용되기 때문에 max함수보다 소트 부하가 적다
 
#### 참고

- 조시형 저자의 개발자를 위한 SQL 튜닝 입문서 친절한 SQL 튜닝
  - https://url.kr/fjm9l2
