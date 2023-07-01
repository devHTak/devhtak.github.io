---
layout: post
title: Spring Proxy Bean
summary: Spring Document
author: devhtak
date: '2023-05-22 21:41:00 +0900'
category: Spring
---
#### Spring Bean은 Proxy로 만들어질까?

- 일반 빈을 조회하였을 때
  - OrderService 및 빈 등록
    ```java
    public class OrderService {
        Map<Long, Order> orders = new HashMap<>();
        public Order saveOrder(Order order) {
            orders.put(order.getId(), order);
            return orders.get(order.getId());
        }
    }
    ```
    ```java
    @Configuration
    public class OrderConfig {
        @Bean
        public OrderService orderService() {
            return new OrderService();
        }
    }
    ```
  - Bean 조회 후 확인
    ```java
    @Test
    void getOrderServiceBeanTest() throws Exception {
        ApplicationContext context = new AnnotationConfigApplicationContext(OrderConfig.class);
        OrderService orderService = (OrderService) context.getBean("orderService");

        Assertions.assertEquals(orderService.getClass(), OrderService.class);
    }
    ```
    - Spring Context를 생성하여 OrderService 확인해본 결과 Proxy 객체가 아닌 순수 객체각 반환되는 것을 확인할 수 있다.
- AOP를 적용한 빈을 조회하였을 때
  - AopServiceAspect 및 AopService, AopConfig
    ```java
    @Aspect
    @Slf4j
    @Configuration
    public class AopServiceAspect {
        @Around(value = "execution(* com.example.bean.AopService.incrementCount())")
        public void aroundService(ProceedingJoinPoint joinPoint) {
          log.info("START SERVICE");
          try {
              joinPoint.proceed();
          } catch (Throwable e) {
              log.error("Around exception: {}", e.getMessage());
          }
          log.info("END SERVICE");
        }
    }
    ```
    ```java
    public class AopService {
        private int count = 0;
        public int incrementCount() {
            return ++count;
        }
    }
    ```
    ```java
    @Configuration
    @EnableAspectJAutoProxy
    public class AopConfig {
        @Bean
        public AopService aopService() {
            return new AopService();
        }
        @Bean
        public AopServiceAspect aopServiceAspect() {
            return new AopServiceAspect();
        }
    }
    ```
  - 빈 등록 확인
    ```java
    @Test
    void getAopServiceBeanTest() throws Exception {
        ApplicationContext context = new AnnotationConfigApplicationContext(AopConfig.class);
        AopService aopService = (AopService) context.getBean("aopService");

        Assertions.assertNotEquals(aopService.getClass(), AopService.class);
    }
    ```
    - AOP를 적용한 빈을 조회하였을 때 Proxy로 생성되어져 있는 것을 확인할 수 있다.
    <img width="562" alt="image" src="https://github.com/devHTak/devhtak.github.io/assets/42403023/bd88f7b1-bfd2-4a02-8524-b44087894a3c">

- 빈을 등록하는 과정에서 AOP 여부를 확인한다
  - Spring Context 가 Bean 후보를 스캔하여 생성할 때, 빈이 생성되는 과정에서 AbstractAutoProxyCreator.wrapIfNeccesary() 함수가 호출되는 데, Proxy로 Wrap 하는 역할을 수행
    ```java
    protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {
      // validation...

      // Create proxy if we have advice.
      Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);
      if (specificInterceptors != DO_NOT_PROXY) {
        this.advisedBeans.put(cacheKey, Boolean.TRUE);
        Object proxy = createProxy(
            bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));
        this.proxyTypes.put(cacheKey, proxy.getClass());
        return proxy;
      }

      this.advisedBeans.put(cacheKey, Boolean.FALSE);
      return bean;
    }
    ```
    - getAdvicesAndAdvisorsForBean 은 추상메소드로 AbstractAdvisorAutoProxyCreator에 구현
    - specificInterceptors에 따라 Proxy 클래스를 생성하여 리턴한다
  - 이 뿐만 아니라 Spring은 Bean의 관계를 파악하고 Proxy로 등록해야할 지를 결정하는 것으로 보인다.
  - Proxy 객체를 만들 수 있는 이유
    - CGLIB Proxy는 빈과 그 빈을 감싸는 프록시 객체는 부모-자식 관계로 주입을 받을 수 있다.
    - 만약 빈을 등록할 때 final class로 상속을 더이상 못하게 한다면 프록시 객체를 생성할 수 없기 때문에 오류가 발생한다.
      ```java
      Could not generate CGLIB subclass of class com.example.bean.AopServic
      ```
- CGLIB vs jdk dynamic proxy
  - jdk dynamic proxy
    - 1.3 버전부터 생긴 기능으로 interface 기반으로 Proxy 생성
    - invocation handler를 상속받아 실체를 구현하는데, invoke 메소드(reflection)을 활용하기 때문에 성능이 떨어진다.
    - MethodMatcher
  - CGLIB
    - CGLIB은 reflection을 사용하지 않고 상속을 사용하여 Proxy할 메서드를 오버라이딩하는 방식
    - MethodInterceptor
