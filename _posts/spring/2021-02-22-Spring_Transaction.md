---
layout: post
title: Spring_@Transactional
summary: 선언적 트랜잭션 @Transactional
author: devhtak
date: '2021-02-21 22:41:00 +0900'
category: Spring
---

#### Transaction

- DB의 상태를 변경시키기 위해 수행하는 작업의 단위

- 특징 (ACID)
  - 원자성(Atomicity)
    - 원자성은 트랜잭션이 데이터베이스에 모두 반영되던가 아니면 전혀 반영되지 않아야 한다는 것이다.
  - 일관성(Consistency)
    - 일관성은 트랜잭션의 작업 처리 결과가 항상 일관성이 있어야 한다는 것이다.
  - 독립성(Isolation)
    - 독립성이란 둘 이상의 트랜잭션이 동시에 실행되고 있을 경우 어떤 하나의 트랜잭션이라도 다른 트랜잭션의 연산에 끼어들 수 없다는 점을 가리킨다.
  - 지속성(Durability)
    - 지속성이란 트랜잭션이 성공적으로 완료됐을 경우, 결과는 영구적으로 반영되어야 한다는 점이다.
    
- Commit과 Rollback 연산
  - Commit
    - 하나의 트랜잭션이 성공적으로 끝났고, DB가 일관성있는 상태에 있을 때, 하나의 트랜잭션이 끝났다는 것을 알려주기 위해 사용
    - commit을 수행하면 트랜잭션이 로그에 저장되며, 후에 Rollback 연산을 수행할 때 최신 commit을 기준으로 한다.
  - Rollback
    - 하나의 트랜잭션 처리가 비정상적으로 종료되어 트랜잭션이 원자성이 깨진 경우, 트랜잭션을 처음부터 다시 시작하거나, 트랜잭션의 부분적으로 연산된 결과를 다시 취소시킨다. 
  
- 출처: https://mommoo.tistory.com/62
  
#### @Transctional

- Spring에서 지원하는 애노테이션으로 선언적 트랜잭션이다.
- 클래스 또는 메서드위에 @Transactional이 추가되면, 이 클래스에 트랜잭션 기능이 적용된 프록시 객체가 생성된다.
- 이 프록시 객체는 @Transactional이 포함된 메소드가 호출될 경우 TransactionManager(PlatformTransactionManager)를 사용하여 트랜잭션을 시작하고, 정상 여부에 따라 commit, rollback한다.

- Spring Framework에서 @Transactional 설정
  - root-context.xml에 transactionManager 설정
    ```xml
    <!-- 트랜젝션 매니저 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"></property>
    </bean>
	
    <!-- @Transactional 애노테이션을 sacn하기 위한 설정 -->
    <tx:annotation-driven/>
    ```
    
  - JavaConfig
    ```java
    @EnableTransactionManagement 
    public class AppConfig { 
        // ... 
        @Bean
        public PlatformTransactionManager transactionManager() throws URISyntaxException, GeneralSecurityException, ParseException, IOException { 
            return new DataSourceTransactionManager(dataSource()); 
        }
    }
    ```
    - DataSourceTransactionManager
      - 단일 DataSource에 대한 PlatformTransactionManager 구현체로 설정된 DataSource당 하나의 스레드 바인딩을 허용한다.

- Spring Boot에서는 별도의 설정이 필요 없으며, 클래스 또는 메소드에 @Transactional을 선언할 수 있다.

- @Transactional 옵션

  |옵션|설명|
  |---|---|
  |propagation|트랜잭션 동작 도중 다른 트랜잭션을 호출할 때 어떻게 할 것인지 지정하는 옵션|
  |isolation|트랜잭션에서 일관성없는 데이터 허용 수준을 설정한다.|
  |noRollbackFor=Exception.class|특정 예외 발생시 rollback하지 않는다.|
  |timeout|지정한 시간 내에 메소드 수행이 완료되지 않으면 rollback한다. (-1일 경우 timeout을 사용하지 않는다.|
  |readOnly|트랜잭션을 읽기 전용으로 설정한다.|
  
  - propagation
    - 트랜잭션 동작 도중 다른 트랜잭션을 호출할 때 어떻게 할 것인지 지정하는 옵션
    
    |옵션|설명|
    |---|---|
    |REQUIRED(Default)|이미 진행중인 트랜잭션이 있다면 해당 트랜잭션 속성을 따르고, 진행중이 아니라면 새로운 트랜잭션을 생성한다|
    |REQUIRES_NEW|항생 새로운 트랜잭션을 생성한다. 이미 진행중인 트랜잭션이 있다면 잠깐 보류하고 해당 트랜잭션 작업을 먼저 진행한다|
    |SUPPORT|이미 진행 중인 트랜잭션이 있다면 해당 트랜잭션 속성을 따르고, 없다면 트랜잭션을 설정하지 않는다.|
    |NOT_SUPPORT|이미 진행중인 트랜잭션이 있다면 보류하고, 트랜잭션 없이 작업을 수행한다.|
    |MANDATORY|이미 진행중인 트랜잭션이 있어야만, 작업을 수행한다. 없다면 Exception을 발생시킨다.|
    |NEVER|트랜잭션이 진행중이지 않을 때 작업을 수행한다. 트랜잭션이 있다면 Exception을 발생시킨다.|
    |NESTED|진행중인 트랜잭션이 있다면 중첩된 트랜잭션이 실행되며, 존재하지 않으면 REQUIRED와 동일하게 실행된다.|
    
  - isolation
    - 트랜잭션에서 일관성없는 데이터 허용 수준을 설정한다.
    
    |옵션|설명|
    |---|---|
    |Default|사용하는 DB 드라이버의 디폴트 설정을 따른다. 대부분 READ_COMMITED를 기본 격리수준으로 설정한다.|
    |READ_UNCOMMITED|가장 낮은 격리 수준이다.  트랜잭션이 커밋되기 전에 그 변화가 다른 트랜잭션에 그대로 노출된다. 하지만 속도가 빠르기 떄문에 데이터의 일관성이 떨어지더라도, 성능 극대화를 위해 의도적으로 사용하기도 한다.|
    |READ_COMMITED|다른 사용자가 데이터를 변경하는 동안 사용자는 데이터를 읽을 수 없다. 하지만 트랜잭션이 읽은 로우를 다른 트랜잭션에서 수정 할 수 있다. 그래서 트랜잭션이 같은 로우를 읽었어도 시간에 따라서 다른 내용이 발견될 수 있다.|
    |REPEATABLE_READ|선행 데이터가 조회한 데이터는 트랜잭션이 종료될때까지 후행 트랜잭션잉 수정 삭제가 불가능. 트랜잭션이 읽은 로우를 다른 트랜잭션에서 수정되는 것을 막아준다. 하지만 새로운 로우를 추가하는 것은 제한하지 않는다.|
    |SERIALIZABLE|동일한 데이터에 대해서 두개의 트랜잭션이 수행 될 수 없다. 가장 강력한 트랜잭션 격리수준이다. 여러 트랜잭션이 동시에 같은 테이블 로우에 액세스하지 못하게 한다. 가장 안전하지만 가장 성능이 떨어진다.|
    
  - rollbackFor
    - 트랜잭션 작업 중 런타임 예외가 발생하면 롤백한다. 반면에 예외가 발생하지 않거나 체크 예외(IOException 등등)가 발생하면 커밋한다.
    - 하지만 원한다면 체크 예외를 롤백 대상으로 삼을 수 있다. rollbackFor 또는 rollbackForClassName 속성 이용
    
  - noRollbackFor
    - rollbackFor 속성과는 반대로 런타임 예외가 발생해도 지정한 런타임 예외면 커밋을 진행한다.
  
  - timeout
    - 트랜잭션에 제한시간을 지정한다. 초 단위로 지정하고, 디폴트 설정으로 트랜잭션 시스템의 제한시간을 따른다. 
    - -1 입력 시, 트랜잭션 제한시간을 사용하지 않는다.

  - readOnly
    - 트랜잭션을 읽기 전용으로 설정한다. 
    - 특정 트랜잭션 안에서 쓰기 작업이 일어나는 것을 의도적으로 방지하기 위해 사용된다. 
    - insert,update,delete 작업이 진행되면 예외가 발생한다.

#### 트랜잭션 격리 수준(Transaction Isolation Level)

- 트랜잭션에서 일관성 없는 데이터를 허용하도록 하는 수준
- Isolation level의 필요성

  - 데이터베이스는 ACID 특징과 같이 트랜잭션이 독립적인 수행을 하도록 한다.
  - 따라서 Locking을 통해, 트랜잭션이 DB를 다루는 동안 다른 트랜잭션이 관여하지 못하도록 막는 것이 필요하다.
  - 하지만 무조건 Locking으로 동시에 수행되는 수많은 트랜잭션들을 순서대로 처리하는 방식으로 구현하게 되면 데이터베이스의 성능은 떨어지게 될 것이다.
  - 그렇다고 해서, 성능을 높이기 위해 Locking의 범위를 줄인다면, 잘못된 값이 처리될 문제가 발생하게 된다.
  - 따라서 최대한 효율적인 Locking 방법이 필요함!

- 선택 시 고려사항
  - Isolation Level에 대한 조정은, 동시성과 데이터 무결성에 연관되어 있음
  - 동시성을 증가시키면 데이터 무결성에 문제가 발생하고, 데이터 무결성을 유지하면 동시성이 떨어지게 됨
  - 레벨을 높게 조정할 수록 발생하는 비용이 증가함
  - 낮은 단계 Isolation Level을 활용할 때 발생하는 현상들
  - 격리 수준에 따른 현상
  
    |격리 수준|DIRTY READ|NON-REPEATABLE READ|PHANTOM READ|
    |---|---|---|---|
    |READ UNCOMMITED|v|v|v|
    |READ COMMITED|-|v|v|
    |REPEATABLE READ|-|-|v|
    |SERIALIZABLE|-|-|-|
    
  - Dirty Read
    - 커밋되지 않은 수정중인 데이터를 다른 트랜잭션에서 읽을 수 있도록 허용할 때 발생하는 현상
    - 어떤 트랜잭션에서 아직 실행이 끝나지 않은 다른 트랜잭션에 의한 변경사항을 보게되는 경우

  - Non-Repeatable Read
    - 한 트랜잭션에서 같은 쿼리를 두 번 수행할 때 그 사이에 다른 트랜잭션 값을 수정 또는 삭제하면서 두 쿼리의 결과가 상이하게 나타나는 일관성이 깨진 현상

  - Phantom Read
    - 한 트랜잭션 안에서 일정 범위의 레코드를 두 번 이상 읽었을 때, 첫번째 쿼리에서 없던 레코드가 두번째 쿼리에서 나타나는 현상
    - 트랜잭션 도중 새로운 레코드 삽입을 허용하기 때문에 나타나는 현상임
    - 완벽한 읽기 일관성 모드를 제공함
    - 다른 사용자는 트랜잭션 영역에 해당되는 데이터에 대한 수정 및 입력 불가능

#### Lock

- Lock을 통해 ACID 원칙을 지켜낸다. 하지만, ACID 원칙을 strict하게 지켜내기 위해서는 동시성이 떨어질 수 있다.
- Database에서는 Isolation level에 따라 다양한 Lock을 제공함으로써 ACID 원칙과 동시성을 선택하게 해준다.

- Lock Mode
  - Exclusive lock mode
    - Write에 대한 lock
    - UPDATE, DELETE 문과 같이 데이터를 수정할 때 데이터에 Lock을 걸어 해당 트랜잭션의 잠금이 해제될 때까지 데이터를 변경할 수 없도록 한다.
    - 또 다른 Lock(Shared lock, Exclusive lock)이 걸릴 수 없다.
  - Shared lock mode
    - Read에 대한 lock
    - 여러 트랜잭션이 데이터를 읽어들일 때 read를 허락하지만 exclusive lock 이 필요(수정이 필요) 하는 트랜잭션이 접근하는 것을 막는다.
    - 일반적인 select 문은 shared lock이 발생하지 않는다. 다만 select ... for share 등 일부 select 쿼리는 read 작업을 수행할 때 S lock을 건다.
    - Exclusive lock이 걸릴 수 없다.

- Lock 설정 레벨(범위)
  - Table
    - 테이블 기준으로 Lock이 걸린다.
    - 주로 DDL 구문을 사용할 때 Lock이 걸리기 때문에 DDL Lock이라고 한다.
  - Column
    - 컬럼 기준으로 Lock을 설정할 수 있지만, Lock 설정 및 해제의 리소스가 많이 들어 일반적으로 잘 사용하지 않는다.
  - Row
    - 1개의 Row를 기준으로 Lock을 설정한다. DML에 대한 Lock으로 가장 일반적으로 사용하는 Lock
    - Record Lock
      - Index record에 걸리는 lock
      - 예시)
        - Test 란 테이블에 index로 col1이 걸려있다
        - Transaction 1.
        
          ```
          UPDATE Test
          SET COL2 = 'TEST'
          WHERE COL1=10
          ```
	  
        - Transaction 2.
          
          ```
          DELETE FROM Test
          WHERE COL1=10
          ```
	  
        - Transaction 1이 COL1=10인 index record에 락을 걸어둔 상태이기 때문에 commit이나 rollback하기 전에 Transaction2가 row를 삭제할 수 없다.      
	
    - Gap Lock 
      - index table에 없는 index에 대하여 lock을 건다.
      - 예시)
        - id가 인덱스로 걸려있는 Test란 테이블이 있다. 여기에 현재 데이터는 3, 7만 존재한다.
          
          ```
          SELECT id FROM Test;
          id
          ---
          3
          7
          ---
          ```
        - Transaction 1.
	  
          ```
          UPDATE Test 
          SET COL1='TEST'
          WHERE id BETWEEN 1 AND 10;
          ```
	
        - Transaction 2.
	
          ```
          INSERT Test(id, COL1) VALUES(2, 'TEST01');
          ```
	
         - 2란 id값은 없지만 1 부터 10까지의 id에 Exclusive Lock을 걸은 상태로 transaction1이 commit이나 rollback을 하기 전까지 insert를 하지 못하게 된다.

- Blocking 과 Deaklock
  - Blocking은 Lock간의 경합이 발생하여 특정 Transaction이 작업을 진행하지 못하고 멈춰선 상태
    - shared lock 간에는 블로킹이 발생하지는 않지만 shared lock + exclusive lock 또는 exclusive lock + exclusive lock 간에는 경합이 발생
    - 한 트랜잭션의 길이를 너무 길게하면 blocking이 발생할 가능성이 높다.
  - Deadlock은 두 트랜잭션이 각각 Lock을 설정하고 다음 서로의 Lock에 접근하여 값을 얻어오려고 하는 경우 발생한다.
    - 교착상태에 빠지면 계속 Lock을 건 상태에서 끝나지 않기 떄문에 DBMS에서 둘 중 한 트랜잭션에 에러를 발생시켜 해결한다.
    - 교착상태를 해결하기 위해서는 DB 접근 순서를 동일하게 하는 것이 중요하다.
      - master -> sub 테이블 순으로 update 

- Consistent Read (기준: InnoDB)
  - Read operation을 수행할 때 현재 DB의 값이 아닌 Snapshot으로부터 읽어오는 것이다.
  - Snapshot은 commit된 변화만이 적용된 상태를 의미
  - InnoDB에서는 쿼리의 로그를 활용하여 Snapshot을 복구하여 가져온다.

- Isolation Level에 따른 Lock (기준: InnoDB)
  - Read Uncommited
    - Consistent Read를 지원하지 않는다.
    - 즉, 해당 시점의 DB를 읽기 때문에 dirty read가 발생한다.
  - Read Commited
    - read operation(select) 마다 snapshot을 다시 뜬다.
    - consisten read를 수행하여 새로 뜬 snapshot을 보기 때문에 dirty read가 발생하지 않는다.
    - record lock을 지원하며 gap lock은 사용하지 않는다.
  - Repeatable Commit
    - 처음 실행하는 read operation 수행시간을 기록한다. 그리고 그 이후 모든 read operation마다 해당 시점을 기준으로 consistent read를 수행한다.
    - record lock과 gap lock을 사용한다.
  - Serializable
    - 모든 select 문을 select ... for share로 변경하여 shared lock을 건다.
    - select문에 lock을 걸기 때문에 deadlock이 걸릴 가능성이 높아 사용할 때 조심해야 한다.
    
- 출처: https://bamdule.tistory.com/51
- 출처: https://www.hanumoka.net/2018/09/11/spring-20180911-spring-Transactional/
- 출처: https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jdbc/datasource/DataSourceTransactionManager.html
- 출처: https://sabarada.tistory.com/121
- 출처: https://docs.oracle.com/database/121/CNCPT/consist.htm#CNCPT88972
- 출처: https://suhwan.dev/2019/06/09/transaction-isolation-level-and-lock/
