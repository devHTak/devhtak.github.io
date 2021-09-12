---
layout: post
title: Exception 처리
summary: Spring MVC Basic
author: devhtak
date: '2021-09-12 21:41:00 +0900'
category: Spring
---

#### Servlet 예외 처리

- Servlet의 예외 처리 지원
  - Exception (예외 발생)
  - response.sendError(HTTP Status Code, Error Message)
  
  - Exception
    - WAS(여기까지 전파) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러(예외발생)
    - 예외를 처리하지 않으면 예외 상황이 WAS까지 전파된다.
    - tomcat과 같은 WAS에서는 기본적으로 처리 페이지에서 상태를 보여주도록 되어 있다.

  - response.sendError(HTTP Status Code, Error Message)
    - WAS(sendError 호출 기록 확인) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러(response.sendError()
    - sendError를 호출하면 예외가 바로 발생하지는 않지만, 서블릿 컨테이너에게 오류가 발생했다는 점을 전달한다.
    
- 오류화면 제공
  - 오류 화면을 만들어서 보여줄 수 있다.
  - Servlet 오류 페이지 등록 방법 1. web.xml
    ```xml
    <web-app>
        <error-page>
            <error-code>404</error-code>
            <location>/error-page/404.html</location>
        </error-page>
        <error-page>
            <error-code>500</error-code>
            <location>/error-page/500.html</location>
        </error-page>
        <error-page>
            <exception-type>java.lang.RuntimeException</exception-type>
            <location>/error-page/500.html</location>
        </error-page>
    </web-app>
    ```
  - Servlet 오류 페이지 등록 방법 2. Spring Boot 기능 제공
    - 빈 등록
      ```java
      @Component
      public class WebServerCustomizer implements WebServerFactoryCustomizer<ConfigurableWebServerFactory> {
          @Override
          public void customize(ConfigurableWebServerFactory factory) {
              ErrorPage errorPage404 = new ErrorPage(HttpStatus.NOT_FOUND, "/errorpage/404");
              ErrorPage errorPage500 = new ErrorPage(HttpStatus.INTERNAL_SERVER_ERROR, "/error-page/500");
              ErrorPage errorPageEx = new ErrorPage(RuntimeException.class, "/errorpage/500");
              factory.addErrorPages(errorPage404, errorPage500, errorPageEx);
          }
      }
      ```
      
    - Controller 처리
      ```java
      @Slf4j
      @Controller
      public class ErrorPageController {
          @RequestMapping("/error-page/404")
          public String errorPage404(HttpServletRequest request, HttpServletResponse response) {
              log.info("errorPage 404");
              return "error-page/404";
          }
          @RequestMapping("/error-page/500")
          public String errorPage500(HttpServletRequest request, HttpServletResponse response) {
              log.info("errorPage 500");
              return "error-page/500";
          }
      }
      ```

- 오류 페이지 작동 원리
  ```
  1. WAS(여기까지 전파) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러(예외발생)
  2. WAS `/error-page/500` 다시 요청 -> 필터 -> 서블릿 -> 인터셉터 -> 컨트롤러(/errorpage/500) -> View
  ```
  - 예외가 발생해서 WAS까지 전파된다.
  - WAS는 오류 페이지 경로를 찾아서 내부에서 오류 페이지를 호출한다. 이때 오류 페이지 경로로 필터, 서블릿, 인터셉터, 컨트롤러가 모두 재호출
  - 웹 브라우저(클라이언트)는 서버 내부에서 이런 일이 일어나는지 전혀 모른다는 점이다. 오직 서버 내부에서 오류 페이지를 찾기 위해 추가적인 호출을 한다
  - ErrorPageController에서 request 객체에 담아온 정보를 확인할 수 있다.
    ```java
    log.info("ERROR_EXCEPTION: ex=", request.getAttribute("javax.servlet.error.exception")); // 예외
    log.info("ERROR_EXCEPTION_TYPE: {}", request.getAttribute("javax.servlet.error.exception_type")); // 예외 타입
    log.info("ERROR_MESSAGE: {}", request.getAttribute("javax.servlet.error.message"));  // 오류 메시지
    //ex의 경우 NestedServletException 스프링이 한번 감싸서 반환
    log.info("ERROR_REQUEST_URI: {}", request.getAttribute("javax.servlet.error.request_uri")); // 클라이언트 요청 URI
    log.info("ERROR_SERVLET_NAME: {}", request.getAttribute("javax.servlet.error.servlet_name")); // 오류가 발생한 서블릿 이름
    log.info("ERROR_STATUS_CODE: {}", request.getAttribute("javax.servlet.error.status_code")); // HTTP 상태 코드
    log.info("dispatchType={}", request.getDispatcherType())
    ```

- 필터 중복 호출 방지
  - 오류가 발생하면 오류 페이지를 출력하기 위해 WAS 내부에서 다시 한번 호출이 발생한다.
  - 예외 호출과 관련하여 필터가 호출되지 않아도 되고, 호출이 필요한 필터가 존재하기 때문에 서블릿은 DispatcherType이라는 추가 정보를 제공하여 관리할 수 있다.
  - DispatcherType
    ```java
    public enum DispatcherType {
        FORWARD, // MVC에서 배웠던 서블릿에서 다른 서블릿이나 JSP 호출할 때 사용
        INCLUDE, // 서블릿에서 다른 서블릿이나 JSP의 결과를 포함할 때
        REQUEST, // 클라이언트 요청
        ASYNC, // 서블릿 비동기 호출
        ERROR // 오류 요청
    }
    ```
  - 오류 호출 필터 등록
    ```java
    @Configuration
    public class WebConfig implements WebMvcConfigurer {
        @Bean
        public FilterRegistrationBean logFilter() {
            FilterRegistrationBean<Filter> filterRegistrationBean = new FilterRegistrationBean<>();
            filterRegistrationBean.setFilter(new LogFilter());
            filterRegistrationBean.setOrder(1);
            filterRegistrationBean.addUrlPatterns("/*");
            filterRegistrationBean.setDispatcherTypes(DispatcherType.REQUEST, DispatcherType.ERROR);
            return filterRegistrationBean;
        }
    }
    ```
    - DispatcherType을 세팅하면 클라이언트 요청은 물론이고, 오류 페이지 요청에서도 필터가 호출된다.
    - 아무것도 넣지 않으면 기본 값이 DispatcherType.REQUEST 이다. 

- 인터셉터 중복 호출 방지
  - 인터셉터는 스프링에서 제공하는 기술로 서블릿에서 제공하는 DispatcherType과는 무관하게 항상 호출된다.
  - 인터셉터를 등록할 때 excludePatters에 오류 페이지를 추가하면 된다.
  
  ```java
  @Configuration
  public class WebConfig implements WebMvcConfigurer {
      @Override
      public void addInterceptors(InterceptorRegistry registry) {
          registry.addInterceptor(new LogInterceptor())
              .order(1)
              .addPathPatterns("/**")
              .excludePathPatterns(
              "/css/**", "/*.ico"
              , "/error", "/error-page/**" //오류 페이지 경로
      );
  }
  ```
  
### Spring Boot Exception 처리


#### 출처

- 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술 \[인프런 김영한님 강의]
