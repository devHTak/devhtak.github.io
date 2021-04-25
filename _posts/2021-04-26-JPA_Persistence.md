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
    - DAO에서 EntityManager의 Persiste 를 실행한다.
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



** 출처: 자바 ORM 표준 JPA 프로그래밍 - 기본편
