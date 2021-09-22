---
layout: post
title: Querydsl 과 JpaRepository
summary: Querydsl
author: devhtak
date: '2021-09-22 21:41:00 +0900'
category: JPA
---

#### 순수 JPA와 Querydsl

- EntityManager와 JPAQueryFactory
  ```java
  @Repository
  public class MemberJpaRepository {
      private final EntityManager em;
      private final JPAQueryFactory queryFactory;

      public MemberJpaRepository(EntityManager em) {
          this.em = em;
          queryFactory = new JPAQueryFactory(em);
      }

      public void save(Member member) {
          em.persist(member);
      }

      public Optional<Member> findById(Long id) {
          Member member = em.find(Member.class, id);
          return Optional.ofNullable(member);
      }

      public List<Member> findAll() {
          return em.createQuery("select m from Member m", Member.class)
                    .getResultList();
      }

      public List<Member> findAllQuerydsl() {
          return queryFactory.selectFrom(QMember.member)
                    .fetch();
      }

      public List<Member> findByUsername(String username) {
          return em.createQuery("select m from Member m where m.username = :username", Member.class)
                      .setParameter("username", username);
                      .getResultList();
      }

      public List<Member> findByUsernameQuerydsl(String username) {
          return queryFactory.selectFrom(QMember.member);
                    .where(QMember.member.username.eq(username))
                    .fetch();
      }
  }
  ```

  - EntityManager를 통해 createQuery를 사용했던 부분을 JPAQueryFactory를 사용할 수 있다.
  - JPAQueryFactory는 빈으로 등록하여 사용해도 된다
    ```java
    @Bean
    public JPAQueryFactory jpaQueryFactory(EntityManager em) {
        return new JPAQueryFactory(em);
    }
    ```
    
- 동적 쿼리와 성능 최적화 조회 (Builder 사용)
  - MemberTeamDto.java
    ```java
    @Data
    public class MemberTeamDto {
        private Long memberId;
        private Long teamId;
        private String username;
        private String teamName;
        private int age;
        
        @QueryProjection
        public MemberTeamDto(Long memberId, Long teamId, String username, String teamName, int age) {
            this.memberId = memberId; this.teamId = teamId;
            this.username = username; this.teamName = teamName;
            this.age = age;
        }
    }
    ```
    - @QueryProjection 을 추가했다. QMemberTeamDto를 생성해야 한다.
    - @QueryProjection 을 사용하면 해당 DTO가 Querydsl을 의존하게 된다.
    - 이런 의존이 싫으면, 해당 에노테이션을 제거하고, Projection.bean(), fields(), constructor() 을 사용하면 된다.
    
  - MemberTeamCondition.java
    ```java
    @Data
    public class MemberTeamCondition {
        private String username;
        private String teamName;
        private Integer ageLoe; // null 을 활용하기 위해 Integer 사용
        private Integer ageGoe;
    }
    ```
    
  - 조회 예제
    ```java
    public List<MemberTeamDto> search1(MemberTeamCondition condition) {
        BooleanBuilder builder = new BooleanBuilder();
        // hasText: null, "" 확인 가능
        if(StringUtils.hasText(condition.getUsername())) {
            builder.and(member.username.eq(condition.getUsername()));
        }
        if(StringUtils.hasText(condition.getTeamName())) {
            builder.and(member.teamName.eq(condition.getTeamName()));
        }
        if(condition.getAgeLoe() != null) {
            builder.and(member.age.loe(condition.getAgeLoe()));
        }
        if(condition.getAgeGoe() != null) {
            builder.and(member.age.goe(condition.getAgeGoe()));
        }
        
        return queryFactory
            .select( new QMemberTeamDto(
                QMember.member.id,
                QTeam.team.id
                QMember.member.username,
                QTeam.team.name,
                QMember.member.age
            )).from(QMember.member)
            .join(QMember.member.team, team)
            .where(builder)
            .fetch();
    }
    ```
  
- 동적 쿼리와 성능 최적화 조회 (Where절 파라미터 사용)
  ```java
  public List<MemberTeamDto> search2(MemberSearchCondition condition) {
      return queryFactory
              .select( new QMemberTeamDto(
                  QMember.member.id,
                  QTeam.team.id
                  QMember.member.username,
                  QTeam.team.name,
                  QMember.member.age
              )).from(QMember.member)
              .join(QMember.member.team, QTeam.team)
              .where(usernameEq(condition.getUsername()),
                      teamNameEq(condition.getTeamName()),
                      ageGoe(condition.getAgeGoe()),
                      ageLoe(condition.getAgeLoe()))
              .fetch();
  }
  private BooleanExpression usernameEq(String username) {
      return StringUtils.isEmpty(username) ? null : QMember.member.username.eq(username);
  }
  private BooleanExpression teamNameEq(String teamName) {
      return StringUtils.isEmpty(teamName) ? null : QTeam.team.name.eq(teamName);
  }
  private BooleanExpression ageGoe(int age) {
      return age == null ? null : QMember.member.age.goe(age);
  }
  private BooleanExpression ageLoe(int age) {
      return age == null ? null : QMember.member.age.loe(age);
  }
  ```
  - where 절에 파라미터 방식을 사용하면 조건 재사용 가능
  - null 체크를 조심하여야 한다.
  

#### 스프링 데이터 JPA 와 Querydsl

#### 스프링 데이터 JPA가 제공하는 Querydsl 기능

#### 출처

- 실전! Querydsl \[인프런 김영한님 강의]
