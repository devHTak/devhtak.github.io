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

- JPA MemberRepository 생성 및 테스트
  ```java
  public interface MemberRepository extends JpaRepository<Member, Long> {
      List<Member> findByUsername(String username);
  }
  ```
  ```java
  @SpringBootTest
  @Transactional
  class MemberRepositoryTest {
      @Autowired EntityManager em;
      @Autowired MemberRepository memberRepository;
      
      @Test
      void basicTest() {
          Member member = new Member("Member1", 10);
          memberRepository.save(member);
          
          Member findMember = memberRepository.findById(member.getId())
                  .get();
          assertThat(member).isEqualTo(findMember);
          
          List<Member> result1 = memberRepository.findAll();
          assertThat(result1).containsExactly(member);
          
          List<Member> result2 = memberRepository.findByUsername("Member1");
          assertThat(result2).containsExactly(member);
      }
  }
  ```
  - Querydsl 전용 기능인 회원 search를 작성할 수 없다. -> 사용자 정의 리포지터리 필요  

- 사용자 정의 리포지터리
  - JpaRepository를 구현하는 MemberRepository 또한 interface이기 때문에 구현 메소드를 생성할 수 없다.
  - 사용자 정의 리포지터리 및 구현체를 사용하여 QueryDsl을 이용
  - 사용자 정의 리포지터리 사용법
    - 사용자 정의 인터페이스 작성
    - 사용자 정의 인터페이스 구현
    - 스프링 데이터 리포지토리에 사용자 정의 인터페이스 상속

  - 사용자 정의 리포지 터리 구성
    ```
    - <<Interface>> MemberRepository -(extends)-> <<Interface>> JpaRepository 
    - <<Interface>> MemberRepositoryCustom -(implements)-> MemberRepositoryImpl
    ```
    - 주의! MemberRepositoryCustom.java의 이름은 상관없다. 하지만 MemberRepositoryImpl.java의 이름은 구현Repository + Impl 로 정해져 있다.
    
  - 사용자 정의 인터페이스 작성
    ```java
    public interface MemberRepositoryCustom {
        public List<MemberTeamDto> search(MemberSearchCondition condition);
    }
    ```
  - 사용자 정의 인터페이스 구현
    ```java
    public class MemberRepositoryImpl implements MemberRepositoryCustom {

        private final JPAQueryFactory queryFactory;
        
        public MemberRepositoryImpl(EntityManager em) {
            queryFactory = new JPAQueryFactory(em);
        }
        
        @Override
        public List<MemberTeamDto> search(MemberSearchCondition condition) {
            // 회원명, 팀명, 나이(ageGoe, ageLoe)
            return queryFactory.select( new QMemberTeamDto(
                    member.id, member.username, member.age,
                    team.id, team.name
                )).from(member)
                .join(member.teatm, team)
                .where(usernameEq(condition.getUsername()
                    ,teamNameEq(condition.getTeamName())
                    ,ageGoe(condition.getAgeGoe())
                    ,ageLoe(condition.getAgeLoe()));
        }
        
        private BooleanExpression usernameEq(String username) {
            return isEmpty(username) ? null : member.username.eq(username);
        }
        private BooleanExpression teamNameEq(String teamName) {
            return isEmpty(teamName) ? null : team.name.eq(teamName);
        }
        private BooleanExpression ageGoe(Integer age) {
            return age == null ? null : member.age.goe(age);
        }
        private BooleanExpression ageLoe(Integer age) {
            return age == null ? null : member.age.loe(age);
        }
    }
    ```
  - 사용자 정의 인터페이스 상속
    ```java
    public interface MemberRepository extends JpaRepository<Member, Long>, MemberRepositoryCustom {
        List<Member> findByUsername(String username);
    }
    ```
    
  - 테스트
    ```java
    @Test
    public void searchTest() {
        Team teamA = new Team("teamA");
        Team teamB = new Team("teamB");
        
        em.persist(teamA);
        em.persist(teamB);
        
        Member member1 = new Member("member1", 10, teamA);
        Member member2 = new Member("member2", 20, teamA);
        Member member3 = new Member("member3", 30, teamB);
        Member member4 = new Member("member4", 40, teamB);
        
        em.persist(member1);
        em.persist(member2);
        em.persist(member3);
        em.persist(member4);
        
        MemberSearchCondition condition = new MemberSearchCondition();
        condition.setAgeGoe(35);
        condition.setAgeLoe(40);
        condition.setTeamName("teamB");
        
        List<MemberTeamDto> result = memberRepository.search(condition);
        assertThat(result).extracting("username").containsExactly("member4");
    }
    ```

- 페이징 활용 1. Querydsl 페이징 연동
  - 스프링 데이터의 Page, Pageable 활용
  - 전체 카운트를 한번에 조회하는 단순한 방법
  - 데이터 내용과 전체 카운트를 별도로 조회하는 방법
  
  - Interface
    ```java
    Page<MemberTeamDto> searchPageSimple(MemberSearchCondition condition,
    Pageable pageable);
    Page<MemberTeamDto> searchPageComplex(MemberSearchCondition condition,
    Pageable pageable);
    ```
  
  - Implements
    ```java
    Page<MemberTeamDto> searchPageSimple(MemberSearchCondition condition,
    Pageable pageable) {
        QueryResults<MemberTeamDto> results = queryFactory
            .select( new QMemberTeamDto(
            
            )).from(member)
            .leftJoin(member.team, team)
            .where( usernameEq(condition.getUsername())
                ,teamNameEq(condition.getTetamName())
                ,ageGoe(condition.getAgeGoe())
                ,ageLoe(condition.getAgeLoe())
            ).offset(pageable.getOffset())
            .limit(pageable.getPageSize())
            .fetchResults();
            
        List<MemberTeamDto> content = results.getResults();
        long total = results.getTotal();
        
        return new PageImpl<>(content, pageable, total);
    }
    Page<MemberTeamDto> searchPageComplex(MemberSearchCondition condition,
    Pageable pageable) {
        List<MemberTeamDto> content = queryFactory
            .select(new QMemberTeamDto(
            
            )).from(member)
            .leftJoin(member.team, team)
            .where(usernameEq(condition.getUsername())
                ,teamNameEq(condition.getTeamName())
                ,ageGoe(condition.getAgeGoe())
                ,ageLoe(condition.getAgeLoe()))
            .offset(pageable.getOffset())
            ,limit(pageable.getLimit())
            ,fetch();
        long count =  queryFactory
            .select(member)
            .from(member)
            .leftJoin(member.team, team)
            .where(usernameEq(condition.getUsername()),
                teamNameEq(condition.getTeamName()),
                ageGoe(condition.getAgeGoe()),
                ageLoe(condition.getAgeLoe()))
            .fetchCount();
        return new PageImpl<>(content, pageable, count);
    }
    ```
    - searchPageSimple
      - fetchResult()를 하게 되면 totalCount를 위해 쿼리가 2번 호출된다
      - totalCount 쿼리에서는 orderBy등 필요없는 쿼리는 삭제된다.
    - searchPageComplex
      - fetch()를 하면 totalCount가 날라가지 않지만, totalCount가 필요하기 때문에 한번 더 쿼리를 작성해주었다.
      - count 쿼리 튜닝을 위해 수정
      
- 페이징 활용 2. CountQuery최적화
  - 사용 가능한 경우 생략
    - 페이지 시작이면서 컨텐츠 사이즈가 페이지 사이즈보다 작을 때
    - 마지막 페이지 일 때 (offset + 컨텐츠 사이즈를 더해서 전체 사이즈를 구함)
  - 스프링 데이터 라이브러리 제공
  - count 쿼리
    ```java
    JPAQuery<Member> countQuery = queryFactory
        .select(member)
        .from(member)
        .leftJoin(member.team, team)
        .where(usernameEq(condition.getUsername())
            .teamNameEq(condition.getTeamName())
            .ageGoe(condition.getAgeGoe())
            .ageLoe(condition.getAgeLoe()))
        
    return PageableExecutionUtils.getPage(content, pageable, () -> countQuery.fetchCount());
    ```
    - fetchCount(); 를 해야 실제 쿼리가 날라간다.
    - PageableExecutionUtils.getPage()에서 조건을 판단하여 필요 없으면 count쿼리를 발생시키지 않는다.

- 페이징 활용 3. 컨트롤러 개발
  ```java
  @GetMapping("/members/simple-paging")
  public Page<MemberTeamDto> searchMemberSimplePaging(MemberSearchCondition condition, Pageable pageable) {
      return memberRepository.searchPageSimple(condition, pageable);
  }
  
  @GetMapping("/members/complex-paging")
  public Page<MEmberTeamDto> searchMemberComplexPaging(MemberSearchCondition condition, Pageable pageble) {
      return memberRepository.searchPageComplex(condition, pageable);
  }
  ```
  - /members/simple-paging?size=5&page=2

- 정렬(Sort)
  - 스프링 데이터 JPA는 자신의 정렬(Sort)을 Querydsl의 정렬(OrderSpecifier)로 편리하게 변경하는
기능을 제공한다.
  - 정렬(Sort)은 조건이 조금만 복잡해져도 Pageable 의 Sort 기능을 사용하기 어렵다. 
  - 루트 엔티티 범위를 넘어가는 동적 정렬 기능이 필요하면 스프링 데이터 페이징이 제공하는 Sort 를 사용하기 보다는 파라미터를 받아서 직접 처리하는 것을 권장한다.
  ```java
  JPAQuery<Member> query = queryFactory
      .selectFrom(member);
      
  for (Sort.Order o : pageable.getSort()) {
      PathBuilder pathBuilder = new PathBuilder(member.getType(), member.getMetadata());
      query.orderBy(new OrderSpecifier(o.isAscending() ? Order.ASC : Order.DESC, pathBuilder.get(o.getProperty())));
  }
  
  List<Member> result = query.fetch();
  ```

#### 출처

- 실전! Querydsl \[인프런 김영한님 강의]
