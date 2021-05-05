---
layout: post
title: JPA. 프록시와 연관관계
summary: JPA - 자바 ORM 표준 JPA 프로그래밍 - 기본편
author: devhtak
date: '2021-05-05 21:41:00 +0900'
category: JPA
---

#### 프록시

- em.find() vs em.getReference()
  - em.find(): 데이터베이스를 통해서 실제 엔티티 객체 조회
  - em.getReference(): 데이터베이스 조회를 미루는 가짜(프록시) 엔티티 객체 조회
  
- 프록시 특징
  - 실제 클래스를 상속 받아서 만들어진다.
  - 실제 클래스와 겉 모양이 같다
  - 사용하는 입장에서는 진짜 객체인지 프록시 객체인지 구분하지 않고 사용하면 된다.

  ![image](https://user-images.githubusercontent.com/42403023/117097647-2bbecb80-ada7-11eb-883b-9a1fd8cded45.png)

  - 프록시 객체는 실제 객체의 참조(target)를 보관
  - 프록시 객체를 호출하면 프록시 객체는 실제 객체의 메소드 호출

- 프록시 객체의 초기화
  ```java 
  Member member = entityManager.getReference(Member.class, 1L);
  member.getName();
  ```
  - getReference로 Proxy 객체를 불렀다.
  - getName으로 MemberProxy 객체에 메소드를 호출하면
  - JPA에서 영속성 컨텍스트의 초기화 요청을 하고, 영속성 컨텍스트는 DB에서 조회하여 실제 엔티티를 생성한다.
  - Proxy객체는 Member객체에 getName()을 호출하여 리턴한다.
  
- 프록시 객체 초기화 특징
  - 프록시 객체는 처음 사용할 때 한 번만 초기화한다.
  - 프록시 객체를 초기화 할 때, 프록시 객체가 실제 엔티티로 바뀌는 것은 아님, 초기화되면 프록시 객체를 통해서 실제 엔티티에 접근 가능
  - 프록시 객체는 원본 엔티티를 상속받음, 따라서 타입 체크시 주의해야함 (== 비교 실패, 대신 instanceof 사용) 
  - 영속성 컨텍스트에 찾는 엔티티가 이미 있으면 em.getReference()를 호출해도 실제 엔티티 반환
  - 영속성 컨텍스트의 도움을 받을 수 없는 준영속 상태일 때, 프록시를 초기화하면 문제 발생 (하이버네이트는 org.hibernate.LazyInitializationException 예외를 터트림)

- 프록시 확인
  - 프록시 인스턴스의 초기화 여부 확인
    ```java
    PersistenceUnitUtil.isLoaded(Object entity) 
    ```
  - 프록시 클래스 확인 방법
    ```java
    entity.getClass().getName() 출력(..javasist.. or HibernateProxy…) 
    ```
  - 프록시 강제 초기화
    ```java
    org.hibernate.Hibernate.initialize(entity); 
    ```
  - 참고: JPA 표준은 강제 초기화 없음
    - 강제 호출: member.getName()

#### 즉시 로딩과 지연로딩

- Member를 조회할 때 Team도 함께 조회해야 할까?
  - 회원과 팀 함께 출력
    ```java
    public void printUserAndTeam(String memberId) {
        Member member = em.find(Member.class, memberId);
        Team team = member.getTeam();
        System.out.println("회원 이름: " + member.getUsername());
        System.out.println("팀 이름: " + team.getName());
    }
    ```
    
  - 회원만 출력
    ```java
    public void printUserAndTeam(String memberId) {
        Member member = em.find(Member.class, memberId);
        System.out.println("회원 이름: " + member.getUsername());
    }
    ```
  - 팀을 조회하기 위해서는 JOIN이 필요하다.
  - 하지만 회원만 출력할 경우 팀에 대한 정보를 조회할 필요가 없다.
  - JPA는 이런 상황에서 Proxy를 활용하여 해결한다.

- 지연로딩
  ```java
  @Entity
  @Getter @Setter
  public class Member {

      @Id @GeneratedValue
      @Column(name = "MEMBER_ID")
      private Long id;

      @ManyToOne(fetch = FetchType.LAZY)
      @JoinColumn(name = "TEAM_ID")
      private Team team;

  }
  ```
  - LAZY를 사용하면 Team에는 프록시 객체가 담기며 실제 사용하는 시점에서 초기화(DB 조회)된다.
    ```java
    Member member = entityManager.find(Member.class, 1L);
    Team team = member.getTeam(); // 프록시 객체
    System.out.println(team.getName()); // 사용하는 시점에서 초기화(DB 조회)된다.
    ```
  
- 즉시로딩
  ```java
  @Entity
  @Getter @Setter
  public class Member {

      @Id @GeneratedValue
      @Column(name = "MEMBER_ID")
      private Long id;

      @ManyToOne(fetch = FetchType.EAGER)
      @JoinColumn(name = "TEAM_ID")
      private Team team;

  }
  ```
  - EAGER를 사용하면 Member를 조회할 때 Team과 Join하여 Team 객체에 바로 담긴다.
    ```java
    Member member = entityManager.find(Member.class, 1L); // Team 객체까지 조회
    ```
    
- 프록시와 즉시 로딩 주의!!
  - 가급적 지연 로딩만 사용하자
  - 즉시 로딩을 적용하면 예상하지 못한 SQL이 발생한다.
    
  - 즉시 로딩은 JPQL에서 N+1 문제를 일으킨다.
    ```
    SELECT m FROM Member m; // 모든 Member에 대한 Team을 SELECT 한다. (SELECT 쿼리가 N + 1개 발생)
    ```
    - JPQL Fetch 조인이나, 엔티티 그래프 기능을 사용하자
  - @ManyToOne, @OneToOne은 기본이 즉시 로딩이다 -> Lazy로 설정
  - @OneToMany, @ManyToMany는 기본이 지연로딩이다.

#### 영속성전이: CASCADE

- 특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들고 싶을 때
  - 예) 부모 엔티티를 저장할 때 자식 엔티티도 함께 저장

  ```java
  @Entity
  @Getter @Setter
  public class Team {

    @Id @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;

    private String name;

    @OneToMany(mappedBy = "team", cascade = CascadeType.PERSIST)
    private List<Member> members = new ArrayList<>();

  }
  ```
  ```java
  @Test
  @Transactional
  void test() {
      Member member1 = new Member();
      Member member1 = new Member();
      Team team = new Team();
      team.addMember(member1);
      team.addMember(member2);

      entityManager.persist(team);

      entityManager.flush();		
  }
  ```
    - team만 저장했지만 member1, member2도 함께 저장된다.

- 영속성 전이: CASCADE 주의!
  - 영속성 전이는 연관관계를 매핑하는 것과 아무 관련이 없음
  - 엔티티를 영속화할 때 연관된 엔티티도 함께 영속화하는 편리함을 제공할 뿐

- CASCADE의 종류
  - ALL: 모두 적용
  - PERSIST: 영속
  - REMOVE: 삭제
  - MERGE: 병합
  - REFRESH: REFRESH 
  - DETACH: DETACH

#### 고아객체

- 고아 객체 제거: 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제
  ```java
  @Entity
  @Getter @Setter
  public class Team {
    @Id @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;
    private String name;
    
    @OneToMany(mappedBy = "team", cascade = CascadeType.PERSIST, orphanRemoval = true)
    private List<Member> members = new ArrayList<>();
  }
  ```
  - orphanRemoval = true 
  ```java
  Team team = em.find(Team.class, id); 
  team.getMembers().remove(0);
  ```
    - 자식 엔티티를 컬렉션에서 제거
      ```
      DELETE FROM CHILD WHERE ID=?
      ```

- 고아 객체 - 주의
  - 참조가 제거된 엔티티는 다른 곳에서 참조하지 않는 고아 객체로 보고 삭제하는 기능
  - 참조하는 곳이 하나일 때 사용해야함! 
  - 특정 엔티티가 개인 소유할 때 사용
  - @OneToOne, @OneToMany만 가능
  - 참고: 개념적으로 부모를 제거하면 자식은 고아가 된다. 따라서 고아 객체 제거 기능을 활성화 하면, 부모를 제거할 때 자식도 함께 제거된다. 
  - 이것은 CascadeType.REMOVE처럼 동작한다

#### 영속성전이 + 고아객체, 생명주기

```
CascadeType.ALL + orphanRemovel=true
```
- 스스로 생명주기를 관리하는 엔티티는 em.persist()로 영속화, em.remove()로 제거
- 두 옵션을 모두 활성화 하면 부모 엔티티를 통해서 자식의 생명
- 주기를 관리할 수 있음
- 도메인 주도 설계(DDD)의 Aggregate Root개념을 구현할 때 유용
