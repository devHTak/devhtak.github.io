---
layout: post
title: JPA와 영속성 관리
summary: JPA - 자바 ORM 표준 JPA 프로그래밍 - 기본편
author: devhtak
date: '2021-04-25 21:41:00 +0900'
category: JPA
---

#### ORM

- Object Realational mapping (객체 관계 매핑)
- 객체는 객체대로 설계하고, 관계형 데이터베이스는 관계형 데이터베이스대로 설계
- ORM 프레임워크가 중간에서 매핑
- 대중적인 언어에는 대부분 ORM 기술이 존재

#### JPA

- Java Persistence API의 약자
- 자바 진영의 ORM 기술 표준
- JPA는 애플리케이션과 JDBC 사이에서 동작한다.
  - Java Application -> JPA -> JDBC API <--> (sql)DB

- JPA 동작
  - 저장
    - DAO에서 EntityManager의 Persist 를 실행한다.
    - JPA는 Persist에 맞게 Entity 분석, Insert SQL 생성, JDBC API 사용 후, 객체와 DB 간의 페러다임 불일치를 해결한다.
    - JDBC API는 DB에 Insert 쿼리를 실행한다.
    
  - 조회
    - DAO에서 JPA에게 find 메소드 실행
    - JPA는 SELECT SQL 생성, JDBC API 사용, ResultSet 매핑, 패러다임 불일치 해결한 후 EntityObject를 DAO에게 return한다.
    - JDBC API는 DB에 SELECT 쿼리를 실행한다.
    
- JPA 사용 이유
  - SQL 중심의 개발에서 객체 중심의 개발
  - 생산성
  - 유지보수
  - 데이터 접근 추상화와 벤더 독립성
    - JPA는 특정 DB에 종속적이지 않다.
    - 표준이 아닌 DB별로 다른 키워드 등을 지원해준다. 
  - 표준
  - 패러다임의 불일치 해결
    - 상속
      - 객체의 상속 <-> DB의 슈퍼타입과 서브타입
        - 만약 서브타입에 Insert하면 JPA가 슈퍼타입, 서브타입에 insert하는 것을 만들어준다.
        - 만약 서브타입을 조회하면 JPA는 슈퍼타입, 서브타입을 함께 조회하도록 해준다.
      - 연관관계와 객체 그래프 탐색
        - 객체의 양방향 관계 <-> DB의 관계
        - JPA를 통해서 객체를 가져오면 관계에 있는 객체를 가져와 준다.
      - 비교하기
        - DB에서 조회하여 받은 객체는 Primary Key는 같으나 JPA는 id가 같으면 == 에서 같도록 해준다.
        
- JPA의 성능 최적화 기능
  - 1차 캐시와 동일성 보장
    - 같은 트랜잭션 안에서는 같은 엔티티를 반환 - 조회 성능을 향상시킨다.
    - DB Isolation Level이 Read commit이여도, 애플리케이션에서 Repeatable Read 보장
  - 트랜잭션을 지원하는 쓰기 지연
    - 트랜잭션을 커밋할 때까지 Insert SQL 을 모음
      - JDBC Batch SQL 기능을 사용하여 한번에 SQL 전송
    - 트랜잭션 커밋 시 UPDATE, DELETE SQL을 실행하고 커밋한다.
      - UPDATE, DELETE로 인한 row 락 시간 최소화
  - 지연로딩과 즉시 로딩
    - 지연로딩: 객체가 실제 사용될 때 로딩
      
      ```java
      Member member = memberRepository.findById(1); // SELECT * FROM MEMBER WHERE ID=1
      Team team = member.getTeam();  // SELECT * FROM TEAM WHERE MEMBER_ID=1
      // 지연 로딩 시 실제 사용할 때 Team객체를 조회한다. 
      ```
    - 즉시로딩: JOIN SQL로 한번에 연관된 객체까지 미리 조회
      
      ```java
      Member member = memberRepository.findById(1); // SELECT M.*, T.* FROM MEMBER M JOIN TEAM T ON M.ID = T.MEMBER_ID WHERE M.ID=1
      Team team = member.getTeam();  
      // 즉시 로딩 시 조인하여 한번에 관계된 객체까지 가져온다.
      ```

- JPA 구동 방식
  - 설정
    - 경로: /src/main/resources/META-INF/persistence.xml
    ```
    <?xml version="1.0" encoding="UTF-8"?> 
    <persistence version="2.2" 
        xmlns="http://xmlns.jcp.org/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
        xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd"> 
        <persistence-unit name="hello"> 
            <properties> 
                <!-- 필수 속성 --> 
                <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/> 
                <property name="javax.persistence.jdbc.user" value="sa"/> 
                <property name="javax.persistence.jdbc.password" value=""/> 
                <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/> 
                <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/> 

                <!-- 옵션 --> 
                <property name="hibernate.show_sql" value="true"/> 
                <property name="hibernate.format_sql" value="true"/> 
                <property name="hibernate.use_sql_comments" value="true"/> 
                <!--<property name="hibernate.hbm2ddl.auto" value="create" />--> 
            </properties> 
        </persistence-unit> 
    </persistence>
    ```
  - 구동 방식
    - Persistence -> 설정 정보 조회 (META-INF/persistence.xml or application.properties)
    - Persistence -> Entity Manager Factory 생성
    - Entity Manager Factory -> Entity Manager 생성
    - 주의할 점
      - Entity Manager Factory는 하나만 생성해서 애플리케이션 전체에서 공유
      - Entity Manager는 Thread 간에 공유하지 않는다. 사용하고 버려야 한다.
      - JPA의 모든 데이터 변경은 트랜잭션 안에서 실행한다.
    
    ```java
    @Override
    public void run(ApplicationArguments args) throws Exception {
        // TODO Auto-generated method stub

        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("hello");		
        EntityManager entityManager = entityManagerFactory.createEntityManager();
        EntityTransaction transaction = entityManager.getTransaction();
        transaction.begin();

        try {
            Member member = new Member(); member.setName("hello");
            entityManager.persist(member);

            Member findMember = entityManager.find(Member.class, member.getId());
            System.out.println(member.getId() + " " + findMember.getId());
            transaction.commit();
        } catch(Exception e) {
            transaction.rollback();
        } finally {
            entityManager.close();
            entityManagerFactory.close();
        }		
    }
    ```

#### Persistence Context

- Entity Manager Factory와 Entity Manager
  - 고객의 요청마다 Entity Manager Factory가 Entity Manager를 생성한다.
  - Entity Manager 는 Connection Pool을 사용하여 DB에 접근한다.

- 영속성 컨텍스트
  - 엔티티를 영구 저장하는 환경이라는 뜻
  - EntityManager.persist(entity);
    - DB에 저장하는 것이 아닌, Persistence Context에 저장, 관리한다는 의미
  - 영속성 컨텍스트는 논리적인 개념으로 눈에 보이지는 않는다.
  - Entity Manager를 통해 영속성 컨텍스트에 접근 가능하다.

- Entity의 생명 주기
  ```java
  // 객체를 생성한 상태 - 비영속 상태
  Member member = new Member();
  member.setId(1);
  member.setUsername("회원1");
  
  // EntityManager 가져오기
  EntityManager entityManager = entityManagerFactory.createEntityManager();
  entityManager.getTransaction().begin();
  
  // 객체를 저장한 상태 - 영속 상태
  entityManager.persist(member);
  
  // 객체를 다시 영속성 컨텍스트에서 삭제 - 준영속
  entityManager.detached(member);
  
  // 객체 삭제 (DB) - 삭제
  entityManager.remove(member);
  
  entityManager.getTransaction().commit();
  entityManager.close();
  ```
  - 비영속(new, transient)
    - 영속성 컨텍스트와 전혀 관계가 없는 새로운 상태
      
  - 영속(managed)
    - 영속성 컨텍스트에 관리되는 상태
      ```
      - new -> persist() -> managed
      - detached -> merge() -> managed
      - removed -> persist() -> managed
      - DB -> find() -> managed
      - managed -> flush() -> DB
      ```
  - 준영속(detached)
    - 영속성 컨텍스트에 저장되었다가 분리되는 상태
      ```
      managed -> detach(), clear(), close() -> detached
      ```
    
  - 삭제(removed)
    - 삭제된 상태
      ```
      - managed -> remove() -> removed
      - removed -> flush() -> DB
      ```

- 영속성 컨텍스트의 이점 1. 1차 캐시
  - 영속성 컨텍스트 안에 1차 캐시가 있으며 Key: Entity의 ID, Value: Entity로 되어 있다.

  ```java
  // 객체를 생성한 상태 - 비영속 상태
  Member member = new Member();
  member.setId(1L);
  member.setUsername("회원1");

  // EntityManager 가져오기
  EntityManager entityManager = entityManagerFactory.createEntityManager();
  entityManager.getTransaction().begin();

  // 객체를 저장한 상태 - 영속 상태
  entityManager.persist(member);
  Member findMember = entityManager.find(Member.class, Member.getId());
  Member newMember = entityManager.find(Member.class, 2L);
  ```
  - SELECT Query가 한번 발생한다.
    - 1차 캐시에 해당 ID가 없는 경우, DB에서 조회하여 1차 캐시에 저장하고 return한다.
    - findMember에 경우 영속성 컨텍스트 1차 캐시에 있기 때문에 바로 return
    - newMember에 경우 1차 캐시에 없기 때문에 DB에 조회한다.
  - 큰 이점은 없다. 
    - EntityManager는 요청에 따라 생성되며 보통 DB Transaction 단위로 구성하기 때문이다.
    - 하나의 비즈니스 로직이 끝나면 영속성 컨텍스트가 끝이나고 1차 캐시도 날라간다.
      
- 영속성 컨텍스트의 이점 2. 동일성 보장
  ```java
  Member findMember = entityManager.find(Member.class, 2L);
  Member newMember = entityManager.find(Member.class, 2L);
  System.out.println(findMember == newMember); // true
  ```
  - 1차 캐시로 반복 가능한 읽기(Repeatable Read) 등급의 트랜잭션 격리 수준을 DB가 아닌 Application 차원에서 제공
    - Non repeatable Read 문제
      - Transaction1이 Member1 조회를 2번 진행한다. 
      - 첫번째 조회한 후, Transaction2가 Member1을 수정완료 하였으면 2번째 조회한 Member1은 다른 데이터를 갖고 있다.
    - 1차 캐시로 2번 read해도 같은 Member를 return하도록 되어 있다.
      
- 영속성 컨텍스트의 이점 3. 트랜잭션을 지원하는 쓰기 지연
  ```java
  // 객체를 생성한 상태 - 비영속 상태
  Member member = new Member();
  member.setId(1L);
  member.setUsername("회원1");

  // EntityManager 가져오기
  EntityManager entityManager = entityManagerFactory.createEntityManager();
  EntityTransaction transaction = entityManager.getTransaction();
  transaction.begin();

  // 이 때 INSERT SQL을 보내지 않는다.
  entityManager.persist(member);

  transaction.commit(); // 커밋하는 순간 INSERT SQL을 보낸다.
  ```

  - 영속 컨텍스트안에 1차 캐시와 더불어 쓰기 지연 SQL 저장소가 있다.
  - entityManager.persist(member);
    - 1차 캐시에 저장과 동시에 INSERT SQL을 생성하여 쓰기 지연 SQL 저장소에 저장한다.
  - transaction.commit();
    - 쓰기 지연 SQL 저장소에 있던 QUERY가 DB에 flush되며 모든 쿼리가 수행된다.
    
- 영속성 컨텍스트의 이점 4. 변경 감지
  ```java
  
  EntityManager entityManager = entityManagerFactory.createEntityManager();
  EntityTransaction transaction = entityManager.getTransaction();
  transaction.begin();

  Member member = entityManager.find(Member.class, 1L);
  member.setUsername("updateJPA");
  
  // entityManager.update(member); 라는 쿼리는 없다.
  transaction.commit(); // 커밋하는 순간 update SQL을 보낸다.
  ``` 
  - commit, flush하는 순간
    - 1차 캐시내에 스냅샷이 저장된다.
    - 엔티티와 스냅샷을 비교하여 수정이 있는 경우 update한다.
  
- 영속성 컨텍스트의 이점 5. 지연 로딩

#### Persistence Context - flush

- 영속성 컨텍스트의 변경 내용을 데이터베이스에 반영

- flush 발생
  - 변경 감지(스냅샷)
  - 수정된 에티티 쓰기 지연 SQL 저장소에 등록
  - 쓰기 지연 SQL 저장소의 Query를 DB에 전송 (등록, 수정, 삭제 쿼리)

- 영속성 컨텍스트를 플러시하는 방법
  - entityManager.flush()
    - 직접 호출
    - 영속성 컨텍스트를 비우지 않고 영속성 컨텍스트의 변경내용을 DB의 동기화하는 것
      - 1차 캐시를 지우지는 않고, 쓰기 지연 SQL 저장소의 Query를 실행한다.    
    
  - transaction.commit()
    - 플러시 자동 호출
    - 트랜잭션이라는 작업 단위가 중요 -> 커밋 직전에만 동기화 진행
  
  - JPQL 쿼리 실행
    - 플러시 자동 호출

- 플러시 모드 옵션
  ```java
  entityManager.setFlushMode(FlushModeType.COMMIT)
  ```
  - FlushModeType.AUTO
    - 커밋이나 쿼리를 실행할 때 플러시(디폴트)
    
  - FlushModeType.COMMIT
    - 커밋할 때에만 플러시

#### Persistence Context - Detached

- 영속 -> 준영속
  - 영속 상태의 엔티티가 영속성 컨텍스트에서 분리(detached)
  - 영속성 컨텍스트가 제공하는 기능을 사용하지 못한다.

- detached 만드는 방법
  
  - entityManager.detach(entity); 
    - 특정 엔티티만 준영속 상태로 전환
    
    ```java
    Member member1 = entityManager.find(Member.class, 1L);
    entityManager.detach(member1);
    member1.setName("detachedJPA");
    ```ㅂ
    - 영속성 컨텍스트에서 관리하지 않기 때문에 setName으로 수정을 해도 update가 발생하지 않는다.
  
  - entityManager.clear(); 
    - 영속성 컨텍스트를 완전히 초기화
    
    ```java
    Member member1 = entityManager.find(Member.class, 1L);
    entityManager.clear();
    Member member2 = entityManager.find(Member.class, 1L);
    ```
    - 영속성 컨텍스트가 초기화되었기 때문에 member2를 조회할 때에도 다시 select query가 발생한다.
  
  - entityManager.close();
    - 영속성 컨텍스트 종료
  
** 출처: 자바 ORM 표준 JPA 프로그래밍 - 기본편
