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

#### 기본키 매핑

- 기본키 매핑 어노테이션
  - @Id
  - @GeneratedValue

- 

- 기본키 매핑 방법
  - 직접 할당: @Id 만 사용
  - 자동 생성: @GeneratedValue
    - 속성: stragegy
      - IDENTITY: 데이터베이스에 위임, MYSQL
        - @GeneratedValue(strategy = GenerationType.IDENTITY)
        - 기본 키 생성을 데이터베이스에 위임
        - MySQL, PostgreSQL, SQL Server, DB2에서 사용
          - 예: MySQL의 AUTO_INCREMENT)
        - JPA는 보통 트랜잭션 커밋 시점에 INSERT SQL 실행하지만 AUTO_INCREMENT는 데이터베이스에 INSERT SQL을 실행한 이후에 ID 값을 알 수 있다
        - IDENTITY 전략은 em.persist() 시점에 즉시 INSERT SQL 실행하고 DB에서 식별자 조회하여 Persistence Context가 관리된다.

      - SEQUENCE: 데이터베이스 시퀀스 오브젝트 사용, Oracle
        - DB 시퀀스는 유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트(예, 오라클 시퀀스)
        - SEQUENCE 또한 DB에서 조회가 가능하다. 저장할 때 해당 SEQUENCE에서 next_value를 조회하여 세팅한다. INSERT는 트랜잭션이 끝날 때 실행한다.
        - 오라클, PostgreSQL, DB2, H2 데이터베이스 사용
        - @SequenceGenerator를 통해서 테이블 별 sequence 를 매핑할 수 있다.
          ```java
          @Entity 
          @SequenceGenerator( 
              name = “MEMBER_SEQ_GENERATOR", 
              sequenceName = “MEMBER_SEQ", //매핑할 데이터베이스 시퀀스 이름
              initialValue = 1, allocationSize = 1) 
          public class Member { 
              @Id 
              @GeneratedValue(strategy = GenerationType.SEQUENCE, 
                  generator = "MEMBER_SEQ_GENERATOR") 
              private Long id; 
          }
          ```
          
          |속성|설명|기본값|
          |---|---|---|
          |name|식별자 생성기 이름|필수|
          |sequenceName|데이터베이스에 등록되어 있는 시퀀스 이름|hibernate_sequence|
          |initialValue|DDL 생성 시에만 사용됨, 시퀀스 DDL을 생성할 때 처음 1 시작하는 수를 지정한다.|1|
          |allocationSize|시퀀스 한 번 호출에 증가하는 수(성능 최적화에 사용됨 데이터베이스 시퀀스 값이 하나씩 증가하도록 설정되어 있으면 이 값을 반드시 1로 설정해야 한다|50|
          |catalog, schema|데이터베이스 catalog, schema 이름|-|
          
      - TABLE: 키 생성용 테이블 사용, 모든 DB에 사용
        - 테이블을 하나 만들어서 거기에서 뽑아 사용한다.
        - 장점으로는 모든 데이터베이스에 적용이 가능하나 단점은 성능이 떨어진다.
        - @TableGenerator 필요
          ```java
          @Entity 
          @TableGenerator( 
              name = "MEMBER_SEQ_GENERATOR", 
              table = "MY_SEQUENCES", 
              pkColumnValue = “MEMBER_SEQ", allocationSize = 1) 
          public class Member { 
              @Id 
              @GeneratedValue(strategy = GenerationType.TABLE, 
                  generator = "MEMBER_SEQ_GENERATOR") 
              private Long id; 
          }
          ```
          
          |속성|설명|기본값|
          |---|---|---|
          |name|식별자 생성기 이름|필수|
          |table|키생성 테이블명|hibernate_sequences|
          |pkColumnName|시퀀스 컬럼명|sequence_name|
          |valueColumnName|시퀀스 값 컬럼명|next_val|
          |pkColumnValue|키로 사용할 값 이름|엔티티 이름|
          |initialValue|초기 값, 마지막으로 생성된 값이 기준이다.|0|
          |allocationSize|시퀀스 한 번 호출에 증가하는 수(성능 최적화에 사용됨)|50|
          |catalog, schema|데이터베이스 catalog, schema 이름|-|
          |uniqueConstraints(DDL)|유니크 제약 조건을 지정할 수 있다.|-|
          
      - AUTO: 방언에 따라 자동 지정, Identity, Sequence, Table 중 DB Vendor에 따라 선택된다.
      
- 권장하는 식별자 전략
  - 기본 키 제약 조건: null 아님, 유일, 변하면 안된다.
  - 미래까지 이 조건을 만족하는 자연키는 찾기 어렵다. 대체키를 사용하자.
  - 예를들어 주민등록번호도 기본 키로 적절하지 않다.
  - 권장: Long 형(10억 넘어도 동작하도록) + 대체키(sequence) + 키 생성전략 사용

