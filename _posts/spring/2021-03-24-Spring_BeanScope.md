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
    @Component
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

#### 웹 스코프

- 웹 스코프의 특징
  - 웹 스코프는 웹 환경에서만 동작한다.
  - 웹 스코프는 프로토타입과 다르게 스프링이 해당 스코프의 종료시점까지 관리한다. 따라서 종료 메서드가 호출된다.

- 웹 스코프의 종류
  
  |웹스코프|설명|
  |---|---|
  |request|HTTP 요청 하나가 들어오고 나갈때까지 유지되는 스코프, 각각의 HTTP 요청마다 별도의 빈 인스턴스가 생성되고, 관리된다.|
  |session|HTTP Session과 동일한 생명주기를 가지는 스코프|
  |application|서블릿 컨텍스트(ServletContext) 와 동일한 생명주기를 가지는 스코프|
  |websocket|웹 소켓과 동일한 생명주기를 가지는 스코프|

  - HTTP request 요청 당 각각 할당되는 request scope

#### request 스코프 예제 만들기

- 웹 환경 필요
  - 웹 스코프는 웹 환경에서만 동작하므로 web 환경이 동작하도록 라이브러리 추가 필요
  - spring-boot-starter-web 라이브러리 추가 필요
  - 라이브러리를 차가하면 스프링 부트는 내장 톰캣 서버를 활용하여 웹 서버와 스프링을 함께 실행시킨다.
  - 스프링 부트는 웹 라이브러리가 없으면 AnnotationConfigApplicationContext를 기반으로 애플리케이션을 구동한다. 
  - 웹 라이브러리가 추가되면 웹과 관련된 설정과 환경이 필요하므로 AnnotationConfigServletWebServerApplicationContext 를 기반으로 애플리케이션을 구동한다.

- 예제

  - 동시에 여러 HTTP 요청이 오면 정확히 어떤 요청이 남긴 로그인지 구분하기 어렵다.
  - 이럴 때 사용하기 딱 좋은 것기 request 스코프이다.
  - UUID를 사용해 HTTP 요청을 구분하자. requestURL 정보도 추가로 넣어 어떤 URL을 요청해서 남은 로그인지 확인하는 예제
  
  - MyLogger.class
    ```java
    @Component
    @Scope(value = "request")
    public class MyLogger {
		private String uuid;
		private String requestURL;
		public void setRequestURL(String requestURL) {
			this.requestURL = requestURL;
		}
		public void log(String message) {
			System.out.println("["+uuid+"] "+requestURL+"] " + message);
		}	
		@PostConstruct
		public void init() {
			uuid = UUID.randomUUID().toString();
			System.out.println("[" + uuid + "] request scope bean create:" + this);
		}
		@PreDestroy 
		public void destroy() {
			System.out.println("[" + uuid + "] request scope bean close:" + this);
		}
    }    
    ```
    - 로그를 출력하기 위한 MyLogger 클래스
    - @Scope를 통해 request 스코프로 지정했다.
    - 이 빈은 HTTP 요청 당 하나씩 생성하고, HTTP 요청이 끝나는 시점에 소멸된다.

  - LogDemoController.java
    
    ```java
    @RestController
    public class LogDemoController {
		private final LogDemoService logDemoService;
		private final MyLogger myLogger;
		@Autowired
		public LogDemoController(LogDemoService logDemoService, MyLogger myLogger) {
			this.logDemoService = logDemoService;
			this.myLogger = myLogger;
		}
		@GetMapping("log-demo")
		public String logDemo(HttpServletRequest request) {
			String requestURL = request.getRequestURL().toString();
			myLogger.setRequestURL(requestURL);		
			myLogger.log("controller test");
			logDemoService.logic("testID");
			return "ok";
		}
    }
    ```
    
    - request 객체를 받아 URL을 가져온다.
    - 이렇게 받은 URL 등을 활용하여 로그를 남긴다.
    - requestURL 등에 로그를 남기는 부분은 공통처리가 가능한 스프링 인터셉터나 서블릿 필터 같은 곳을 활용하는 것이 좋다.

  - LogDemoService.java
    
    ```java
    @Service
    public class LogDemoService {
		private final MyLogger myLogger;
		@Autowired
		public LogDemoService(MyLogger myLogger) {
			this.myLogger = myLogger;
		}
		public void logic(String id) {
			myLogger.log("service id = " + id);
		}
    }
    ```
    
    - 비즈니스 로직이 있는 서비스 계층에서도 로그를 출력했다.
    - request scope을 사용하지 않고 파라미터로 해당 정보를 서비스 계층에 넘긴다면, 파라미터가 많아진다.
    - 더 큰 문제는 requestURL과 괕은 웹과 관련된 정보가 웹과 관련없는 서비스 계층까지 넘어가게 된다.
    - 웹과 관련된 부분은 컨트롤러까지만 사용해야 한다. 서비스 계층은 웹 기술에 종속되지 않고, 가급적 순수하게 유지하는 것이 유지보수 관점에서 좋다.
    - requestscope의 MyLogger 덕분에 해당 부분을 파라미터로 넘기지 않고, MyLogger의 멤버 변수에 저장해서 코드와 계층을 깔끔하게 유지할 수 있다.

  - 오류가 발생한다.
    ```
    Error creating bean with name 'myLogger': Scope 'request' is not active for the current thread; 
    consider defining a scoped proxy for this bean if you intend to refer to it from a singleton;
    ```
    - 스프링 애플리케이션을 실행하는 시점에 싱글톤 빈은 생성해서 주입이 가능하지만, request 스코프 빈은아직 생성되지 않는다.
    - 이 빈은 실제 고객의 요청이 와야 생성할 수 있다.
    - 즉, 톰캣에 띄울 때 요청이 없는 상태로 빈이 생성되지 않았다. 하지만 Controller와 Service 에서 MyLogger 빈을 주입받아야 되기 때문에 빈이 생성되지 않았다는 오류가 발생하게 된다.
    
#### 해결방법 1. Provider

- Provider를 활용하여 빈의 생성 시점(스코프)을 미룰 수 있다.
- LogDemoController.java
  
  ```java
	@RestController
	public class LogDemoController {

		private final LogDemoService logDemoService;
		private final ObjectProvider<MyLogger> myLogger;

		@Autowired
		public LogDemoController(LogDemoService logDemoService, ObjectProvider<MyLogger> myLogger) {
			this.logDemoService = logDemoService;
			this.myLogger = myLogger;
		}

		@GetMapping("log-demo")
		public String logDemo(HttpServletRequest request) {
			String requestURL = request.getRequestURL().toString();
			MyLogger logger = myLogger.getObject();
			logger.setRequestURL(requestURL);

			logger.log("controller test");
			logDemoService.logic("testID");
			return "ok";
		}
	}
  ```
- LogDemoService.java
  ```java
	@Service
	public class LogDemoService {
		private final ObjectProvider<MyLogger> myLoggerProvider;

		@Autowired
		public LogDemoService(ObjectProvider<MyLogger> myLoggerProvider) {
			this.myLoggerProvider = myLoggerProvider;
		}

		public void logic(String id) {
			MyLogger logger = myLoggerProvider.getObject();
			logger.log("service id = " + id);
		}
	}
  ```
  
  - ObjectProvider 덕분에 ObjectProvider.getObject() 를 호출하는 시점까지 request scope 빈의 생성을 지연할 수 있다.
  - ObjectProvider.getObject() 를 호출하시는 시점에는 HTTP 요청이 진행중이므로 request scope 빈의 생성이 정상 처리된다.
  - ObjectProvider.getObject() 를 LogDemoController , LogDemoService 에서 각각 한번씩 따로 호출해도 같은 HTTP 요청이면 같은 스프링 빈이 반환된다

- 결과
  ```
  [b0d7fc2c-f1e5-4392-9589-70299350c6a3] request scope bean create:com.example.web.MyLogger@5b37e6fd
  [b0d7fc2c-f1e5-4392-9589-70299350c6a3] http://localhost:8080/log-demo] controller test
  [b0d7fc2c-f1e5-4392-9589-70299350c6a3] http://localhost:8080/log-demo] service id = testID
  [b0d7fc2c-f1e5-4392-9589-70299350c6a3] request scope bean close:com.example.web.MyLogger@5b37e6fd
  [7f41c158-4693-4164-8bea-9e476bafd94f] request scope bean create:com.example.web.MyLogger@75b97525
  [7f41c158-4693-4164-8bea-9e476bafd94f] http://localhost:8080/log-demo] controller test
  [7f41c158-4693-4164-8bea-9e476bafd94f] http://localhost:8080/log-demo] service id = testID
  [7f41c158-4693-4164-8bea-9e476bafd94f] request scope bean close:com.example.web.MyLogger@75b97525
  ```
  - request 당 다른 UUID, MyLogger가 찍히는 것을 확인할 수 있다.
    
#### 해결방법 2. Scope와 Proxy

- Proxy 사용하기
  ```java
  @Component
  @Scope(value="request", proxyMode=ScopedProxyMode.TARGET_CLASS) 
  public class MyLogger {
  }
  ```
  - proxyMode=ScopedProxyMode.TARGET_CLASS 를 추가하였다.
    - 적용 대상이 인터페이스가 아닌 클래스면 TARGET_CLASS를 선택
    - 인터페이스인 경우 INTERFACES를 선택
  - 이렇게 하면 MyLogger의 가짜 프록시 클래스를 만들고, HTTP Request와 상관없이 가짜 프록시 클래스를 다른 빈에 미리 주입한다.

  ```java
  private final MyLogger myLogger;
  ```
  - ObjectProvider를 사용하지 않아도 된다.

- 웹 스코프와 프록시 동작 원리
  - 먼저 주입된 myLogger를 확인해보자.
    ```java
    System.out.println("myLogger = " + myLogger.getClass())
    ```
  - 출력 결과
    ```
    myLogger = class hello.core.common.MyLogger$$EnhancerBySpringCGLIB$$b68b726d
    ```
  - CGLIB이라는 라이브러리로 내 클래스를 상속 받은 가짜 프록시 객체를 만들어 주입한다.
    - @Scope의 proxyMode = ScopedProxyMode.TARGET_CLASS를 설정하면 스프링 컨테이너는 CGLIB 라는 바이트 코드를 조작하는 라이브러리를 사용하여 MyLogger를 상속받은 가짜 프록시 객체를 생성한다.
    - 결과를 확인하면 우리가 등록한 순수한 MyLogger클래스가 아니라 CGLIB이라는 클래스로 만들어진 객체가 대신 등록하는 것을 확인할 수 있다.
    - 그리고 스프링 컨테이너에 "myLogger"라는 이름으로 진짜 대신에 가짜 프록시 객체를 등록
    - ac.getBean("myLogger", MyLogger.class)로 조회해도 프록시 객체가 조회되는 것을 확인할 수 있다.
    - 그래서 의존관계 주입도 이 가짜 프록시 객체가 주입된다.

  - 가짜 프록시 객체는 요청이 오면 그때 내부에서 진짜 빈을 요청하는 위임 로직이 들어있다.
    - 가짜 프록시 객체는 내부에 진짜 myLogger를 찾는 방법을 알고 있다.
    - 클라이언트가 myLogger.logic() 을 호출하면 사실은 가짜 프록시 객체의 메서드를 호출한 것이다.
    - 가짜 프록시 객체는 request 스코프의 진짜 myLogger.logic() 를 호출한다.
    - 가짜 프록시 객체는 원본 클래스를 상속 받아서 만들어졌기 때문에 이 객체를 사용하는 클라이언트 입장에서는 사실 원본인지 아닌지도 모르게, 동일하게 사용할 수 있다(다형성)

  - 동작 정리
    - CGLIB라는 라이브러리로 내 클래스를 상속 받은 가짜 프록시 객체를 만들어서 주입한다.
    - 이 가짜 프록시 객체는 실제 요청이 오면 그때 내부에서 실제 빈을 요청하는 위임 로직이 들어있다.
    - 가짜 프록시 객체는 실제 request scope와는 관계가 없다. 그냥 가짜이고, 내부에 단순한 위임 로직만 있고, 싱글톤 처럼 동작한다.

  - 특징 정리
    - 프록시 객체 덕분에 클라이언트는 마치 싱글톤 빈을 사용하듯이 편리하게 request scope를 사용할 수 있다.
    - 사실 Provider를 사용하든, 프록시를 사용하든 핵심 아이디어는 진짜 객체 조회를 꼭 필요한 시점까지 지연처리 한다는 점이다.
    - 단지 애노테이션 설정 변경만으로 원본 객체를 프록시 객체로 대체할 수 있다. 이것이 바로 다형성과 DI 컨테이너가 가진 큰 강점이다.
    - 꼭 웹 스코프가 아니어도 프록시는 사용할 수 있다.

  - 주의점
    - 마치 싱글톤을 사용하는 것 같지만 다르게 동작하기 때문에 결국 주의해서 사용해야 한다.
    - 이런 특별한 scope는 꼭 필요한 곳에만 최소화해서 사용하자, 무분별하게 사용하면 유지보수하기 어려워진다.

** 출처: 김영한님 - 스프링 핵심 원리 기본편 강의
