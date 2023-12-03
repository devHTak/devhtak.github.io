---
layout: post
title: Spring Singleton Container
summary: 김영한님 - 스프링 핵심 원리_기본편
author: devhtak
date: '2021-03-15 21:41:00 +0900'
category: Spring
---

#### 앱 애플리케이션과 싱글톤

- 스프링은 태생이 기업용 온라인 서비스 기술을 지원하기 위해 탄생했다.
- 대부분의 스프링 애플리케이션은 웹 애플리케이션이다. 물론 웹이 아닌 애플리케이션 개발도 얼마든지 개발할 수 있다.
- 웹 애플리케이션은 보통 여러 고객이 동시에 요청한다.

```java
public class SingletonTest {
    @Test
    @DisplayName("스프링 없는 순수한 DI 컨테이너")
    void pureContainer() {
        AppConfig appConfig = new AppConfig();
        //1. 조회: 호출할 때 마다 객체를 생성
        MemberService memberService1 = appConfig.memberService();
        //2. 조회: 호출할 때 마다 객체를 생성
        MemberService memberService2 = appConfig.memberService();
        //참조값이 다른 것을 확인
        System.out.println("memberService1 = " + memberService1); System.out.println("memberService2 = " + memberService2);
        //memberService1 != memberService2
        assertThat(memberService1).isNotSameAs(memberService2);
    }
}
```

- 우리가 만든 스프링 없는 순수 DI 컨테이너인 AppConfig는 요청을 할때마다 객체를 새로 생성한다.
- 고객 트래픽이 초당 100이 나오면 100개 객체가 생성되고 소멸된다. -> 메모리 낭비
- 해결방은은 객체가 딱 1개만 생성되고 공유하도록 설계하면 된다. -> 싱글톤 패턴

#### 싱글톤 패턴

- 클래스의 인스턴스가 딱 1개만 생성되는 것을 보장하는 디자인 패턴
- 그래서 객체 인스턴스를 2개 이상 생성하지 못하도록 막는다.
  - private 생성자를 사용하여 외부에서 임의로 new 키워드를 사용하지 못하도록 막는다.

```java
public class SingletonService {
    //1. static 영역에 객체를 딱 1개만 생성해둔다
    private static final SingletonService instance = new SingletonService();
    
    //2. 생성자를 private으로 선언해서 외부에서 new 키워드를 사용한 객체 생성을 못하게 막는다.
    private SingletonService() {}
    
    //3. public으로 열어서 객체 인스터스가 필요하면 이 static 메서드를 통해서만 조회하도록 허용한다.
    public static SingletonService getInstance() {
        return instance;
    }
    
    public void logic() {
        System.out.println("싱글톤 객체 로직 호출");
    }
}
```
  - static 영역에 객체 instance를 미리 하나 생성해서 올려둔다.
  - 딱 1개의 객체 인스턴스만 존재해야 하므로, 생성자를 private으로 막아서 혹시라도 외부에서 new 키워드로 객체 인스턴스가 생성되는 것을 막는다.
  - 이 객체 인스턴스가 필요하면 오직 getInstance() 메서드를 통해서만 조회할 수 있다. 이 메서드를 호출하면 항상 같은 인스턴스를 반환한다.  

```java
@Test
@Displayname("싱글톤 패턴을 적용한 객체 사용")
void singletonServiceTest() {
    // 호출할 때마다 같은 객체를 반환한다.
    SingletonService singletonService1 = SingletonService.getInstance();
    SingletonService singletonService2 = SingletonService.getInstance();
    
    // singletonService1 == singletonService2
    // equals -> Object의 equal
    // same -> == 
    assertThat(singletonService1).isSameAs(singletonService2);
}
```

  - private으로 new 키워드를 막아두었다.
  - 호출할 때 마다 같은 객체 인스턴스를 반환하는 것을 확인할 수 있다.
  - 참고: 싱글톤 패턴을 구현하는 방법은 여러가지가 있다. 여기서는 객체를 미리 생성해두는 가장 단순하고 안전한 방법을 선택했다.
  - 싱글톤 패턴을 적용하면 고객의 요청이 올 때 마다 객체를 생성하는 것이 아니라, 이미 만들어진 객체를 공유해서 효율적으로 사용할 수 있다. 
  - 하지만 싱글톤 패턴은 다음과 같은 수 많은 문제점들을 가지고 있다
    - 싱글톤 패턴을 구현하는 코드 자체가 많이 들어간다.
    - 의존관계상 클라이언트가 구체 클래스에 의존한다. DIP를 위반한다.
    - 클라이언트가 구체 클래스에 의존해서 OCP 원칙을 위반할 가능성이 높다.
    - 테스트하기 어렵다.
    - 내부 속성을 변경하거나 초기화 하기 어렵다.
    - private 생성자로 자식 클래스를 만들기 어렵다.
    - 결론적으로 유연성이 떨어진다.
    - 안티패턴으로 불리기도 한다.

#### 싱글톤 컨테이너

- 스프링 컨테이너는 싱글톤 패턴의 문제점을 해결하면서, 객체 인스턴스를 싱글톤(1개만 생성)으로 관리한다.
- 지금까지 우리가 학습한 스프링 빈이 바로 싱글톤으로 관리되는 빈이다.

- 싱글톤 컨테이너
  - 스프링 컨테이너는 싱글턴 패턴을 적용하지 않아도, 객체 인스턴스를 싱글톤으로 관리한다.
    - 이전에 설명한 컨테이너 생성 과정을 잘 살펴보자. 컨테이너는 객체를 하나만 생성해서 관리한다.
  - 스프링 컨테이너는 싱글톤 컨테이너 역할을 한다. 이렇게 싱글톤 객체를 생성하고 관리하는 기능을 싱글톤 레지스트리라고 한다.
  - 스프링 컨테이너의 이런 기능 덕분에 싱글턴 패턴의 모든 단점을 해결하면서 객체를 싱글톤으로 유지할 수 있다.
  - 싱글톤 패턴을 위한 지저분한 코드가 들어자기 않아도 된다.
  - DIP, OCP, 테스트, private 생성자로부터 자유롭게 싱글톤을 사용할 수 있다.

- 스프링 컨테이너를 사용하는 테스트 코드

  ```java
  @Test
  @DisplayName("스프링 컨테이너와 싱글톤")
  void sprintContainer() {
      ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

      // 조회 -> 호출할 때마다 같은 객체를 봔환
      MemberService memberService1 = ac.getBean("memberService", MemberService.class);
      MemberService memberService2 = ac.getBean("memberService", MemberService.class);

      // memberService1 == memberService2
      assertThat(memberService1).isSameAs(memberService2);
  }
  ```
  
- 싱글톤 컨테이너 적용 후

  - 스프링 컨테이너 덕분에 고객의 요청이 올 때마다 객체를 생성하는 것이 아니라, 이미 만들어진 객체를 공유해서 효율적으로 재사용할 수 있다.
  - 참고, 스프링의 기본 빈 등록 방식은 싱글톤이지만, 싱글톤 방식만 지원하는 것이 아니다. 빈스코프를 설정하여 요청할 때마다 새로운 객체를 생성해서 반환하는 기능도 제공한다.

#### 싱글톤 방식의 주의점

- 싱글톤 패턴이든, 스프링 같은 싱글톤 컨테이너를 사용하든, 객체 인스턴스를 하나만 생성해서 공유하는 싱글톤 방식은 여러 클라이언트가 하나의 같은 객체 인스턴스를 공유한다. 
- 그래서 싱글톤 객체는 상태를 유지(stateful)하게 설계하면 안된다.
- 무상태(stateless)로 설계해야 한다.
  - 특정 클라이언트에 의존적인 필드가 있으면 안된다.
  - 특정 클라이언트가 값을 변경할 수 있는 필드가 있으면 안된다.
  - 가급적 읽기만 가능해야 한다.
  - 필드 대신에 자바에서 공유되지 않는 지역변수, 파라미터, ThreadLocal 등을 사용해야 한다.
- 스프링 빈의 필드에 공유 값을 설정하면 정말 큰 장애가 발생할 수 있다.

- 예제
  ```java
  public class StatefulService {
      private int price; //상태를 유지하는 필드
      public void order(String name, int price) {
          System.out.println("name = " + name + " price = " + price);
          this.price = price; //여기가 문제!
      }
      public int getPrice() {
          return price;
      }
  }
  ```
  ```java
  public class StatefulServiceTest {
      @Test
      void statefulServiceSingleton() {
          ApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);
          StatefulService statefulService1 = ac.getBean("statefulService", StatefulService.class); 
          StatefulService statefulService2 = ac.getBean("statefulService", StatefulService.class);
   
          //ThreadA: A사용자 10000원 주문
          statefulService1.order("userA", 10000);
          //ThreadB: B사용자 20000원 주문
          statefulService2.order("userB", 20000);
          //ThreadA: 사용자A 주문 금액 조회
          int price = statefulService1.getPrice();
          //ThreadA: 사용자A는 10000원을 기대했지만, 기대와 다르게 20000원 출력
          System.out.println("price = " + price);
   
          Assertions.assertThat(statefulService1.getPrice()).isEqualTo(20000);
      }
      static class TestConfig {
         @Bean
         public StatefulService statefulService() {
              return new StatefulService();
          }
      }
  }
  ```

  - 최대한 단순히 설명하기 위해, 실제 쓰레드는 사용하지 않았다.
  - ThreadA가 사용자A 코드를 호출하고 ThreadB가 사용자B 코드를 호출한다 가정하자.
  - StatefulService 의 price 필드는 공유되는 필드인데, 특정 클라이언트가 값을 변경한다.
  - 사용자A의 주문금액은 10000원이 되어야 하는데, 20000원이라는 결과가 나왔다.
  - 실무에서 이런 경우를 종종 보는데, 이로인해 정말 해결하기 어려운 큰 문제들이 터진다.(몇년에 한번씩 꼭 만난다.)
  - 진짜 공유필드는 조심해야 한다! 스프링 빈은 항상 무상태(stateless)로 설계하자.

- 예제 해결
```java
public class StatefulService {
    // private int price; //상태를 유지하는 필드
    public void order(String name, int price) {
        System.out.println("name = " + name + " price = " + price);
        // this.price = price; //여기가 문제!
        return price;
    }
}
```
```java
public class StatefulServiceTest {
    @Test
    void statefulServiceSingleton() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);
        StatefulService statefulService1 = ac.getBean("statefulService", StatefulService.class); 
        StatefulService statefulService2 = ac.getBean("statefulService", StatefulService.class);
   
        //ThreadA: A사용자 10000원 주문
        int userAPrice = statefulService1.order("userA", 10000);
        //ThreadB: B사용자 20000원 주문
        int userBPrice = statefulService2.order("userB", 20000);
        
        assertEquals(10000, userAPrice);
        assertEquals(20000, userBPrice);
    }
    static class TestConfig {
       @Bean
       public StatefulService statefulService() {
            return new StatefulService();
       }
    }
}
```

#### @Configuration과 싱글톤

```java
@Configuration
public class AppConfig {
    @Bean
    public MemberService memberService() {
        //1번
        System.out.println("call AppConfig.memberService")
        return new MemberServiceImpl(memberRepository());
    }
    @Bean
    public OrderService orderService() {
        //1번
        System.out.println("call AppConfig.orderService")
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }
    @Bean
    public MemberRepository memberRepository() {
        //2번? 3번?
        System.out.println("call AppConfig.memberRepository")
        return new MemoryMemberRepository();
    }
    
    @Bean
    public DiscountPolicy discountPolicy() {
        return new RateDiscountPolicy();
    }
}
```
- AppConfig.class에서 등록한 빈은 항상 객체가 생성되도록 하였다.
    - @Bean memberService()를 호출하면 memberRepository()가 호출되고, new MemoryMemberRepository()가 호출된다.
    - @Bean orderService()를 호출하면 memberRepository()가 호출되고, new MemoryMemberRepository()가 호출된다.
- 결과적으로 싱글톤이 깨진 것처럼 보이는데, 스프링 컨테이너는 해당 문제를 어떻게 해결할까?

- 테스트 코드
  ```java
  public class ConfigurationSingletonTest {
      @Test
      void configurationTest() {
          ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
          
          MemberServiceImpl memberService = ac.getBean("memberService", MemberServiceImpl.class);
          OrderServiceImpl orderService = ac.getBean("orderService", OrderServiceImpl.class);
          MemberRepository memberRepository = ac.getBean("memberRepository", MemberRepository.class);
          
          //모두 같은 인스턴스를 참고하고 있다.
          assertThat(memberService.getMemberRepository()).isSameAs(memberRepository);
          assertThat(orderService.getMemberRepository()).isSameAs(memberRepository);
      }
  }
  ```
  
  - 스프링 컨테이너가 각각 @Bean을 호출해서 스프링 빈을 생성했다.
  - memberRepository()는 다음과 같이 총 3번 호출되는 것 아닐까?
    - 스프링 컨테이너가 스프링 빈을 등록하기 위해 @Bean이 붙어있는 memberRepository() 호출
    - memberService() 로직에서 memberRepository() 호출
    - orderService() 로직에서 memberRepository() 호출
  - 하지만, 출력 결과 1번만 호출된다.
    ```
    call AppConfig.memberService
    call AppConfig.memberRepository
    call AppConfig.orderService
    ```

#### Configuration과 바이트코드 조작의 마법

- 스프링 컨테이너는 싱글톤 레지스트리다. 따라서 스프링 빈이 싱글톤이 되도록 보장해야 한다.
- 하지만 스프링이 자바 코드를 어떻게 하기 어렵다. 그래서 스프링은 클래스의 바이트코드를 조작하는 라이브러리를 사용한다.

```java
@Test
void configurationDeep() {
    ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
    //AppConfig도 스프링 빈으로 등록된다.
    AppConfig bean = ac.getBean(AppConfig.class);

    System.out.println("bean = " + bean.getClass());
    //출력: bean = class hello.core.AppConfig$$EnhancerBySpringCGLIB$$bd479d70
}
```
  - 사실 AnnotationConfigApplicationContext 에 파라미터로 넘긴 값은 스프링 빈으로 등록된다. 
  - 그래서 AppConfig 또한 스프링 빈이 된다.
  - 출력 값 확인
    - 빈이 아닌 경우
      ```
      class hello.core.AppConfig
      ```
    - 빈으로 등록 된 경우
      ```
      bean = class hello.core.AppConfig$$EnhancerBySpringCGLIB$$bd479d70
      ```
      - CGLIB이 붙은 것이 보인다.
      - 스프링이 CGLIB이라는 바이트코드 조작 라이브러리를 사용하여 AppConfig 클래스를 상속받은 임의의 다른 클래스를 만들고, 그 다른 클래스를 스프링 빈으로 등록한 것

- AppConfig@CGLIB 예상 코드
  - 실제로는 CGLIB의 내부 기술을 사용하는 데 매우 복잡하다.

  ```java
  @Bean
  public MemberRepository memberRepository() {
      if (memoryMemberRepository가 이미 스프링 컨테이너에 등록되어 있으면?) {
          return 스프링 컨테이너에서 찾아서 반환;
      } else {
          // 스프링 컨테이너에 없으면 기존 로직을 호출해서 MemoryMemberRepository를 생성하고 스프링 컨테이너에 등록
          return 반환
      }
  }
  ```
  - @Bean이 붙은 메서드마다 이미 스프링 빈이 존재하면 존재하는 빈을 반환하고, 스프링 빈이 없으면 생성해서 스프링 빈으로 등록하고 반환하는 코드가 동적으로 만들어진다.
  - 덕분에 싱글톤이 보장되는 것이다.
  - 참고, AppConfig@CGLIB은 AppConfig의 자식이므로 빈을 조회할 때 AppConfig 타입으로 조회할 수 있다.

- @Configuration을 적용하지 않고 @Bean만 적용하면 어떻게 될까??

  - memberRepository()는 다음과 같이 총 3번 호출된다.

- 정리
  - @Configuration 없이 @Bean만 사용해도 스프링 빈으로 등록되지만, 싱글톤을 보장하지 않는다.
    - @Configuration이 없으면, AppConfig@CGLIB이 생성되지 않는다.
  - memberRepository() 처럼 의존관계 주입이 필요해서 메서드를 직접 호출할 때 싱글톤을 보장하지 않는다.
  - 크게 고민할 것이 없다. 스프링 설정 정보는 항상 @Configuration 을 사용하자.

#### SpringApplication 파헤치기
- SpringApplication.run(class)
  ```java
  public ConfigurableApplicationContext run(String... args) {
		//... 
		try {
			ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
			ConfigurableEnvironment environment = prepareEnvironment(listeners, bootstrapContext, applicationArguments);
			Banner printedBanner = printBanner(environment);
			context = createApplicationContext(); // Application Type에 따라 ApplicationContext를 생성한다.(Default, Servlet, Reactive)
			context.setApplicationStartup(this.applicationStartup);
			prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner); // Spring Bean Factory 생성
			refreshContext(context); 
			afterRefresh(context, applicationArguments);
			Duration timeTakenToStartup = Duration.ofNanos(System.nanoTime() - startTime);
			if (this.logStartupInfo) {
				new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), timeTakenToStartup);
			}
			listeners.started(context, timeTakenToStartup);
			callRunners(context, applicationArguments);
		} 
		// ...
		return context;
	}
  ```
  - context = createApplicationContext();
    - ApplicationContextFactory를 통해 Application Type에 따라 ApplicationContext를 생성한다.(Default, Servlet, Reactive)
    - AnnotationConfigApplicationContext
      - scan, register 메소드
  - prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
    - Context 내에 BeanFactory를 가져온다.
      - GenericApplicationContext 생성자에서 DefaultListableBeanFactory beanFactory 를 생성한다.
      - BeanFactory를 생성할 때 BeanNameAware, BeanFactoryAware, BeanClassLoaderAware를 세팅
    - Boot 기준 Singleton Bean 생성 및 BeanFactoryPostProcessor 등을 등록한다.
  - refreshContext(context);
    - ApplicationContext에 refresh() 호출
    - AbstractApplicationContext.refresh()
      ```java
      @Override
	  public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			// ...
			try {
				// Allows post-processing of the bean factory in context subclasses.
				postProcessBeanFactory(beanFactory);
				StartupStep beanPostProcess = this.applicationStartup.start("spring.context.beans.post-process");
				// Invoke factory processors registered as beans in the context.
				invokeBeanFactoryPostProcessors(beanFactory);
				// Register bean processors that intercept bean creation.
				registerBeanPostProcessors(beanFactory);
				beanPostProcess.end();
				// Initialize message source for this context.
				initMessageSource();
				// Initialize event multicaster for this context.
				initApplicationEventMulticaster();
				// Initialize other special beans in specific context subclasses.
				onRefresh();
				// Check for listener beans and register them.
				registerListeners();
				// Instantiate all remaining (non-lazy-init) singletons.
				finishBeanFactoryInitialization(beanFactory);
				// Last step: publish corresponding event.
				finishRefresh();
			}
			// ...
		}
	  }  
      ```
      - 과정
        - refresh 준비 단계
        - BeanFactory 준비 단계
        - BeanFactory의 후처리 진행
        - BeanFactoryPostProcessor 실행
        - BeanPostProcessor 등록
        - MessageSource 및 Event Multicaster 초기화
        - onRefresh(웹 서버 생성)
        - ApplicationListener 조회 및 등록
        - 빈들의 인스턴스화 및 후처리
        - refresh 마무리 단계
      - BeanFactoryPostProcessor 실행
        - BeanFactoryPostProcessor는 빈을 탐색하는 것처럼 빈 팩토리가 준비된 후에 해야하는 후처리기들을 실행된다. 대표적으로 싱글톤 객체로 인스턴스화할 빈을 탐색하는 작업 진행
        - PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors 이 실제 작업하는 메소드로 postProcessBeanDefinitionRegistry, processConfigBeanDinitions 메소드를 타고 들어가면
          - ConfigurationClassParser의 parse 메소드 실행, parsing 하는 작업 중에 ComponentScanAnnotationParser.parse 메소드까지 찾아 들어가면, Scanner에 basePackage를 기반으로 scan을 진행한다.
          - AbstractBeanFactory.doGetBean 메소드를 살펴보면 빈이 아직 등록되지 않았다면 빈을 생성하는 것을 볼 수 있다.
      - BeanPostProcessor
        - 빈들이 생성되고 나서 빈의 내용이나 빈 자체를 변경하기 위한 빈 후처리기인 BeanPostProcessor를 등록
        - 대표적으로 @Value, @PostConstruct, @Autowired 등이 BeanPostProcessor에 의해 처리되며 이를 위한 BeanPostProcessor 구현체들이 등록된다
      - onRefresh();
        - template method 로 구현체에 따른다.
        - themeSource = ResourceBundleThemeSource 로 세팅(GenericWebApplicationContext)
        - WebServerFactory 를 통해 Tomcat Server 객체를 만들고 설정 값들을 세팅한다.(ServletWebServerApplicationContext)
  - afterRefresh(context, applicationArguments);
    - refresh 후 처리를 진행하며 과거에는 애플리케이션 컨텍스트 생성 후에 초기화 작업을 위한 ApplicationRunner, CommandLineRunner를 호출하는 callRunners()가 내부에 존재했다.
    - 현재는 별도의 단계로 빠져서 메소드가 비어있는 상태
- 참고: 영한님의 스프링핵심원리-기본편 강의
