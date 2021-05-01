---
layout: post
title: JPA와 연관관계
summary: JPA - 자바 ORM 표준 JPA 프로그래밍 - 기본편
author: devhtak
date: '2021-04-30 21:41:00 +0900'
category: JPA
---

#### 연관관계가 필요한 이유

![image](https://user-images.githubusercontent.com/42403023/116777878-a63ad300-aaa9-11eb-9d1c-0dcd4e82a5ba.png)

- 객체를 테이블에 맞추어 모델링한다면 문제점이 있다.
  ```java
  //팀 저장
  Team team = new Team();
  team.setName("TeamA");
  em.persist(team);
  
  //회원 저장
  Member member = new Member();
  member.setName("member1");
  member.setTeamId(team.getId());
  em.persist(member);
  
  //조회
  Member findMember = em.find(Member.class, member.getId()); 
  //연관관계가 없음
  Long teamId = findMember.getTeamId();
  Team findTeam = em.find(Team.class, teamId);
  ```
  - Team을 조회할 때 member에서 저장한 team_id를 통해 다시 조회해야 한다.
  - 객체에 reference를 조회 한다면 아래와 같은 방법으로 바로 조회하지 않을까?
    ```java
    Team team = member.getTeam();
    ```
    
- 객체를 테이블에 맞추어 데이터 중심으로 모델링한다면, 협력 관계를 만들 수 없다.
  - 테이블은 외래 키로 조인을 사용하여 연관된 테이블을 찾는다.
  - 객체는 참조를 사용해서 연관된 객체를 찾는다.
  - 테이블과 객체 사이에는 이런 큰 간견이 있다.

#### 단방향 연관관계

![image](https://user-images.githubusercontent.com/42403023/116777947-39740880-aaaa-11eb-8aac-4734b645e7ba.png)
```java
@Entity
public class Member { 

    @Id @GeneratedValue
    private Long id;

    @Column(name = "USERNAME")
    private String name;

    private int age;

    // @Column(name = "TEAM_ID")
    // private Long teamId;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;
}
```
- 이렇게 객체 지향 모델링을 한다면, 참조로 연관관계 조회가 가능하다.
  - 객체 그래프 탐색
  ```java
  //조회
  Member findMember = em.find(Member.class, member.getId()); 
  //참조를 사용해서 연관관계 조회
  Team findTeam = findMember.getTeam();
  ```

#### 양방향 연관관계와 연관관계의 주인

![image](https://user-images.githubusercontent.com/42403023/116777984-6e805b00-aaaa-11eb-9cf3-a559cf1edc98.png)

- Member.class
  ```java
  @Entity
  public class Member { 
      @Id @GeneratedValue
      private Long id;
      
      @Column(name = "USERNAME")
      private String name;
 
      private int age;
 
      @ManyToOne
      @JoinColumn(name = "TEAM_ID")
      private Team team;
  }
  ```

- Team.class
  ```java
  @Entity
  public class Team {
 
      @Id @GeneratedValue
      private Long id;
      
      private String name;
 
      @OneToMany(mappedBy = "team")
      List<Member> members = new ArrayList<Member>();
  }  
  ```

- 양방향의 경우 반대 방향으로 객체 그래프 탐색이 가능하다.
  ```java
  //조회
  Team findTeam = em.find(Team.class, team.getId()); 
  int memberSize = findTeam.getMembers().size(); //역방향 조회
  ```
  
- 연관관계의 주인과 mappedBy
  - 객체와 테이블이 관계를 맺는 차이
    - 객체의 양방향 연관관계: 2개(회원 -> 팀, 팀 -> 회원)
      - 객체의 양방향 관계는 사실 양방향 관계가 아니라 서로 다른 단뱡향 관계 2개다.
      - 객체를 양방향으로 참조하려면 단방향 연관관계를 2개 만들어야 한다. 
    - 테이블의 연관관계 = 1개 (회원 <-> 팀, 양방향 가능)
      - 테이블은 외래 키 하나로 두 테이블의 연관관계를 관리
      - MEMBER.TEAM_ID 외래 키 하나로 양방향 연관관계 가짐(양쪽으로 조인할 수 있다.)

  - 연관관계의 주인(Owner)
    - 객체의 두 관계중 하나를 연관관계의 주인으로 지정
    - 연관관계의 주인만이 외래 키를 관리(등록, 수정) 
    - 주인이 아닌쪽은 읽기만 가능
    - 주인은 mappedBy 속성 사용X 
    - 주인이 아니면 mappedBy 속성으로 주인 지정

  - 누가 주인인가?
    - 외래 키가 있는 있는 곳을 주인으로 정해라
    - 여기서는 Member.team이 연관관계의 주인

  - 편의 메소드
    - 순수 객체 상태를 고려해서 항상 양쪽에 값을 설정하자
    - 연관관계 편의 메소드를 생성하자
    - 양방향 매핑시에 무한 루프를 조심하자
      ```java
      @Entity
      public class Team {

          @Id @GeneratedValue
          private Long id;

          private String name;

          @OneToMany(mappedBy = "team")
          List<Member> members = new ArrayList<Member>();
          
          // 추가 편의 메소드
          public void addMember(Member member) {
              members.add(member);
              member.setTeam(this);
          }
          
          // 삭제 편의 메소드
          public void removeMember(Member member) {
              if(members.contains(member)) {
                  members.remove(member);
                  member.setTeam(null);
              }
          }
      }  
      ```
      
#### 연관관계 매핑 시 고려사항 3가지

- 다중성
- 단방향, 양방향
- 연관관계 주인

#### 다중성

- 다대일: @ManyToOne
- 일대다: @OneToMany
- 일대일: @OneToOne
- 다대다: @ManyToMany
  - 다대다는 실무에서 사용하지 않는다.
  - 연관 객체(테이블)을 생성하여 사용

#### 단방향, 양방향

- 테이블
  - 외래 키 하나로 양쪽 조인 가능
  - 사실 방향이라는 개념이 없다.
  
- 객체
  - 참조용 필드가 있는 쪽으로만 참조 가능
  - 한쪽만 참조하면 단방향
  - 양쪽이 서로 참조하면 양방향
  
#### 연관관계의 주인

- 테이블은 외래키 하나로 두 테이블이 연관관계를 맺는다.
- 객체 양방향 관계는 A->B, B->A 처럼 참조가 2군데
- 객체 양방향 관계는 참조가 2군데 있다. 둘중 테이블의 외래키를 관리할 곳을 지정해야 한다.
- 연관관계의 주인: 외래 키를 관리하는 참조
- 주인의 반대편: 외래 키에 영향을 주지 않음, 단순 조회만 가능

#### 다대일(@ManyToOne)

- 단방향
  ![image](https://user-images.githubusercontent.com/42403023/116778363-6c1f0080-aaac-11eb-9f2f-6ac14a7432a0.png)

  - 가장 많이 사용하는 연관관계
  - 다대일의 반대는 일대다

- 양방향
  ![image](https://user-images.githubusercontent.com/42403023/116778388-8658de80-aaac-11eb-80d2-399124692303.png)
  
  - 외래 키가 있는 쪽이 연관관계의 주인
  - 양쪽을 서로 참조하도록 개발

#### 일대다

- 단방향
  ![image](https://user-images.githubusercontent.com/42403023/116778513-480fef00-aaad-11eb-85ec-93838a954180.png)
  
  - 일대다 단방향은 일대다(1:N)에서 일(1)이 연관관계의 주인
  - 테이블 일대다 관계는 항상 다(N) 쪽에 외래 키가 있음
  - 객체와 테이블의 차이 때문에 반대편 테이블의 외래 키를 관리하는 특이한 구조
  - @JoinColumn을 꼭 사용해야 함. 그렇지 않으면 조인 테이블 방식을 사용함(중간에 테이블을 하나 추가함)
  
  - 일대다 단방향 매핑의 단점
    - 엔티티가 관리하는 외래 키가 다른 테이블에 있음
    - 연관관계 관리를 위해 추가로 UPDATE SQL 실행
  - 일대다 단방향 매핑보다는 다대일 양방향 매핑을 사용하자

- 양방향
  ![image](https://user-images.githubusercontent.com/42403023/116778543-78f02400-aaad-11eb-8e4a-8daa60fc4e13.png)

  - 이런 매핑은 공식적으로 존재하지 않는다.
  - @JoinColumn(insertable=false, updatable=false) 
  - 읽기 전용 필드를 사용해서 양방향 처럼 사용하는 방법
  - 다대일 양방향을 사용하자

