---
layout: post
title: SQL Tuning. Join
summary: SQL Tuning
author: devhtak
date: '2021-11-16 21:41:00 +0900'
category: SQL
---
 
#### NL Join

- Nested Loop Join(NL Join): 인덱스를 이용한 조인

- 기본 메커니즘
  - 예시
    - 사원과 고객 테이블이 있다
    - 2020년 이후 입사한 사원이 관리하는 고객을 추출하면
      ```
      SELECT e.사원명, c.고객명, c.전화번호
      FROM 사원 e, 고객 c
      WHERE e.입사일자 >= TO_DATE('2020', 'YYYY')
      AND    c.관리사원번호 = e.사원번호
      ```
    - 기본적인 Nested Loop Join 방식으로 PL/SQL로 중접루프문의 수행구조는 아래와 같다
      ```
      begin
          For outer in ( select 사원번호, 사원명 from 사원 where 입사일자 >= TO_DATE('2020', 'YYYY') )
          loop -- outer loop
              for inner in (select 고객명, 전화번호 from 고객 where 관리사원번호 = outer.사원번호)
                 loop
                           dbms_output.put_line(outer.사원명 || '  : ' || inner.고객명);
                  end loop;
              end loop;
      end
      ```
  - 일반적으로 NL 조인은 Outer, Inner 양쪽 테이블 모두 인덱스를 이용한다.
    - Outer 쪽 테이블은 사이즈가 크지 않으면 인덱스를 이용하지 않아도 된다.
      - Table Full Scan이 일어나도 한번에 그치기 때문이다.
    - Inner 쪽 테이블은 인덱스를 사용해야 한다
      - Outer에서 읽은 건수만큼 Table Full Scan 발생
- NL 조인 실행계획 제어
  - 실행계획
    ```
    SELECT STATEMENT OPTIMIZER=ALL_ROWS
        NESTED LOOPS
            TABLE ACCESS (BY INDEX ROWID) OF '사원' (TABLE)
                INDEX (RANGE SCAN) OF '사원_X1' (INDEX)
            TABLE ACCESS (BY INDEX ROWID) OF '고객' (TABLE)
                INDEX (RANGE SCAN) OF '고객_X1' (INDEX)
    ```
  - 힌트
    ```
    SELECT /*+ ordered use_nl(B) use_nl( C ) use_hash(D)*/
        *
    FROM A, B, C, D
    WHERE ….
    ```
    - Ordered: 조인을 순서대로 진행
    - Leading(C, A, B, D): from 절 순서와는 달리 C->A->B->D 순으로 조인하라는 의미
    - Use_*: * 방식으로 조인할 것을 명시
    - A->B->C->D 순으로 조인하며, A, B와 C를 조인할 때에는 NL방식, D를 조인할 때에는 해쉬방식으로 조인하라는 뜻이다.
- NL 조인 수행 과정 분석
  ```
  SELECT /*+ ordered use_nl( c ) index( e ) index( c ) */
      *
  FROM 사원 e, 고객 c
  WHERE c.관리사원번호 = e.사원번호  -- 1
  AND       e.입사일자 >= TO_DATE('20210101', 'YYYYMMDD') -- 2
  AND    e.부서코드 = 'Z123' -- 3
  AND    c.최종주문금액 >= 20000 -- 4

  -- 사원_PK: 사원번호
  -- 사원_X1: 입사일자
  -- 고객_PK: 고객번호
  -- 고객_X1: 관리사원번호
  -- 고객_X2: 최종주문금액
  ```
  - 둘 다 인덱스를 이용하여 액세스하지만 어떤 인덱스를 사용할지는 옵티마이저가 결정한다
    ```
    SELECT STATEMENT
        NESTED LOOPS
            TABLE ACCESS (BY INDEX ROWID) OF '사원' (TABLE)
                INDEX (RANGE SCAN) OF '사원_X1' (INDEX)
            TABLE ACCESS (BY INDEX ROWID) OF '고객' (TABLE)
                INDEX (RANGE SCAN) OF '고객_X1' (INDEX)
    ```
  - 사원_X1, 고객_X1 인덱스를 사용한 것을 볼 수 있다
  - 조건절 비교 순서: 2 -> 3 -> 1 -> 4
    - 2: 입사일자 조건에 만족하는 레코드를 찾으려고 사원_X1 인덱스를 스캔
    - 3: 사원_X1 인덱스에 읽은 ROWID 로 사원 테이블을 액세스하여 부서코드에 대한 필터 조건을 만족하는지 확인
    - 1: 사원 테이블에서 읽은 사원번호 값으로 조인 조건을 만족하는 고객 레코드를 고객_X1 인덱스로 스캔한다
    - 4: 고객_X1 인덱스에서 읽은 ROWID 로 고객테이블을 액세스해서 최종주문금액에 대한 필터 조건을 만족하는지 확인한다
  - 각 단계를 모두 완료하고 넘어가는 것이 아닌, 한 레코드 씩 순차적으로 진행

- NL 조인 튜닝 포인트
  - NL 조인 튜닝 포인트에 따라 각 단계의 수행 일량을 분석하여 과도한 랜덤 액세스가 발생하는 지점을 파악
    - 조인 순서 변경하여 랜덤 액세스를 줄일 수 있는지
    - 더 효과적인 인덱스가 있는지, 필요하다면 인덱스 추가 또는 구성 변경 고려
  - NL 조인으로 좋은 성능을 내기 어렵다고 판단된다면
    - 소트 머지 조인이나 해시 조인 검토
- NL 조인 특징 요약
  - 랜덤 액세스 위주의 조인 방식
  - 한 레코드씩 순차적으로 진행
    - 부분범위 처리가 가능한 상황에서는 빠른 성능을 가져올 수 있다.
  - 다른 조인 방식과 비교할 때 인덱스 구성 전략이 중요
- NL 조인 확장 메커니즘
  - 오라클은 NL 조인 성능을 높이기 위해 테이블 Prefetch, Batch I/O 기능을 도입했다
    - Table Prefetch: 인덱스를 이용해 테이블을 액세스하다가 디스크 I/O가 필요해지면, 이어서 곧 읽게 될 블록까지 미리 읽어서 버퍼 캐시에 적재하는 기능
    - Batch I/O: 디스크 I/O Call을 미뤘다가 읽을 블록이 일정량 쌓이면 한꺼번에 처리하는 기능
  - 표현 방식
    - 테이블 Prefetch 실행계획
      ```
      TABLE ACCESS (BY INDEX ROWID) OF '고객' (TABLE)
          NESTED LOOPS
              TABLE ACCESS (BY INDEX ROWID) OF '사원' (TABLE)
                  INDEX (RANGE SCAN) OF '사원_X1' (INDEX)
                      INDEX (RANGE SCAN) OF '고객_X1' (INDEX)
      ```
    - 배치 I/O 실행계획
      ```
      NESTED LOOPS
          NESTED LOOPS
              TABLE ACCESS (BY INDEX ROWID) OF '사원' (TABLE)
                  INDEX (RANGE SCAN) OF '사원_X1' (INDEX)
                      INDEX (RANGE SCAN) OF '고객_X1' (INDEX)
                  TABLE ACCESS (BY INDEX ROWID) OF '고객' (TABLE)
      ```
      
#### 소트 머지 조인

- 조인 컬럼에 인덱스가 없거나 대량 데이터 조인이어서 인덱스가 효과적이지 않을 때 소트 머지 조이, 해쉬 조인을 선택한다

- SGA vs PGA
  - SGA
    - 공유 메모리 영역인 SGA 에 캐시된 데이터는 여러 프로세스가 공유할 수 있지만, 동시에 액세스할 수 없다
    - 동시에 액세스하는 프로세스를 직렬화하기 위한 Lock 매커니즘으로 Latch가 존재한다
  - PGA
    - 오라클 서버 프로세스는 SGA 에 공유된 데이터를 읽고 쓰면서 동시에 자신만의 고유 메모리 영역을 갖는 데 해당 메모리 영역을 PGA라고 한다
    - 다른 프로세스와 공유하지 않기 때문에 래치 메커니즘이 불필요하며 같은 양의 데이터를 읽더라도 SGA 버퍼캐시에서 읽을 때 보다 빠르다

- 기본 메커니즘
  - 두 단계로 진행
    - 소트 단계: 양쪽 집합을 조인 컬럼 기준으로 정렬
    - 머지 단계: 정렬한 양쪽 집합을 서로 머지
  - 예시
    ```
    SELECT /*+ ordered use_merge( c ) */
      *
    FROM 사원 e, 고객 c
    WHERE c.관리사원번호 = e.사원번호
    AND e.입사일자>= TO_DATE('20210101', 'YYYYMMDD')
    AND e.부서코드 = 'Z123'
    AND c.최종주문금액 >= 20000
    ```
    
    - 힌트
      - Use_merge(table) 로 힌트를 작성한다
    - 사원 테이블에 대한 조건을 바탕으로 읽은 후 조인 컬럼인 사원번호 순으로 정렬하고 해당 결과 집합은 PGA 영역에 할당된 Sort Area에 저장
    - 고객 테이블에 대한 조건을 바탕으로 읽은 후 조인 컬럼인 관리사원번호 순으로 저장, PGA 영역에 저장
    - 정렬한 결과가 PGA에 담을 수 없을 정도로 크면, Temp 테이블스페이스에 저장
    - PGA에 저장한 사원테이블을 스캔하며 PGA or Temp에 있는 고객 데이터와 조인

   ```
   Begin
      for outer in (select * from PGA_sorted_사원) 
      loop -- outer
          for inner in (select * from PGA_sorted_고객 where 관리사원번호=outer.사원번호)
          loop -- inner
              dbms_output.putline('…');    
          end loop;
      end loop;
   end
   ```

    - 사원 데이터를 기준으로 고객 데이터를 매번 full scan 할 필요가 없다
    - 조인 대상 레코드가 시작되는 지점을 쉽게 찾을 수 있다
			 
- 소트 머지 조인이 빠른 이유
  - 소트 머지 조인과 NL 조인에 차이점은 미리 정렬해 둔 자료구조를 이용한다는 점
  - 하지만 NL 조인은 조인과정에서 액세스 하는 모든 블록을 랜덤 액세스 방식으로 db buffer cache를 경유해서 읽는다 (래치 획득 및 캐시 버퍼 체인 스캔 과정)
  - 소트 연산이 추가로 수행하기 때문에 소트에 대한 부하로 소량에 데이터에는 이슈가 있을 수 있다
  - 조인 컬럼에 대한 인덱스 유무에도 영향을 받지 않는다.
		
- 소트 머지 조인의 주용도
  - 소트 머지 조인보다 성능이 좋은 해쉬 조인이 나타나면서 소트 머지 조인에 인기는 줄어들었다.
  - 다만, 아래와 같은 경우에는 소트 머지 조인이 주로 사용된다.
  - 조인 조건식이 등치(=) 조건이 아닌 대량 데이터 조인
  - 조인 조건식이 아에 없는 조인(Cross Join)
			 
- 소트 머지 조인 제어하기
  - 실행 계획
    ```
    SELECT STATEMENT OPTIMIZER=ALL_ROWS
      MERGE JOIN
        SORT (JOIN)
            TABLE ACCESS (BY INDEX ROWID) OF '사원' (TABLE)
                INDEX (RANGE SCAN) OF '사원_X1' (INDEX)
        SORT (JOIN)
            TABLE ACCESS (BY INDEX ROWID) OF '고객' (TABLE)
                INDEX (RANGE SCAN) OF '고객_X1' (INDEX)
    ```
			 
    - 양쪽 테이블을 각각 소트한 후, 위쪽 사원 테이블 기준으로 아래쪽 고객 테이블과 머지 조인한다

#### 해시 조인

- 소트 머지 조인과 해시 조인은 조인 과정에서 인덱스를 이용하지 않기 때문에 대량 데이터를 조인할 때 NL 조인보다 빠르고, 일정한 성능을 보인다
- 소트 머지 조인은 항상 양쪽 테이블을 정렬하는 부담이 있는데, 해시 조인은 해당 부담도 없다

- 기본 메커니즘
  - 두 단계로 나누어 진행
    - Build 단계: 작은 쪽 테이블(build input) 을 읽어 해시 테이블(해시 맵) 생성
    - Probe 단계:  큰 쪽 테이블(probe input) 을 읽어 해시 테이블을 탐색하며 조인
  -  예시
    ```
    SELECT /*+ ordered use_hash( c )  */
      *
    FROM 사원 e, 고객 c
    WHERE c.관리사원번호 = e.사원번호
    AND e.입사일자 >= TO_DATE('20210101', 'YYYYMMDD')
    AND e.부서코드 = 'Z123'
    AND c.최종주문금액 >= 20000
    ```
    - Build 단계: 조인컬럼인 사원번호를 해시 테이블 키로 사용하여 해시 테이블 생성. 해시 테이블은 PGA 영역에 할당된 Hash Area에 저장되며 공간이 큰 경우 Temp 테이블스페이스에 저장
    - Probe 단계: 고객 데이터를 하나씩 읽어 앞서 생성한 테이블을 탐색. 즉, 관리사원번호를 해시 함수에 입력하여 반환된 값으로 해시 체인을 찾고, 해시 체인을 스캔하여 같은 사원번호를 찾는다
    ```
    Begin
      for outer in (select * from 고객 where 최종주문금액 >= 20000)
      loop -- outer
        for inner in (select * from PGA_사원_생성맵 where 사원번호 = outer.관리사원번호)
        loop
          dbms_output.put_line(…);
        end loop;
      end loop;
    end
    ```
- 해시 조인이 빠른 이유
  - NL 조인 비교
    - 해시 테이블이 빠른 이유는 소트 머지 조인과 같이 해시 테이블을 PGA 영역에 할당하기 때문이다
    - NL 조인은 Outer 테이블 레코드마다 Inner 테이블 레코드를 읽기 위해 래치 획득 및 캐시버퍼 체인 스캔 과정을 반복하지만 해시 조인은 래치 획득 과정없이 PGA에서 빠르게 데이터를 탐색하고 조인
    - 물론 Build Input, Probe Input을 읽을 때에는 DB 버퍼캐시를 경우하며, 이 때 인덱스를 이용하기도 한다.
  - 소트 머지 조인 비교
    - 소트 머지 조인처럼 양쪽 집합을 미리 정렬하는 부하가 없다
    - 물론 해시 테이블을 생성하는 비용이 수반되지만, 둘 중 작은 집합을 Build Input을 삼기 때문에 부담이 크지 않다

- 대용량 Build Input 처리
  - 만약 두 테이블 모두 대용량 테이블이어서 인메모리 해시 조인이 불가능한 상황에는 Divide & Conquer 방식을 사용하자
  - 파티션 단계
    - 조인하는 양쪽 집합의 조인 컬럼에 해시 함수 적용
    - 반환된 해시값에 따라 동적으로 파티셔닝한다. (여러 개의 서브 집합으로 분할함으로써 파티션 짝을 생성하는 단계) 
  - 조인 단계
    - 파티션 짝에 대해 하나씩 조인을 수행
    - 각각에 대해 build input, probe input은 독립적으로 결정
    - 모든 파티션 짝에 대한 처리를 마칠 때까지 반복

- 해시 조인 실행계획 제어
  ```
  SELECT STATEMENT Optimizer=ALL_ROWS
    HASH JOIN
      TABLE ACCESS (BY INDEX ROWID) OF '사원' (TABLE)
        INDEX (RANGE SCAN) OF '사원_x1' (INDEX)
      TABLE ACCESS (BY INDEX ROWID) OF '고객' (TABLE)
       INDEX (RANGE SCAN) OF '고객_n1' (INDEX)
  ```
  - 위쪽 사원 데이터로 해시 테이블을 생성한 후, 아래 고객 테이블에서 읽은 조인 키 값으로 해시 테이블을 탐색하면서 조인
  - 힌트에서 swap_join_inputs로 build input을 명시저으로 선택할 수 있다
    - /*+ leading( e ) use_hash( c ) swap_join_inputs( c ) */

- 조인 메서드 선택 기준
  - 데이터 양에 따른 조인 방식 선택 기준
    - 소량 데이터 조인: NL 조인
    - 대량 데이터 조인: 해시 조인
    - 대량 데이터 조인이지만 조인 조건식이 등치가 아니거나, 카테시안 곱인 경우: 소트 머지 조인
    - 대량은 NL 조인 기준으로 최적화하였지만, 랜덤 액세스가 많아 성능이 저하되는 경우를 의미한다

  - 수행 빈도
    - (최적화된) NL 조인과 해시 조인 성능이 같으면 NL 조인
    - 해시 조인이 약간 더 빨라도 NL 조인
    - NL 조인보다 해시 조인이 매우 빠른 경우, 해시 조인
    - NL 조인이 우선순위가 높은 이유는 인덱스는 영구 저장되는 자료구조이고, PGA에 저장되는 데이터는 조인이 끝나면 소멸하는 자료구조이다.

#### 참고

- 조시형 저자의 개발자를 위한 SQL 튜닝 입문서 친절한 SQL 튜닝
  - https://url.kr/fjm9l2
