---
layout: post
title: Querydsl 프로젝션, 동적 쿼리, 벌크 연산, Function 
summary: Querydsl
author: devhtak
date: '2021-09-21 21:41:00 +0900'
category: JPA
---

#### 프로젝션

- 프로젝션: select 대상 지정
- 프로젝션 대상이 하나인 경우
  ```java
  List<String> result = queryFactory
      .select(member.username)
      .from(member)
      .fetch();
  ```
  - 프로젝션 대상이 하나면 타입을 명확하게 지정할 수 있다.
  - 프로젝션 대상이 둘 이상이면 튜플이나 DTO로 조회

- 튜플 조회
  - 프로젝트 대상이 둘 이상일 때 사용
  - com.querydsl.core.Tuple
    - Tuple이 Repository단을 넘어 Service단에서 사용하는 것은 좋은 설계가 아니다.
    - DTO를 생성하여 사용해주는 것이 좋은 설계
  ```java
  List<Tuple> result = queryFactory
      .select(member.username, member.age)
      .from(member)
      .fech();
  ```

- JPQL을 활용한 DTO 조회
  - MemberDto.java
    ```java
    @Getter @Setter
    @NoArgsConstructor @AllArgsConstructor
    public class MemberDto {
        private String username;
        private int age;
    }
    ```
    
  - DTO 조회 코드
    ```java
    List<MemberDto> result = em.createQuery(
            "select new study.querydsl.dto.MemberDto(m.username, m.age) from Member m"
            , MemberDto.class)
        .getResult();
    ```
    - new 명령어를 사용하여 생성자 방식으로 가져와야 한다
    - Full Package 경로를 사용하여야 한다.

- Querydsl에서 DTO 사용
  - com.querydsl.core.types.Projection
  - 지원 방법
    - 프로퍼티 접근
      ```java
      List<MemberDto> result = queryFactory
          .select(Projections.bean(MemberDto.class,
              member.username, member.age))
          .from(member)
          .fetch();
      ```

    - 필드 직접 접근
      ```java
      List<MemberDto> result = queryFactory
          .select(Projections.fields(MemberDto.class,
              member.username, member.age))
          .from(member)
          .fetch();
      ```

    - 생성자 사용
      ```java
      List<MemberDto> result = queryFactory
          .select(Projections.constructor(MemberDto.class,
              member.username, member.age))
          .from(member)
          .fetch();
      ```
  - 별칭이 다를 때
    ```java
    @Data
    public class UserDto {
        private String name;
        private int age;
    }

    List<UserDto> result = queryFactory
        .select( Projections.fields(UserDto.class
            , member.username.as("name")
            , ExpressionUtils.as(
                JPAExpressions
                    .select(memberSub.age.max())
                    .from(memberSub), "age")))
        .from(member)
        .fetch();
    ```
    - 프로퍼티나, 필드 접근 생성 방식에서 이름이 다를 때 해결 방안
    - ExpressionUtils.as(source, alias): 필드나 서브 쿼리에 별칭 적용
    - username.as("memberName"): 필드에 별칭 적용

- 프로젝션 결과 반환 - @QueryProjection
  - 예시
    - 생성자 + @QueryProjection
      ```java
      @Data
      public class MemberDto {
          private String username;
          private int age;
          
          @QueryProjection
          public MemberDto(String username, int age) {
              this.username = username; this.age = age;
          }
      }
      
      List<MemberDto> result = queryFactory
          .select(QMemberDto(member.username, member.age))
          .from(member)
          .fetch();
      ```
      - 컴파일러로 타입을 체크할 수 있는 가장 안전한 방법
      - DTO에 Q파일을 생성해야 하는 단점이 있다.
      - MemberDto에서 Querydsl에 대한 의존성을 갖게 된다.

- distinct
  ```java
  List<String> result = queryFactory
      .select(member.username).distinct()
      .from(member)
      .fetch();
  ```
  ```
  select distinct m.username
  from member m
  ```
  - distinct가 적용된다

#### 동적 쿼리

- 동적 쿼리 사용하는 방법
  - BooleanBuilder 사용
  - Where 다중 파라미터 사용

- BooleanBuilder 사용
  ```java
  BooleanBuilder builder = new BooleanBuilder();
  if(usernameCond != null) {
      builder.and(member.username.eq(usernameCond));
  }
  if(ageCond != null) {
      builder.and(member.age.eq(ageCond));
  }
  
  List<Member> result = queryFactory
      .selectFrom(member)
      .where(builder)
      .fetch();
  ```
- Where 다중 파라미터 사용 - BooleanExpression
  ```java
  List<Member> result = queryFactory
      .selectFrom(member)
      .where(usernameEq(usernameCond), ageEq(ageCond))
      .fetch();
  ```
  ```java
  private BooleanExpression usernameEq(String usernameCond) {
      return usernameCond == null ? null : member.username.eq(usernameCond);
  }
  private BooleanExpression ageEq(Integer ageCond) {
      return ageCond == null ? null : member.age.eq(ageCond);
  }
  ```
  - where 조건에 null 값은 무시된다.
  - 메서드를 다른 쿼리에서도 재활용 할 수 있다.
  - 쿼리 자체의 가독성이 높아진다.
  - 조합 가능
    ```java
    // .where(allEq(usernameEq, ageCond)
    private BooleanExpression allEq(String usernameEq, Integer ageCond) {
        return usernameEq(usernameCond).and(ageEq(ageCond));
    }
    ```
    - null 체크는 주의해서 처리해야 함

#### 수정, 삭제 벌크 연산

- 건건히 수정하는 것보다 한번에 수정하는 것이 더 성능이 좋다.
- JPA에서 이런 연산을 벌크 연산이라 한다.

- 문자열 수정
  ```java
  long count = queryFactory
      .update(member)
      .set(member.username, "non-user")
      .where(member.age.lt(28))
      .execute();
  ```

- 기존 숫자에 더하기
  ```java
  long count = queryFactory
      .update(member)
      .set(member.age, member.age.add(1))
      .execute();
  ```
  - 곱하기: multiply(x);

- 삭제
  ```java
  long count = queryFactory
      .delete(member)
      .where(member.age.gt(18))
      .execute();
  ```
  
- JPQL 배치와 마찬가지로, 영속성 컨텍스트에 있는 엔티티를 무시하고 실행되기 때문에 배치 쿼리를 실행하고 나면 영속성 컨텍스트를 초기화 하는 것이 안전하다.
  ```java
  long count = queryFactory
      .update(member)
      .set(member.username, "non-user")
      .where(member.age.lt(28))
      .execute();
  /*
  PersistenceContext: Member1 // DB: non-user
  PersistenceContext: Member2 // DB: non-user
  PersistenceContext: Member3 // DB: Member3
  PersistenceContext: Member4 // DB: Member4
  해당 쿼리가 진행하면 PersistenceContext와 DB에 대한 결과가 다르다
  다시 조회해 와도 Repeatable Read가 유지되기 때문에 PersistenceContext가 우선순위가 된다.
  PersistenceContext를 초기화 해야 한다.
  */
  em.flush();
  em.clear();
  ```
  
#### function 호출

- SQL function은 JPA와 같이 Dialect에 등록된 내용만 호출할 수 있다.
- member M으로 변경하는 replace 함수 사용
  ```java
  String result = queryFactory
      .select(Expressions.stringTemplate("function('replace', {0}, {1}, {2})", member.username, "member", "M"))
      .from(member)
      .fetchFirst();
  ```
- 소문자로 변경해서 비교해라.
  ```java
  String result = queryFactory
      .select(member.username)
      .from(member)
      .where(member.username.eq(
          Expressions.stringTemplate("function('lower', {0})", member.username)))
      .fetchFirst();
  ```
  - lower 같은 ansi 표준 함수들은 querydsl이 상당부분 내장하고 있다. 따라서 다음과 같이 처리해도
결과는 같다.
    ```java
    .where(member.username.eq(member.username.lower()))
    ```

- Custom Function 사용 방법
  - Dialect 설정
    - 사용하는 DB에 따른 Dialect가 등록되어 있다.
    - application.yml
      ```
      spring:
        jpa:
          properties:
            hibernate:
              dialect: org.hibernate.H2Dialect
      ```
  - Dialect 상속받아 커스텀 펑션 사용
    ```java
    public class MyH2Dialect extends H2Dialect {
        public MyH2Dialect() {
            super();
            registerFunction("customFunction", new StandardSQLFunction("h2_func_ex", new StringType()));
        }
    }
    ```
  - application.yml에 다시 등록
    ```java
    spring:
      jpa:
        properties:
          hibernate:
            #dialect: org.hibernate.H2Dialect
            dialect: com.querydsl.MyH2Dialect
    ```

#### 출처

- 실전! Querydsl \[인프런 김영한님 강의]
- https://055055.tistory.com/83
