---
layout: post
title: Spring 의존관계 주입
summary: 김영한님 - 스프링 핵심 원리_기본편
author: devhtak
date: '2021-03-19 21:41:00 +0900'
category: Spring
---

#### 다양한 의존 관계 주입 방법

- 의존 관계 주입 4가지 방법
  - 생성자 주입
  - 수정자 주입(setter)
  - 필드주입
  - 일반 메서드 주입

- 생성자 주입
  - 이름 그대로 생성자를 통해서 의존 관계를 주입받는 방법
  - 지금까지 우리가 진행했던 방법이 바로 생성자 주입
  - 특징
    - 생성자 호출 시점에 딱 1번만 호출되는 것이 보장된다.
    - "불변, 필수" 의존관계에서 사용
  - 중요! 생성자가 딱 1개만 있으면, @Autowired를 생략해도 가능하다.

  ```java
  @Component
  public class OrderServiceImpl implements OrderService {
      private final MemberRepository memberRepository;
      private final DiscountPolicy discountPolicy;
      
      @Autowired // 생략 가능
      public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
          this.memberRepository = memberRepository;
          this.discountPolicy = discountPolicy;
      }
      //...
  }
  ```
  
- 수정자 주입(setter)
  - setter라 불리는 필드의 값을 변경하는 수정자 메서드를 통해서 의존관계를 주입하는 방법
  - 특징
    - "선택, 변경" 가능성이 있는 의존관계에 사용
    - 자바 빈 프로퍼티 규약의 수정자 메서드 방식을 사용하는 방법
  - 참고. @Autowired의 기본 동작은 주입할 대상이 없으면 오류가 발생한다. 주입할 대상이 없어도 동작하게 하려면 @Autowired(requried=false)로 지정
  - 참고. 자바빈 프로퍼티, 자바에서 과거부터 필드의 값을 직접 변경하지 않고, getter/setter를 통해 값을 읽거나 수정하는 규칙을 만들었는데, 그것을 자바빈 프로퍼티 규약이다.

  ```java
  @Component
  public class OrderServiceImpl implements OrderService {
      private MemberRepository memberRepository;
      private DiscountPolicy discountPolicy;
      
      @Autowired
      public void setMemberRepository(MemberRepository memberRepository) {
          this.memberRepository = memberRepository;     
      }
      @Autowired
      public void setDiscountPolicy(DiscountPolicy discountPolicy) {
          this.discountPolicy = discountPolicy;
      }
      //...
  }
  ```
  
- 필드 주입
  - 필드에 바로 주입하는 방법
  - 특징
    - 코드가 간결해서 많은 개발자들을 유혹하지만 외부에서 변경이 불가능해서 테스트하기 어렵다는 단점이 있다.
    - DI 프레임워크가 없으면 아무것도 할 수 없다.
    - 사용하지 말자.
      - 애플리케이션의 실제 코드와 관계없는 테스트 코드에서만 사용
      - 스프링 설정을 목적으로 하는 @Configuration같은 곳에서만 특별한 용도로 사용

  ```java
  @Component
  public class OrderServiceImpl implements OrderService {
      @Autowired
      private MemberRepository memberRepository;
      @Autowired
      private DiscountPolicy discountPolicy;
      //...
  }
  ```
  
- 일반 메서드 주입
  - 일반 메서드를 통해서 주입 받을 수 있다.
  - 특징
    - 한번에 여러 필드를 주입 받을 수 있다.
    - 일반적으로 잘 사용하지 않는다.
  - 참고. 의존관계 자동 주입은 스프링 컨테이너가 관리하는 스프링 빈이어야 동작한다. 스프링 빈이 아닌 Member 같은 클래스에서 @Autowired 코드를 적용해도 아무 기능도 동작하지 않는다.
  
  ```java
  @Component
  public class OrderServiceImpl implements OrderService {
      private MemberRepository memberRepository;
      private DiscountPolicy discountPolicy;
      
      @Autowired
      public void init(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
          this.memberRepository = memberRepository;
          this.discountPolicy = discountPolicy;
      }
      //...
  }
  ```

#### 옵션 처리

- 주입할 스프링 빈이 없어도 동작해야 할 때가 있다.
- @Autowired만 사용하면 required 옵션이 true로 되어 있어서 자동 주입 대상이 없으면 오류가 발생한다.

- 자동 주입 대상을 옵션으로 처리하는 방법은 다음과 같다
  - @Autowired(required=false): 자동 주입할 대상이 없으면 수정자 메서드 자체가 호출 안된다.
  - org.springframework.lang.@Nullable: 자동 주입할 대상이 없으면 null이 입력된다.
  - Optional<>: 자동 주입할 대상이 없으면 Optional.empty가 입력된다.

- 예제, Member가 스프링 빈이 아니다.

  ```java
  @Autowired(required = false)
  public void setNoBean1(Member member) {
      System.out.println("setNoBean1 = " + member);
  }

  @Autowired
  public void setNoBean2(@Nullable Member member) {
      System.out.println("setNoBean2 = " + member);
  }

  @Autowired(required = false)
  public void setNoBean3(Optional<Member> member) {
      System.out.println("setNoBean3 = " + member);
  }
  ```
  - setNoBean1()은 @Autowired(required=false)이므로 호출 자체가 안된다.
  - 출력 결과
    ```
    setNoBean2 = null
    setNoBean3 = Optional.empty
    ```

- 참고. @Nullable, Optional은 스프링 전반에 걸쳐서 지원된다. 예를 들어서 생성자 자동 주입에서 특정 필드에만 사용해도 된다.
  
#### 생성자 주입을 선택해라!

- 과거에는 수정자 주입과 필드 주입을 많이 사용했다.
- 최근에는 스프링을 포함한 DI 프레임워크 대부분이 생성자 주입을 권장한다.

- 특징1. 불변
  - 대부분의 의존관계 주입은 한번 일어나면 애플리케이션 종료시점까지 의존 관계를 변경할 일이 없다.
  - 오히려 대부분의 의존관계는 애플리케이션 종료 전까지 변하면 안된다.(불변)
  - 수정자 주입을 사용하면 setter 메서드를 public으로 열어두어야 한다.
  - 누군가 실수로 변경할 수도 있고, 변경하면 안되는 메서드를 열어두는 것은 좋은 설계 방법이 아니다.
  - 생성자 주입은 객체를 생성할 때 딱 1번만 호출되므로 이후에 호출되는 일이 없다. 따라서 불변하게 설계할 수 있다.

- 누락
  - 프레임워크 없이 순수한 자바 코드를 단위 테스트 하는 경우에 수정자 의존관계인 경우
  - 예제
    - 수정자 주입
      ```java
      public class OrderServiceImpl implements OrderService {
           private MemberRepository memberRepository; 
           private DiscountPolicy discountPolicy;
           
           @Autowired
           public void setMemberRepository(MemberRepository memberRepository) {
              this.memberRepository = memberRepository;
           }
           
           @Autowired
           public void setDiscountPolicy(DiscountPolicy discountPolicy) {
              this.discountPolicy = discountPolicy;
           }
           
           //...
      }
      ```
    - 테스트 코드
      ```java
      @Test
      void createOrder() {
          OrderServiceImpl orderService = new OrderServiceImpl();
          orderService.createOrder(1L, "itemA", 10000);
      }
      ```
      - NullPointExceptoin이 발생한다.
      - Spring을 사용하지 않았기 때문에 의존관계 주입이 누락되었다.

- final 키워드
  - 생성자 주입을 사용하면 필드에 final 키워드를 사용할 수 있다.
  - 그래서 생성자에서 혹시라도 값이 설정되지 않는 오류를 컴파일 시점에 막아준다.
  - 예제
    ```java
    @Component
    public class OrderServiceImpl implements OrderService {
       private final MemberRepository memberRepository;
       private final DiscountPolicy discountPolicy;
       
       @Autowired
       public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
          this.memberRepository = memberRepository;
       }
       
       //...
    }
    ```
    - 생성자에서 DisocuntPolicy를 세팅하지 않았다.
    - 자바는 컴파일 시점에서 다음 오류를 발생시킨다.
      ```
      java: variable discountPolicy might not have been initialized
      ```
    - 컴파일 오류는 세상에서 가장 빠르고, 좋은 오류다!!

  - 참고. 수정자 주입을 포함한 나머지 주입 방식은 모두 생성자 이후에 호출되므로, 필드에 final 키워드를 사용할 수 없다.
    - 오직, 생성자 주입 방식만 final 키워드를 사용할 수 있다.

- 정리
  - 생성자 주입 방식을 선택하는 이유는 여러가지가 있지만, 프레임워크에 의존하지 않고, 순수한 자바 언어의 특징을 잘 살리는 방법이기도 하다.
  - 기본으로 생성자 주입을 사용하고, 필수 값이 아닌 경우에는 수정자 주입 방식을 옵션으로 부여하면 된다. 생성자 주입과 수정자 주입을 동시에 사용할 수 있다.
  - 항상 생성자 주입을 선택해라! 그리고 가끔 옵션이 필요하면 수정자 주입을 선택해라. 필드 주입은 사용하지 않는게 좋다

#### 롬복과 최신 트랜드

- 대부분이 불변이고, 다음과 같이 생성자에 final 키워드를 사용한다.
- 롬복을 활용하면 간편하게 생성자 주입을 할 수 있다.
- 예시
  ```java
  @Component
  @RequiredArgsConstructor
  public class OrderServiceImpl implements OrderService {
      private final MemberRepository memberRepository;
      private final DiscountPolicy discountPolicy;
  }
  ```
  - @RequiredArgsConstructor 기능을 사용하면 final이 붙은 필드를 모아서 생성자를 자동으로 만들어준다.

#### 조회 빈이 2개 이상일 때 해결 방법

- 문제 발생
  - @Autowired는 타입(Type)으로 조회한다.

    ```java
    @Autowired
    private DiscountPolicy discountPolicy;
    ```

  - 타입으로 조회하기 때문에 마치 다음 코드와 유사하게 동작한다.
    - 실제로는 더 많은 기능 제공

    ```java
    ac.getBean(DiscountPolicy.class);
    ```

  - 스프링 빈 조회에서 학습했듯이 타입으로 조회하면 선택된 빈이 2개 이상일 때 문제가 발생한다.
    - DiscountPolicy 하위 타입인 FixDiscountPolicy, RateDiscountPolicy 둘 다 스프링 빈으로 선언

      ```java
      @Component
      public class FixDiscountPolicy implements DiscountPolicy {}
      ```
      ```java
      @Component
      public class RateDiscountPolicy implements DiscountPolicy {}
      ```
    - NoUniqueBeanDefinitionException 오류가 발생한다.
    - 이 때 하위 타입으로 빈을 주입 받을 수 있지만 DIP에 위배되며 유연성이 떨어진다.

- 해결 방법 1. @Autowired 필드명
  - @Autowired는 타입 매칭을 시도하고, 이 때 여러 빈이 있으면 필드 이름, 파라미터 이름으로 빈 이름을 추가 매칭한다.
    ```java
    @Autowired
    private DiscountPolicy disocuntPolicy;
    ```
    - 해당에 경우 필드 이름(discountPolicy)가 상위 타입과 동일하기 때문에 NoUniqueBeanDefinitionException 발생
    ```java
    @Autowired
    private DiscountPolicy rateDiscountPolicy;
    ```
    - 해당에 경우 필드 이름(rateDiscountPolicy)이므로 등록된 빈 중 RateDiscountPolicy가 등록된다.

  - 타입 매칭을 먼저 시도하고, 그 결과에 여러 빈이 있을 때 추가로 동작하는 기능

- 해결 방법 2. @Qualifier 사용
  - @Qualifier는 추가 구분자를 붙여주는 방법이다.
  - 주입 시 추가적인 방법을 제공하는 것인지 빈 이름을 변경하는 것은 아니다.
  
  - 주입 시 @Qualifier를 붙여주고 등록한 이름을 적어준다.
    
    ```java
    @Component
    @Qualifier("mainDiscountPolicy")
    public class RateDiscountPolicy implements DiscountPolicy {}
    ```
    ```java
    @Component
    @Qualifier("fixDiscountPolicy")
    public class FixDiscountPolicy implements DiscountPolicy {}
    ```
  
  - 생성자 자동 주입 예시
    
    ```java
    @Autowired
    public OrderService(MemberRepository memberRepository, @Qualifier("mainDiscountPolicy") DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
    ```
  
  - 수정자 자동 주입 예시
    
    ```java
    @Autowired
    public DiscountPolicy setDiscountPolicy(@Qualifier("mainDiscountPolicy") DiscountPolicy discountPolicy) {
        this.discountPolicy = discountPolicy;
    }
    ```
  
  - @Qualifier로 주입할 때 @Qualifier("mainDiscountPolicy")를 못찾으면 어떻게 될까?
    - 먼저 @Qualifier("mainDiscountPolicy") 가 붙은 빈을 찾는다.
    - 없다면 mainDiscountPolicy라는 이름의 스프링 빈을 추가로 찾는다.
    - 하지만 경험상 @Qualifier는 매칭하여 @Qualifier를 찾는 용도로만 사용하는게 면확하고 좋다.

- 해결 방법 3. @Primary
  - @Primary는 우선순위를 정하는 방법이다.
  - @Autowired 시에 여러 빈이 매칭되면 @Primary 우선권을 가진다.
    
    ```java
    @Component
    @Primary
    public class RateDiscountPolicy implements DiscountPolicy {}
    
    @Component
    public class FixDiscountPolicy implements DiscountPolicy {}
    ```
  - DiscountPolicy 타입에 생성자 주입은 우선순위가 높은 RateDiscountPolicy가 동작한다.

- @Primary vs @Qualifier
  - @Qualifier는 빈 등록, 빈 주입시에 모두 사용하여 코드가 길어지는 불편한 점이 있다.
  - 주 사용 빈은 @Primary로 등록하여 사용하고, 특별한 경우에 사용할 경우 @Qualifier를 사용하자
    - 만약 Main DB의 커넥션을 획득하는 빈은 @Primary로 사용하고, Sub DB의 커넥션을 획득하는 빈은 @Qualifier를 지정한다.
    - Sub DB의 커넥션 정보를 주입받는 경우에만 @Qualifier를 지정하여 사용할 수 있게끔 한다.
  - 즉, 우선순위는 @Qualifier가 @Primary보다 높다.
    - @Primary는 기본값처럼 동작하며, @Qualifier는 매우 상세하게 동작한다.
    - 스프링은 좁은 범위의 선택권에 우선 순위를 높게 주기 때문에 빈을 주입 받는 곳에 @Qualifier가 있으면 해당 타입의 빈을 주입한다.

#### 조회한 빈이 모두 필요할 때: List, Map

- 의도적으로 해당 타입의 스프링 빈이 다 필요한 경우가 있다.
  - 할인 서비스를 제공할 때 클라이언트가 할인의 종류를 선택할 수 있다고 가정해보자.
  - 스프링을 사용하면 전략 패턴을 매우 간단하게 구현할 수 있다.

```java
public class AllBeanTest {
    @Test
    void findAllBean() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(AutoAppConfig.class, DiscountService.class);
        DiscountService discountService = ac.getBean(DiscountService.class);
        Member member = new Member(1L, "test", Grade.VIP);
        assertNotNull(discountService);
        
        int rateDiscountPrice = discountService.discount(member, 10000, "rateDiscountPolicy");
        assertEquals(rateDicountPrice, 1000);
    }
    
    static class DiscountService {
        Map<String, DiscountPolicy> policyMap;
        List<DiscountPolicy> policyList;
        
        @Autowired
        public DiscountService(Map<String, DiscountPolicy> policyMap, List<DiscountPolicy> policyList) {
            this.policyMap = policyMap;
            this.policyList = policyList;
        }
        
        public int discount(Member member, int price, String dicountPolicyCode) {
            DiscountPolicy discountPolicy = policyMap.get(discountPolicyCode);
            return discountPolicy.discount(member, price);
        }
    }
}
```
- 로직 분석
  - DiscountService는 Map 또는 List로 모든 DiscountPolicy를 주입받는다. 이 때, fixDiscountPolicy, rateDiscountPolicy가 주입된다.
  - discount() 메서드는 discountCode(key)로 해당하는 DiscountPolicy(value)를 찾아서 실행한다.

- 주입 분석
  - Map<String, DiscountPolicy>: map의 키에 스프링 빈의 이름을 넣어주고, 그 값으로 DiscountPolicy 타입으로 조회한 모든 스프링 빈을 담아준다.
  - List<DiscountPolicy>: DiscountPolicy 타입으로 조회한 모든 스프링 빈을 담아준다.
  - 만약 해당하는 타입의 스프링 빈이 없으면, 빈 컬렉션이나 Map을 주입한다.

#### 빈 주입 과정 - 어떻게 빈을 찾는가?
- LifeCycle
  - 스프링 컨테이너 생성 -> 스프링 빈 생성 -> 의존성 주입 -> 초기화 콜백(빈의 의존관계 주입이 완료된 후 호출) -> 사용 -> 소멸전 콜백(빈이 소멸되기 직전 호출) -> 종료
- BeanPostProcessor
  ```java
  @Nullable
	default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}
	@Nullable
	default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}
  ```
  - 빈의 인스턴스가 만들어지는 라이프사이클에서 빈의 초기화 단계의 이전과 이후에 다른 부가적인 작업을 할 수 있게 해주는 인터페이스이다.
  - postProcessBeforeInitialization 메소드와 postProcessAfterInitialization 메소드가 존재하는데 초기화 단계 이전과 이후에 해당 메서드가 실행되어 빈을 찾을 수 있다.

- AutowiredAnnotationBeanPostProcessor
  ```java
  public void processInjection(Object bean) throws BeanCreationException {
  		Class<?> clazz = bean.getClass();
  		InjectionMetadata metadata = findAutowiringMetadata(clazz.getName(), clazz, null);
  		try {
  			metadata.inject(bean, null, null);
  		}
  		catch (BeanCreationException ex) {
  			throw ex;
  		}
  		catch (Throwable ex) {
  			throw new BeanCreationException(
  					"Injection of autowired dependencies failed for class [" + clazz + "]", ex);
  		}
	}
  ```
  - BeanPostProcessor 에 구현 객체로 필드, 메서드, 클래스에 대한 의존관계를 주입해준다.
  - processInjection 메서드를 보면 빈의 클래스 정보를 가져와 Metadata를 찾아 주입하는 역할을 한다.
  - inject 메서드를 보면 리플렉션을 통해 빈을 주입한다.

- AbstractAutowireCapableBeanFactory
  - BeanFactory는 BeanPostProcessor 타입의 빈을 꺼내 일반적인 빈들을 @Autowired 로 의존관계 주입이 필요한 빈들에게 @Autowired를 처리하는 로직을 적용한다.
    
** 출처: 인프런 스프링핵심원리 - 기본편, 김영한님 강연
