---
layout: post
title: Spring DB. Transaction
summary: Spring DB
author: devhtak
date: '2023-05-21 21:41:00 +0900'
category: Spring
---

#### Transaction
- 트랜잭션이란
  - 트랜잭션(Transaction)을 이름 그대로 번역하면 거래한다는 뜻
  - 데이터베이스에서 트랜잭션은 하나의 거래를 안전하게 처리하도록 보장해주는 것을 뜻한다
    - 모든 작업이 성공해서 데이터베이스에 정상 반영하는 것을 커밋(Commit)
    - 작업 중 하나라도 실패해서 거래 이전으로 되돌리는 것을 롤백(Rollback)

- 트랜잭션 특징(ACID)
  - 원자성 (Actomicity)
    - 트랜잭션 내에서 실행한 작업들은 마치 하나의 작업인 것처럼 모두 성공하거나 모두 실패
  - 일관성 (Consistency)
    - 모든 트랜잭션은 일관성 있는 데이터베이스 상태를 유지
    - 예를 들어 데이터베이스에서 정한 무결성 제약 조건을 항상 만족해야한다.
  - 격리성(Isolation)
    - 동시에 실행되는 트랜잭션들이 서로에게 영향을 미치지 않도록 격리한다.
    - 예를들어 동시에 같은 데이터를 수정하지 못하도록 해야합니다.
    - 격리성은 동시성과 관련된 성능 이슈로 인해 트랜잭션 격리 수준(Isolation level)을 선택할 수 있습니다.
  - 지속성 (Durability)
    - 트랜잭션을 성공적으로 끝내면 그 결과가 항상 기록되어야 합니다.
    - 중간에 시스템에 문제가 발생해도 데이터베이스 로그 등을 사용해서 성공한 트랜잭션 내용을 복구해야 합니다.

- 트랜잭션 격리 수준(Isolation Level)
  - READ UNCOMMITED
    - 각 트랜잭션의 변경 내용이 Commit 이나 Rollback 여부에 상관없이 변경 중 다른 트랜잭션(작업) 에 그대로 보여진다
    - Dirty Read 
      - 다른트랜잭션이 수정중인 데이터를 볼 수 있는 현상을 말합니다.
      - 데이터가 나타났다가 사라졌다하는 현상을 초래할 수 있다
  - READ COMMITED
    - DB 에서 Default 로 사용하는 Isolation Level
    - 어떠한 트랜잭션에서 데이터를 변경하더라도 커밋이 완료된 데이터만 조회하기 때문에 Dirty Read 현상이 발생하지 않는다.
    - NON_REPEATABLE READ
      - 하나의 트랜잭션 내에서 동일한 Select 를 날렸을 때 항상 같은결과를 보장해야한다는 정합성
      - A 트랜잭션 작업 중 다른 B 트랜잭션의 커밋이 완료되어 데이터가 변경되면, 변경된 데이터로 Select를 하기 때문에 조회결과가 바뀔 수 있다
  - REPEATABLE READ
    - 언두(Undo)영역에 백업된 이전 데이터를 통해 트랜잭션 내에서는 동일한 결과를 보여 주도록 보장하여 Non-Repeatable Read 문제를 해결
    - Phantom Read
      - select ~ for update 와 같은 쓰기 잠금을 거는 경우 다른 트랜잭션에 수행한 변경 작업에 의해 레코드가 보였다가 안보였다가 하는 현상을 말합니다.
  - SERIALIZABLE 란
    - 가장 단순하며 엄격한 격리 수준으로 동시 처리 성능도 다른 트랜잭션 격리 수준보다 떨어집니다.
    - 한 트랜잭션에서 읽고 쓰고있는 레코드는 다른트랜잭션에서 아예 접근할 수 없도록 Lock을 걸어버리는 격리 수준입니다.
    - Serializable 에서는 모든 부정합 문제가 발생하지 않습니다.

- 데이터베이스 연결 구조와 DB 세션
  - 사용자는 웹 어플리케이션 (WAS) 나 DB 접근 툴 같은 클라이언트를 사용하여 DB 서버에 접근 가능
  - 클라이언트는 데이터베이스 서버에 연결을 요청하고 커넥션을 맺는다.
    - 이때 데이터베이스 서버는 내부에 세션이라는 것을 만듭니다.
  - 앞으로 해당 커넥션을 통한 모든 요청은 이 세션을 통해 실행
    - 세션은 트랜잭션을 시작하고, 커밋 또는 롤백을 통해 트랜잭션을 종료
  - 이후에 새로운 트랜잭션을 다시 시작할 수 있습니다.
  - 사용자가 커넥션을 닫거나, 또는 DBA가 세션을 강제로 종료하면 세션은 종료됩니다.

- 트랜잭션 정리
  - 원자성
    - 트랜잭션 내에서 실행한 작업들은 마치 하나의 작업인 것처럼 모두 성공하거나 모두 실패해야 한다
    - 트랜잭션의 원자성 덕분에 여러 SQL 명령어를 마치 하나의 작업인 것처럼 처리 가능
  - 오토 커밋
    - 만약 오토 커밋 모드로 동작하는데, 계좌이체 중간에 실패하면 쿼리 하나 실행할 때마다 커밋이 되기 때문에 데이터 정합성이 깨지게 된다.
  - 트랜잭션 시작
    - 이런 종류의 작업은 꼭 수동 커밋 모드를 사용해서 수동으로 커밋, 롤백 할 수 있도록 해야합니다.
    - 보통 이렇게 자동 커밋모드에서 수동 커밋 모드로 전환하는 것을 트랜잭션의 시작이라 합니다.

#### 트랜잭션 추상화 - PlatformTransactionManager

- 트랜잭션은 구현 기술마다 제공하는 방법이 다르다
  - JDBC 자동 커밋 취소: connection.setAutoCommit(false);
  - JPA: transaction.begin()
- 스프링의 트랜잭션 추상화 인터페이스를 활용하여 원하는 구현체를 DI 해 사용한다
  - PlatformTransactionManager
    - 구현체로는 JDBC 트랜잭션 매니저: JdbcTransactionManager, JPA 트랜잭션 매니저: JpaTransactionManager 등이 있다
  ```java
  public interface PlatformTransactionManager extends TransactionManager{
      TransactionStatus getTransaction(@Nullable TransactionDefinition definition) throws TransactionException;
      void commit(TransactionStatus status) throws TransactionException;
      void rollback(TransactionStatus status) throws TransactionException;
  }
  ```
- 예제
  ```java
  @Autowired
  private PlatformTransactionManager transactionManager;
  public void orderItem(String itemId, String memberId, int qty) throws SQLException {
      TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());
      try {
          buisinessLogic(itemId, memberId, qty);
          transactionManager.commit(status);
      } catch (Exception e) {
          transactionManager.rollback(status);
      }
  }
  ```
 
- 트랜잭션 동기화 매니저를 통해 트랜잭션 종료까지 커넥션을 동기화(유지)해주는 기능을 제공한다.
  - 쓰레드 로컬: 쓰레드가 각자 스택을 갖는 것 처럼 싱글톤인 스프링 빈도 각자의 저장소를 갖는 것
  - 동기화 매니저는 이 쓰레드 로컬을 사용하여 커넥션을 저장하기 때문에 커넥션을 동기화한다
  - 순서
    - 비즈니스 로직에서 Transaction을 시작하면 TransactionManager는 DataSource에 ConnectionPool에서 Connection 을 획득
    - setAutoCommit(false)를 통해 수동 커밋 모드로 변경
    - 동기화 매니저에 커넥션을 보관하면, 트랜잭션 동기화 매너지는 쓰레드 로컬에 커넥션을 보관한다
    - 비즈니스 로직을 실행하면 커넥션을 동기화 매니저에서 획득하여 SQL을 데이터베이스에 전달하여 실행
    - 종료를 위해서 트랜잭션 동기화 매니저로부터 동기화된 커넥션을 획득, DB에 commit, rollback 등을 진행한다
    - 트랜잭션 동기화 매니저와 쓰레드 로컬을 정리하고, 수동 모드에서 다시 자동 커밋 모드로 되돌린다. connection.close()를 호출해 커넥션 풀 반환

- DataSourceUtils
  ```java
  private void close(Connection connection, Statement statement, ResultSet resultSet) {
      JdbcUtils.closeResultSet(resultSet);
      JdbcUtils.closeStatement(statement);
      DataSourceUtils.releaseConnection(connection, dataSource);
  }

  private Connection getConnection() throws SQLException {
      return DataSourceUtils.getConnection(dataSource);
  }
  ```
  - TransactionManager를 사용해 커넥션을 이용할 때 사용
  - getConnection()은 트랜잭션 동기화 매니저를 사용하며, 만약 관리중인 커넥션이 있으면 이를 반환하고, 없다면 새로운 커넥션을 생성하여 반환
  - releaseConnection() 은 트랜잭션을 사용하기 위해 동기화된 커넥션은 닫지 않고, 유지하지만, 트랜잭션 동기화 매니저가 관리하는 커넥션이 없다면 바로 닫는다

#### TransactionTemplate

- 트랜잭션이 추상화된 인터페이스를 사용하더라도, 동일한 골격을 생성하는 것은 동일하다.
- 템플릿 콜백 패턴을 활용하면 코드 중복을 해결할 수 있다
  - 스프링에서는 TransactionTemplate을 통해 이미 구현된 트랜잭션 관련 코드를 한곳에 몰아 놓고, 우리는 비즈니스 로직만 작성해 주입하면 된다
- TransactionTemplate
  ```java
  @Override
	@Nullable
	public <T> T execute(TransactionCallback<T> action) throws TransactionException {
    Assert.state(this.transactionManager != null, "No PlatformTransactionManager set");

    if (this.transactionManager instanceof CallbackPreferringPlatformTransactionManager) {
      return ((CallbackPreferringPlatformTransactionManager) this.transactionManager).execute(this, action);
    }
    else {
      TransactionStatus status = this.transactionManager.getTransaction(this);
      T result;
      try {
        result = action.doInTransaction(status);
      } catch (RuntimeException | Error ex) {
        // Transactional code threw application exception -> rollback
        rollbackOnException(status, ex);
        throw ex;
      } catch (Throwable ex) {
        // Transactional code threw unexpected exception -> rollback
        rollbackOnException(status, ex);
        throw new UndeclaredThrowableException(ex, "TransactionCallback threw undeclared checked exception");
      }
      this.transactionManager.commit(status);
      return result;
    }
  }
  ```
  - action 인자를 넘겨주면 된다.
  ```java
  public void orderItemV2(String itemId, String memberId, int qty) {
      template.executeWithoutResult(status -> buisinessLogic(itemId, memberId, qty));
  }
  ```

#### 트랜잭션 AOP
- 트랜잭션 프록시
  - Client -> Proxy 호출(Trasaction 시작 - buisiness logic(서비스) - Transaction 종료) -> Service(Buisiness Logic) -> Repository
  - 서비스에서 직접 트랜잭션을 처리하는 것이 아닌 프록시가 대신 트랜잭션 관련 로직을 처리한다
  - Spring에서 직접 AOP를 구현해도 되지만, @Transactional 애노테이션을 통해 자동으로 AOP를 적용해준다
- 과정
  - 프록시 호출하면 트랜잭션 시작 부분에서 스프링 컨테이너를 통해 트랜잭션 매니저를 획득한다
  - 트랜잭션 매니저를 통해 트랜잭션을 획득(transactionManager.getTransaction()), DataSource에서 Connection 획득, setAutoCommit(false)를 통해 수동 커밋 모드로 변경
  - 트랜잭션 동기화 매니저에 커넥션을 보관
  - 실제 서비스(비즈니스 로직)를 호출하고, Repository에서는 트랜잭션 동기화 매니저에서 커넥션을 획득하여 SQL 실행

- Spring Boot
  - Spring Boot 이전에는 DataSource와 TransactionManager를 빈으로 등록해주어야 했다
    ```java
    @Bean
    DataSource dataSource() {
        return new DriverManagerDataSource(URL, USERNAME, PASSWORD);
    }
    @Bean
    PlatformTransactionManager platformTransactionManager() {
        return new DataSourceTransactionManager(dataSource());
    }
    ```
  - SpringBoot에서는 빈을 등록하는 과정을 자동화 해두었다(@EnableAutoConfiguration)
    - application.yml 에 datasource 관련 정보를 작성하면, 지정된 속성을 참고하여 데이터 소스와 트랜잭션 매니저를 자동으로 생성
    
#### JdbcTemplate 예외 처리

- Exception은 CheckedException, RuntimeException(UncheckedException) 으로 나뉜다
  - RuntimeException 하위 예외들은 런타임 예외로 컴파일러가 확인하지 않고, 실행 시점에 발생
- RuntimeException 활용
  - 반드시 잡아야하는 비즈니스 로직 상의 예외를 제외하고는 기본적으로 런타임 예외를 사용하여 예외를 생성한다.
  - 예외를 핸들링하는 하는 책임은 ControllerAdvice 에게 있다.
    - Service, Repository 에 책임과는 다르기 때문에 단일 책임 원칙(SRP)에 어긋난다.
- Spring 예외 변환기와 JdbcTemplate
  - SQLErrorCodeSQLExceptionTranslator
    ```java
    @Override
    @Nullable
    protected DataAccessException doTranslate(String task, @Nullable String sql, SQLException ex) {
	SQLException sqlEx = ex;
	if (sqlEx instanceof BatchUpdateException && sqlEx.getNextException() != null) {
		SQLException nestedSqlEx = sqlEx.getNextException();
		if (nestedSqlEx.getErrorCode() > 0 || nestedSqlEx.getSQLState() != null) {
			sqlEx = nestedSqlEx;
		}
	}

	// First, try custom translation from overridden method.
	DataAccessException dae = customTranslate(task, sql, sqlEx);
	if (dae != null) {
		return dae;
	}

	// Next, try the custom SQLException translator, if available.
	SQLErrorCodes sqlErrorCodes = getSqlErrorCodes();
	if (sqlErrorCodes != null) {
		SQLExceptionTranslator customTranslator = sqlErrorCodes.getCustomSqlExceptionTranslator();
		if (customTranslator != null) {
			DataAccessException customDex = customTranslator.translate(task, sql, sqlEx);
			if (customDex != null) {
				return customDex;
			}
		}
	}
	// ...
    }
    ```
    - CheckedException 에 경우 메소드에서 예외를 던져야 하기 때문에 추상화의 의미를 퇴색시킨다.
    - 스프링 예외를 분석하여 자동으로 스프링의 런타임 예외로 변환해주는 예외 변환기를 제공하며 DataAccessException 리턴ㅓ
    
- JdbcTemplate에서 transalteException
  ```java
  protected DataAccessException translateException(String task, @Nullable String sql, SQLException ex) {
	DataAccessException dae = getExceptionTranslator().translate(task, sql, ex);
	return (dae != null ? dae : new UncategorizedSQLException(task, sql, ex));
  }
  ```
  - transalteException은 catch 문에서 실행되며, Translator를 활용하여 CheckedException을 UncheckedException 을 변환하는 역할을 한다.

#### @Transactional
- 스프링은 PlatformTransactionManager로 여러 트랜잭션 구현체를 추상화 두었다.
  - Spring Boot는 @AutoConfiguration을 통해 사용 기술에 따라 자동으로 TransactionManager를 등록해준다.
- @Transactional
  - AOP가 적용되어 트랜잭션과 비즈니스 로직을 분리할 수 있다.
- Proxy로 인한 트랜잭션이 적용되지 않은 문제 발생 가능!!
  - 트랜잭션 어노테이션을 적용 하면
    - Client -> Transaction Proxy 호출(PlatformTransactionManager를 통해 Transaction을 얻고, 여기사 DataSource(CP)에서 커넥션을 얻고, 이는 동기화 매니저에 보관)
    - Transaction Proxy는 @Transactional이 적용된 서비스에 비즈니스 로직을 호출
    - 비즈니스 로직 수행 중 데이터 로직을 수행하면 데이터 로직은 동기화 매니저에 보관한 커넥션을 획득하여 사용
  - 비즈니스 로직 내부에서 메소드를 호출한다면? -> Proxy 객체를 통해 호출하는 것이 아니기 때문에 트랜잭션이 적용될 수 없다.
    ```java
    @Service
    public class TransactionExampleService {
    	public void extenralCall() {
	    log.info("external");
	    internalCall(); // 내부에서 호출
	}
	@Transactional
	public void internalCall() {
	    log.info("internal");
	    // transaction logic
	}
    }
    ```
    - this.internalCall()을 호출하기 때문에 트랜잭션 로직이 탈 수 없다.
  - 해결 방법
    - 가장 간단한 방법은 내부 호출을 끄집어 외부 호출로 변경
      ```java
      @RequiredArgsConstructor
      public class OuterTransactionExampleService {
          private final InternalTransactionExampleService service;
          public void externalCall() {
	      log.info("external");
              service.internalCall();
	  }
      }
      @Service
      public class InternalTransactionExampleService {
      	  @Transactional
	  public void internalCall() {
	      log.info("internal");
	      // transaction logic
	  }
      }
      ```
    - @Transactional이 걸리는 메소드는 public 메소드에만 적용이 된다.
- rollbackFor 사용 필요한 경우
  - 기본적으로 RuntimeException는 롤백, CheckedException은 커밋되기 때문에, 필요한 예외에 경우 설정해주어야 한다.

#### 트랜잭션 전파
- 스프링은 논리 트랜잭션과 물리 트랜잭션 개념이 있다.
  - 트랜잭션이 적용된 외부 로직이 트랜잭션이 적용된 내부 로직을 호출한다면,
  - 논리트랜잭션은 외부 로직, 내부 로직을 따로 생각하고, 물리트랜잭션은 외부, 내부 로직을 하나의 트랜잭션으로 생각한다.
- 논리 트랜잭션들은 하나의 물리 트랜잭션으로 묶인다.
  - 모든 논리 트랜잭션이 커밋되어야 물리 트랜잭션이 커밋된다.
  - 하나의 논리 트랜잭션이 롤백되면 물리 트랜잭션은 롤백된다.
- 외부와 내부 트랜잭션의 분리 - REQUIRES_NEW
  - 외부 로직과 내부 로직 모두 새로운 커낵션을 획득하여 트랜잭션을 진행
  - 분리된 커넥션이기 때문에 내부 롤백, 외부 커밋처럼 다르게 가져갈 수 있다.
  - 새로운 커넥션을 얻기 때문에 DB 커넥션 낭비될 수 있기 때문에 잘 판단해야 한다.
- 이 외 Propagation 속성
  - MANDATORY, NEVER, NESTED
  - SUPPORT(트랜잭션이 없으면 그래도 진행, 기존 트랜잭션이 있으면 참여하여 진행), NOT_SUPPORT(트랜잭션이 있던, 없던 보류 시키고 트랜잭션 없이 진행)
