---
layout: post
title: Spring MVC - 응답
summary: 김영한님 - 스프링 MVC - 백엔드 웹 개발 핵심 기술
author: devhtak
date: '2021-04-18 21:41:00 +0900'
category: Spring
---

#### HTTP Response

- 응답 데이터
  - 정적 리소스
    - 예) 브라우저에 정적인 HTML, CSS, JS 를 제공할 때는, 정적 리소스를 사용한다.
  - 뷰 템플릿
    - 예) 웹 브라우저에 동적인 HTML을 제공할 때는 뷰 템플릿을 사용한다.
  - HTTP 메시지 사용
    - HTTP API를 제공하는 경우에는 HTML이 아닌 데이터를 전달해야 하므로, HTTP 메시지 바디에 JSON 같은 형식으로 데이터를 실어 보낸다.

#### HTTP Response - 정적 리소스

- 정적 리소스
  - 스프링 부트는 클래스패스의 다음 디렉토리에 있는 정적 리소스를 제공한다.
    ```
    /static, /public, /resources, /META-INF/resources
    ```
    - src/main/resources는 리소스를 보관하는 곳이고, 또 클래스패스의 시작 경로이다.
    - 다음 디렉토리에 리소스를 넣으면 스프링 부트가 정적 리소스로 서비스를 제공한다.

  - 정적 리소스 경로
    ```
    src/main/resources/static
    ```
    - 만약 src/main/resources/static/basic/hello-form.html에 파일이 있으면 http://localhost:8080/biasic/hello-form.html 로 접근할 수 있다.

  - 정적 리소스는 해당 파일 변경 없이 그대로 서비스한다.


#### HTTP Response - 뷰 템플릿

- 뷰 템플릿
  - 뷰 템플릿을 거쳐서 HTML 이 생성되고, 뷰가 응답을 만들어서 전달한다.
  - 일반적으로 HTML을 동적으로 생성하는 용도로 사용하지만, 다른 것도 가능하다.
  - 뷰 템플릿 경로
    ```
    src/main/resources/template
    ```
  - 예제
    - 뷰 템플릿 생성
      - src/main/resources/template/response/hello.html
        ```
        <!DOCTYPE html>
        <html xmlns:th="http://www.thymeleaf.org">
        <head>
        <meta charset="UTF-8">
        <title>Insert title here</title>
        </head>
        <body>

        <p th:text="${data}">empty</p>

        </body>
        </html>
        ```
    - ResponseViewController - 뷰를 호출하는 컨트롤러
      ```java
      @Controller
      public class ResponseViewController {

          @RequestMapping("/response-view-v1")
          public ModelAndView responseViewV1() {
              ModelAndView mav = new ModelAndView("response/hello.html")
                  .addObject("data", "hello!");
              return mav;
          }

          @RequestMapping("/response-view-v2")
          public String responseViewV2(Model model) {
              model.addAttribute("data", "hello!");
              return "response/hello";
          }

          @RequestMapping("/response-view-v3")
          public void responseViewV3(Model model) {
              model.addAttribute("data", "hello!");
          }
      }
      ```
      
      - V1: ModelAndView에 Viewname을 세팅하여 return한다.
      - V2: String을 View로 반환
        - @ResponseBody가 없으면 HTTP 메시지가 아닌 View Resolver가 실행되어 View를 찾고 렌더링 한다.
        - @ResponseBody가 있으면 View Resolver를 실행하지 않고, HTTP 메시지 바디에 직접 문자열이 입력된다.
      - V3: void를 반환하는 경우
        - @Controller 를 사용하고, HttpServletResponse , OutputStream(Writer) 같은 HTTP 메시지 바디를 처리하는 파라미터가 없으면 요청 URL을 참고해서 논리 뷰 이름으로 사용
          - 요청 URL: /response/hello
          - 실행: templates/response/hello.html
        - 참고로 이 방식은 명시성이 너무 떨어지고 이렇게 딱 맞는 경우도 많이 없어서, 권장하지 않는다.

- 참고: Tymeleaf 설정
  - 의존성
    ```
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
    ```
  - 스프링 부트가 자동으로 ThymeleafViewResolver와 필요한 스프링 빈들을 등록한다.
  - 설정: application.properties
    ```
    spring.thymeleaf.prefix=classpath:/templates/
    spring.thymeleaf.suffix=.html
    ```
    - prefix, suffix 설정

#### HTTP Response - HTTP API, 메시지 바디에 직접 입력

- HTTP API를 제공하는 경우에는 HTML이 아니라 데이터를 전달해야 하므로 HTTP 메시지 바디에 JSON같은 형식으로 데이터를 실어 보낸다.
- 참고
  -  HTML이나 뷰 템플릿을 사용해도 HTTP 응답 메시지 바디에 HTML 데이터가 담겨서 전달된다. 
  -  정적 리소스나 뷰 템플릿을 거치지 않고, 직접 HTTP 응답 메시지를 전달하는 경우를 말한다.

```java
@Controller
public class ResponseBodyController {
	
    @GetMapping("/response-body-string-v1")
    public void responseBodyV1(HttpServletResponse response) throws IOException {
        response.getWriter().write("ok");
    }

    @GetMapping("/response-body-string-v2")
    public HttpEntity<String> responseBodyV2() {
        return new HttpEntity<String>("ok");
    }

    @GetMapping("/response-body-string-v3")
    @ResponseBody
    public String responseBodyV3() {
        return "ok";
    }

    @GetMapping("/response-body-json-v1")
    public ResponseEntity<HelloData> responseBodyJsonV1() {
        HelloData helloData = new HelloData();
        helloData.setUsername("test");
        helloData.setAge(10);
        return new ResponseEntity<HelloData>(helloData, HttpStatus.OK);
    }

    @ResponseStatus(HttpStatus.OK)
    @ResponseBody
    @GetMapping("/response-body-json-v2")
    public HelloData responseBodyJsonV2() {
        HelloData helloData = new HelloData();
        helloData.setUsername("test");
        helloData.setAge(10);		
        return helloData;
    }
}
```
- String
  - V1: 서블릿을 다룰 때 처럼 HttpServletResponse를 직접 사용할 수 있다.
  - V2: HttpEntity와 상속받은 ResponseEntity를 활용하여 사용할 수 있다.
    - ResponseEntity를 하면, HttpStatus를 사용하여 응답코드를 지정할 수 있다.
  - V3: @ResponseBody를 사용하면 view를 생성하지 않고 string을 리턴한다.

- JSON
  - V1: HttpEntity와 상속받은 ResponseEntity를 반환할 수 있다.
    - Message Converter를 사용하여 JSON 형식으로 변환되어사 반환된다.
  - V2: 객체 자체를 반환할 수 있다. 필요하다면 @ResponseStatus 애노테이션을 활용하여 응답 코드를 지정할 수 있다.
    - 동적으로 응답 코드를 생성하기 위해서는 ResponseEntity를 사용하는 것이 좋다.

- @ RestController
  - @Controller 대신에 @RestController 애노테이션을 사용하면, 해당 컨트롤러에 모두 @ResponseBody 가 적용되는 효과가 있다. 
  - 따라서 뷰 템플릿을 사용하는 것이 아니라, HTTP 메시지 바디에 직접 데이터를 입력한다. 
  - 이름 그대로 Rest API(HTTP API)를 만들 때 사용하는 컨트롤러이다.
  - 참고로 @ResponseBody 는 클래스 레벨에 두면 전체에 메서드에 적용되는데, @RestController 에노테이션 안에 @ResponseBody 가 적용되어 있다.

#### HTTP Message Converter

- Spring MVC는 다음의 경우 HTTP Message Converter를 적용한다.
  - HTTP 요청: @RequestBody, HttpEntity(RequestEntity)
  - HTTP 응답: @ResponseBody, HttpEntity(ResponseEntity)

- HTTP Message Converter Interface
  - org.springframework.http.converter.HttpMessageConverter
  
  ```java
  package org.springframework.http.converter;
  public interface HttpMessageConverter<T> {
      boolean canRead(Class<?> clazz, @Nullable MediaType mediaType);
      
      boolean canWrite(Class<?> clazz, @Nullable MediaType mediaType);
      
      List<MediaType> getSupportedMediaTypes();
      
      T read(Class<? extends T> clazz, HttpInputMessage inputMessage) throws IOException, HttpMessageNotReadableException;

      void write(T t, @Nullable MediaType contentType, HttpOutputMessage outputMessage) throws IOException, HttpMessageNotWritableException;
  }
  ```

    - HTTP MessageConverter는 HTTP 요청, 응답 둘 다 사용된다.
    - canRead(), canWrite(): 메시지 컨버터가 해당 클래스, 미디어 타입을 지원하는 지 체크
    - read(), write(): 메시지 컨버터를 통해서 메시지를 읽고 쓰는 기능

- Spring Boot 기본 메시지 컨버터
  - (일부 생략)
  ```
  0 = ByteArrayHttpMessageConverter
  1 = StringHttpMessageConverter 
  2 = MappingJackson2HttpMessageConverter
  ```
    - 스프링 부트는 다양한 메시지 컨버터를 제공하는 데, 대상 클래스 타입과 미디어 타입 둘을 체크해서 사용여부를 결정한다.
      - MediaType -> 요청(HTTP request header: Content-Type, @RequestMapping parameter: consume)
      - MediaType -> 응답(HTTP request header: Accept, @RequestMapping parameter: produce)
    - 주요 메시지 컨버터
      - ByteArrayHttpMessageConverter: byte[] 데이터를 처리한다.
        - 클래스 타입: byte[], 미디어 타입: */*
        - 요청 예: @RequestBody byte[] data
        - 응답 예: @ResponseBody return byte[], 쓰기 미디어 타입: application/octet-stream
      - StringHttpMessageConverter: String 문자로 데이터를 처리한다.
        - 클래스 타입: String , 미디어타입: */*
        - 요청 예) @RequestBody String data
        - 응답 예) @ResponseBody return "ok" 쓰기 미디어타입 text/plain
      - MappingJackson2HttpMessageConverter: application/json
        - 클래스 타입: 객체 또는 HashMap , 미디어타입 application/json 관련
        - 요청 예) @RequestBody HelloData data
        - 응답 예) @ResponseBody return helloData 쓰기 미디어타입 application/json 관련

- HTTP 요청 데이터 읽기
  - HTTP 요청이 오고, 컨트롤러에서 @RequestBody , HttpEntity 파라미터를 사용한다.
  - 메시지 컨버터가 메시지를 읽을 수 있는지 확인하기 위해 canRead() 를 호출한다.
    - 대상 클래스 타입을 지원하는가.
      - 예) @RequestBody 의 대상 클래스 ( byte[] , String , HelloData )
    - HTTP 요청의 Content-Type 미디어 타입을 지원하는가.
      - 예) text/plain , application/json , */*
  - canRead() 조건을 만족하면 read() 를 호출해서 객체 생성하고, 반환한다.

- HTTP 응답 데이터 생성
  - 컨트롤러에서 @ResponseBody , HttpEntity 로 값이 반환된다.
  - 메시지 컨버터가 메시지를 쓸 수 있는지 확인하기 위해 canWrite() 를 호출한다.
    - 대상 클래스 타입을 지원하는가.
      - 예) return의 대상 클래스 ( byte[] , String , HelloData )
    - HTTP 요청의 Accept 미디어 타입을 지원하는가.(더 정확히는 @RequestMapping 의 produces)
      - 예) text/plain , application/json , */*
  - canWrite() 조건을 만족하면 write() 를 호출해서 HTTP 응답 메시지 바디에 데이터를 생성한다.

#### HttpConverter에 위치, feat ArgumentResolver, ReturnValueHandler

- RequestMappingHandlerAdapter 동작 방식
  - DispatcherServlet이 RequestMappingHandlerAdapter 호출
  - RequestMappingHandlerAdapter가 Handerl 호출
    - Handler 파라미터에 맞는 Argument를 생성해야 한다.
    - ArgumentResolver가 지원
    - HttpServletRequest, Model, @RequestParam, @ModelAttribute, @RequestBody, HttpEntity...
  - Handler가 RequestMappingHandlerAdapter에게 결과 return
    - Handler가 Return할 때 ReturnValueHandler가 반환 값을 변환한다.
    - ModelAndView, @ResponseBody, HttpEntity ...

- ArgumentResolver
  - 애노테이션 기반의 컨트롤러는 매우 다양한 파라미터를 사용할 수 있었다.
    - HttpServletRequest , Model, @RequestParam , @ModelAttribute, @RequestBody , HttpEntity 등
  - 파라미터를 유연하게 처리할 수 있는 이유가 바로 ArgumentResolver 덕분이다.
  - 애노테이션 기반 컨트롤러를 처리하는 RequestMappingHandlerAdaptor 는 바로 이 ArgumentResolver 를 호출해서 컨트롤러(핸들러)가 필요로 하는 다양한 파라미터의 값(객체)을 생성한다. 
  - 파리미터의 값이 모두 준비되면 컨트롤러를 호출하면서 값을 넘겨준다.
  - 스프링은 30개가 넘는 ArgumentResolver 를 기본으로 제공한다.
  
  ```java
  public interface HandlerMethodArgumentResolver {
      boolean supportsParameter(MethodParameter parameter);
      @Nullable
      Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer, NativeWebRequest webRequest, @Nullable WebDataBinderFactory  binderFactory) throws Exception;
  }
  ```
  - 동작 방식
    - ArgumentResolver 의 supportsParameter() 를 호출해서 해당 파라미터를 지원하는지 체크
    - 지원하면 resolveArgument() 를 호출해서 실제 객체를 생성한다. 
    - 이렇게 생성된 객체가 컨트롤러 호출시 넘어가는 것이다.
    - 여러분이 직접 이 인터페이스를 확장해서 원하는 ArgumentResolver 를 만들 수도 있다. 
  
- ReturnValueHandler
  - HandlerMethodReturnValueHandler 를 줄여서 ReturnValueHandle 라 부른다.
  - ArgumentResolver 와 비슷한데, 이것은 응답 값을 변환하고 처리한다.
  - 컨트롤러에서 String으로 뷰 이름을 반환해도, 동작하는 이유가 바로 ReturnValueHandler 덕분이다.
  - 스프링은 10여개가 넘는 ReturnValueHandler 를 지원한다.
    - 예) ModelAndView , @ResponseBody , HttpEntity , String

- HTTP Message Converter 위치
  - ArgumentResolver에서 @RequestBody, HttpEntity등을 만나면 HTTP Message Converter를 호출한다.
  - ReturnValueHandler에서 @ResponseBody, HttpEntity등을 만나면 HTTP Message Converter를 호출한다.
  - 요청
    - @RequestBody 를 처리하는 ArgumentResolver 가 있고, HttpEntity 를 처리하는 ArgumentResolver 가 있다. 
    - 이 ArgumentResolver 들이 HTTP 메시지 컨버터를 사용해서 필요한 객체를 생성하는 것이다.
  - 응답
    - @ResponseBody 와 HttpEntity 를 처리하는 ReturnValueHandler 가 있다.
    - 여기에서 HTTP 메시지 컨버터를 호출해서 응답 결과를 만든다.
  - 확장
    - 스프링은 다음을 모두 인터페이스로 제공한다. 따라서 필요하면 언제든지 기능을 확장할 수 있다.
      - HandlerMethodArgumentResolver
      - HandlerMethodReturnValueHandler
      - HttpMessageConverter
    - 스프링이 필요한 대부분의 기능을 제공하기 때문에 실제 기능을 확장할 일이 많지는 않다. 
    - 기능 확장은 WebMvcConfigurer 를 상속 받아서 스프링 빈으로 등록하면 된다. 
      ```java
      @Bean
      public WebMvcConfigurer webMvcConfigurer() {
          return new WebMvcConfigurer() {
              @Override
              public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
                  //...
              }
	      
              @Override
              public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
                  //...
              }
          };
      }
      ```
      
** 출처: 스프링 MVC - 백엔드 웹 개발 핵심 기술
