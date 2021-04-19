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
    |READ_COMMITED|트랜잭션이 커밋하지 않은 정보는 읽을 수 없다. 하지만 트랜잭션이 읽은 로우를 다른 트랜잭션에서 수정 할 수 있다. 그래서 트랜잭션이 같은 로우를 읽었어도 시간에 따라서 다른 내용이 발견될 수 있다.|
    |READ_UNCOMMITED|가장 낮은 격리 수준이다.  트랜잭션이 커밋되기 전에 그 변화가 다른 트랜잭션에 그대로 노출된다. 하지만 속도가 빠르기 떄문에 데이터의 일관성이 떨어지더라도, 성능 극대화를 위해 의도적으로 사용하기도 한다.|
    |REPEATABLE_READ|트랜잭션이 읽은 로우를 다른 트랜잭션에서 수정되는 것을 막아준다. 하지만 새로운 로우를 추가하는 것은 제한하지 않는다.|
    |SERIALIZABLE|가장 강력한 트랜잭션 격리수준이다. 여러 트랜잭션이 동시에 같은 테이블 로우에 액세스하지 못하게 한다. 가장 안전하지만 가장 성능이 떨어진다.|
    
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
  
- 출처: https://bamdule.tistory.com/51
- 출처: https://www.hanumoka.net/2018/09/11/spring-20180911-spring-Transactional/
- 출처: https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jdbc/datasource/DataSourceTransactionManager.html
