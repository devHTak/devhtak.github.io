---
layout: post
title: Spring Container and Bean
summary: 김영한님 - 스프링 핵심 원리_기본편
author: devhtak
date: '2021-03-09 21:41:00 +0900'
category: Spring
---

#### Spring Container

```java
ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
```

- ApplicationContext를 스프링 컨테이너라 한다.
- 기존에는 개발자가 AppConfig를 사용해서 직접 객체를 생성하고 DI를 했지만, 이제는 스프링 컨테이너를 통해서 사용한다.
  ```java
  MemberService memberServiceBefore = appConfig.memberService(); // 기존 방식
  MemberService memberServiceAfter = context.getBean("memberService", MemberService.class); // Spring Container활용
  ```
- 스프링 컨테이너는 @Configuration이 붙은 AppConfig를 설정(구성) 정보로 사용한다.
  - 여기서 @Bean이라 적힌 메서드를 모두 호출하여 반환된 객체를 스프링 컨테이너에 등록
  - 이렇게 스프링 컨테이너에 등록된 객체를 Spring Bean이라 한다.
- 스프링 빈은 @Bean이 붙은 메서드의 명을 스프링 빈의 이름으로 사용한다. (memberService, orderService)
- 이전에는 개발자가 필요한 객체를 AppConfig를 사용해서 직접 조회했지만, 이제부터는 스프링 컨테이너를 통해서 필요한 스프링 빈(객체)를 찾아야 한다.
  - 스프링 빈은 applicationContext.getBean() 메서드를 사용해여 찾을 수 있다.
- 기존에는 개발자가 직접 자바코드로 모든 것을 했다면 이제부터는 스프링 컨테이너에 객체를 스프링 빈으로 등록하고, 스프링 컨테이너에서 스프링 빈을 찾아서 사용하도록 변경하였다.


#### Spring Container 생성

```java
ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
```
- ApplicationContext는 인터페이스이다.
- Spring Container는 XML을 기반으로 만들 수 있고, 애노테이션 기반의 자바 설정 클래스로 만들 수 있다.
- 직전에 AppConfig를 사용했던 방식이 애노테이션 기반의 자바 설정 클래스로 스프링 컨테이너를 만든 것
- 자바 설정 클래스를 기반으로 스프링 컨테이너(ApplicationContext)를 만들어보자
  - new AnnotationConfigApplicationContext(AppConfig.class);
  - 해당 클래스는 ApplicationContext 인터페이스의 구현체다.

- 스프링 컨테이너 생성 과정
  - 스프링 컨테이너 생성
    - Spring Container 생성(AppConfig.class) 
    - Spring Container는 AppConfig.class에 구성정보를 활용하여 스프링 빈 저장소를 만든다.
    - Spring Container 안에 있는 스프링 빈 저장소에는 빈 이름과 빈 객체로 key, value 형태로 저장된다.
  - 스프링 빈 등록
    - 스프링 컨테이너에는 파라미터로 넘어온 설정 클래스 정보를 사용해서 스프링 빈을 등록한다.
    - 빈 이름
      - 빈 이름은 기본 메서드 이름을 사용한다.
      - 빈 이름을 직접 부여할 수도 있다. ex) @Bean(name="memberService")
      - 주의!! 빈 이름은 항상 다른 이름으로 부여해야 한다. 같은 이름을 부여하면 다른 빈이 무시되거나 기존 빈을 덮어버리거나 설정에 따라 오류 발생
  - 스프링 빈 의존관계 설정 - 준비
  - 스프링 빈 의존관계 설정 - 완료
    - 스프링 컨테이너는 설정 정보를 참고해서 의존관계를 주입한다.
    - 단순히 자바 코드를 호출하는 것 같지만, 차이가 있다. 이 차이는 뒤에 싱글톤 컨테이너에서 설명한다.
    - ex) memberService -> memberRepository <- orderService -> discountPolicy
      - memberService에서 memberRepository를 생성한다.
      - orderService는 memberRepository와 discountPolicy를 생성한다.
  
- 스프링 빈 의존관계 설정 준비 및 완료에서 참고할 부분
  - 스프링은 빈을 생성하고, 의존관계를 주입하는 단계가 나눠져 있다.
  - 그런데 이렇게 자바 코드로 스프링 빈을 등록하면 생성자를 호출하면서 의존관계 주입도 한번에 처리된다.
  - 여기서는 개념적으로 나누어 설명했다.
  
#### 컨테이너에 등록된 모든 빈 조회

```java
public class ApplicationContextInfoTest {
    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
    @Test
    @DisplayName("모든 빈 조회")
    void findAllBean() {
        String[] beanName = ac.getBeanDefinitionNames();

        Arrays.stream(beanName).forEach((b) -> {
            Object bean = ac.getBean(b);
            System.out.println("name: " + b + ", object: " + bean);
        });
    }

    @Test
    @DisplayName("애플리케이션 빈 조회")
    void findApplicationBean() {
        String[] beanName = ac.getBeanDefinitionNames();

        Arrays.stream(beanName).forEach((b) -> {
            BeanDefinition beanDefinition = ac.getBeanDefinition(b);
            if(beanDefinition.getRole() == BeanDefinition.ROLE_APPLICATION) {
                Object bean = ac.getBean(b);
                System.out.println("name: " + b + ", object: " + bean);
            }
        });
    }
}
```
- findAllBean() -> 모든 빈 출력하기
  - 실행하면 스프링에 등록된 모든 빈 정보를 출력할 수 있다.
  - ac.getBeanDefinitionNames(): 스프링에 등록된 모든 빈 이름을 조회한다.
  - ac.getBean(): 빈 이름으로 빈 객체(인스턴스)를 조회한다.

- findApplicationBean() -> 애플리케이션 빈 출력하기
  - 스프링이 내부에서 사용하는 빈은 제외하고, 내가 등록한 빈만 출력
  - 스프링이 내부에서 사용하는 빈은 getRole()로 구분할 수 있다.
    - BeanDefinition.ROLE_APPLICATOIN: 일반 사용자가 정의한 빈
    - BeanDefinition.ROLE_INFRASTRUCTURE: 스프링 내부에서 사용하는 빈

#### 스프링 빈 조회 - 기본

```java
public class ApplicationContextInfoTest {	
    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);	
    @Test
    @DisplayName("빈 이름으로 조회")
    void findBeanByName() {
        MemberService memberService = ac.getBean("memberService", MemberService.class);
        assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
    }

    @Test
    @DisplayName("빈 이름으로 조회 실패")
    void findBeanByNameFailed() {
        assertThrows(NoSuchBeanDefinitionException.class, ()-> {
            ac.getBean("xxx", MemberService.class);
        });
    }

    @Test
    @DisplayName("이름없이 타입으로 조회")
    void findBeanByType() {
        MemberService memberService = ac.getBean(MemberService.class);
        assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
    }

    @Test
    @DisplayName("구체 타입으로 조회")
    void findBeanByDefiniteType() {
        MemberServiceImpl memberServiceImpl = ac.getBean(MemberServiceImpl.class);
        assertNotNull(memberServiceImpl);
    }
}
```

- 스프링 컨테이너에서 스프링 빈을 찾는 가장 기본적인 조회 방법
  - ac.getBean(빈이름, 타입)
  - ac.getBean(타입)
  - 조회 대상 스프링이 빈이 없으면 예외 발생
    - NoSuchBeanDefinitionException: No bean named 'XXXX' available

#### 스프링 빈 조회 - 동일한 타입이 둘 이상

```java
public class ApplicationContextInfoTest {
    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
    
    @Test
    @DisplayName("타입 조회 시 중복 타입이 있는 경우 오류 발생")
    void findBeanByTypeDuplicate() {
        ac = new AnnotationConfigApplicationContext(SameBeanConfig.class);
        assertThrows(NoUniqueBeanDefinitionException.class, ()-> {
            ac.getBean(MemberRepository.class);
        });
    }

    @Test
    @DisplayName("타입 조회 시 중복 타입이 있는 경우 이름으로 조회")
    void findDuplicateBeanByName() {
        ac = new AnnotationConfigApplicationContext(SameBeanConfig.class);
        MemberRepository memberRepository = ac.getBean("memberRepository1", MemberRepository.class);
        assertNotNull(memberRepository);
    }

    @Test
    @DisplayName("특정 타입 모두 조회")
    void findAllDuplicateBeans() {
        ac = new AnnotationConfigApplicationContext(SameBeanConfig.class);

        Map<String, MemberRepository> memberRepositories = ac.getBeansOfType(MemberRepository.class);
        memberRepositories.keySet().forEach((name) -> {
            System.out.println("key: " + name + ", bean: " + memberRepositories.get(name));
        });

        assertEquals(2, memberRepositories.size());
    }

    @Configuration
    static class SameBeanConfig {
        @Bean 
        public MemberRepository memberRepository1() {
            return new InMemoryMemberRepository();
        }
        @Bean
        public MemberRepository memberRepository2() {
            return new InMemoryMemberRepository();
        }
    }
}
```

- 같은 타입의 빈이 두개 이상 등록
  - NoUniqueBeanDefinitionException 발생, @Qualifier, @Primary 등을 활용하여 해결 필요
  - ApplicationContext를 통하여 여러 상황에 맞게 조회할 수 있다. 
- 타입으로 조회시 같은 타입의 스프링 빈이 둘 이상이면 오류가 발생한다. 이때는 빈 이름을 지정하자.
- ac.getBeanOfType()을 사용하면 해당 타입의 모든 빈을 조회할 수 있다. return 타입은 Map<String, Object>

#### 스프링 빈 조회 - 상속 관계

- 부모 타입으로 조회하면, 자식 타입도 함께 조회한다.
- 그래서 Object타입으로 조회하면 모든 스프링 빈을 조회한다.

```java
@Configuration
static class TestConfig {
    @Bean
    public DiscountPolicy rateDiscountPolicy() {
      return new RateDiscountPolicy();
    }

    @Bean
    public DiscountPolicy fixDiscountPolicy() {
      return new FixedDiscountPolicy();
    }
}

@Test
@DisplayName("부모 타입으로 조회, 자식이 둘 이상 있으면 중복 오류 발생")
void findBeanByParentTypeDuplicate() {
    ac = new AnnotationConfigApplicationContext(TestConfig.class);
    assertThrows(NoUniqueBeanDefinitionException.class, () -> {
        ac.getBean(DiscountPolicy.class);
    });
}

@Test
@DisplayName("부모 타입으로 조회, 자식이 둘 이상 있으면 빈 이름을 지정하면 된다.")
void findBeanByParentTypeBeanName() {
    ac = new AnnotationConfigApplicationContext(TestConfig.class);

    DiscountPolicy rateDiscountPolicy = ac.getBean("rateDiscountPolicy", DiscountPolicy.class);
    assertThat(rateDiscountPolicy).isInstanceOf(RateDiscountPolicy.class);
}

@Test
@DisplayName("특정 하위 타입으로 조회")
void findBeanByChildType() {
    ac = new AnnotationConfigApplicationContext(TestConfig.class);

    RateDiscountPolicy rateDiscountPolicy = ac.getBean(RateDiscountPolicy.class);
    assertNotNull(rateDiscountPolicy);
}

@Test
@DisplayName("부모 타입으로 자식 타입 모두 조회")
void findAllBeanByParentType() {
    ac = new AnnotationConfigApplicationContext(TestConfig.class);
    Map<String, DiscountPolicy> beansOfType = ac.getBeansOfType(DiscountPolicy.class);

    assertEquals(2, beansOfType.size());
    beansOfType.keySet().stream().forEach((k)-> {
        assertThat(beansOfType.get(k)).isInstanceOf(DiscountPolicy.class);
    });
}

@Test
@DisplayName("Object 타입으로 자식 타입 모두 조회")
void findAllBeanByObjectType() {
    ac = new AnnotationConfigApplicationContext(TestConfig.class);
    Map<String, Object> beansOfType = ac.getBeansOfType(Object.class);

    beansOfType.keySet().stream().forEach((k)-> {
        assertThat(beansOfType.get(k)).isInstanceOf(Object.class);
    });
}
```
- TestConfig에서 
- findBeanByParentTypeDuplicate: 단순히 부모타입으로 조회하면, 여러 자식타입이 있기 때문에 예외가 발생한다.
- findBeanByParentTypeBeanName: 필요로하는 자식타입을 지정하여 조회
- findBeanByChildType: 구체적인 자식 타입으로 조회
- findAllBeanByParentType: getBeansOfType을 통해 부모 타입으로 모두 조회하여 Map 객체로 가져온다.
- findAllBeanByObjectType: Object 타입으로 조회하면 스프링에 등록된 모든 빈을 가져온다.

#### BeanFactory와 ApplicationContext

- BeanFactory와 ApplicationContext 상속 구조
  - BeanFactory <- ApplicationContext <- AnnotationConfigApplicationContext
    - BeanFactory: interface
    - ApplicationContext: interface

- BeanFactory
  - 스프링 컨테이너의 최상위 인터페이스
  - 스프링 빈을 관리하고 조회하는 역할을 담당
  - getBean()을 제공한다.
  - 지금까지 우리가 사용했던 대부분의 기능(빈 조회)은 BeanFactory에서 제공하는 기능

- ApplicationContext
  - BeanFactory 기능을 모두 상속받아 제공한다.
  - 빈을 관리하고 검색하는 기능을 BeanFactory가 제공하고, 애플리케이션을 개발할 때 필요하는 수 많은 부가기능을 제공한다.
  
  ```java
  public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,MessageSource, ApplicationEventPublisher, ResourcePatternResolver
  ```
  - MessageSource : 국제화 기능
    - 국가별 언어를 제공하는 기능
  - EnvironmentCapable: 환경 변수
    - 로컬, 개발, 운영 등 환경을 구분하여 처리 가능
  - ApplicationEventPublisher: 애플리케이션 이벤트
    - 이벤트를 발행하고 구독하는 모델을 편리하게 지원
  - ResourceLoader: 편리한 리소스 조회
    - 파일, 클래스패스, 외부 등에서 리소스를 편리하게 조회

#### 다양한 설정 형식 지원 - 자바 코드, XML

- 스프링 컨테이너는 다양한 형식의 설정 정보를 받아드릴 수 있게 유연하게 설계되어 있다.
- 상속 구조
  - BeanFactory <- ApplicationContext <- AnnotationConfigApplicationContext(AppConfig.class), GenericXmlApplicationContext(appConfig.xml), XxxApplicationContext(appConfig.xxx)
    - BeanFactory: interface
    - ApplicationContext: interface

- Annotation 기반 자바 코드 설정 사용
  - 예제에서 계속 사용
  - new AnnotationConfigApplicationContext(AppConfig.class)
  - AnnotationConfigApplicationContext 클래스를 사용하면서 자바 코드로된 설정 정보를 넘기면 된다.

- XML 설정 사용
  - 아직 레거시 프로젝트들에서는 XML로 되어있고, 또 XML을 사용하면 컴파일 없이 빈 설정 정보를 변경할 수 있는 장점이 있다.
  - GenericXmlApplicationContext를 사용하여 xml 설정파일을 넘기면 된다.

  - XmlAppConfig 사용 자바 코드
    - 테스트 코드 먼저 작성
      - GenericXmlApplicationContext를 사용하여 XML 설정 파일을 읽어와 빈을 구성한다.
      ```java
      @Test
      void xmlAppContext() {
          ApplicationContext ac = new GenericXmlApplicationContext("appConfig.xml");

          MemberService memberService = ac.getBean("memberService", MemberService.class);
          assertThat(memberService).isInstanceOf(MemberService.class);
      }
      ```
      
    - appConfig.xml
      - src/main/resources/appConfig.xml 로, resources 밑에 xml 파일을 둔다.
      ```xml
      <?xml version="1.0" encoding="UTF-8"?>
      <beans xmlns="http://www.springframework.org/schema/beans"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

        <bean id="memberService" class="com.example.member.service.MemberServiceImpl">
          <constructor-arg name="memberRepository" ref="memberRepository"></constructor-arg>
        </bean>

        <bean id="memberRepository" class="com.example.member.repository.InMemoryMemberRepository"></bean>

        <bean id="orderService" class="com.example.order.service.OrderServiceImpl">
          <constructor-arg name="memberService" ref="memberService" ></constructor-arg>
          <constructor-arg name="discountPolicy" ref="discountPolicy"></constructor-arg>		
        </bean>

        <bean id="discountPolicy" class="com.example.order.discount.RateDiscountPolicy"></bean>
      </beans>
      ```
      - 추가적으로 정보가 필요하면 스프링 공식 레퍼런스를 참고

#### 스프링 빈 설정 메타 정보 - BeanDefinition

- 스프링은 다양한 설정 형식을 지원하기 위해 BeanDefinition이라는 추상화를 사용했다.
- 쉽게 생각하면 역할과 구현을 개념적으로 나눈 것이다.
  - xml을 읽어서 BeanDefinition을 만들면 된다.
  - 자바 코드를 읽어서 BeanDefinition을 만들면 된다.
  - 자바, XML이 아니여도 BeanDefinition을 알면 도니다.
- BeanDefinition을 빈 설정 메타정보라고 한다.
  - @Bean, <bean> 당 각각 하나씩 메타정보가 생성된다.
- 스프링 컨테이너는 이 메타정보를 기반으로 스프링 빈을 생성한다.
  - Spring Container -> Bean Definition <- AppConfig.class, appConfig.xml, appConfig.xxx

- 코드 레벨로 들어가보자
  - Java 기반
    - ApplicationContext 구현체인 AnnotationConfigApplicationContext는 AnnotatedBeanDefinitionReader를 의존하고 있다.
      ```java
      public class AnnotationConfigApplicationContext extends GenericApplicationContext implements AnnotationConfigRegistry {
          private final AnnotatedBeanDefinitionReader reader;
          private final ClassPathBeanDefinitionScanner scanner;
          // ...
      ```
    - AnnotatedBeanDefinitionReader는 AppConfig.class에 설정 정보를 읽어서 BeanDefinition(빈 메타 정보)를 생성한다.
  - XML 기반
    - GenericXmlApplicationContextsms는 XmlBeanDefinitionReader를 의존하고 있다.
    - XmlBeanDefinitionReader는 appConfig.xml에 설정 정보를 읽어서 BeanDefinition(빈 메타 정보)를 생성한다.

- BeanDefinition에 세팅된 정보 확인
```java
public class BeanDefinitionTest {
    AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
    
    @Test
    @DisplayName("빈 설정 메타정보 확인")
    void findApplicationBean() {
        String[] beanDefinitionNames = applicationContext.getBeanDefinitionNames();
        Arrays.stream(beanDefinitionNames).forEach((name) -> {
            BeanDefinition beanDefinition = applicationContext.getBeanDefinition(name);

            if(beanDefinition.getRole() == BeanDefinition.ROLE_APPLICATION) {
                System.out.println("beanDefinitionName = " + name + ", beanDefinition = " + beanDefinition);
            }
        });
    }
}
```
- 자체적으로 생성한 빈 확인
  ```
  // AppConfig.class
  beanDefinitionName = appConfig, 
  beanDefinition = Generic bean: class [com.example.AppConfig$$EnhancerBySpringCGLIB$$34464e38]; 
  scope=singleton; abstract=false; lazyInit=null; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; 
  factoryBeanName=null; factoryMethodName=null; initMethodName=null; destroyMethodName=null
  // MemberService.class  
  beanDefinitionName = memberService, 
  beanDefinition = Root bean: class [null]; 
  scope=; abstract=false; lazyInit=null; autowireMode=3; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=appConfig; 
  factoryMethodName=memberService; initMethodName=null; destroyMethodName=(inferred); defined in com.example.AppConfig
  // OrderService.class
  beanDefinitionName = orderService, 
  beanDefinition = Root bean: class [null]; 
  scope=; abstract=false; lazyInit=null; autowireMode=3; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=appConfig; 
  factoryMethodName=orderService; initMethodName=null; destroyMethodName=(inferred); defined in com.example.AppConfig
  ```
  - BeanDefinition 정보
    - BeanClassName: 생성할 빈의 클래스 명(자바 설정처럼 팩토리 역할의 빈을 사용하면 없음)
    - factoryBeanName: 팩토리 역할의 빈을 사용할 경우 이름, 예) appConfig
    - factoryMethodName: 빈을 생성할 팩토리 메서드 지정 예) memberService
    - Scope: 싱글톤(기본값)
    - lazyInit: 스프링 컨테이너를 생성할 때 빈을 생성하는 것이 아닌, 실제 빈을 사용할 때까지 최대한 생성을 지연처리하는 지 여부
    - InitMethodName: 빈을 생성하고, 의존 관계를 적용한 뒤에 호출되는 초기화 메서드 명
    - DestroyMethodName: 빈의 생명주기가 끝나서 제거하기 직전에 호출되는 메서드 명
    - Constructor arguments, Properties: 의존관계 주입에서 사용 (자바 설정처럼 팩토리 역할의 빈을 사용하면 없음)
  - FactoryBean으로 AppConfig로 등록되어 있다. 
    - 현재 AppConfig.java에서 등록한 방식처럼 @Bean을 메서드로 등록하는 방식을 FactoryBean 을 통해 등록하는 방식

- 출처: 김영한님의 스프링 핵심 기본편 강의 및 강의자료
