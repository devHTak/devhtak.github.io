---
layout: post
title: Spring DI, IoC
summary: Spring Basic
author: devhtak
date: '2021-05-13 21:41:00 +0900'
category: Spring
---

#### IoC (Inversion Of Control)

- IoC는 제어의 역전이란 단어로 GoF의 디자인 패턴에서 언급되었다.
- 예시
  - 기존 방식
    ```java
    public class A {
        private B b;

        public Person() {
            this.b = new B();
        }
    }
    ```
  - IoC 적용된 방식
    ```java
    public class A {

        @Autowired
        private B b;
    }
    ```
  - 기존 방식은 개발자가 생성자를 통해 객체를 생성했다.
  - IoC가 적용된 방식을 보면 컨테이너에서 관리하는 Bean을 주입시키므로 사용할 수 있도록 했다.
- IoC의 이점
  - 역할과 책임을 분리할 수 있다.
  - 즉, 빈을 생성하고 주입하는 역할과 책임을 컨테이너에게 위임하므로써 변경에 유연한 코드 구조를 가져갈 수 있다.
    - 예시. 주문할 때 기존에는 고정된 퍼센트로 할인을 진행했다가 구매 가격에 따른 할인 정책으로 변경하였다.
      ```java
      public class Orders {
          // private Discount disount = new FixedDiscount();
          private Discount disount = new DynamicDiscount(); 
      }
      ```
    - 할인 정책이 변경될 때마다 Discount 가 참조하는 객체가 변경된다.
    - 하지만 IoC를 활용하여 DynamicDiscount를 주요 빈으로 등록하면 해당 소스 수정없이 진행할 수 있다.
  - 수 많은 객체들을 편리하게 관리할 수 있다.
  
  #### Spring 의 DI
  
  - 토비의 스프링
    ```
    IoC가 매우 느슨하게 정의돼서 폭넓게 사용되는 용어라는 점이다. 
    때문에 스프링을 IoC 컨테이너라고만 해서는 스프링이 제공히는 기능의 특정을 명확하게 설명하지 못한다. 
    … 생략 … 
    몇몇 사람의 제안으로 스프링이 제공하는 IoC 방식을 핵심을 짚어주는 의존관계 주입 DependencyIniection이라는，
    좀 더 의도가 명확히 드러나는 이름을 사용하 기 시작했다.
    … 생략 … 
    스프링이 여타 프레임워크와 차별화돼서 제공해주는 기능은 의존관계 주입이라는 새로운 용어를 사용할 때 분명하게 드러난다. 
    …생략…
    DI는 오브젝트 레퍼런스를 외부로부터 제공(주입)받고 이를 통해 여타 오브젝트와 다이내믹하게 의존관계가 만들어지는 것이 핵심
    ```

- 의존성
  - 의존한다는 말은, 서로 다른 객체간에 레퍼런스 참조가 되어 있다는 의미
  - 이때, A->B에 의존관계에 있을 때 B객체에 변경사항이 A 객체에게 영향을 끼치는 구조
  - 예시
    ```
    public class Orders {
        private Discount discount = new FixedDiscount();
        private int totalPrice;
        
        // 여기서 discount의 discount메서드가 변경되었다면 Orders에 getTotalPrice() 함수도 영향을 받는다.
        public int getTotalPrice() {
            return discount.discount(totalPrice); 
        }
    }
    ```

- 주입
  - 외부로부터 객체의 주소(레퍼런스) 값을 전달받게 되어 객체가 참조되어지는 방식

- 의존성 주입
  - 의존관계에 있는 객체들이 있을 때 외부(스프링 컨테이너)에서 객체에 레퍼런스를 전달하여 사용하고자 하는 객체에서 코드를 사용 가능하게 한다.
  - 토비의 스프링에서 말하는 3가지 조건
    - 클래스 모델이나 코드에는 런타임 시점의 의존관계가 드러나지 않는다. 그러기 위해서는 인터페이스에만 의존하고 있어야 한다.
    - 런타임 시점의 의존관계는 컨테이너나 팩토리 같은 제3의 존재가 결정한다.
    - 의존관계는 사용할 오브젝트에 대한 레퍼런스를 외부에서 제공(주입)해줌으로써 만들어진다.
    ```
    DI는 오브젝트 레퍼런스를 외부로부터 제공(주입)받고 이를 통해 여타 오브젝트와 다이내믹하게 의존관계가 만들어지는 것이 핵심 
    …생략…
    주입받는 메소드 따라미터가 이미 특정 클래스 타입으로 고정되어 있다면 DI가 일어날 수 없다. 
    DI에서 말하는 주입은 다이내믹하게 구현 클래스를 결정해서 제공받을 수 있도록 인터페이스 타입 의 파라미터를 통해 이뤄져야 한다.
    ```
  - 핵심은 DI는 클래스타입이 고정되어 있지 않고 인터페이스 타입의 파라미터를 통해 다이나믹하게 구현 클래스를 결정해서 제공 받을수 있어야 한다.
    ```java
    public class Orders {
        private Discount discount = new FixedDiscount();
        
        @Autowired
        public Orders(Discount discount) {
            this.discount = discount;
        }      
    }
    ```
    - 동적으로 결정한 Discount 구현체를 injection 해주어야 한다.

#### 출처

- 블로그: https://biggwang.github.io/2019/08/31/Spring/IoC,%20DI%EB%9E%80%20%EB%AC%B4%EC%97%87%EC%9D%BC%EA%B9%8C/
- 토비의 스프링
