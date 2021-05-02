---
layout: post
title: JPA와 상속관계
summary: JPA - 자바 ORM 표준 JPA 프로그래밍 - 기본편
author: devhtak
date: '2021-05-02 21:41:00 +0900'
category: JPA
---

#### 상속관계 매핑

![image](https://user-images.githubusercontent.com/42403023/116807011-b6b38200-ab6b-11eb-8b6d-58d61f50d774.png)

- 관계형 데이터베이스는 상속관계가 없다.
- 대신 슈퍼타입, 서브타입 관계라는 모델링 기번이 객체 상속과 유사하다.
- 상속관계 매핑: 객체의 상속 구조와 DB의 슈퍼, 서브타입 관계를 매핑한다.

- 슈퍼, 서브타입 논리 모델을 실제 물리 모델로 구현하는 방법
  - 조인 전략: 각각 테이블로 변환
  - 단일 테이블 전략: 통합 테이블로 변환
  - 구현 클래스마다 테이블 전략: 서브타입 테이블로 변환
  - 주요 어노테이션
    - @Inheritance(strategy = InheritanceType.XXX)
      - JOINED
      - SINGLE_TABLE
      - TABLE_PER_CLASS
    - @DiscrminatorColumn(name="DTYPE")
    - @DiscriminatorValue("XXX")

#### 조인 전략

![image](https://user-images.githubusercontent.com/42403023/116807042-f8442d00-ab6b-11eb-95ec-02942433a5b5.png)

- 장점
  - 테이블 정규화
  - 외래 키 참조 무결성 제약조건 활용 가능
  - 저장공간 효율화

- 단점
  - 조회시 조인을 많이 사용, 성능 저하
  - 조회 쿼리가 복잡
  - 데이터 저장시 INSERT SQL 2번 호출

- 예시
  - Product -> Album, Movie
    ```java
    @Entity
    @Inheritance(strategy = InheritanceType.JOINED)
    @DiscriminatorColumn(name = "DTYPE")
    public class Product {
        @Id @GeneratedValue
        private Long id;

        private String name;

        private int price;
    }
    ```
    ```java
    @Entity
    public class Album extends Product {
        private String artist;
    }
    ```
    ```java
    @Entity
    public class Movie extends Product {
      private String director;
      private String actor;
    }
    ```
  - 테이블 생성
    ```
    Hibernate:     
    create table album (
       artist varchar(255),
        id bigint not null,
        primary key (id)
    )
    Hibernate: 
    create table album (
       artist varchar(255),
        id bigint not null,
        primary key (id)
    )
    Hibernate: 
    create table product (
       dtype varchar(31) not null,
        id bigint not null,
        name varchar(255),
        price integer not null,
        primary key (id)
    )
    ```
    
#### 단일테이블 전략

- 장점
  - 조인이 필요 없으므로 일반적으로 조회 성능이 빠르며 조회 쿼리가 단순하다
   
- 단점
  - 자식 엔티티가 매핑한 컬럼은 모두 null 허용
  - 단일 테이블에 모든 것을 저장하므로 테이블이 커질 수 있다. 상황에 따라서 조회 성능이 오히려 느려질 수 있다.

- 예시
  - Product -> Album, Movie
    ```java
    @Entity
    @Inheritance(strategy = InheritanceType.SINGLE_TABLE)
    @DiscriminatorColumn(name = "DTYPE")
    public class Product {
        @Id @GeneratedValue
        private Long id;

        private String name;

        private int price;
    }
    ```
    ```java
    @Entity
    public class Album extends Product {
        private String artist;
    }
    ```
    ```java
    @Entity
    public class Movie extends Product {
      private String director;
      private String actor;
    }
    ```
  - 결과 테이블
    ```
    Hibernate:     
    create table product (
       dtype varchar(31) not null,
        id bigint not null,
        name varchar(255),
        price integer not null,
        artist varchar(255),
        actor varchar(255),
        director varchar(255),
        primary key (id)
    )
    ```
    
#### 구현 클래스마다 테이블 전략

- 이 전략은 DB 설계자와 ORM 전문가 둘 다 추천하지 않는다.

- 장점
  - 서브 타입을 명확하게 구분해서 처리할 때 효과적
  - not null 제약조건 사용 가능

- 단점
  - 여러 자식 테이블을 함께 조회할 때 성능이 느리다.(UNION SQL 필요)
  - 자식 테이블을 통합해서 쿼리하기 어렵다.

#### @MappedSuperclass

- 공통 매핑 정보(id, name 등)가 필요할 때 사용한다.

![image](https://user-images.githubusercontent.com/42403023/116807940-00eb3200-ab71-11eb-8413-4dd1beb75ac2.png)

- 특징
  - 상속관계 매핑되지 않는다.
  - 엔티티가 아니다. 테이블과 매핑되지 않는다.
  - 부모 클래스를 상속받는 자식 클래스에 매핑 정보만 제공
  - 조회, 검색 불가
    ```
    // BaseEntity baseEntity = entityManager.find(BaseEntity.class, entity.getId());
    ```
  - 직접 생성해서 사용할 일이 없으므로 추상 클래스 권장
  - 테이블과 관계 없고, 단순히 엔티티가 공통으로 사용하는 매핑 정보를 모으는 역할
  - 주로 등록일, 수정일, 등록자, 수정자 같은 전체 엔티티에서 공통으로 적용하는 정보를 모을 때 사용
  - 참고: @Entity 클래스는 엔티티나 @MappedSuperclass로 지정한 클래스만 상속 가능

- 예시
  - 모든 엔티티에 생성자, 생성 일시, 수정자, 수정 일시를 추가하자
  
    ```java
    @Getter @Setter
    @MappedSuperclass
    public class BaseEntity {
        private String createdBy;
        private LocalDateTime createdDate;
        private String modifiedBy;
        private LocalDateTime modifiedDate;
    }
    ```
    ```java
    public class Member extends BaseEntity { ... }
    ```
    ```java
    public class Orders extends BaseEntity { ... }
    ```
  - 생성 쿼리
    ```
    Hibernate: 
    create table member (
       member_id bigint not null,
        created_by varchar(255),
        created_date timestamp,
        modified_by varchar(255),
        modified_date timestamp,
        city varchar(255),
        name varchar(255),
        street varchar(255),
        zipcode varchar(255),
        locker_id bigint,
        primary key (member_id)
    )
    Hibernate:     
    create table orders (
       order_id bigint not null,
        created_by varchar(255),
        created_date timestamp,
        modified_by varchar(255),
        modified_date timestamp,
        order_date timestamp,
        order_status varchar(255),
        delivery_id bigint,
        member_id bigint,
        primary key (order_id)
    )
    ```
