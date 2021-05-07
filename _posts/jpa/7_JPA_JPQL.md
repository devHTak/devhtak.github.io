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

#### 출처

김영한님의 자바 ORM 표준 JPA 프로그래밍 - 기본편 강의
