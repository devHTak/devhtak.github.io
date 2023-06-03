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

- 두 객체에 차이점
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

