#### Spring Bean은 Proxy로 만들어질까?

- 일반 빈을 조회하였을 때,

- AOP를 적용한 빈을 조회하였을 때,

- AOP를 적용한 빈을 조회하였을 때 Proxy로 생성되어져 있는 것을 확인할 수 있다.
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

