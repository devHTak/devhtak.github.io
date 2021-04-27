---
layout: post
title: JPA와 영속성 관리
summary: JPA - 자바 ORM 표준 JPA 프로그래밍 - 기본편
author: devhtak
date: '2021-04-25 21:41:00 +0900'
category: JPA
---

#### Entity Mapping

- 객체와 테이블 매핑
  - @Entity, @Table

- 필드와 컬럼 매핑
  - @Column

- 기본 키 매핑
  - @Id

- 연관관계 매핑
  - @ManyToOne
  - @OneToMany

#### 객체와 테이블 매핑

```java
@Entity
@Getter @Setter
@Table(name = "TB_MEMBER")
public class Member {
    @Id @GeneratedValue
    private Long id;

    private String name;
}
```
- @Entity
  - @Entity가 붙은 클래스는 JPA가 관리, 엔티티라 한다.
  - JPA를 사용해서 테이블과 매핑할 클래스는 @Entity 필수
  - 주의점
    - 기본 생성자 필수(파라미터가 없는 public 또는 protected 생성자)
    - final 클래스, enum, interface, inner 클래스 사용하면 안된다.
    - 저장할 필드에 final 사용하면 안된다.
  - 속성: name
    - JPA에서 사용할 엔티티 이름을 지정
    - 기본값: 클래스 이름 그대로 사용
    - 같은 클래스 이름이 없으면 기본값을 사용

- @Table
  - 엔티티와 매핑할 테이블 지정
  - 속성. name: 매핑할 테이블 이름, 기본으로는 엔티티 이름을 사용한다.
  - 속성. catalog: 데이터베이스 catalog 매핑
  - 속성. schema: 데이터베이스 스키마 매핑
  - 속성. uniqueConstraints(DDL): DDL 생성 시 유니크 제약 조건 생성

- 데이터베이스 스키마 자동 생성
  - DDL을 애플리케이션 실행 시점에 자동 생성할 수 있다. 
  - 테이블 중심 -> 객체 중심
  - 데이터베이스에 맞는 적절한 DDL 생성 
    - VARCHAR2, NUMBER 등 DB에 의존적인 keyword를 사용하여 생성한다.
  - 이렇게 생성된 DDL은 개발 장비에서만 사용
  - 생성된 DDL은 운영서버에는 사용하지 않거나, 적절히 다듬은 후 사용
  
  - spring.jpa.hibernate.ddl-auto 속성
    - create: 애플리케이션을 실행할 때 table을 새로 생성한다.
    - create-drop: create와 같으나 종료 시점에서 테이블을 drop한다.
    - update: 변경부분만 반영한다.
    - validate: 엔티티와 테이블이 정상 매핑되었는지 확인
    - none: 사용하지 않음
    - 주의점
      - 운영환경에서는 절대 create, create-drop, update 사용하면 안된다.
      - 개발 초기단계는 create 또는 update
      - 테스트 서버는 update 또는 validate
      - 스테이징과 운영서버는 valdiate 또는 none

- DDL 생성 기능
  - 컬럼에 제약 조건 추가
    ```java
    @Column(nullable=false, length=10)
    ```
  - 테이블에 유니크 제약 조건 추가
    ```java
    @Table(uniqueConstraints={
        @UniqueConstraint(name="NAME_AGE_UNIQUE", columnNames={"NAME", "AGE"})
    })
    ```
  - DDL 생성 기능은 DDL을 자동 생성할 때만 사용되고 JPA의 실행 로직에는 영향을 주지 않는다.
  
#### 필드와 컬럼 매핑

```java
@Entity 
public class Member { 
    @Id 
    private Long id; 

    @Column(name = "name") 
    private String username; 

    private Integer age; 

    @Enumerated(EnumType.STRING) 
    private RoleType roleType; 

    @Temporal(TemporalType.TIMESTAMP) 
    private Date createdDate; 

    @Temporal(TemporalType.TIMESTAMP) 
    private Date lastModifiedDate; 

    @Lob 
    private String description; 
}
```

- @Column
  
  |속성|설명|기본값|
  |---|---|---|
  |name|필드와 매핑할 테이블의 컬럼 이름|객체의 필드 이름|
  |insertable, updatable|등록, 변경 가능 여부|TRUE|
  |nullable(DDL)|null 값의 허용 여부를 설정한다. false로 설정하면 DDL 생성 시에 not null 제약조건이 붙는다.|
  |unique(DDL)|@Table의 uniqueConstraints와 같지만 한 컬럼에 간단히 유니크 제약조건을 걸 때 사용한다.
  |columnDefinition(DDL)|데이터베이스 컬럼 정보를 직접 줄 수 있다. ex) varchar(100) default ‘EMPTY'|필드의 자바 타입과 방언 정보를 사용해서 적절한 컬럼 타입|
  |length(DDL)|문자 길이 제약조건, String 타입에만 사용한다.|255|
  |precision, scale(DDL)|BigDecimal 타입에서 사용한다(BigInteger도 사용할 수 있다).precision은 소수점을 포함한 전체 자 릿수를, scale은 소수의 자릿수다. 참고로 double, float 타입에는 적용되지 않는다. 아주 큰 숫자나 정밀한 소수를 다루어야 할 때만 사용한다.|precision=19, scale=2|

- @Enumerated
  - enum 타입 매핑할 때 사용
  - Enum에는 @Enumerable 사용
  - EnumType.STRING을 주자 (ORDINAL 사용하면 안된다.
  - 속성. value
    - EnumType.ORDINAL: enum 순서를 데이터베이스에 저장
    - EnumType.STRING: enum 이름을 데이터베이스에 저장 
    - 기본값: EnumType.ORDINAL

- @Temporal
  - 날짜 타입(java.util.Date, java.util.Calendar)을 매핑할 때 사용
  - 참고: LocalDate, LocalDateTime을 사용할 때는 생략 가능(최신 하이버네이트 지원) 
  - 속성. value
    - TemporalType.DATE: 날짜, 데이터베이스 date 타입과 매핑 (예: 2013–10–11) 
    - TemporalType.TIME: 시간, 데이터베이스 time 타입과 매핑 (예: 11:11:11) 
    - TemporalType.TIMESTAMP: 날짜와 시간, 데이터베이스, timestamp 타입과 매핑(예: 2013–10–11 11:11:11) 

- @Lob
  - 데이터베이스 BLOB, CLOB 타입과 매핑 
  - @Lob에는 지정할 수 있는 속성이 없다. 
  - 매핑하는 필드 타입이 문자면 CLOB 매핑, 나머지는 BLOB 매핑
    - CLOB: String, char[], java.sql.CLOB 
    - BLOB: byte[], java.sql. BLOB 

- @Transient
  - 필드 매핑X 
  - 데이터베이스에 저장X, 조회X 
  - 주로 메모리상에서만 임시로 어떤 값을 보관하고 싶을 때 사용
  
  ```java
  @Transient
  private Integer temp; 
  ```
