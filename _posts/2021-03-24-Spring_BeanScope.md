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


    
** 출처: 김영한님 - 스프링 핵심 원리 기본편 강의
