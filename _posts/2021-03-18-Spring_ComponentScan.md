---
layout: post
title: Spring @ComponentScan
summary: 김영한님 - 스프링 핵심 원리_기본편
author: devhtak
date: '2021-03-18 21:41:00 +0900'
category: Spring
---

#### 컴포넌트 스캔과 의존관계 자동 주입 시작하기

- 지금까지 스프링 빈을 등록할 때는 자바 코드의 @Bean이나 XML <bean> 등을 통해서 설정 정보에 직접 등록할 스피링 빈을 나열했다.
- 예제에서는 몇개가 안되지만, 이렇게 등록해야 할 스프링 빈이 수십, 수백개가 되면 일일이 등록하기 어렵고, 설정 정보도 커지고, 누락하는 문제가 발생한다.
- 그래서 스프링은 설정 정보가 없어도 자동으로 스프링 빈을 등록하는 컴포넌트 스캔이라는 기능을 제공한다.
- 또 의존관계에서 자동으로 주입하는 @Autowired라는 기능도 제공한다.

- 컴포넌트 스캔과 자동 주입
  - 기존 AppConfig.java는 과거 코드와 테스트를 유지하기 위해 남겨두고, 새로운 AuthAppConfig.java를 만들었다.
  ```java
  @Configuration
  @ComponentScan(
      excludeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = Configuration.class)
      // 기존 예제를 안전하게 유지하기 위해서 Configuration 애노테이션을 제외했다
  )
  public class AutoAppConfig {

  }
  ```
  
  ```
  참고, 컴포넌트 스캔을 사용하면 @Configuration 이 붙은 설정 정보도 자동으로 등록하기 때문에, AppConfig, TestConfig 등 앞에서 만들었던 설정 정보도 함께 등록되고 실해된다.
  그래서 excludedFilters를 사용하여 설정정보는 컴포넌트 스캔 대상에서 제외했다. 보통 설정 정보를 컴포넌트 스캔 제외하지 않지만, 
  기존 예제 코드를 최대한 남기고 유지하기 위해 해당 방법을 선택했다.
  ```
  
  - 컴포넌트 스캔은 이름 그대로 @Component 애노테이션이 붙은 클래스를  스캔해서 스프링 빈으로 등록한다.
  - 참고, @Configuration 이 컴포넌트 스캔 대상이 되는 이유는 @Configuration 안에서 살펴보면 @Component 애노테이션이 붙어있다.
  
  - 각 클래스에 @Component를 추가하자
    - MemberServiceImpl.java
      ```java
      @Component
      public class MemberServiceImpl implements MemberService {
          @Autowired
          private final MemberRepository memberRepository;
          
          // ...
      }
      ```
    - RateDiscountPolicy.java
      ```
      @Component
      public class RateDiscountPolicy implements DiscountPolicy {
          // ...
      }
      ```
  
    - OrderServiceImpl.java
      ```java
      @Component
      public class OrderServiceImpl implements OrderService {
          private final MemberService memberService;
          private final DiscountPolicy discountPolicy;
          @Autowired
          public OrderServiceImpl(MemberService memberService, DiscountPolicy discountPolicy) {
              super();
              this.memberService = memberService;
              this.discountPolicy = discountPolicy;
          }

          @Override
          public Order createOrder(Long memberId, String itemName, int itemPrice) {
              // TODO Auto-generated method stub
              Member member = memberService.findByMember(memberId);

              int discountedPrice = discountPolicy.discount(member, itemPrice);
              return new Order(memberId, itemName, itemPrice, discountedPrice);
          }
      }
      ```
      - Component를 붙이면 빈으로 등록된다.
      - 빈으로 등록 받은 객체를 사용하기 위해 MemberService, DiscountPolicy에 @Autowired로 자동 주입되도록 하였다.
      - 테스트 코드
        ```java
        @Test
        void basicScan() {
            ApplicationContext ac = new AnnotationConfigApplicationContext(AutoAppConfig.class);

            MemberService memberService = ac.getBean(MemberService.class);
            assertThat(memberService).isInstanceOf(MemberService.class);
        }
        ```
        - AnnotationConfigApplicationContext를 사용하는 것은 기존과 동일
        - 설정 정보로 AutoAppConfig 클래스를 넘겨준다.
        
- 컴포넌트 스캔과 자동 의존관계 주입 동작
  - @ComponentScan
    - @ComponentScan은 @Component가 붙은 모든 클래스를 스프링 빈으로 등록한다.
    - 이 때 스프링 빈의 기본 이름은 클래스 명을 사용하되 맨 앞글자만 소문자를 사용한다.
      - 빈 이름 기본 전략: MemberServiceImpl 클래스 -> memberServiceImpl (앞 대문자만 소문자로)
      - 빈 이름 직접 지정: 만약 스프링 빈의 이름을 직접 등록하고 싶다면 @Copmonent("memberService2")로 부여 가능

  - @Autowired
    - 생성자에 @Autowired를 지정하면, 스프링 컨테이너가 자동으로 해당 스프링 빈을 찾아서 주입
    - 이 때 기본 조회 전략은 타입이 같은 빈을 찾아서 주입한다.
      - getBean(MemberRepository.class) 와 동이라하다고 이해하면 된다.
      - 생성자에 파라미터가 많아도 다 찾아서 자동으로 주입한다.

#### 탐색 위치와 기본 스캔 대상(basePackages)

```java
@ComponentScan(
	basePackages = "com.example.member",
)
```
- 탐색할 패키지의 시작 위치 지정
  - 모든 자바 클래스를 다 컴포넌트 스캔하면 시간이 오래 걸린다.
  - 꼭 필요한 위치부터 탐색하도록 시작 위치를 지정할 수 있다.
  - basePackages: 탐색할 패키지의 시작 위치를 지정한다. 이 패키지를 포함해서 하위 패키지를 모두 탐색
    - basePackages = {"com.example.member", "com.example.order"} 이런 방식으로 여러 위치를 지정할 수 있다.
  - basePackageClasses: 지정한 클래스의 패키지를 탐색 시작 위치로 지정한다.
- 만약 지정하지 않으면 @ComponentScan 이 붙은 설정 정보 클래스의 패키지가 시작 위치가 된다.

- 권장 방법
  - 패키지 위치를 지정하지 않고, 설정 정보 클래스의 위치를 프로젝트 최상단에 두는 것
    - 예시
      ```
      com.hello
      com.hello.service
      com.hello.repository
      ```
      - 해당 프로젝트 구조로 되어 있으면, com.hello 를 프로젝트 시작 루트로 여기에 AppConfig같은 메인 설정 정보를 둔다.
      - @ComponentScan 애노테이션을 붙이고, basePackages 지정은 생략한다.
      - 이렇게 하면 com.hello를 포함한 하위는 모두 자동으로 컴포넌트 스캐의 대상이 된다.
    
  - 스프링 부트도 해당 방법을 기본으로 제공
    ```java
    @SpringBootApplication
    public class Application {
        public static void main(String[] args) {
            SpringApplication.run(Application.class, args);
        }
    }
    ```
    - @SpringBootApplication
      - @Configuration
      - @EnableAutoConfiguration
      - @ComponentScan

- 컴포넌트 스캔 기본 대상
  
  |애노테이션|사용 위치|부가 기능|
  |---|---|---|
  |@Component|컴포넌트 스캔에서 사용||
  |@Controller: 스프링 MVC 컨트롤러에서 사용|스프링 MVC 컨트롤러로 인식|
  |@Service|스프링 비즈니스 로직에서 사용|특별한 처리는 없으나 개발자들에게 비즈느시 계층을 인식하는데 도움이 된다|
  |@Repository|스프링 데이터 접근 계층에서 사용|데이터 계층의 예외를 스프링 예외로 변환|
  |@Configuration|스프링 설정 정보에서 사용|스프링 빈이 싱글톤을 유지하도록 추가 처리를 한다|
    
  - 참고, useDefaultFilters 옵션은 기본으로 켜져있는데, 이 옵션을 끄면 기본 스캔 대상들이 제외된다.

#### 필터

- includeFilters: 컴포넌트 스캔 대상을 추가로 지정
- excludeFilters: 컴포넌트 스캔에서 제외할 대상을 지정

- 예제
  - @MyExcludeComponent.java
    ```java
    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    public @interface MyExcludeComponent {}
    ```

  - @MyIncludeComponent.java
    ```java
    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    public @interface MyIncludeComponent {}
    ```
    
  - ExcludeBean.java
    ```java
    @Component
    @MyExcludeComponent
    public class ExcludeBean {}
    ```

  - IncludeBean.java
    ```java
    @Component
    @MyIncludeComponent
    public class IncludeBean {}
    ```

  - Test
    ```java
    public class ComponentFilterTest {	
        @Test
        void filterScan() {
            ApplicationContext ac = new AnnotationConfigApplicationContext(ComponentFilterAppConfig.class);
            IncludeBean bean1 = ac.getBean("includeBean", IncludeBean.class);
            assertNotNull(bean1);
            assertThrows(NoSuchBeanDefinitionException.class, ()-> ac.getBean("excludeBean", ExcludeBean.class));					
        }
        @ComponentScan(
            includeFilters = @Filter(type = FilterType.ANNOTATION, classes = MyIncludeComponent.class),
            excludeFilters = @Filter(type = FilterType.ANNOTATION, classes = MyExcludeComponent.class)
        )
        static class ComponentFilterAppConfig {}
    }
    ```
    - ComponentFilterAppConfig에서 includeFilter로 MyIncludeComponent 애노테이션을 등록하였고, excludeFilter로 MyExcludeComponent 애노테인셔을 등록하였다.
    - bean을 가져올 때 MyExcludeBean 애노테이션을 사용한 ExcludeBean을 가져올 때 오류가 발생한다.

- FilterType 옵션
  - FilterType은 5가지 옵션이 있다.
  
  |Annotation|Description|예제|
  |---|---|---|
  |ANNOTATION|디폴트, 애노테이션을 인식해서 동작|org.example.SomeAnnotation|
  |ASSIGNABLE_TYPE|지정한 타입과 자식 타입을 인식해서 동작|org.example.SomeClass|
  |ASPECTJ|AspectJ 패턴 사용|org.example..\*Service+|
  |REGEX|정규 표현식|org\.example\.Default\.\*|
  |CUSTOM|TypeFilter 라는 인터페이스를 구현하여 처리|org.example.MyTypeFilter|
  
- 참고, includeFilters는 @Component를 사용하면 충분하기 때문에 사용할 일이 거의 없다.
- excludeFilters는 여러가지 이유로 간혹 사용할 때가 있지만 많지는 않다.

#### 중복 등록과 충돌

- 컴포넌트 스캔에서 같은 빈 이름을 등록하면 어떻게 될까
  - 자동 빈 등록 vs 자동 빈 등록
  - 수동 빈 등록 vs 자동 빈 등록

- 자동 빈 등록 vs 자동 빈 등록
  - 컴포넌트 스캔에 의해 자동으로 스프링 빈이 등록되는데, 그 이름이 같은 경우 스프링은 오류를 발생한다.
  - ConflictingBeanDefinitionException 발생
  
- 수동 빈 등록 vs 자동 빈 등록
  - 만약 수동 빈 등록과 자동 빈 등록에서 빈 이름이 충돌되면 어떻게 될까?
  - 자동 빈 등록
    ```java
    @Component
    public class MemoryMemberRepository implements MemberRepository {}
    ```
  - 수동 빈 등록
    ```java
    @Configuration
    @ComponentScan(
        excludeFilters = @Filter(type = FilterType.ANNOTATION, classes =Configuration.class)
    )
    public class AutoAppConfig {
       @Bean(name = "memoryMemberRepository")
       public MemberRepository memberRepository() {
          return new MemoryMemberRepository();
       }
    }
    ```
  - 해당 경우 수동 빈 등록이 우선권을 가진다.
  - 수동 빈이 자동 빈을 오버라이딩 해버린다.
    ```
    Overriding bean definition for bean 'memoryMemberRepository' with a different definition: replacing
    ```
  - 하지만, 오버라이딩 되기를 의도적으로 원해서 개발하는 경우는 드물다. 이런 경우 잡기 어려운 버그가 만들어진다.
  - 그래서, 최근 스프링 부트는 수동 빈 등록과 자동 빈 등록이 충돌나면 오류가 발생하도록 기본 값을 바꾸웠다.
    ```
    Consider renaming one of the beans or enabling overriding by setting spring.main.allow-bean-definition-overriding=true
    ```

- 참고: 인프런 이영한님의 스프링 핵심원리 - 기본편 강의
