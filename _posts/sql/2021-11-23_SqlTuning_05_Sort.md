---
layout: post
title: SQL Tuning. Join
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

- Union VS Union ALL
  - Union, Minus, Distinct 연산은 중복 레코드를 제거하기 위해 소트 연산을 발생시키므로 필요한 경우에 사용해야 한다.
  - Union vs Union ALL
    - Union은 중복 데이터를 삭제하기 위해 sort 연산을 진행하며 Union ALL은 sort 연산 없이 데이터를 합친다.
  - 즉, Union은 데이터 중복이 있을 경우에만 사용해야 하며, 중복이 없는 경우에는 성능을 위해서 Union ALL을 사용해야 한다.
    
- Exists 활용
  - 

- 조인 방식 변경


#### Index를 이용한 Sort 연산 생략

#### Sort Area 를 적게 사용하도록 SQL 작성
 
#### 참고

- 조시형 저자의 개발자를 위한 SQL 튜닝 입문서 친절한 SQL 튜닝
  - https://url.kr/fjm9l2
