---
layout: post
title: 객체 지향 원리 적용과 스프링
summary: 김영한님 - 스프링 핵심 원리_기본편
author: devhtak
date: '2021-03-08 21:41:00 +0900'
category: Spring
---

#### 들어가기

- 스프링을 사용하지 않고 객체지향 원리를 적용하여 설계한 후 문제점을 파악하고 해결하는 방식으로 진행한다.
- 이 글은 인프런에 김영한님의 스프링핵심원리 강의 및 강의자료를 바탕으로 작성하였습니다.

#### 요구사항

- 비즈니스 요구사항과 설계
  - 회원
    - 회원을 가입하고 조회할 수 있다.
    - 회원은 일반과 VIP 두가지 등급이 있다.
    - 회원 데이터는 자체 DB를 구축할 수 있고, 외부 시스템과 연동할 수 있다.
  
  - 주문과 할인 정책
    - 회원은 상품을 주문할 수 있다.
    - 회원 등급에 따라 할인 정책을 적용할 수 있다.
    - 할인 정책은 모든 VIP는 1000원을 할인해주는 고정 금액 할인을 적용해달라.
    - 할인 정책은 변경 가능성이 높다. 회사의 기본 할인 정책을 아직 정하지 못했고, 오픈 직전까지 고민을 미루고 싶다. 최악의 경우 할인을 적용하지 않을수도 있다.(미확정)
        
#### 회원 도메인 설계

- 클라이언트 -> 회원 서비스 -> 회원 저장소로 나누어 생각한다.
- 회원 서비스에는 회원 가입, 회원 조회 기능이 있다.
- 저장소에 경우 자체 DB, 외부 시스템과 연동 등 변경 가능성이 있으므로 역할과 구현을 나누어 구현하자 

#### 회원 도메인 개발

- Member.java
  ```java
  public class Member {	
      private Long id;
      private String name;
      private Grade grade;
      public Member() {}
      public Member(Long id, String name, Grade grade) {
          super();
          this.id = id;
          this.name = name;
          this.grade = grade;
      }
      public Long getId() {
         return id;
      }
      public void setId(Long id) {
          this.id = id;
      }
      public String getName() {
          return name;
      }
      public void setName(String name) {
          this.name = name;
      }
      public Grade getGrade() {
          return grade;
      }
      public void setGrade(Grade grade) {
          this.grade = grade;
      }
  }
  ```
- Grade.java
  ```java
  public enum Grade {
	    BASIC, VIP;  
  }
  ```
- MemberService.java
  ```java
  public interface MemberService {
	    void join(Member member);
      Member findByMember(Long memberId);
  }
  ```
- MemberServiceImpl.java
  ```java
  public class MemberServiceImpl implements MemberService {	
      private MemberRepository memberRepository = new InMemoryMemberRepository();

      @Override
      public void join(Member member) {
          // TODO Auto-generated method stub
          memberRepository.save(member);
      }

      @Override
      public Member findByMember(Long memberId) {
          // TODO Auto-generated method stub
          return memberRepository.findById(memberId);
      }
  }
  ```

- MemberRepository.java
  ```java
  public interface MemberRepository {	
      void save(Member member);
      Member findById(Long memberId);
  }
  ```
  
- InMemoryMemberRepository.java
  ```java
  public class InMemoryMemberRepository implements MemberRepository {	
      private static Map<Long, Member> datasource = new HashMap<>();
      @Override
      public void save(Member member) {
          // TODO Auto-generated method stub
          datasource.put(member.getId(), member);
      }
      @Override
      public Member findById(Long memberId) {
          // TODO Auto-generated method stub
          return datasource.get(memberId);
      }
  }
  ```

#### 회원 도메인 테스트

- OrderServiceTest.java
  ```java
  public class OrderServiceTest {	
      private MemberService memberService = new MemberServiceImpl();
      private OrderService orderService = new OrderServiceImpl();
      @Test
      void createOrderByVIP() {
          Member member1 = new Member(1L, "testA", Grade.VIP);
          memberService.join(member1);
          Order newOrder = orderService.createOrder(1L, "testProduct", 5000);

          assertEquals(5000, newOrder.getItemPrice());
          assertEquals(4000, newOrder.calculatedPrice());
      }

      @Test
      void createOrderBASIC() {
          Member member2 = new Member(2L, "testB", Grade.BASIC);
          memberService.join(member2);
          Order newOrder = orderService.createOrder(2L, "testProduct", 5000);

          assertEquals(5000, newOrder.getItemPrice());
          assertEquals(5000, newOrder.calculatedPrice());
      }
  }
  ```

#### 주문과 할인 도메인 설계

- 주문 생성: 클라이언트는 주문 서비스에 주문 생성을 요청한다.
- 회원 조회: 할인을 위해서 회원 등급이 필요하다. 그래서 주문 서비스는 회원 저장소에서 회원을 조회한다.
- 할인 적용: 주문 서비스는 회원 등급에 따른 할인 여부를 할인 정책에 위임한다.
- 주문 결과 반환: 주문 서비스는 할인 결과를 포함한 주문 결과를 반환한다.

- 클라이언트 -> 주문 서비스 -> (회원 조회) -> 회원 저장소,
                          -> (할인 적용) -> 할인 정책 역할
- 할인 정책 역할은 고정할인정책이나 비율할인정책 등으로 변경가능하므로 역할과 구현을 나누어 구현하자

#### 주문과 할인 도메인 개발

- Order.java
  ```java
  public class Order {	
      private Long memberId;
      private String itemName;
      private int itemPrice;
      private int discountedPrice;
      public Order() {}
      public Order(Long memberId, String itemName, int itemPrice, int discountedPrice) {
          super();
          this.memberId = memberId;
          this.itemName = itemName;
          this.itemPrice = itemPrice;
          this.discountedPrice = discountedPrice;
      }
      /*
       * @Return 할인 가격 
       */
      public int calculatedPrice() {
          return this.itemPrice - this.discountedPrice;
      }
      public Long getMemberId() {
          return memberId;
      }
      public void setMemberId(Long memberId) {
          this.memberId = memberId;
      }
      public String getItemName() {
          return itemName;
      }
      public void setItemName(String itemName) {
          this.itemName = itemName;
      }
      public int getItemPrice() {
          return itemPrice;
      }
      public void setItemPrice(int itemPrice) {
          this.itemPrice = itemPrice;
      }
      public int getDiscountedPrice() {
          return discountedPrice;
      }
      public void setDiscountedPrice(int discountedPrice) {
          this.discountedPrice = discountedPrice;
      }	
  }
  ```
  
- OrderService.java
  ```java
  public interface OrderService {	
      public Order createOrder(Long memberId, String itemName, int itemPrice);
  }
  ```
  
- OrderServiceImpl.java
  ```java
  public class OrderServiceImpl implements OrderService {	
      private MemberService memberService = new MemberServiceImpl();
      private DiscountPolicy discountPolicy = new FixedDiscountPolicy();

      @Override
      public Order createOrder(Long memberId, String itemName, int itemPrice) {
          // TODO Auto-generated method stub
          Member member = memberService.findByMember(memberId);

          int discountedPrice = discountPolicy.discount(member, itemPrice);
          return new Order(memberId, itemName, itemPrice, discountedPrice);
      }
  }
  ```
  
- DiscountPolicy.java
  ```java
  public interface DiscountPolicy {	
      public int discount(Member member, int price);
  }
  ```

- FixedDiscountPolicy.java
  ```java
  public class FixedDiscountPolicy implements DiscountPolicy{
      @Override
      public int discount(Member member, int price) {
          // TODO Auto-generated method stub
          if(member.getGrade() == Grade.VIP)
              return 1000;
          return 0;
      }
  }
  ```
  
- RateDiscountPolicy.java
  ```java
  public class RateDiscountPolicy implements DiscountPolicy {
	    private int discountPercent = 10;
      @Override
      public int discount(Member member, int price) {
          // TODO Auto-generated method stub
          if(member.getGrade() == Grade.VIP) {
              return (price * discountPercent / 100);
          }
          return 0;
      }
  }
  ```
  
#### 회원 도매인 설계및 개발의 문제점

- 이 코드의 설계상 문제점은 무엇일까?
- 다른 저장소로 변경할 때 OCP 원칙을 잘 준수할까?
- DIP를 잘 지키고 있을까?
- 의존관계가 인터페이스뿐만 아닌 구현까지 모두 의존하는 문제점이 있다.
