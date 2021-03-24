---
layout: post
title: Spring Bean Scope
summary: 김영한님 - 스프링 핵심 원리_기본편
author: devhtak
date: '2021-03-24 21:41:00 +0900'
category: Spring
---

#### 빈스코프란

- 싱글톤으로 생성되는 빈은 스프링 컨테이너의 시작과 함께 생성되어 스프링 컨테이너가 종료될 때까지 유지된다.
- 하지만 다양한 스코프를 지원한다.

- 빈 스코프

  |스코프|지원 범위|
  |---|---|
  |싱글톤|기본 스코프, 스프링 컨테이너의 시작과 종료까지 유지되는 가장 넓은 범위의 스코프|
  |프로토타입|스프링 컨테이너는 프로토타입 빈의 생성과 의존관계 주입까지만 관여하고 더는 관리하지 않는 매우 짧은 범위의 스코프|
  |(web)request|웹 요청이 들어오고 나갈 때까지 유지되는 스코프|
  |(web)session|웹 세션이 생성되고 종료될 때까지 유지되는 스코프|
  |(web)application|웹의 서블릿 컨텍스트와 같은 범위로 유지되는 스코프|
  
- 스코프 지정 방법
  - 컴포넌트 스캔 자동 등록
    
    ```java
    @Scope("prototype")
    @Scomponent
    public class PrototypeBean() {}
    ```
    
  - 수동 등록
    
    ```java
    @Scope("prototype")
    @Bean
    public PrototypeBean prototypeBea() {}
    ```

#### 프로토타입 스코프

- 싱글톤 빈을 조회하면 스프링 컨테이너는 항상 같은 인스턴스의 스프링 빈을 반환한다.
- 프로토타입 스코프를 스프링 컨테이너에 조회하면 스프링 컨테이너는 항상 새로운 인스턴스를 생성해서 반환한다.

- 싱글톤 빈 요청
  - 싱글톤 스코프의 빈을 스프링 컨테이너에 요청한다.
  - 스프링 컨테이너는 본인이 관리하는 스프링 빈을 반환한다.
  - 이후에 스프링 컨테이너에 같은 요청이 와도 같은 객체 인스턴스의 스프링 빈을 반환한다.

- 프로토타입 빈 요청
  - 프로토타입 스코프의 빈을 스프링 컨테이너에 요청한다.
  - 스프링 컨테이너는 이 시점에 프로토타입 빈을 생성하고, 필요한 의존관계를 주입한다.
  - 스프링 컨테이너는 생성한 프로토타입 빈을 클라이언트에 반환한다.
  - 이후에 스프링 컨테이너에 같은 요청이 오면 항상 새로운 프로토타입 빈을 생성해서 반환한다.

- 정리
  - 스프링 컨테이너는 프로토타입 빈을 생성하고, 의존관계 주입, 초기화까지만 처리한다.
  - 프로토타입 빈을 관리할 책임은 프로토타입 빈을 받은 클라이언트에 있다. 그래서 @PostDestroy 같은 종료 메서드가 호출되지 않는다.

- 예제
  - 싱글톤 스코프 빈 테스트
    
    ```java
    public class SingletonTest {	
        @Test
        void singletonBeanFind() {
            AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SingletonBean.class);

            SingletonBean singletonBean1 = context.getBean(SingletonBean.class);
            SingletonBean singletonBean2 = context.getBean(SingletonBean.class);

            assertEquals(singletonBean1, singletonBean2);

            context.close();
        }

        @Scope("singleton")
        static class SingletonBean {
            @PostConstruct
            public void init() {
                System.out.println("Singleton.init");
            }

            @PreDestroy
            public void destroy() {
                System.out.println("Singleton.destroy");
            }
        }
    }
    ```
    ```
    Singleton.init
    10:10:33.158 [main] DEBUG org.springframework.context.annotation.AnnotationConfigApplicationContext - Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@78123e82, started on Wed Mar 24 10:10:33 KST 2021
    Singleton.destroy
    ```
    - SingletonBean에 @Component 등록을 하지 않은 것은 ApplicationContext를 생성할 때 인자로 입력하였기 때문에 기본적으로 ComponentScan 역할을 하기 때문이다. 
    - 싱글톤 객체는 동일한 객체이며, 생성 및 종료 한번 호출된다.
  
  - Prototype 빈 테스트
    
    ```java
    public class PrototypeTest {
        @Test
        void prototypeTest() {
            AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(PrototypeBean.class);

            PrototypeBean prototypeBean1 = context.getBean(PrototypeBean.class);
            PrototypeBean prototypeBean2 = context.getBean(PrototypeBean.class);

            assertNotEquals(prototypeBean1, prototypeBean2);
            context.close();  
        }

        @Scope("prototype")
        static class PrototypeBean {
            @PostConstruct
            public void init() {
                System.out.println("Prototype.init");
            }

            @PreDestroy
            public void destroy() {
                System.out.println("Prototype.destroy");
            }
        }
    }
    ```
    ```
    Prototype.init
    Prototype.init
    10:12:00.973 [main] DEBUG org.springframework.context.annotation.AnnotationConfigApplicationContext - Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@78123e82, started on Wed Mar 24 10:12:00 KST 2021
    ```
    - PrototypeBean을 생성할 때마다 초기화 함수가 호출되며, 종료 될 때 @PreDestroy 함수가 호출되지 않는다.
    - Prototype으로 빈을 생성할 때 @PreDestroy가 호출되지 않으므로, 리소스 반납 등 빈이 종료될 때에 해야할 부분을 클라이언트가 처리해 주어야 한다.
    - prototypeBean1, prototypeBean2 은 다르다.

#### 프로토타입 스코프 - 싱글톤 빈과 함께 사용시 문제점

- 스프링 컨테이너에 프로토타입 스코프의 빈을 요청하면 항상 새로운 객체 인스턴스를 생성하여 반환한다.
- 하지만, 싱글톤 스코프의 빈과 함께 사용할 때 의도한 대로 잘 동작하지 않는다.

- 예시
  - Prototype 스코프의 빈인 PrototypeBean을 갖는 싱글톤 스코프의 빈 ClientBean 이 있다.
  - ClientBean은 싱글톤이므로 스프링 컨테이너 생성 시점에 함께 생성되고, 의존관계 주입도 발생한다.
  - 이 때, 의존관계 주입을 받는 PrototypeBean은 스코프가 Prototype이지만, ClientBean이 생성되는 단 한번만 의존관계를 주입받기 때문에, 여러 ClientBean 객체를 생성해도 동일한 PrototypeBean을 사용하게 된다.
  
  - 테스트 코드
    
    ```java
    public class SingletonPrototypeTest {	
        @Test
        void singletonPrototypeTest() {
            AnnotationConfigApplicationContext context = 
                new AnnotationConfigApplicationContext(ClientBean.class, PrototypeBean.class);

            ClientBean clientBean1 = context.getBean(ClientBean.class);
            assertEquals(1, clientBean1.logic());

            ClientBean clientBean2 = context.getBean(ClientBean.class);
            assertEquals(2, clientBean2.logic());

            context.close();
        }

        @Scope("singleton")
        static class ClientBean {
            private final PrototypeBean prototypeBean;
            @Autowired
            public ClientBean(PrototypeBean prototypeBean) {
                this.prototypeBean = prototypeBean;
            }
            public int logic() {
                prototypeBean.addCount();
                return prototypeBean.getCount();
            }
            @PostConstruct
            public void init() {
                System.out.println("ClientBean.init");
            }
            @PreDestroy
            public void destory() {
                System.out.println("ClientBean.destroy");
            }
        }

        @Scope("prototype")
        static class PrototypeBean {
            private int count;
            public PrototypeBean() {
                this.count = 0;
            }
            public void addCount() {
                this.count += 1;
            }
            public int getCount() {
                return this.count;
            }
            @PostConstruct
            public void init() {
                System.out.println("PrototypeBean.init");
            }
            @PreDestroy
            public void destroy() {
                System.out.println("PrototypeBean.destroy");
            }
        }
    }    
    ```
    ```
    PrototypeBean.init
    11:03:19.669 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Autowiring by type from bean name 'singletonPrototypeTest.ClientBean' via constructor to bean named 'singletonPrototypeTest.PrototypeBean'
    ClientBean.init
    11:03:19.717 [main] DEBUG org.springframework.context.annotation.AnnotationConfigApplicationContext - Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@78123e82, started on Wed Mar 24 11:03:19 KST 2021
    ClientBean.destroy
    ```
    - ClientBean을 생성하기 전에 PrototypeBean을 생성하여 주입받는다.
    - ClientBean 빈을 활용하여 clientBean1, clientBean2 객체를 생성할 때에는 이미 스프링 컨테이너가 관리하고 있는 빈을 주입하기 때문에 한번만 생성된다.
    - 그래서 PrototypeBean 또한 하나의 객체로 사용되기 때문에 count의 값이 계속 커진다.

#### 프로토타입 스코프 - 싱글톤 빈과 함께 사용시 Provider로 문제 해결

- 해결방법 1. 스프링 컨테이너에 요청
  - 가장 간단한 방법은 싱글톤 빈이 프로토타입 빈을 사용할 때마다 스프링 컨테이너에 새로 요청하는 것이다.
  
  ```java
  @Scope("singleton")
	static class ClientBean {
      private final ApplicationContext applicationContext;
      @Autowired
      public ClientBean(ApplicationContext applicationContext) {
        this.applicationContext = applicationContext;
      }
      public int logic() {
          PrototypeBean prototypeBean = applicationContext.getBean(PrototypeBean.class);
          prototypeBean.addCount();
          return prototypeBean.getCount();
      }
      @PostConstruct
      public void init() {
          System.out.println("ClientBean.init");
      }
      @PreDestroy
      public void destory() {
          System.out.println("ClientBean.destroy");
      }
  }
  ```
  ```
  ClientBean.init
  PrototypeBean.init
  PrototypeBean.init
  11:25:20.432 [main] DEBUG org.springframework.context.annotation.AnnotationConfigApplicationContext - Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@78123e82, started on Wed Mar 24 11:25:20 KST 2021
  ClientBean.destroy
  ```
    - 의존관계를 외부에서 주입(DI) 받는 것이 아닌 직접 필요한 의존관게를 찾는 것을 Dependency Lookup(DL) 의존관계 조회(탐색)이라 한다.
    - 그런데 이렇게 스프링의 애플리케이션 컨텍스트 전체를 주입받게 되면, 스프링 컨테이너에 종속적인 코드가 되고, 단위 테스트도 어려워진다.
    - 지금 필요한 기능은 지정한 프로토타입 빈을 컨테이너에서 대신 찾아주는 DL 기능을 제공하는 것이 있으면 된다.

- 해결 방법 2. ObjectFactory, ObjectProvider
  - 스프링에서 만든 DL 서비스를 제공하는 것이 ObjectProvider이다.
  - 과거에는 ObjectFactory였는데, 여러 편의 기능을 추가하여 ObjectProvider가 만들어 졌다.
  
  ```java
  private final ObjectProvider<PrototypeBean> provider;
		
  @Autowired
  public ClientBean(ObjectProvider<PrototypeBean> provider) {
      this.provider = provider;
  }

  public int logic() {
      PrototypeBean prototypeBean = provider.getObject();
      prototypeBean.addCount();
      return prototypeBean.getCount();
  }
  ```
    - provider.getObject()를 통해서 항상 새로운 프로토타입 빈이 생성된다.
    - ObjectProvider의 getObject() 메서드는 내부에서 스프링 컨테이너를 통해 지정한 타입의 빈을 찾아서 반환한다.(DL)
    - 기능이 단순하여 단위테스트, mock 코드를 만들기 쉬워진다.
  
  - 특징
    - ObjectFactory: 기능이 단순, 별도의 라이브러리 필요 없음, 스프링에 의존
    - ObjectProvider: ObjectFactory 상속, 옵션, 스트림 처리등 편의 기능이 많고, 별도의 라이브러리가 필요 없으며 스프링에 의존

- 해결 방법3. JSR-330 Provider
  - 마지막 방법은 javax.inject.Provider라는 JSR-330 자바 표준을 사용하는 방법이다.
  - 의존성을 추가해야 한다.
    - maven
      ```
      <dependency>
          <groupId>javax.inject</groupId>
          <artifactId>javax.inject</artifactId>
          <version>1</version>
      </dependency>
      ```
  - 예시 코드
    ```java
    private final Provider<PrototypeBean> provider;

    @Autowired
    public ClientBean(Provider<PrototypeBean> provider) {
        this.provider = provider;
    }

    public int logic() {
        PrototypeBean prototypeBean = provider.get();
        prototypeBean.addCount();
        return prototypeBean.getCount();
    }
    ```
    - 실행해보면 provider.get() 을 통해서 항상 새로운 프로토타입 빈이 생성되는 것을 확인할 수 있다.
    - provider 의 get() 을 호출하면 내부에서는 스프링 컨테이너를 통해 해당 빈을 찾아서 반환한다. (DL)
    - 자바 표준이고, 기능이 단순하므로 단위테스트를 만들거나 mock 코드를 만들기는 훨씬 쉬워진다.
    - Provider 는 지금 딱 필요한 DL 정도의 기능만 제공한다.
  
  - 특징
    - get() 메서드 하나로 기능이 매우 단순하다.
    - 별도의 라이브러리가 필요하다.
    - 자바 표준이므로 스프링이 아닌 다른 컨테이너에서도 사용할 수 있다.

```
참고: 스프링이 제공하는 메서드에 @Lookup 애노테이션을 사용하는 방법도 있지만, 이전 방법들로 충분하고, 고려해야할 내용도 많아서 생략하겠다.
```
```
참고: 실무에서 자바 표준인 JSR-330 Provider를 사용할 것인지, 아니면 스프링이 제공하는 ObjectProvider를 사용할 것인지 고민이 될 것이다.
  ObjectProvider는 DL을 위한 편의 기능을 많이 제공해주고 스프링 외에 별도의 의존관계 추가가 필요 없기 때문에 편리하다. 만약(정말 그럴일은 거의 없겠지만) 코드를 스프링이 아닌 다른 컨테이너에서도 사용할 수 있어야 한다면 JSR-330 Provider를 사용해야한다. 
  스프링을 사용하다 보면 이 기능 뿐만 아니라 다른 기능들도 자바 표준과 스프링이 제공하는 기능이 겹칠때가 많이 있다.
  대부분 스프링이 더 다양하고 편리한 기능을 제공해주기 때문에, 특별히 다른 컨테이너를 사용할 일이 없다면, 스프링이 제공하는 기능을 사용하면 된다.
```
  
** 출처: 김영한님 - 스프링 핵심 원리 기본편 강의
