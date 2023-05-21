---
layout: post
title: Spring DB. JDBC의 이해
summary: Spring DB
author: devhtak
date: '2023-05-21 21:41:00 +0900'
category: Spring
---

#### JDBC
- JDBC란?
  - Java Database Connectivity 는 자바에서 데이터베이스에 접속할 수 있도록 하는 API
- JDBC 등장 이유
  - Application 와 DB
    - 커넥션 연결: 주로 TCP/IP를 이용하여 커넥션 연결
    - SQL 전달: 애플리케이션 서버는 DB가 이해할 수 있는 SQL을 연결된 커넥션을 통해 DB에 전달
    - 결과 응답: DB는 전달된 SQL을 수행하고 그 결과를 응답, 애플리케이션 서버는 응답 결과를 활용
  - 문제점 - 각각의 데이터베이스 사용 방법이 다르고, 접근 방법이 달라 DB를 변경할 때마다 접근 코드도 함께 변경되어야 한다
  - 이런 문제를 해결하기 위하여 JDBC 등장
    - JDBC 표준 인터페이스(Connection, Statement, ResultSet) -> (상속)MySqlDriver, OracleDriver ..
    - java.sql.Connection - 연결
    - java.sql.Statement- SQL을 담은 내용
    - java.sql.ResultSet - SQL 요청 응답
- JDBC로 인한 문제 해결
  - DB 변경 시, 불필요한 애플리케이션의 코드도 변경되어야 했던 문제 해결
    - 데이터베이스 연결 로직을 jdbc 인터페이스에 의존하게 됨으로써, 다른 데이터베이스 종료로 변경하여도, jdbc 구현 라이브러리만 변경해주면 됩니다.
    - 어플리케이션의 비지니스 로직은 영향을 받지 않습니다.
  - DB 마다 달랐던 커넥션 연결, SQL 전달, 응답 방법을 JDBC 표준 라이브러리가 동일하게 제공하기 때문에 따로 학습하지 않아도 된다.
- JDBC 한계
  - jdbc 표준 인터페이스 등장으로 일반적인 부분은 공통화 하였지만, 각각 데이터베이스마다 제공하는 기능과 특별한 양식들은 다르기 때문에 결국, 한계가 있었다고 합니다.
    - 페이징 SQL - 각 db 마다 사용방법이 다름
  - 데이터베이스 연결 코드는 JDBC 를 사용함으로 써, 변경하지 않아도 되었지만 SQL 문은 해당 데이터베이스에 맞게 수정해주어야 하는 한계점이 존재합니다.
    - ORM가 상당부분 해결

#### SQL Mapper vs ORM
- SQL Mapper
  - 장점: JDBC를 편리하게 사용하도록 도와준다.
    - JDBC의 반복 코드를 제거해준다.
    - SQL 응답 결과를 객체로 편리하게 변환해준다.
  - 단점: 개발자가 SQL을 직접 작성해야한다. (어디까지나 ORM 에 비해서)
- ORM
  - ORM은 객체를 관계형 데이터베이스 테이블과 매핑
  - 개발자는 반복적인 SQL을 직접 작성하지 않고, ORM 기술이 개발자 대신에 SQL을 동적으로 만들어 실행
  - 추가로 각각의 데이터베이스마다 다른 SQL을 사용하는 문제도 중간에서 해결해준다.
- SQL Mapper, ORM 모두 base low level 에서는 JDBC 사용

#### JDBC Connection
- JDBC 커넥션을 얻어, H2 DB 연결 예제
- ConnectionConst.class
  ```java
  public class ConnectionConst {
      public static final String URL = "jdbc:h2:tcp://localhost/~/test";
      public static final String USERNAME = "sa";
      public static final String PASSWORD = "";
  }
  ```
  - H2 DB 접속정보 저장
- DBConnectionUtil.class
  ```java
  @Slf4j
  public class DBConnectionUtil {
      public static Connection getConnection() {
          Connection connection = null;
          try {
              connection = DriverManager.getConnection(ConnectionConst.URL, ConnectionConst.USERNAME, ConnectionConst.PASSWORD);
              log.info("Connection = {}, class = {}", connection, connection.getClass());
          } catch (SQLException e) {
              throw new IllegalStateException(e);
          }
          return connection;
      }
  }
  ```
  - JDBC를 이용해서, 실제 h2 에 연결할 커넥션을 얻어오는 코드
  - JAVA 에서 데이터베이스에 연결하려면 JDBC가 제공하는 DriverManager.getConnection(..) 메소드를 사용해서 얻어와야한다
  - DB 인식 방법
    - DriverManager 가 External Libraries 에 있는 현재 다운되어진 라이브러리 중 디비 라이브러리를 찾아옵니다.
    - DriverManager.getConnection() 은 데이터베이스 드라이버를 찾아 해당 드라이버가 제공하는 커넥션을 반환
- DriverManager 이해
  - JDBC DriverManager란, JDBC 드라이버 세트를 관리하기 위한 기본 서비스
  - DriverManager 초기화는 느리게 수행되며 스레드 컨텍스트 클래스 로더를 사용하여 서비스 공급자를 찾는다.
  - getConnection 메소드가 호출되면 DriverManager 는 초기화 시 로드된 드라이버와 현재 애플리케이션과 동일한 클래스 로더를 사용하여 명시적으로 로드된 드라이버 중에서 적절한 드라이버를 찾으려고 시도
- getConnection 과정
  - 애플리케이션 로직에서 커넥션이 필요하면 DriverManager.getConnection() 을 호출
  - DriverManager 는 라이브러리에 등록된 드라이버 목록을 자동으로 인식
    - 해당 드라이버에게 순서대로 다음 정보를 넘겨서 커넥션을 획득할 수 있는지 확인
      - URL, 이름, 비밀번호 등 접속에 필요한 추가 정보
    - 여기서 각각의 드라이버는 URL 정보를 체크해서 본인이 처리할 수 있는 요청인지 확인
    - 예를 들어서 URL이 jdbc:h2 로 시작하면 이것은 h2 데이터베이스에 접근하기 위한 규칙
    - 따라서 H2 드라이버는 본인이 처리할 수 있으므로 실제 데이터베이스에 연결해서 커넥션을 획득하고 이 커넥션을 클라이언트에 반환
    - 반면에 URL이 jdbc:h2 로 시작했는데 MySQL 드라이버가 먼저 실행되면 이 경우 본인이 처리할 수 없다는 결과를 반환하게 되고, 다음 드라이버에게 순서가 넘어간다.
  - 이렇게 찾은 커넥션 구현체가 클라이언트에 반환

#### JDBC를 활용한 간단한 CRUD
- MemberRepositoryV0
  ```java
  public Member save(Member member) throws SQLException {
      String sql = "insert into member(member_id, money) values (?, ?)";

      Connection connection = null;
      PreparedStatement preparedStatement = null; // 이걸로 쿼리를 날림?

      try {
          connection = getConnection();
          preparedStatement = connection.prepareStatement(sql);

          //파라미터 바인딩
          preparedStatement.setString(1, member.getMemberId());
          preparedStatement.setInt(2, member.getMoney());
          preparedStatement.executeUpdate();
          return member;

      } catch (SQLException e) {
          log.error("db error", e);
          throw e;
      } finally {
          //db 연결을 끊어줘야함
          preparedStatement.close();
          //여기서 익셉션이 터지면 connection 이 안끊어짐 그래서 다 묵어야함

          connection.close();


          //요래 해야함
          close(connection, preparedStatement, null);
      }
  }

  public Member findById(String memberId) throws SQLException {
      String sql = "select * from member where member_id = ?";

      Connection connection = null;
      PreparedStatement preparedStatement = null;
      ResultSet resultSet = null;

      try {
          connection = getConnection();
          preparedStatement = connection.prepareStatement(sql);
          preparedStatement.setString(1, memberId);

          resultSet = preparedStatement.executeQuery();

          if (resultSet.next()) {
              Member member = new Member();
              member.setMemberId(resultSet.getString("member_id"));
              member.setMoney(resultSet.getInt("money"));
              return member;
          }else{
              throw new NoSuchElementException("member not found memberId : " + memberId);
          }
      } catch (SQLException e) {
          log.error("error : ", e);
          throw e;
      }finally {
          close(connection,preparedStatement,resultSet);
      }
  }

  public void update(String memberId, int money) throws SQLException {
      String sql = "update member set money=? where member_id=?";


      Connection connection = null;
      PreparedStatement preparedStatement = null;

      try {
          connection = getConnection();
          preparedStatement = connection.prepareStatement(sql);
          preparedStatement.setInt(1, money);
          preparedStatement.setString(2, memberId);

          int resultSize = preparedStatement.executeUpdate();
          log.info("resultSize = {}", resultSize);

      } catch (SQLException e) {
          log.error("error : ", e);
          throw e;
      }finally {
          close(connection,preparedStatement,null);
      }
  }

  public void delete(String memberId) throws SQLException {
      String sql = "delete from member where member_id=?";


      Connection connection = null;
      PreparedStatement preparedStatement = null;

      try {
          connection = getConnection();
          preparedStatement = connection.prepareStatement(sql);
          preparedStatement.setString(1, memberId);

          int resultSize = preparedStatement.executeUpdate();
          log.info("resultSize = {}", resultSize);

      } catch (SQLException e) {
          log.error("error : ", e);
          throw e;
      }finally {
          close(connection,preparedStatement,null);
      }
  }
  private void close(Connection con, Statement stmt, ResultSet resultSet) {

      //이렇게 해야 Exception 이 터져도 catch 로 잡기 때문에 각각의 close 가 다른 close 에 영향을 주지 않음
      if (resultSet != null) {
          try {
              resultSet.close();
          } catch (SQLException e) {
              log.error("error", e);
          }
      }
      if (stmt != null) {
          try {
              stmt.close();
          } catch (SQLException e) {
              log.error("error", e);
          }
      }
      if (con != null) {
          try {
              con.close();
          } catch (SQLException e) {
              log.error("error", e);
          }
      }
  }

  private static Connection getConnection() {
      return DBConnectionUtil.geConnection();
  }
  ```
  - 커넥션을 얻은 후 
  - 커넥션을 닫을 객체를 null 로 초기화하는 이유는, SQLException 은 Checked Exception 이기 때문에, try catch 처리 필요
  - PreparedStatement 객체를 이용해 쿼리를 디비에 전달
  - 모든 처리가 완료되었다면, 연결된 커넥션을 순차적으로 역순으로 close() 호출
  - ResultSet 은 디비에서 불러온 결과를 저장해서 cursor 를 이용해 값 조회
- TestCode
  ```java
  MemberRepositoryV0 repositoryV0 = new MemberRepositoryV0();
  @Test
  void crud() throws SQLException {

      //save
      Member memberV0 = new Member("memberV0", 100000);
      repositoryV0.save(memberV0);

      //findById
      Member findMember = repositoryV0.findById(memberV0.getMemberId());
      log.info("findMember = {}", findMember);

      //2개는 다른 인스턴스지만, isEqualTo 가  equals() 를 호출해 값만 비교 하기 때문에 true
      //@Data 롬복은, 모든 상태 값을 비교할수 있는 equals 와 고유한 hashCode 를 자동으로 만들어준다.
      assertThat(findMember).isEqualTo(memberV0);


      //update
      int updateMoney = 200000;
      repositoryV0.update(memberV0.getMemberId(), updateMoney);
      Member updateMember = repositoryV0.findById(memberV0.getMemberId());
      assertThat(updateMember.getMoney()).isEqualTo(updateMoney);


      //delete
      repositoryV0.delete(memberV0.getMemberId());
      assertThatThrownBy(() -> repositoryV0.findById(memberV0.getMemberId()))
          .isInstanceOf(NoSuchElementException.class);
  }
  ```
