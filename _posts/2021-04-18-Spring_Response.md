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




** 출처: 스프링 MVC - 백엔드 웹 개발 핵심 기술
