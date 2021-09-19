---
layout: post
title: Querydsl 기본 문법
summary: Querydsl
author: devhtak
date: '2021-09-19 21:41:00 +0900'
category: JPA
---

#### JPQL VS Querydsl

```java
@Test
public void startJPQL() {
    //member1을 찾아라.
    String qlString = "select m from Member m where m.username = :username";
    Member findMember = em.createQuery(qlString, Member.class)
        .setParameter("username", "member1")
        .getSingleResult();
    assertThat(findMember.getUsername()).isEqualTo("member1");
}

@Test
public void startQuerydsl() {
    //member1을 찾아라.
    JPAQueryFactory queryFactory = new JPAQueryFactory(em);
    QMember m = new QMember("m");
    Member findMember = queryFactory
        .select(m)
        .from(m)
        .where(m.username.eq("member1"))//파라미터 바인딩 처리
        .fetchOne();
    assertThat(findMember.getUsername()).isEqualTo("member1");
}
```

- JPA의 EntityManager 로 JPAQueryFactory 생성
  - Querydsl은 JPQL 빌더
  - JPAQueryFactory 의 동시성 문제
    - JPAQueryFactory를 생성할 때 제공하는 EntityManager에 달려있다.
    - EntityManager는 트랜잭션마다 별도의 영속성 컨텍스트를 제공하기 때문에 동시성 문제는 걱정하지 않아도 된다.
- JPQL: 문자(실행 시점 오류), Querydsl: 코드(컴파일 시점 오류)
- JPQL: 파라미터 바인딩 직접, Querydsl: 파라미터 바인딩 자동 처리

#### Q-Type

- 생성 방법
  ```java
  QMember qMember = new QMember("m"); //별칭 직접 지정
  QMember qMember = QMember.member; //기본 인스턴스 사용
  ```
  - QMember.member를 static import로 하면 편리하게 사용할 수 있다.
  - 해당 설정을 하면 JPQL을 볼 수 있다.
    ```
    spring.jpa.properties.hibernate.use_sql_comments: true
    ```

#### 검색 조건

- 기본 검색 쿼리
  ```java
  @Test
  public void search() {
   Member findMember = queryFactory
       .selectFrom(member)
       .where(member.username.eq("member1")
       .and(member.age.eq(10)))
       .fetchOne();
   assertThat(findMember.getUsername()).isEqualTo("member1");
  }
  ``` 
  - 참고, select(), from()은 selectFrom()으로 합칠 수 있다.

- JPQL 이 제공하는 모든 검색 조건 제공
  ```java
  member.username.eq("member1"); // username = 'member1'
  member.username.ne("member1"); // username != 'member1'
  member.username.eq("member1").not(); // username != 'member1'
  
  member.username.isNotNull(); // member is not null
  member.username.isNull(); // member is null
  
  member.age.in(10, 20); // age in (10, 20)
  member.age.notIn(10, 20); // age not in (10, 20)
  member.age.between(10, 30); // age between 10 and 30
  
  member.age.goe(30); // age >= 30
  member.age.gt(30); // age > 30
  member.age.loe(30); // age <= 30
  member.age.lt(30); // age < 30
  
  member.username.like("member%"); // member like 'member%'
  member.username.contains("member"); // member like '%member%'
  member.username.startsWith("member"); // member like 'member%'
  // ...
  ``` 
- AND 조건을 파라미터로 처리
  ```java
  @Test
  public void searchAndParam() {
      List<Member> result1 = queryFactory
          .selectFrom(member)
          .where(member.username.eq("member1"),
              member.age.eq(10))
          .fetch();
      
    List<Member> result2 = queryFactory
          .selectFrom(member)
          .where(member.username.eq("member1").and(member.age.eq(10)))
          .fetch();    
  }
  ```
  - where() 에 파라미터로 검색조건을 추가하면 AND 조건이 추가됨

#### 결과 조회

- fetch() : 리스트 조회, 데이터 없으면 빈 리스트 반환
- fetchOne() : 단 건 조회
  - 결과가 없으면 : null
  - 결과가 둘 이상이면 : com.querydsl.core.NonUniqueResultException 발생
- fetchFirst() : limit(1).fetchOne()
- fetchResults() : 페이징 정보 포함, total count 쿼리 추가 실행
- fetchCount() : count 쿼리로 변경해서 count 수 조회

```java
//List
List<Member> fetch = queryFactory
    .selectFrom(member)
    .fetch();
//단 건
Member findMember1 = queryFactory
    .selectFrom(member)
    .fetchOne();
//처음 한 건 조회
Member findMember2 = queryFactory
    .electFrom(member)
    .fetchFirst();
//페이징에서 사용
QueryResults<Member> queryResults = queryFactory
    .selectFrom(member)
    .fetchResults();
List<Hello> results = queryResults.getResults(); // paging 처리 등에서 사용되는 값들을 가져올 수 있다.
queryResults.getLimit();
queryResults.getOffset();

//count 쿼리로 변경
long count = queryFactory
    .selectFrom(member)
    .fetchCount();
```

#### 정렬

```java
/**
 * 회원 정렬 순서
 * 1. 회원 나이 내림차순(desc)
 * 2. 회원 이름 올림차순(asc)
 * 단 2에서 회원 이름이 없으면 마지막에 출력(nulls last)
 */
@Test
public void sort() {
    em.persist(new Member(null, 100));
    em.persist(new Member("member5", 100));
    em.persist(new Member("member6", 100));
  
    List<Member> result = queryFactory
        .selectFrom(member)
        .where(member.age.eq(100))
        .orderBy(member.age.desc(), member.username.asc().nullsLast())
        .fetch();
    Member member5 = result.get(0);
    Member member6 = result.get(1);
    Member memberNull = result.get(2);

    assertThat(member5.getUsername()).isEqualTo("member5");
    assertThat(member6.getUsername()).isEqualTo("member6");
    assertThat(memberNull.getUsername()).isNull();
}
```
- desc(), asc() : 일반 정렬
- nullLast(), nullFirst(): null 데이터의 순서 부여


#### 페이징

- 조회 건수 제한
  ```java
  @Test
  public void paging1() {
       List<Member> result = queryFactory
           .selectFrom(member)
           .orderBy(member.username.desc())
           .offset(1) //0부터 시작(zero index)
           .limit(2) //최대 2건 조회
           .fetch();
       assertThat(result.size()).isEqualTo(2);
  }
  ```
  
- 전체 조회 수
  ```java
  @Test
  public void paging2() {
      QueryResults<Member> queryResults = queryFactory
          .selectFrom(member)
          .orderBy(member.username.desc())
          .offset(1)
          .limit(2)
          .fetchResults();
      assertThat(queryResults.getTotal()).isEqualTo(4);
      assertThat(queryResults.getLimit()).isEqualTo(2);
      assertThat(queryResults.getOffset()).isEqualTo(1);
      assertThat(queryResults.getResults().size()).isEqualTo(2);
  }
  ```
  - 주의: count 쿼리가 실행되니 성능상 주의!
    - 실무에서 페이징 쿼리를 작성할 때, 데이터를 조회하는 쿼리는 여러 테이블을 조인해야 한다.
    - 하지만 count 쿼리는 조인이 필요 없는 경우도 있다. 그런데 이렇게 자동화된 count 쿼리는 원본 쿼리와 같이 모두 조인을 해버리기 때문에 성능이 안나올 수 있다. 
    - count 쿼리에 조인이 필요없는 성능 최적화가 필요하다면, count 전용 쿼리를 별도로 작성해야 한다.

#### 집합

- 집합 함수

  ```java
  /**
   * JPQL
   * select
   * COUNT(m), //회원수
   * SUM(m.age), //나이 합
   * AVG(m.age), //평균 나이
   * MAX(m.age), //최대 나이
   * MIN(m.age) //최소 나이
   * from Member m
   */
  @Test
  public void aggregation() throws Exception {
      List<Tuple> result = queryFactory
          .select(member.count(),
              member.age.sum(),
              member.age.avg(),
              member.age.max(),
              member.age.min())
          .from(member)
          .fetch();
      
      Tuple tuple = result.get(0);
      assertThat(tuple.get(member.count())).isEqualTo(4);
      assertThat(tuple.get(member.age.sum())).isEqualTo(100);
      assertThat(tuple.get(member.age.avg())).isEqualTo(25);
      assertThat(tuple.get(member.age.max())).isEqualTo(40);
      assertThat(tuple.get(member.age.min())).isEqualTo(10);
  }
  ```
  - JPQL이 제공하는 모든 집합 함수를 제공한다.
  - tuple은 프로젝션과 결과반환에서 설명한다.

- GroupBy 사용
  ```java
  /**
   * 팀의 이름과 각 팀의 평균 연령을 구해라.
   */
  @Test
  public void group() throws Exception {
      List<Tuple> result = queryFactory
          .select(team.name, member.age.avg())
          .from(member)
          .join(member.team, team)
          .groupBy(team.name)
          .fetch();
    
      Tuple teamA = result.get(0);
      Tuple teamB = result.get(1);
      assertThat(teamA.get(team.name)).isEqualTo("teamA");
      assertThat(teamA.get(member.age.avg())).isEqualTo(15);
      assertThat(teamB.get(team.name)).isEqualTo("teamB");
      assertThat(teamB.get(member.age.avg())).isEqualTo(35);
  }
  ```

#### 조인

#### 서브쿼리

#### case 문

#### 상수, 문자 더하기

####  출처

- 실전! Querydsl \[인프런 김영한님 강의]
