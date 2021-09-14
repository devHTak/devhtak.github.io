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
  
#### Spring Boot Exception 처리

- Servlet에서는 복잡한 과정으로 예외 처리를 진행했다.
  - WebServerCustomizer
  - ErrorPage 추가
  - ErrorPageController

- Spring Boot는 기본 제공
  - /error 경로 하위에 ErrorPage를 자동으로 등록
  - BasicErrorController 라는 스프링 컨트롤러를 자동으로 등록

- 오류 페이지 화면만 BasicErrorController가 제공하는 룰에 맞게 등록하면 된다.
  - 정적 HTML이면 정적 리소스, 뷰 템플릿을 사용해서 동적으로 오류 화면을 만들고 싶으면 뷰 템플릿 경로에 오류 페이지 파일을 만들어서 넣어두기만 하면 된다.
  - 뷰 선택 우선순위
    - 1순위 뷰 템플릿
      - resources/templates/error/500.html
      - resources/templates/error/5xx.html
    - 2순위 정적 리소스( static , public )
      - resources/static/error/400.html
      - resources/static/error/404.html
      - resources/static/error/4xx.html
    - 3순위 적용 대상이 없을 때 뷰 이름( error )
      - resources/templates/error.html
  
- BasicErrorController
  - BasicErrorController는 해당 정보를 model에 담아서 뷰에 전달한다.
    ```
    * timestamp: Fri Feb 05 00:00:00 KST 2021
    * status: 400
    * error: Bad Request
    * exception: org.springframework.validation.BindException
    * trace: 예외 trace
    * message: Validation failed for object='data'. Error count: 1
    * errors: Errors(BindingResult)
    * path: 클라이언트 요청 경로 (`/hello`)
    ```
    
  - 보안상에 문제로 많은 정보를 보여주지 않는 것이 좋다. application.properties에서 오류 정보를 model에 담을지를 선택할 수 있다.
    ```
    // application.properties
    erver.error.include-exception=false # exception 포함 여부( true , false )
    server.error.include-message=never # message 포함 여부
    server.error.include-stacktrace=never # trace 포함 여부
    server.error.include-binding-errors=never # errors 포함 여부
    # 파리미터로 받는 값
    # never : 사용하지 않음
    # always :항상 사용
    # on_param : 파라미터가 있을 때 사용
    ```
    - 실무에서는 이것들을 노출하면 안된다! 사용자에게는 이쁜 오류 화면과 고객이 이해할 수 있는 간단한 오류 메시지를 보여주고 오류는 서버에 로그로 남겨서 로그로 확인해야 한다  

  - 스프링 부트 오류 관련 옵션
    - server.error.whitelabel.enabled=true : 오류 처리 화면을 못 찾을 시, 스프링 whitelabel 오류 페이지 적용
    - server.error.path=/error : 오류 페이지 경로, 스프링이 자동 등록하는 서블릿 글로벌 오류 페이지 경로와 BasicErrorController 오류 컨트롤러 경로에 함께 사용된다.
  - 확장 포인트
    - 에러 공통 처리 컨트롤러의 기능을 변경하고 싶으면 ErrorController 인터페이스를 상속 받아서 구현하거나 BasicErrorController 상속 받아서 기능을 추가하면 된다.

#### API 예외 처리

- 시작
  - API에서는 오류 상황 시 오류 화면을 보여주는 것이 아닌, 오류 응답 스펙을 정하고 JSON 데이터를 내려주어야 한다.
  ```java
  @RequestMapping(produces = MediaType.TEXT_HTML_VALUE)
  public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {}
  @RequestMapping
  public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {}
  ```
  - 요청 Header의 Accept 헤더 값에 따라 ModelAndView를 통해 HTML 오류 화면을 보여줄 수도 있고, JSON 타입의 오류 데이터를 보낼 수 있다.

- HandlerExceptionResolver
  - 스프링 MVC는 컨트롤러(핸들러) 밖으로 예외가 던져진 경우 예외를 해결하고, 동작을 새로 정의할 수 있는 방법을 제공한다.
  - 컨트롤러 밖으로 던져진 예외를 해결하고, 동작 방식을 변경하고 싶으면 HandlerExceptionResolver 를 사용하면 된다.
  - ExceptionResolver 적용 전
    
    ![image](https://user-images.githubusercontent.com/42403023/133262092-2fd7df51-efa7-4bbb-b434-0836b21026f0.png)
    
    - 이미지 출처: 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술 \[인프런 김영한님 강의]
  - ExceptionResolver 적용 후
    
    ![image](https://user-images.githubusercontent.com/42403023/133262150-260a3d3f-5dc7-4ca8-8e5d-ccdcc9849c9b.png)
    
    - 이미지 출처: 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술 \[인프런 김영한님 강의]

  - HandlerExceptionResolver와 Customizing
    ```java
    public interface HandlerExceptionResolver {
        ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response,Object handler, Exception ex);
    }
    ```
    - handler: 핸들러(컨트롤러) 정보
    - Exception ex: 핸들러(컨트롤러)에서 발생한 예외
    ```java
    @Slf4j
    public class MyHandlerExceptionResolver implements HandlerExceptionResolver {
        @Override
        public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
            try {
                if (ex instanceof UserException) {
                    log.info("UserException resolver to 400");
                    String acceptHeader = request.getHeader("accept");
                    response.setStatus(HttpServletResponse.SC_BAD_REQUEST);
                    
                    if ("application/json".equals(acceptHeader)) {
                        Map<String, Object> errorResult = new HashMap<>();
                        errorResult.put("ex", ex.getClass());
                        errorResult.put("message", ex.getMessage());
                        String result = objectMapper.writeValueAsString(errorResult);
                        response.setContentType("application/json");
                        response.setCharacterEncoding("utf-8");
                        response.getWriter().write(result);
                        return new ModelAndView();
                    } else {
                        //TEXT/HTML
                        return new ModelAndView("error/500");
                    }
                }
            } catch (IOException e) {
                log.error("resolver ex", e);
            }
            return null;
        }
    }
    
    // WebMvcConfigurer 를 통해 ExceptionResolver 등록
    @Override
    public void extendHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
        resolvers.add(new MyHandlerExceptionResolver());
    }
    ```
    - ExceptionResolver 가 ModelAndView 를 반환하는 이유는 마치 try, catch를 하듯이, Exception을 처리해서 정상 흐름 처럼 변경하는 것이 목적
    - IllegalArgumentException 이 발생하면 response.sendError(400) 를 호출해서 HTTP 상태 코드를 400으로 지정하고, 빈 ModelAndView 를 반환
    - 빈 ModelAndView를 반환하면 뷰를 렌더링 하지 않고, 정상 흐름으로 서블릿이 리턴된다.
    
- 스프링이 제공하는 ExceptionResolver
  - HandlerExceptionResolverCompsoite에 다음과 같이 스프링 부트가 제공하는 ExceptionResolver가 등록되어 있다.
    - ExceptionHandlerExceptionResolver
    - ResponseStatusExceptionResolver
    - DefaultHandlerExceptionResolver

  - ResponseStatusExceptionResolver
    - 역할
      - @ResponseStatus 가 달려있는 예외 처리
      - ResponseStatusException 예외 처리

    - @ResponseStatus 예제
      ```java
      @ResponseStatus(code = HttpStatus.BAD_REQUEST, reason = "잘못된 요청 오류")
      public class BadRequestException extends RuntimeException {}
      ```
      - BadRequestException 이 발생하면 400 BadRequest 상태코드가 발생하고, message로 잘못된 요청 오류가 발생
      - reason에는 MessageSource 에서 매세지 코드를 찾아서 보여주는 기능도 제공한다.

    - ResponseStatusException
      ```java
      @GetMapping("/api/response-status-ex2")
      public String responseStatusEx2() {
          throw new ResponseStatusException(HttpStatus.NOT_FOUND, "error.bad", new IllegalArgumentException());
      }
      ```
    
  - DefaultHandlerExceptionResolver
    - 스프링 내부에서 발생하는 스프링 예외를 해결
    
    ![image](https://user-images.githubusercontent.com/42403023/133266285-ee13ed62-414e-4d3a-9144-1e9d5dfbd41a.png)

    - 이런 방식으로 내부에서 발생 가능한 예외에 대한 처리를 하고 있다.

  - ExceptionHandlerExceptionResolver -> @ExceptionHandler

#### 출처

- 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술 \[인프런 김영한님 강의]
