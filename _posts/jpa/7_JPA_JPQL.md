---
layout: post
title: JPA. JPQL
summary: JPA - 자바 ORM 표준 JPA 프로그래밍 - 기본편
author: devhtak
date: '2021-05-06 21:41:00 +0900'
category: JPA
---

#### 객체지향 쿼리 언어

- JPQL
- JPA Criteria
- QueryDSL
- Native SQL
- JDBC 직접 사용
  - MyBatis, SpringJdbcTemplate

#### JPQL(Java Persistence Query Language) 소개

- JPQL
  - JPA를 사용하면 엔티티 객체를 중심으로 개발
  - 문제는 검색쿼리
    - 검색할 때에 테이블이 아닌 엔티티 객체를 대상으로 검색
    - 모든 DB 데이터를 객체로 변환하여 검색하는 것은 불가능하다.
    - 애플리케이션이 필요한 데이터만 DB에서 불러오려면 결국 검색 조건이 포함된 SQL이 필요하다.

  - JPA는 SQL을 추상화한 JPQL이라는 객체 지향 쿼리 언어를 제공한다.
  - SQL과 문법 유사
    - SELECT, FROM, WHERE, GROUP BY, HAVING, JOIN 지원
  - JPQL은 엔티티 객체를 대상으로 쿼리
  - SQL은 데이터베이스 테이블을 대상으로 쿼리
  ```java
  String jpql = "SELECT m FROM Member m WHERE m.name like '%hello%'";
  List<Member> result = em.createQuery(jpql, Member.class).getResultList();
  ```
  ```
  // 실행된 SQL
  select 
      m.id as id, 
      m.age as age, 
      m.USERNAME as USERNAME, 
      m.TEAM_ID as TEAM_ID 
  from 
      Member m 
  where 
      m.age>18
  ```
  
  - 테이블이 아닌 객체를 대상으로 검색하는 객체 지향 쿼리
  - SQL을 추상화해서 특정 데이터베이스 SQL에 의존하지 않는다.
  - JPQL을 한마디로 정의하면 객체 지향 SQL

#### JPQL 문법

- JPQL 문법
  ```java
  select m from Member as m where m.age > 18
  ```
  - 엔티티와 속성은 대소문자 구분O (Member, age) 
  - JPQL 키워드는 대소문자 구분X (SELECT, FROM, where) 
  - 엔티티 이름 사용, 테이블 이름이 아님(Member) 
  - 별칭은 필수(m) (as는 생략가능)

- TypeQuery, Query 객체
  - TypeQuery: 반환 타입이 명확할 때 사용
    ```java
    TypedQuery<Member> query =  em.createQuery("SELECT m FROM Member m", Member.class); 
    ```
  - Query: 반환 타입이 명확하지 않을 때 사용
    ```java
    Query query =  em.createQuery("SELECT m.username, m.age from Member m");
    ```

- 결과 조회 API
  - query.getResultList(): 결과가 하나 이상일 때 리스트 반환
    - 결과가 없으면 빈 리스트 반환
  - query.getSingleResult(): 결과가 정학히 하나일 때 단일 객체 반환
    - 결과가 없으면 NoResultException 발생
    - 결과가 둘 이상이면 NonUniqueResultException 발생

- 파라미터 바인딩 - 이름 기준, 위치 기준
  - 이름 기준
    ```java
    SELECT m FROM Member m where m.username=:username 
    query.setParameter("username", usernameParam);
    ```
  - 위치 기준
    ```java
    SELECT m FROM Member m where m.username=?1 
    query.setParameter(1, usernameParam);
    ```
  - 이름 기준으로 사용하자
    
- 집합과 정렬
  - GROUP BY, HAVING, ORDER BY 키워드를 사용할 수 있다.
    ```java
    select
       COUNT(m), //회원수
       SUM(m.age), //나이 합
       AVG(m.age), //평균 나이
       MAX(m.age), //최대 나이
       MIN(m.age) //최소 나이
    from Member m
    ```
    
#### 프로젝션

- SELECT 절에 조회할 대상을 지정하는 것
  - 프로젝션 대상: 엔티티, 임베디드 타입, 스칼라 타입(숫자, 문자등 기본 데이터 타입)
    ```java
    SELECT m FROM Member m // 엔티티 프로젝션
    SELECT m.team FROM Member m // 엔티티 프로젝션
    SELECT m.address FROM Member m // 임베디드 타입 프로젝션
    SELECT m.username, m.age FROM Member m // 스칼라 타입 프로젝션
    ```
    
  - DISTINCT로 중복 제거

- 여러 값 조회
  ```java
  SELECT m.username, m.age FROM Member m 
  ```
  - 첫번째 방법: Query 타입으로 조회
  - 두번째 방법: Object[] 타입으로 조회
  - 세번째 방법: new 명령어로 조회
    - 단순 값을 DTO로 바로 조회
      ```java
      SELECT new jpabook.jpql.UserDTO(m.username, m.age) FROM Member m 
      ```
    - 패키지 명을 포함한 전체 클래스 명 입력
    - 순서와 타입이 일치하는 생성자 필요

#### 페이징

- JPA는 페이징을 다음 두 API로 추상화하였다.
  - setFirstResult(int startPosition): 조회 시작 위치(0부터 시작)
  - setMaxResults(int maxResult): 조회할 데이터 수

- 페이징 API 예시
  ```java
  TypedQuery<Member> query1 = em.createQuery(
      "SELECT m FROM Member as m ORDER BY m.age desc", Member.class
  )
      .setFirstResult(0)
      .setMaxResults(2);
  List<Member> resultList = query1.getResultList();
  ```
  - age가 높은 순으로 0번째 부터 2개 조회한다.

- SQL 벤더별로 페이징 처리하는 것이 다르지만 해당 DB에 맞게 쿼리를 생성한다.

#### 조인

- 조인 종류
  - 내부 조인
    ```
    SELECT m FROM Member M [INNER] JOIN m.team t
    ```
    - m.team으로 조인하였기 때문에 기본 Foreign key로 조인하게 된다.
  - 외부 조인
    ```
    SELECT m FROM Member m LEFT [OUTER] JOIN m.team t
    ```

  - 세타 조인
    ```
    SELECT count(m) from Member m, Team t where m.username = t.name
    ```
  
- 조인 - ON절
  - ON 절을 활용한 조인(JPA 2.1부터 지원)
    - 조인 대상 필터링
      ```
      회원과 팀을 조인하면서, 팀 이름이 A인 팀만 조인
      JPQL:
      SELECT m, t FROM Member m LEFT JOIN m.team t on t.name = 'A' 
      SQL:
      SELECT m.*, t.* FROM 
      Member m LEFT JOIN Team t ON m.TEAM_ID=t.id and t.name='A'
      ```
    - 연관관계 없는 엔티티 외부 조인  
      ```
      회원의 이름과 팀의 이름이 같은 대상 외부 조인
      JPQL:
      SELECT m, t FROM
      Member m LEFT JOIN Team t on m.username = t.name
      SQL:
      SELECT m.*, t.* FROM 
      Member m LEFT JOIN Team t ON m.username = t.name
      ```
      
#### 서브 쿼리

- 서브 쿼리 지원 함수
  - \[NOT] EXISTS (subquery): 서브쿼리에 결과가 존재하면 참
  - {ALL | ANY | SOME} (subquery) 
  - ALL 모두 만족하면 참
  - ANY, SOME: 같은 의미, 조건을 하나라도 만족하면 참
  - \[NOT] IN (subquery): 서브쿼리의 결과 중 하나라도 같은 것이 있으면 참

- 예시
  - 나이가 평균보다 많은 회원
    ```
    select m from Member m
    where m.age > (select avg(m2.age) from Member m2) 
    ```
  - 한 건이라도 주문한 고객
    ```
    select m from Member m
    where (select count(o) from Order o where m = o.member) > 0 
    ```
  - 팀A 소속인 회원
    ```
    select m from Member m
    where exists (select t from m.team t where t.name = ‘팀A') 
    ```
  - 전체 상품 각각의 재고보다 주문량이 많은 주문들
    ```
    select o from Order o 
    where o.orderAmount > ALL (select p.stockAmount from Product p) 
    ```
  - 어떤 팀이든 팀에 소속된 회원
    ```
    select m from Member m 
    where m.team = ANY (select t from Team t)
    ```
    
- JPA 서브 쿼리 한계
  - JPA는 WHERE, HAVING 절에서만 서브 쿼리 사용 가능
  - SELECT 절도 가능(하이버네이트에서 지원) 
  - *** FROM 절의 서브 쿼리는 현재 JPQL에서 불가능
    - 조인으로 풀 수 있으면 풀어서 해결

#### JPQL 타입 표현

- 다양한 타입 표현
  - 문자: 'HELLO', 'She' 's' 
    ```
    SELECT m.username || ' hello' FROM Member m
    ```
  - 숫자: 10L(Long), 10D(Double), 10F(Float)
    ```
    SELECT age + 10 FROM Member m
    ```
  - Boolean: TRUE, FALSE 
  - ENUM: jpabook.MemberType.Admin (패키지명 포함)
    ```
    SELECT m FROM Member m WHERE m.type = jpabook.MemberType.Admin
    
    SELECT m FROM Member m WHERE m.type= :type"
    ```
    - 하드코딩 보다는 setParameter로 type을 지정하는 것이 편리하다.
  -  엔티티 타입: TYPE(m) = Member (상속 관계에서 사용)JPQL 기타
    ```
    SELECT i FROM Item i WHERE TYPE(i) = Book
    ```
- JPQL 기타
  - SQL과 문법이 같은 식
  - EXISTS, IN 
  - AND, OR, NOT 
  - =, >, >=, <, <=, <> 
  - BETWEEN, LIKE, IS NULL
  
- 조건식 - CASE 식
  - 기본 case 식
    ```
    select
        case when m.age <= 10 then '학생요금'
            when m.age >= 60 then '경로요금'
            else '일반요금'
        end
    from Member m
    ```
  - 단순 case 식
    ```
    select
        case t.name 
            when '팀A' then '인센티브110%'
            when '팀B' then '인센티브120%'
            else '인센티브105%'
        end
    from Team t
    ```

- COALESCE: 하나씩 조회해서 null이 아니면 반환
  ```
  사용자 이름이 없으면 이름 없는 회원을 반환JPQL 기본 함수
  select coalesce(m.username,'이름 없는 회원') from Member m
  ```
- NULLIF: 두 값이 같으면 null 반환, 다르면 첫번째 값 반환
  ```
  사용자 이름이 ‘관리자’면 null을 반환하고 나머지는 본인의 이름을 반환
  select NULLIF(m.username, '관리자') from Member m
  ```
  
#### JPQL 기본 함수

- CONCAT: || 또는 concat() 을 사용하여 문자열 붙이기 가능
- SUBSTRING : 문자열 자르기
- TRIM: 문자열 앞, 뒤 공백 삭제
- LOWER, UPPER: 소, 대문자 변환
- LENGTH: 문자열 길이
- LOCATE: 문자열 위치 반환 (1부터 시작)
- ABS, SQRT, MOD: 수학 메서드

- 사용자 정의 함수 호출
  - 하이버네이트는 사용전 방언에 추가해야 한다.
  - Oracle의 Procedure, Function 등을 지원하기 위해 사용
  - 사용하는 DB 방언을 상속받고, 사용자 정의 함수를 등록한다.
    ```
    select function('group_concat', i.name) from Item i
    ```

#### 경로 표현식

- 점(.)을 찍어 객체 그래프를 탐색하는 것
  ```
  SELECT m.username // -> 상태 필드
  FROM Member m
      join m.team t // -> 단일 값 연관 필드
      join m.orders o // -> 컬렉션 값 연관 필드
  WHERE t.name='팀A'
  ```
  
- 경로 표현식 용어 정리
  - 상태 필드(state field): 단순히 값을 저장하기 위한 필드(ex m.username)
  - 연관 필드(association field): 연관관계를 위한 필드
    - 단일 값 연관 필드: @ManyToOne, @OneToOne, 대상이 엔티티(ex. m.team)
    - 컬렉션 값 연관 필드: @OneToMany, @ManyToMany, 대상이 컬렉션(ex. m.orders)

- 경로 표현식 특징
  - 상태 필드(state field): 경로 탐색의 끝, 탐색 X
  - 단일 값 연관 경로: 묵시적 내부 조인 발생, 탐색 O
    ```
    // JPQL
    SELECT m.team.name FROM Member m
    // QUERY
    SELECT team.id, team.name FROM Member INNER JOINT Team ON Member.team_id=Team.id
    ```
    - m.team에 경우 추가로 계속 탐색이 가능하다.    
  - 컬렉션 값 연관 경로: 묵시적 내부 조인 발색, 탐색 X
    - FROM 절에서 명시적 조인을 통해 별칭을 얻으면 별칭을 통해 탐색 가능
      ```
      // JPQL
      SELECT t.members FROM Team t
      // QUERY
      SELECT member.id, member.username FROM Team team INNER JOIN Member ON team.id=Member.team_id
      ```
      - 컬렉션이기 때문에 인덱스 접근 등이 안된다. 즉, 탐색이 되지 않는다.
      - size와 같은 컬렉션 함수를 사용할 수 있다.
        ```
        // JPQL
        SELECT t.members.size FROM Team t
        // QUERY
        SELECT (SELECT COUNT(m.id) FROM Member m WHERE t.id=m.team_id) FROM Team t
        ```
  - 묵시적 내부 조인이 발생하지 않도록 조심해야 한다. 조인이 발생하는 상황 파악이 어렵고 조인은 SQL 튜닝에 중요 포인트가 된다.
    - 항상 내부 조인이 발생하며, 컬렉션은 경로 탐색의 끝이된다. 명시적 조인을 통해 별칭을 얻어야 추가 탐색이 가능하다.
    - 경로 탐색은 주로 SELECT, WHERE 절에서 사용하지만 묵시적 조인으로 인해 SQL의 FROM(JOIN) 절에 영향을 준다.
    - 가급적 묵시적 조인 대신 명시적 조인을 사용하자

- 명시적 조인과 묵시적 조인
  - 명시적 조인: join 키워드 직접 사용
    - SELECT m FROM Member join m.team t
  - 묵시적 조인: 경로 표현식에 의해 묵시적으로 SQL 조인 발생 (내부 조인만 가능)
    - SELECT m.team FROM Member m

#### Fetch JOIN

- Fetch JOIN
  - SQL 조인 종류가 아니다.
  - JPQL에서 성능 최적화를 위해 제공하는 기능
  - 연관된 엔티티나 컬렉션을 SQL 한 번에 함께 조회하는 기능
  - join fetch 명령어 사용
  - Fetch join ::= \[LEFT \[OUTER] | INNER] JOIN FETCH 조인 경로

- 엔티티 페치 조인
  - SQL 한번에 회원을 조회하면서 연관된 팀도 함께 조회
  - SQL을 보면 회원 뿐 아니라 팀도 함께 SELECT
  ```
  // JPQL
  SELECT m FROM Member m JOIN FETCH m.team
  // SQL
  SELECT m.*, t.* FROM Member m JOIN Team t ON m.team_id = t.id
  ```

- 페치 조인 예지
  - fetch 조인을 사용하지 않은 경우
    ```java
    String sql = "SELECT a FROM Account a";
    List<Account> accounts = em.createQuery(sql, Account.class)
        .getResultList();

    for(Account account: accounts) {
        System.out.println(account.getName() +" " + account.getTeam().getName());
    }
    // AccountA, 팀A(SQL)
    // AccountB, 팀A(1차 캐시)
    // AccountC, 팀B(SQL)
    ```
    - 쿼리
      ```
      select
        account0_.id as id1_0_,
        account0_.age as age2_0_,
        account0_.name as name3_0_,
        account0_.team_id as team_id4_0_ 
      from
        account account0_
      ```
      - Account만 가져온다. (FetchType이 LAZY이다)
      - Account에 Team을 사용하기 위해서는 다시 가져와야 하며 1차 캐시에 있는 경우 1차캐시를 사용한다.
      
  - fetch 조인을 사용한 경우
    ```java
    tring sql = "SELECT a FROM Account a JOIN FETCH a.team";
    List<Account> accounts = em.createQuery(sql, Account.class)
        .getResultList();

    for(Account account: accounts) {
      Team team = account.getTeam();
      System.out.println(account.getName() +" " + team.getName());
    }
    ```
    - 쿼리
      ```
      select
        account0_.id as id1_0_0_,
        team1_.id as id1_14_1_,
        account0_.age as age2_0_0_,
        account0_.name as name3_0_0_,
        account0_.team_id as team_id4_0_0_,
        team1_.name as name2_14_1_ 
      from
        account account0_ 
      inner join
        team team1_ 
            on account0_.team_id=team1_.id
      ```
      - Account와 연관된 Team까지 다 가져온다.
 
- 컬렉션 페치 조인
  - 컬렉션 패치 조인은 일대다 관계에 컬렉션을 페치 조인할 때 사용된다.
    ```
    // JPQL
    SELECT t
    FROM Team t JOIN FETCH t.members
    WHERE t.name='TeamA'
    // SQL
    SELECT t.*, m.*
    FROM Team t
    INNER JOIN Member m ON t.id = m.team_id
    WHERE t.name='TeamA'
    ```
    
- JPQL의 Distinct
  - SQL의 DISTINCT는 중복된 결과를 제거하는 명령
    - SQL의 distinct 키워드는 모든 column이 동일해야 가능하다.
  - JPQL의 DISTINCT는 2가지 기능을 제공한다.
    - SQL의 DISTINCT를 추가
    - 애플리케이션에서 엔티티 중복 제거
  - 예시
    - 하나의 Team에 2명의 Account가 있다.
    - Team과 Member를 조회하는 경우
      ```java
      String sql = "SELECT t FROM Team t JOIN FETCH t.accounts";
      List<Team> teams = em.createQuery(sql, Team.class)
          .getResultList();

      for(Team team: teams) {
          System.out.print(team.getName()+"'s members: ");
          for(Account account: team.getAccounts()) {
              System.out.print(account.getName() + " ");
          }
          System.out.println();
      }
      ```
      ```
      //출력
      TeamA's members: AccountA AccountB 
      TeamA's members: AccountA AccountB 
      TeamB's members: AccountC
      ```
      - AccountA와 AccountB가 TeamA에 있기 때문에 중복되어 나온다.
        - SQL에서는 Team1 | Account1, Team1 | Account2 2개의 row로 나오지만,
        - JPA에서 객체로 관리하는 경우 List를 조회할 수 있기 때문에 같은 결과로 보이게 된다.
      - 이를 해결하기 위해 JPQL에서 DISTINCT는 엔티티 중복 제거 기능을 추가했다
    - DISTINCT 추가
      ```java
      String sql = "SELECT distinct t FROM Team t JOIN FETCH t.accounts";
      List<Team> teams = em.createQuery(sql, Team.class)
          .getResultList();

      for(Team team: teams) {
          System.out.print(team.getName()+"'s members: ");
          for(Account account: team.getAccounts()) {
              System.out.print(account.getName() + " ");
          }
          System.out.println();
      }
      ```
      ```
      // 출력
      TeamA's members: AccountA AccountB 
      TeamB's members: AccountC 
      ```
      - 중복되는 TeamA가 사라졌다.

- 페치 조인과 일반 조인의 차이
  - 일반 조인 실행 시 연관된 엔티티를 함께 조회하지 않음
  - 일반 조인 예시
    - Team과 Account를 조인했지만, Team의 컬럼만 가져온다.
    - JPQL
      ```
      SELECT t
      FROM Team t JOIN t.Account a
      WHERE t.name='TeamA'
      ```
    - SQL
      ```
      SELECT T.*
      FROM TEAM T INNER JOIN ACCOUNT A ON T.ID=A.TEAM_ID
      WHERE T.NAME='TeamA'
      ```
  - JPQL은 결과를 반환할 때 연관관계 고려하지 않는다.
  - 단지 SELECT 절에 지정한 엔티티만 조회할 뿐이다.
  - 여기서는 팀 엔티티만 조회하고, 회원 엔티티는 조회하지 않는다.
  - 페치 조인 예시
    - Team만 조회하였지만 연관된 Account도 조회된다.
    - JPQL
      ```
      SELECT t
      FROM Team t JOIN FETCH t.Account a
      WHERE t.name='TeamA'
      ```
    - SQL
      ```
      SELECT T.*, A.*
      FROM TEAM T INNER JOIN ACCOUNT A ON T.ID=A.TEAM_ID
      WHERE T.NAME='TeamA'
      ```
  - 페치 조인을 사용할 때만 연관된 엔티티도 함께 즉시 로딩으로 조회
  - 페치 조인은 객체 그래프를 SQL 한번에 조회하는 개념

- 페치 조인의 한계
  - 페치 조인 대상에는 별칭을 줄 수 없다. 
    - 하이버네이트는 가능, 가급적 사용하지 않는 것이 좋다.
    - F 
  - 둘 이상의 컬렉션은 페치 조인 할 수 없다. 
  - 컬렉션을 페치 조인하면 페이징 API(setFirstResult, setMaxResults)를 사용할 수 없다. 
    - 일대일, 다대일 같은 단일 값 연관 필드들은 페치 조인해도 페이징 가능
    - 하이버네이트는 경고 로그를 남기고 메모리에서 페이징(매우 위험)
      - 대량의 데이터에 경우 메모리에 모든 데이터가 남고, 거기에서 페이징 처리를 하기 때문에 문제가 발생할 수 있다.
    - @BatchSize 로 해결
      - size를 지정하여 설정한 size만큼 데이터를 가져온다.
      - 많은 양의 전체 데이터를 가져오지 않기 때문에 페이징 처리가 가능해진다.

- 페치 조인의 특징
  - 연관된 엔티티들을 SQL 한 번으로 조회 - 성능 최적화
  - 엔티티에 직접 적용하는 글로벌 로딩 전략보다 우선함
    - @OneToMany(fetch = FetchType.LAZY) //글로벌 로딩 전략
  - 실무에서 글로벌 로딩 전략은 모두 지연 로딩
  - 최적화가 필요한 곳은 페치 조인 적용

#### 다형성 쿼리

- TYPE 키워드 예시
  - Item을 상속받는 Album, Movie, Book이 있다.
  - Item 중에 Book, Movie를 조회
    ```
    // JPQL
    SELECT i FROM Item i
    WHERE TYPE(i) IN (Book, Movie)
    // SQL
    SELECT i.* FROM Item i
    WHERE i.DTYPE in ('B', 'M')
    ```

- TREAT 키워드
  - 자바의 타입 캐스팅과 유사
  - 상속 구조에서 부모 타입을 특정 자식 타입으로 다룰 때 사용
  - FROM, WHERE, SELECT 사용
  - 예시
    - 부모인 Item과 자식 Book이 있다.
    - JPQL
      ```
      // JPQL
      SELECT i FROM Item i
      WHERE treat(i as Book).author = 'kim'
      ```
    - 단일 테이블(SINGLE_TABLE) 전략
      ```
      // SQL 
      SELECT i.* FROM Item i
      WHERE i.DTYPE='B' AND i.author='kim'      
      ```
    - JOIN 테이블 전략
      ```
      //SQL
      SELECT product0_.*,
          product0_1.*,
          product0_2.*,
          product0_3.*
      from
          product product0_ 
      left outer join
          album product0_1_ 
              on product0_.id=product0_1_.id 
      inner join
          book product0_2_ 
              on product0_.id=product0_2_.id 
      left outer join
          movie product0_3_ 
              on product0_.id=product0_3_.id 
      where
          product0_2_.author='Kim'
      ```
    - 클래스 별 테이블(TABLE_PER_CLASS) 전략
      ```
      select
          product0_.*
          from
              ( select
                  id,
                  created_by,
                  created_date,
                  modified_by,
                  modified_date,
                  name,
                  price,
                  null as artist,
                  null as author,
                  null as isbn,
                  null as actor,
                  null as director,
                  0 as clazz_ 
              from
                  product 
              union
              all select
                  id,
                  created_by,
                  created_date,
                  modified_by,
                  modified_date,
                  name,
                  price,
                  artist,
                  null as author,
                  null as isbn,
                  null as actor,
                  null as director,
                  1 as clazz_ 
              from
                  album 
              union
              all select
                  id,
                  created_by,
                  created_date,
                  modified_by,
                  modified_date,
                  name,
                  price,
                  null as artist,
                  author,
                  isbn,
                  null as actor,
                  null as director,
                  2 as clazz_ 
              from
                  book 
              union
              all select
                  id,
                  created_by,
                  created_date,
                  modified_by,
                  modified_date,
                  name,
                  price,
                  null as artist,
                  null as author,
                  null as isbn,
                  actor,
                  director,
                  3 as clazz_ 
              from
                  movie 
          ) product0_ 
      where
          product0_.author='Kim'
      ```

#### 출처

김영한님의 자바 ORM 표준 JPA 프로그래밍 - 기본편 강의
