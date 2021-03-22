---
layout: post
title: Spring Bean Lifecycle Callback
summary: 김영한님 - 스프링 핵심 원리_기본편
author: devhtak
date: '2021-03-21 21:41:00 +0900'
category: Spring
---

#### 빈 생명주기 콜백

- DB Connection pool 또는 Network Socket처럼 애플리케이션 시작 시점에 필요한 연결을 미리 해두고, 애플리케이션 종료 시점 연결을 모두 종료하는 작업을 진행하려면 객체의 초기화와 종료 작업이 필요하다.
- 예제) Network Client를 생성할 때 정보를 보자
  
  ```java
  public class NetworkClient {	
    private String url;
    public NetworkClient() {
        System.out.println("생성자 호출 url: " + url);
        connect();
        call("init");
    }
    public void setUrl(String url) {
        this.url = url;
    }
    public void connect() {
        System.out.println("connect: " + url);
    }
    public void call(String message) {
        System.out.println("call: " + url + " message: " + message);
    }
    public void disconnect() {
        System.out.println("close: " + url);
    }
  }
  ```
  - 테스트
  ```java
  public class BeanLifecycleTest {	
      @Test
      public void lifecycleTest() {
          ConfigurableApplicationContext ac = new AnnotationConfigApplicationContext(LifecycleConfig.class);
          NetworkClient networkClient = ac.getBean(NetworkClient.class);
          ac.close(); // 스프링 컨테이너 종료 -> ConfigurableApplicationContext 필요
      }

      @Configuration
      static class LifecycleConfig {
          @Bean
          public NetworkClient networkClient() {
              NetworkClient client = new NetworkClient();
              client.setUrl("http://hello-spring.dev");

              return client;
          }
      }
  }
  ```
  - 결과
  ```
  생성자 호출 url: null
  connect: null
  call: null message: init
  ```
  - 생성자 부분을 보면 URL 정보 없이 connect가 호출된다.
  - 객체 생성 단게에서 url 정보를 받지 않기 때문에 url이 null이고, 객체를 생성한 다음에 외부에서 setter를 통해 url이 존재한다.
  
- 스프링 빈은 다음과 같은 라이프 사이클을 갖는다.
  - 객체 생성 -> 의존관계 주입 (생성자 주입에 경우 예외, 빈이 파라미터로 받기 때문에 미리 생성된 빈을 주입받은 후에 생성이 가능하기 때문)
  - 스프링 빈은 객체를 생성한 후에 의존관계 주입이 다 끝난 다음에야 필요한 데이터를 사용할 수 있는 준비가 완료된다.
  - 따라서, 의존관계 주입이 완료된 후에 초기화 작업이 진행해야 된다.
  
- 스프링은 의존관계 주입이 완료된 후, 스프링 빈에게 콜백 메서드를 통해서 초기화 시점을 알려주는 기능을 제공한다.
- 또한, 스프링은 스프링 컨테이너가 종료되기 직전에 소멸 콜백을 주어 종료 작업을 진행할 수 있다.

- 스프링 빈의 이벤트 라이프사이클
  - 스프링 컨테이너 생성 -> 스프링 빈 생성 -> 의존관계 주입 -> 초기화 콜백 -> 사용 -> 소멸 전 콜백 -> 스프링 종료
  - 초기화 콜백: 빈이 생성되고, 빈의 의존관계 주입이 완료된 후 호출
  - 소멸전 콜백: 빈이 소멸되기 직전에 호출

참고. 객체의 생성과 초기화를 분리하자. 
  ```
  생성자는 필수 정보를 받아 메모리를 할당해서 객체를 생성하는 책임을 갖는다.
  초기화는 이렇게 생성된 값들을 활용하여 동작, 수행하는 책임을 갖는다.
  생성자 안에 무거운 초기화 작업을 함께 하는 것이 아닌 나누어 관리하는 것이 유지보수 관점에서 좋다.
  ```

참고. 싱글톤 빈은 스프링 컨테이너가 종료될 때 함께 종료되기 때문에, 컨테이너가 종료될 때 소멸 전 콜백이 일어난다.
참고. 생명주기가 짧은 빈들도 있다. 컨테이너와 무관하게 해당 빈이 종료되기 직전에 소멸 전 콜백이 일어난다.

- 스프링 빈 생명주기 콜백 지원 3가지 방법
  - 인터페이스(InitializingBean, DisposableBean)
  - 설정 정보에 초기화 메소드, 종료 메소드 지정
  - @PostConstruct, @PreDestroy 애노테이션 지원

#### 방법 1. 인터페이스(InitializingBean, DisposableBean) 활용

- 예제 NetworkClient를 바로 보자

  ```java
  public class NetworkClient implements InitializingBean, DisposableBean{	
      private String url;

      public NetworkClient() {
          System.out.println("생성자 호출 url: " + this.url);
      }

      public void setUrl(String url) {
          this.url = url;
      }

      public void connect() {
        System.out.println("connect: " + url);
      }

      public void call(String message) {
          System.out.println("call: " + url + " message: " + message);
      }

      public void disconnect() {
          System.out.println("close: " + url);
      }

      @Override
      public void destroy() throws Exception {
          // TODO Auto-generated method stub
          disconnect();
      }
      @Override
      public void afterPropertiesSet() throws Exception {
          // TODO Auto-generated method stub
          connect();
          call("init");
      }
  }
  ```
- 결과

  ```
  생성자 호출 url: null
  connect: http://hello-spring.dev
  call: http://hello-spring.dev message: init
  11:04:49.382 [main] DEBUG org.springframework.context.annotation.AnnotationConfigApplicationContext - Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@67c33749, started on Mon Mar 22 11:04:49 KST 2021
  close: http://hello-spring.dev
  ```
  
  - 출력 결과를 보면 초기화 메서드가 주입 완료 후에 적절하게 호출된 것을 확인할 수 있다.
  - 그리고 스프링 컨테이너의 종료가 호출되자 소멸 메서드가 호출된 것도 확인할 수 있다.

- 초기화, 소멸 인터페이스 단점
  - 해당 인터페이스는 스프링 전용 인터페이스다. 해당 코드가 스프링 전용 인터페이스에 의존한다.
  - 초기화, 소멸 메서드의 이름을 변경할 수 없다.
  - 내가 코드를 고칠 수 없는 외부 라이브러리에 적용할 수 없다.

참고. 인터페이스를 사용하는 초기화, 종료 방법은 스프링 초창기 방법으로 지금은 거의 사용하지 않는다.

#### 방법2. 빈 설정 정보에 초기화 메소드, 종료 메소드 지정

- 설정 정보에 @Bean(initMethod="init", destroyMethod="close") 처럼 초기화, 소멸 메소드를 지정할 수 있다.
- 예제 NetworkClient를 바로 보자
  
  ```java
  public class NetworkClient {	
      private String url;
      public NetworkClient() {
          System.out.println("생성자 호출 url: " + this.url);
      }
      public void setUrl(String url) {
          this.url = url;
      }
      public void connect() {
          System.out.println("connect: " + url);
      }
      public void call(String message) {
          System.out.println("call: " + url + " message: " + message);
      }
      public void disconnect() {
          System.out.println("close: " + url);
      }
      public void close(){
          disconnect();
      }
      public void init() {
          connect();
          call("init");
      }
  }
  ```
  - 테스트 코드, Config 파일에 빈을 등록할 때 설정 정보를 등록하였다.
  
  ```java
  public class BeanLifecycleTest {
      @Test
      public void lifecycleTest() {
          ConfigurableApplicationContext ac = new AnnotationConfigApplicationContext(LifecycleConfig.class);
          NetworkClient networkClient = ac.getBean(NetworkClient.class);
          ac.close();
      }

      @Configuration
      static class LifecycleConfig {
          @Bean(initMethod = "init", destroyMethod = "close")
          public NetworkClient networkClient() {
              NetworkClient client = new NetworkClient();
              client.setUrl("http://hello-spring.dev");

              return client;
          }
      }
  }
  ```
  - 테스트 결과
  
  ```
  생성자 호출 url: null
  connect: http://hello-spring.dev
  call: http://hello-spring.dev message: init
  16:39:17.732 [main] DEBUG org.springframework.context.annotation.AnnotationConfigApplicationContext - Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@67c33749, started on Mon Mar 22 16:39:17 KST 2021
  close: http://hello-spring.dev
  ```
  
- 설정 정보 사용 특징
  - 메서드 이름을 자유롭게 줄 수 있다.
  - 스프링 빈이 스프링 코드에 의존하지 않는다.
  - 코드가 아니라 설정 정보를 사용하기 때문에 코드를 고칠 수 없는 외부 라이브러리에도 초기화, 종료 메서드를 적용할 수 있다.

- 종료 메서드 추론
  - @Bean의 destroyMethod 속성에는 아주 특별한 기능이 있다.
  - 라이브러리 대부분, close, shutdown이라는 이름의 종료 메서드를 사용한다.
  - @Bean의 destroyMethod는 기본값이 (inferred)(추론)으로 등록되어 있다.
  - 이 추론 기능은 close, shutdown이라는 이름의 메서드를 자동으로 호출해준다. 이름 그대로 종료 메서드를 추론해서 호출한다.
  - 따라서 직접 스프링 빈으로 등록하면 종료 메서드는 따로 적어주지 않아도 잘 동작한다.
  - 추론 기능을 사용하기 싫다면 destroyMethod="" 처럼 빈 공백을 지정하면 된다.
  
#### 방법3. @PostConstructor, @PreDestroy 애노테이션 지정

- 예제 NetworkClient를 바로 보자
  
  ```java
  public class NetworkClient {	
    private String url;
    public NetworkClient() {
        System.out.println("생성자 호출 url: " + this.url);
    }
    public void setUrl(String url) {
        this.url = url;
    }
    public void connect() {
        System.out.println("connect: " + url);
    }
    public void call(String message) {
        System.out.println("call: " + url + " message: " + message);
    }
    public void disconnect() {
        System.out.println("close: " + url);
    }
    @PreDestroy
    public void clear() {
        disconnect();
    }
    @PostConstruct
    public void init() {
        connect();
        call("init");
    }
  }
  ```
  - 테스트 실행 결과
  
  ```
  생성자 호출 url: null
  connect: http://hello-spring.dev
  call: http://hello-spring.dev message: init
  16:46:25.985 [main] DEBUG org.springframework.context.annotation.AnnotationConfigApplicationContext - Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@8317c52, started on Mon Mar 22 16:46:25 KST 2021
  close: http://hello-spring.dev
  ```
    - @PostConstruct, @PreDestroy 두 애노테이션을 사용하면 가장 편리하게 초기화와 종료를 실행할 수 있다.

- @PostConstruct, @PreDestroy 애노테이션 특징
  - 최신 스프링에서 권장하는 방법
  - Annotation 하나만 붙이면 되므로 편리하다.
  - 패키지를 잘보면, javax.annotation.PostConstruct이다. 스프링에 종속되지 않는 자바 표준이기 때문에 스프링이 아닌 다른 컨테이너에서도 동작한다.
  - ComponentScan과 잘어울린다.
  - 유일한 단점은 외부 라이브러에는 적용하지 못한다. 외부 라이브러리를 초기화, 종료하는 데 사용한다면 @Bean의 설정을 사용하자. 

** 출처: 김영한님 스프링 핵심원리 기본편 강의
