---
layout: post
title: JPA N+1 문제와 해결방법
summary: JPA
author: devhtak
date: '2023-05-20 21:41:00 +0900'
category: JPA
---

#### N+1 문제 재현

- JPA를 사용하다 보면 흔하게 마주치는 문제가 N+1 문제이다
- fetch 전략이 Eager인 경우
  - 조회할 때 연관관계를 갖는 객체에 대한 조회 쿼리에서 N+1 문제 발생 
- fetch 전략이 Lazy인 경우
  - 연관관계를 맺은 객체를 사용할 때 N+1 문제가 발생할 수 있다.
- 예제
  - Team, Member 다대 일 관계
    ```java
    @Entity
    @Getter
    @Setter
    @NoArgsConstructor
    public class Team {

        @Id
        @GeneratedValue
        private Long id;

        private String name;

        @OneToMany(mappedBy = "team", cascade = CascadeType.ALL)
        private List<Member> members = new ArrayList<>();

        public Team(String name) {
            this.name = name;
        }

        public void addMember(Member member) {
            this.members.add(member);
            member.setTeam(this);
        }

        public void deleteMemger(Member member) {
            this.members.remove(member);
            member.setTeam(null);
        }
    }
    ```
    ```java
    @Entity
    @Getter
    @Setter
    @NoArgsConstructor
    public class Member {

        @Id
        @GeneratedValue
        private Long id;

        @ManyToOne(fetch = FetchType.LAZY)
        private Team team;

        private String name;

        public Member(String name, Team team) {
            this.name = name;
            this.team = team;
        }
    }
    ```
  - service
    ```java
    @Service
    @RequiredArgsConstructor
    public class TeamService {
        private final TeamRepository teamRepository;

        @Transactional(readOnly = true)
        public List<String> findAllMemberName() {
            return teamRepository.findAll()
                      .flatMap(team -> team.getMembers().stream())
                      .map(Member::getName)
                      .collect(Collectors.toList());
        }
    }
    ```
- N+1 문제 발생
  ```java
  @SpringBootTest
  class TeamServiceTest {
      @Autowired
      private TeamRepository teamRepository;

      @Autowired
      private TeamService teamService;

      @AfterEach
      public void after() {
          teamRepository.deleteAll();
      }

      @BeforeEach
      public void beforeEach() {
          List<Team> teams = IntStream.range(0, 10)
                              .mapToObj(i -> {
                                  Team team = new Team("Team" + i);
                                  team.addMember(new Member("Member" + i, team));
                                  return team;
                              }).collect(Collectors.toList());

          teamRepository.saveAll(teams);
      }

      @Test
      @DisplayName("N+1 예외 발생")
      public void nPlusOne() throws Exception {
          // given
          List<String> memberNames = teamService.findAllMemberName();

          //then
          assertEquals(memberNames.size(), 10);
      }
  }
  ```
  ```
  2023-05-20 14:12:57.650 DEBUG 2883 --- [    Test worker] org.hibernate.SQL : select team0_.id as id1_1_, team0_.name as name2_1_ fromteam team0_
  2023-05-20 14:12:57.707 DEBUG 2883 --- [    Test worker] org.hibernate.SQL: select members0_.team_id as team_id3_0_0_, members0_.id as id1_0_0_, members0_.id as id1_0_1_, members0_.name as name2_0_1_, members0_.team_id as team_id3_0_1_ from member members0_ where members0_.team_id=?
  2023-05-20 14:12:57.707 DEBUG 2883 --- [    Test worker] org.hibernate.SQL: select members0_.team_id as team_id3_0_0_, members0_.id as id1_0_0_, members0_.id as id1_0_1_, members0_.name as name2_0_1_, members0_.team_id as team_id3_0_1_ from member members0_ where members0_.team_id=?
  2023-05-20 14:12:57.707 DEBUG 2883 --- [    Test worker] org.hibernate.SQL: select members0_.team_id as team_id3_0_0_, members0_.id as id1_0_0_, members0_.id as id1_0_1_, members0_.name as name2_0_1_, members0_.team_id as team_id3_0_1_ from member members0_ where members0_.team_id=?
  2023-05-20 14:12:57.707 DEBUG 2883 --- [    Test worker] org.hibernate.SQL: select members0_.team_id as team_id3_0_0_, members0_.id as id1_0_0_, members0_.id as id1_0_1_, members0_.name as name2_0_1_, members0_.team_id as team_id3_0_1_ from member members0_ where members0_.team_id=?
  2023-05-20 14:12:57.707 DEBUG 2883 --- [    Test worker] org.hibernate.SQL: select members0_.team_id as team_id3_0_0_, members0_.id as id1_0_0_, members0_.id as id1_0_1_, members0_.name as name2_0_1_, members0_.team_id as team_id3_0_1_ from member members0_ where members0_.team_id=?
  2023-05-20 14:12:57.707 DEBUG 2883 --- [    Test worker] org.hibernate.SQL: select members0_.team_id as team_id3_0_0_, members0_.id as id1_0_0_, members0_.id as id1_0_1_, members0_.name as name2_0_1_, members0_.team_id as team_id3_0_1_ from member members0_ where members0_.team_id=?
  2023-05-20 14:12:57.707 DEBUG 2883 --- [    Test worker] org.hibernate.SQL: select members0_.team_id as team_id3_0_0_, members0_.id as id1_0_0_, members0_.id as id1_0_1_, members0_.name as name2_0_1_, members0_.team_id as team_id3_0_1_ from member members0_ where members0_.team_id=?
  2023-05-20 14:12:57.707 DEBUG 2883 --- [    Test worker] org.hibernate.SQL: select members0_.team_id as team_id3_0_0_, members0_.id as id1_0_0_, members0_.id as id1_0_1_, members0_.name as name2_0_1_, members0_.team_id as team_id3_0_1_ from member members0_ where members0_.team_id=?
  2023-05-20 14:12:57.707 DEBUG 2883 --- [    Test worker] org.hibernate.SQL: select members0_.team_id as team_id3_0_0_, members0_.id as id1_0_0_, members0_.id as id1_0_1_, members0_.name as name2_0_1_, members0_.team_id as team_id3_0_1_ from member members0_ where members0_.team_id=?
  2023-05-20 14:12:57.707 DEBUG 2883 --- [    Test worker] org.hibernate.SQL: select members0_.team_id as team_id3_0_0_, members0_.id as id1_0_0_, members0_.id as id1_0_1_, members0_.name as name2_0_1_, members0_.team_id as team_id3_0_1_ from member members0_ where members0_.team_id=?
  ```  
  - 이런 로그가 Team별로 총 10개가 반복된다!!

#### 해결 방법

1. Fetch Join
- 조회 시 바로 가져오고 싶은 Entity 필드를 지정
  ```java
  @Query("select t from Team t join fetch t.members")
  List<Team> findAllByJoinFetch();
  ```
- 쿼리 한번으로 필요한 결과를 조회하여 사용할 수 있다
  ```java
  2023-05-20 14:37:06.793 DEBUG 3004 --- [    Test worker] org.hibernate.SQL: select team0_.id as id1_1_0_, members1_.id as id1_0_1_, team0_.name as name2_1_0_, members1_.name as name2_0_1_, members1_.team_id as team_id3_0_1_, members1_.team_id as team_id3_0_0__, members1_.id as id1_0_0__  from team team0_  inner join member members1_ on team0_.id=members1_.team_id
  ```
- Eager, Lazy 조회를 해야하기 때문에 쿼리에서 표현하는 것은 불필요하다라고 생각할 수 있다.

2. @EntityGraph
- @EntityGraph에 attributePaths에 쿼리 수행시 바로 가져올 필드명을 지정하면 Lazy가 아닌 Eager 조회로 가져오게 된다.
  ```java
  @EntityGraph(attributePaths = {"members"})
  @Query("select t from Team t")
  List<Team> findAllEntityGraph();
  ```
- 쿼리 한번으로 필요한 결과를 조회한다.
  ```
  2023-05-20 14:50:03.139 DEBUG 3227 --- [    Test worker] org.hibernate.SQL: select team0_.id as id1_1_0_, members1_.id as id1_0_1_, team0_.name as name2_1_0_, members1_.name as name2_0_1_, members1_.team_id as team_id3_0_1_, members1_.team_id as team_id3_0_0__, members1_.id as id1_0_0__  from team team0_ left outer join member members1_ on team0_.id=members1_.team_id 
  ```
  
#### 우회 방법
1. Batch Size 지정
- N+1이 하는방법이 아닌 N+1 문제가 발생하여도 in 연산자를 사용하여 쿼리 수를 줄이는 방법
- @BatchSize 애노테이션 또는 설정(application.yml)
  ```yml
  spring:
    jpa:
      properties:
        hibernate:
          default_batch_fetch_size: 1000
  ```
- 로그 확인
  ```
  2023-05-20 15:12:37.887 DEBUG 3340 --- [    Test worker] org.hibernate.SQL : select team0_.id as id1_1_, team0_.name as name2_1_  from team team0_
  2023-05-20 15:12:37.893 DEBUG 3340 --- [    Test worker] org.hibernate.SQL : select members0_.team_id as team_id3_0_1_, members0_.id as id1_0_1_, members0_.id as id1_0_0_, members0_.name as name2_0_0_, members0_.team_id as team_id3_0_0_ from member members0_ where members0_.team_id in ( ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
  ```

#### Fetch Join과 @EntityGraph 사용 시 주의사항과 해결방법

1. Cartesian Product
- 주의사항
  - 쿼리를 살펴보면 Fetch Join은 inner join, @EntityGraph는 outer join을 사용한다
    - 카테시안 곱(Cartesian Product)가 발생하여 Member의 수만큼 Team이 중복 발생하게 된다
- 해결 방안
  - List 대신 Set을 사용하여 중복 삭제
    ```java
    @Query("select t from Team t join fetch t.members m")
    Set<Team> findAllFetchJoin();
    ```
  - join query에 대하여 distinct 를 주어 중복 삭제
    ```java
    @Query("select distinct t from Team t join fetch t.members m")
    List<Team> findAllFetchJoin();
    ```
2. Fetch Join 과 pagenation
    
#### 출처

- https://programmer93.tistory.com/83
- https://jojoldu.tistory.com/165
