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
      private final MemberRepository memberRepository = new InMemoryMemberRepository();

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
      private final MemberService memberService = new MemberServiceImpl();
      private final DiscountPolicy discountPolicy = new FixedDiscountPolicy();

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
  
#### 테스트

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

- RateDiscountPolicyTest.java
  ```java
  public class RateDiscountPolicyTest {
      DiscountPolicy discountPolicy = new RateDiscountPolicy();
	
      @Test
      @DisplayName("VIP 10% 할인 적용 확인")
      void discountTestVIP() {
          // GIVEN
          Member member = new Member(1L, "test", Grade.VIP);
          // WHEN
          int discount = discountPolicy.discount(member, 90001);
	  //THEN
	  assertEquals(9000, discount);
      }
      @Test
      @DisplayName("BASIC은 0 할인 적용 확인")
      void discountTestBasic() {
          // GIVEN
	  Member member = new Member(1L, "test", Grade.BASIC);	
	  // WHEN
	  int discount = discountPolicy.discount(member, 10000);	
	  // THEN
	  assertEquals(0, discount);
      }
  }
  ```

#### 도매인 설계및 개발의 문제점

- 이 코드의 설계상 문제점은 무엇일까?
- 다른 저장소로 변경할 때 OCP 원칙을 잘 준수할까?
- DIP를 잘 지키고 있을까?
- 의존관계가 인터페이스뿐만 아닌 구현까지 모두 의존하는 문제점이 있다.

- 즉, DiscountPolicy를 FixedDiscountPolicy에서 RateDiscountPolicy로 변경하고자 한다.
  ```java
  // private final DiscountPolicy discountPolicy = new FixedDiscountPolicy();
  private final DiscountPolicy discountPolicy = new RateDiscountPolicy();
  ```
  - 이처럼 OrderService에서 소스 변경이 필요하다.
  - 즉, 클래스 다이어그램으로 보면 OrderService가 DiscountPolicy 인터페이스만 의존하는 것이 아닌, 구현체(클래스)도 의존하는 것 -> DIP 위반! 
  - 소스 변경을 한다. -> OCP 위반!

#### DIP, OCP 해결 방안 -> 누군가가 클라이언트인 OrderServiceImpl에 DiscountPolicy의 구현 객체를 대신 생성하고 주입해주어야 한다.

#### 관심사의 분리

- 애플리케이션을 하나의 공연이라고 생각해보자.
  - 각각의 인터페이스를 배역이고, 구현체를 배우라고 생각해보자.
  - 배역에 맞는 배우를 선택하는 기획자의 책임은 다른 역할이다.
  - 여기서 관심사를 분리하자는 것은 기획자의 책임을 확실히 분리하는 것이다.

- AppConfig의 등장
  - 애플리케이션의 전체 동작 방식을 구성(config)하기 위해, 구현 객체를 생성하고, 연결하는 책임을 가지는 별도의 설정 클래스를 만들자.
  - AppConfig는 애플리케이션의 실제 동작에 필요한 구현 객체를 생성한다.
    - MemberServiceImpl
    - MemoryMemberRepository
    - OrderServiceImpl
    - FixedDiscountPolicy
  - AppConfig는 생성한 객체 인스턴스의 참조를 생성자를 통해서 주입(연결)해준다.
    - MemberServiceImpl -> MemoryMemberRepository
    - OrderServiceImpl -> MemoryMemberRepository, FixedDiscountPolixy
    
  - 생성자를 메소드로 분리하였다.(memberRepository(), discountPolicy())
    - 이후 발생할 코드 중복 제거 및 역할에 따른 구현이 확실하게 보이도록 하였다.
    
  ```java
  public class AppConfig {
      // 생성자 주입
      public MemberService memberService() {
          return new MemberServiceImpl(memberRepository());
      }
      public OrderService orderService() {
          return new OrderServiceImpl(this.memberService(), discountPolicy());
      }
      private MemberRepository memberRepository() {
          return new InMemoryMemberRepository();
      }	
      private DiscountPolicy discountPolicy() {
	  return new FixedDiscountPolicy();
      }
  }
  ```

- MemberServiceImpl과 OrderServiceImpl
  - 단지 인터페이스만 의존한다.
  - 생성자를 통해 어떤 구현 객체가 주입되는지는 알 수 없고, AppConfig에서 결정된다.
  - 의존 관계에 대한 고민은 외부에 맡기고 실행에만 집중하면 된다.
  
  ```java
  public class MemberServiceImpl implements MemberService {
      private final MemberRepository memberRepository;
      public MemberServiceImpl(MemberRepository memberRepository) {
          super();
	  this.memberRepository = memberRepository;
      }
      // ...
  }
  ```
  
  ```java
  public class OrderServiceImpl implements OrderService {
      private final MemberService memberService;
      private final DiscountPolicy discountPolicy;
      public OrderServiceImpl(MemberService memberService, DiscountPolicy discountPolicy) {
          super();
	  this.memberService = memberService;
	  this.discountPolicy = discountPolicy;
      }
      
      // ...
  }
  ```

#### 좋은 객체 지향 설계의 5가지 원칙의 적용

- 여기서는 SRP, DIP, OCP 적용

- SRP 단일 책임 원칙
  - 한 클래스는 하나의 책임만 가져야 한다.
  - 클라이언트 객체는 직접 구현 객체를 생성하고, 연결하고, 실행하는 다양한 책임을 가지고 있다.
  - SRP 단일 책임 원칙을 따라 관심사를 분리하였다.(Config)
  - 구현 객체를 생성하고, 연결하는 책임은 AppConfig가 담당한다.
  - 클라이언트 객체는 실행하는 책임만 담당

- DIP 의존관계 역전 원칙
  - "추상화에 의존해야지, 구체화에 의존하면 완된다."
  - 의존성 주입은 이 원칙을 따르는 방법 중 하나다.
  - 기존 코드에서는 추상화 인터페이스 뿐만 아니라 구체화 구현 클래스에도 함께 의존했다.
    ```java
    private final DiscountPolicy discountPolicy = new FixedDiscountPolicy();
    ```
  - AppConfig로 책임을 분리하며 클라이언트 코드가 DiscountPolicy 추상화 인터페이스에만 의존하도록 코드를 변경했다.
  - 하지만 클라이언트 코드는 인터페이스만으로는 아무것도 실행할 수 없다.
  - AppConfig가 FixedDiscountPolicy 객체 인스턴스를 클라이언트 코드 대신 생성하여 클라이언트 코드에 의존관계를 주입했다.

- OCP 
  - 소프트웨어 요소는 확장에는 열려 있으나 변경에는 닫혀있어야 한다.
  - 다형성 사용하고 클라이언트가 DIP를 지킨다.
  - 애플리케이션을 사용 영역과 구성 영역으로 나누었다. (OrderServiceImpl + AppConfig)
  - AppConfig가 의존관계를 FixedDiscountPolicy -> RateDiscountPolicy로 변경하여도 클라이언트 코드에 주입하므로 클라이언트 코드는 변경하지 않아도 된다.
  - 소프트웨어 요소를 새롭게 확장해도 사용 영역의 변경은 닫혀있다.

- 출처: 김영한님의 Spring Core 강의 및 강의자료 정리
