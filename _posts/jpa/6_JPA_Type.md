---
layout: post
title: JPA. 값 타입
summary: JPA - 자바 ORM 표준 JPA 프로그래밍 - 기본편
author: devhtak
date: '2021-05-06 21:41:00 +0900'
category: JPA
---

#### 값 타입

- JPA의 데이터 타입 분류
  - 엔티티 타입
    - @Entity로 정의하는 객체
    - 데이터가 변해도 식별자로 지속해서 추적 가능
    - 예) 회원 엔티티의 키나 나이 값을 변경해도 식별자로 인식 가능

  - 값 타입
    - int, Integer, String 처럼 단순히 값으로 사용하는 자바 기본 타입이나 객체
    - 식별자가 없고 값만 있으므로 변경시 추적 불가
    - 예) 숫자 100을 200으로 변경하면 완전히 다른 값으로 대체

- 값 타입 분류
  - 기본 값 타입
    - 자바 기본 타입(int, long, double)
    - 래퍼 클래스(Integer, Long)
    - String
  - 임제디드 타입(Embedded type, 복합 값 타입)
  - 컬렉션 값 타입(collection value type)

- 기본 값 타입
  - 예): String name, int age
  - 생명주기를 엔티티의 의존
  - 예) 회원을 삭제하면 이름, 나이 필드도 함께 삭제
  - 값 타입은 공유하면X 
  - 예) 회원 이름 변경시 다른 회원의 이름도 함께 변경되면 안됨
  - 참고: 자바의 기본 타입은 절대 공유X
    - int, double 같은 기본 타입(primitive type)은 절대 공유X 
    - 기본 타입은 항상 값을 복사함
    - Integer같은 래퍼 클래스나 String 같은 특수한 클래스는 공유 가능한 객체이지만 변경X

#### Embedded Type(복합 값 타입)

- Embedded Type
  - 새로운 값 타입을 직접 정의할 수 있음
  - JPA는 임베디드 타입(embedded type)이라 함
  - 주로 기본 값 타입을 모아서 만들어서 복합 값 타입이라고도 함
    - 회원 엔티티는 이름, 근무 기간, 집 주소를 가진다.
    - 회원에 주소, 기간등 재사용이 가능하고 계산 등 메소드가 필요할 때 객체로 묶어서 사용할 수 있다.
  - int, String과 같은 값 타입

- Embedded Type 사용법
  - Address.java
    ```java
    @Embeddable
    public class Address {

      @Column(name = "ADDRESS_CITY")
      private String city;
      @Column(name = "ADDRESS_STREET")
      private String street;
      @Column(name = "ADDRESS_ZIPCODE")
      private String zipcode;
    }
  ```
- Member.java
  ```java
  @Entity
  @Getter @Setter
  public class Member {
      @Id @GeneratedValue
      @Column(name = "MEMBER_ID")
      private Long id;

      @Embedded
      private Address address;
  }
  ```
  - @Embeddable: 값 타입을 정의하는 곳에 표시
  - @Embedded: 값 타입을 사용하는 곳에 표신
  - 마찬가지로 @Column을 통해 컬럼명을 따로 줄 수 있다.
  - 기본생성자 필수!

- 사용 장점
  - 재사용
  - 높은 응집도
  - Period.isWork()처럼 해당 값 타입만 사용하는 의미 있는 메소드를 만들 수 있음
  - 임베디드 타입을 포함한 모든 값 타입은, 값 타입을 소유한 엔티티에 생명주기를 의존함

- 임베디드 타입과 테이블 매핑
  - 임베디드 타입은 엔티티의 값일 뿐이다. 
  - 임베디드 타입을 사용하기 전과 후에 매핑하는 테이블은 같다. 
  - 객체와 테이블을 아주 세밀하게(find-grained) 매핑하는 것이 가능
  - 잘 설계한 ORM 애플리케이션은 매핑한 테이블의 수보다 클래스의 수가 더 많음

- @AttributeOverride: 속성 재정의
  ```java
  @Embedded
  @AttributeOverrides({
      @AttributeOverride(name="city", column=@Column(name="HOME_CITY")),
      @AttributeOverride(name="street", column=@Column(name="HOME_STREET")),
      @AttributeOverride(name="zipcode", column=@Column(name="HOME_ZIPCODE")),
  })
  private Address homeAddress;

  @Embedded
  @AttributeOverrides({
      @AttributeOverride(name="city", column=@Column(name="WORK_CITY")),
      @AttributeOverride(name="street", column=@Column(name="WORK_STREET")),
      @AttributeOverride(name="zipcode", column=@Column(name="WORK_ZIPCODE")),
  })
  private Address workAddress;
  ```
  - 한 엔티티에서 같은 값 타입을 사용하면? 컬럼명이 중복된다.
  - @AttributeOverrides, @AttributeOverride를 사용하여 컬럼명 속성을 재정의

- 임베디드 타입과 null
  - 임베디드 타입의 값이 null이면 매핑한 컬럼 값은 모두 null

#### 값 타입과 불변 객체

```
값 타입은 복잡한 객체 세상을 조금이라도 단순화하려고만든 개념이다.
따라서 값 타입은 단순하고 안전하게 다룰 수 있어야 한다.
```

- 값 타입 공유 참조
  - 임베디드 타입 같은 값 타입을 여러 엔티티에서 공유하면 위험함
  - 부작용(side effect) 발생
    - Member1과 Member2에 같은 Address를 세팅했을 때, Member1에 Address만 변경해도 Member2에 Address 변경까지 발생할 수 있다.)
      ```java
      Address address = new Address();
      
      Member member1 = new Member();
      member1.setAddress(address);
      
      Member member2 = new Member();
      member2.setAddress(address);
      
      member1.getAddress().setCity("newCity"); // member2도 변경된다.
      ```
      
- 객체 타입의 한계
  - 항상 값을 복사해서 사용하면 공유 참조로 인해 발생하는 부작용을 피할 수 있다.
  - 문제는 임베디드 타입처럼 직접 정의한 값 타입은 자바의 기본 타입이 아니라 객체 타입이다. 
  - 자바 기본 타입에 값을 대입하면 값을 복사한다. 
  - 객체 타입은 참조 값을 직접 대입하는 것을 막을 방법이 없다. 
  - 객체의 공유 참조는 피할 수 없다.

- 불변 객체
  - 객체 타입을 수정할 수 없게 만들면 부작용을 원천 차단
  - 값 타입은 불변 객체(immutable object)로 설계해야함
  - 불변 객체: 생성 시점 이후 절대 값을 변경할 수 없는 객체
  - 생성자로만 값을 설정하고 수정자(Setter)를 만들지 않으면 됨
  - 참고: Integer, String은 자바가 제공하는 대표적인 불변 객체
    ```java
    @Embeddable
    public class Address {
        @Column(name = "ADDRESS_CITY")
        private String city;
        @Column(name = "ADDRESS_STREET")
        private String street;
        @Column(name = "ADDRESS_ZIPCODE")
        private String zipcode;
        
        public String getCity() {
            return city;
        }
        private void setCity(String city) {
            this.city = city;
        }
        public String getStreet() {
            return street;
        }
        private void setStreet(String street) {
            this.street = street;
        }
        public String getZipcode() {
            return zipcode;
        }
        private void setZipcode(String zipcode) {
            this.zipcode = zipcode;
        }	
    }
    ```
    - setter를 private으로 만들어 외부에서 값을 변경하지 못하도록 하였다.

#### 값 타입의 비교

- 값 타입의 비교
  - 값 타입: 인스턴스가 달라도 그 안에 값이 같으면 같은 것으로 봐야 함
    ```java
    int a = 10; 
    int b = 10;
    Address address1 = new Address(“서울시”) 
    Address address2 = new Address(“서울시”)
    
    System.out.println(a == b); // true
    System.out.println(address1 == address2); // false
    System.out.println(address1.equals(address2)); // false
    ```
    
- 동일성과 동등성
  - 동일성(identity) 비교: 인스턴스의 참조 값을 비교, == 사용
  - 동등성(equivalence) 비교: 인스턴스의 값을 비교, equals() 사용
  - 값 타입은 a.equals(b)를 사용해서 동등성 비교를 해야 함
  - 값 타입의 equals() 메소드를 적절하게 재정의(주로 모든 필드 사용)

- equals와 hashCode 재정의하라!
  ```java
  @Override
  public boolean equals(Object o) {
      if(this == o) return true;
      if(o == null || getClass() != o.getClass()) return false;
      Address address = (Address)o;
      return Object.equals(city, address.getCity()) & Object.equals(street, address.getStreet()) && Object.equals(zipCode, address.getZipCode());
  }

  @Override
  public int hashCode() { return Objects.hash(city, street, zipCode);}
  ```
  
#### 값 타입 컬렉션

- 값 타입 컬렉션
  - 값 타입을 하나 이상 저장할 때 사용
  - @ElementCollection, @CollectionTable 사용
  - 데이터베이스는 컬렉션을 같은 테이블에 저장할 수 없다. 
  - 컬렉션을 저장하기 위한 별도의 테이블이 필요함
  - 참고: 값 타입 컬렉션은 영속성 전에(Cascade) + 고아 객체 제거 기능을 필수로 가진다고 볼 수 있다

- 값 타입 컬렉션의 제약사항
  - 값 타입은 엔티티와 다르게 식별자 개념이 없다. 
  - 값은 변경하면 추적이 어렵다. 
  - 값 타입 컬렉션에 변경 사항이 발생하면, 주인 엔티티와 연관된 모든 데이터를 삭제하고, 값 타입 컬렉션에 있는 현재 값을 모두 다시 저장한다. 
  - 값 타입 컬렉션을 매핑하는 테이블은 모든 컬럼을 묶어서 기본키를 구성해야 함: null 입력X, 중복 저장X

- 값 타입 컬렉션 대안
  - 실무에서는 상황에 따라 값 타입 컬렉션 대신에 일대다 관계를 고려
  - 일대다 관계를 위한 엔티티를 만들고, 여기에서 값 타입을 사용
  - 영속성 전이(Cascade) + 고아 객체 제거를 사용해서 값 타입 컬렉션 처럼 사용
  - EX) AddressEntity

- 예제
  ```java
  @Entity
  public class Member extends BaseEntity {
      @Id @GeneratedValue
      @Column(name = "MEMBER_ID")
      private Long id;

      @OneToOne(fetch = FetchType.LAZY)
      @JoinColumn(name = "LOCKER_ID")
      private Locker locker;

      private String name;

      @Embedded
      private Address address;

      @ElementCollection
      @CollectionTable(name = "FAVORITE_FOOD", joinColumns = 
          @JoinColumn(name="MEMBER_ID"))
      @Column(name="FOOD_NAME")
      private Set<String> favoriteFoods = new HashSet<>();

      @ElementCollection
      @CollectionTable(name = "Address", joinColumns = 
          @JoinColumn(name="MEMBER_ID"))
      private List<Address> addressHistory = new ArrayList<>();	
  }
  ```
  - 테스트 생성 확인
    ```
    create table address (
        member_id bigint not null,
        address_city varchar(255),
        address_street varchar(255),
        address_zipcode varchar(255)
    )
    create table favorite_food (
        member_id bigint not null,
        food_name varchar(255)
    )
    create table member (
        member_id bigint not null,
        created_by varchar(255),
        created_date timestamp,
        modified_by varchar(255),
        modified_date timestamp,
        address_city varchar(255),
        address_street varchar(255),
        address_zipcode varchar(255),
        name varchar(255),
        locker_id bigint,
        primary key (member_id)
    )
    ```
    - CollectionTable로 생성한 address, favorite_food가 생성되었다.
    - favorite_food에 경우 List<String>에 저장되는 값이 @Column(name= "food_name")으로 들어가도록 되었다.

#### 출처

김영한님의 자바 ORM 표준 JPA 프로그래밍 - 기본편 

  
