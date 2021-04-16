---
layout: post
title: Spring MVC - 요청과 응답
summary: 김영한님 - 스프링 MVC - 백엔드 웹 개발 핵심 기술
author: devhtak
date: '2021-04-16 21:41:00 +0900'
category: Spring
---

#### @Controller, @RestController

- @Controller
  ```java
  @Target({ElementType.TYPE})
  @Retention(RetentionPolicy.RUNTIME)
  @Documented
  @Component
  public @interface Controller {

    /**
     * The value may indicate a suggestion for a logical component name,
     * to be turned into a Spring bean in case of an autodetected component.
     * @return the suggested component name, if any (or empty String otherwise)
     */
    @AliasFor(annotation = Component.class)
    String value() default "";

  }
  ```
  - Controller임을 명시하는 애노테이션
  - 자동으로 빈에 등록된다.
  - @Controller는 반환 값이 String이면 뷰 이름으로 인식되며, ViewResolver가 뷰를 찾고 뷰가 렌더링된다.   

- @RestController
  ```java
  @Target(ElementType.TYPE)
  @Retention(RetentionPolicy.RUNTIME)
  @Documented
  @Controller
  @ResponseBody
  public @interface RestController {

    /**
     * The value may indicate a suggestion for a logical component name,
     * to be turned into a Spring bean in case of an autodetected component.
     * @return the suggested component name, if any (or empty String otherwise)
     * @since 4.0.1
     */
    @AliasFor(annotation = Controller.class)
    String value() default "";

  }
  ```
  - 반환값으로 뷰를 찾는 것이 아닌, HTTP Message Body에 바로 입력한다.
  - @Controller + @ResponseBody 복합 애노테이션 

#### @RequestMapping

- Dispatcher Servlet이 핸들러를 찾을 때 핸들러 매핑(RequestMappingHandlerMapping), 핸들러 어답터(RequestMappingHandlerAdapter)에 의해 찾아지는 핸들러
- URL을 주어 URL 호출이 오면 해당 메서드가 실행된다.
- HTTP Method 매핑 축약
  - GET(조회) : @GetMapping
  - POST(삽입): @PostMapping
  - PUT(업데이트): @PutMapping
  - PATCH(업데이트): @PatchMapping
  - DELETE(삭제): @DeleteMapping
    - 참고: PUT과 PATCH 차이점
      - PUT은 객체의 모든 변수를 업데이트 하며, PATCH는 일부분을 업데이트 한다.
    - 참고: POST는 요청이 날라올 때마다 결과 값이 달라진다.
      - 새로운 객체가 insert된다.

- 특정 파라미터 조건 매핑
  ```java
  /**
   * 파라미터로 추가 매핑
   * params="mode",
   * params="!mode"
   * params="mode=debug"
   * params="mode!=debug" (! = )
   * params = {"mode=debug","data=good"} 
   */
  @GetMapping(value = "/mapping-param", params = "mode=debug")
  public String mappingParam() {
      log.info("mappingParam");
      return "ok";
  }
  ```
  - localhost:8080/mapping-param?mode=debug 와 같이 요청해야 된다.

- 특정 헤더 조건 매핑
  ```java
  /**
   * 특정 헤더로 추가 매핑
   * headers="mode",
   * headers="!mode"
   * headers="mode=debug"
   * headers="mode!=debug" (! = )
   */
  @GetMapping(value = "/mapping-header", headers = "mode=debug")
  public String mappingHeader() {
      log.info("mappingHeader");
      return "ok";
  }
  ```
  - request header에 mode = debug가 있어야 매핑된다.

- 미디어 타입 조건 매핑 - HTTP Request Header에 Content-Type: consume

  ```java
  /**
   * Content-Type 헤더 기반 추가 매핑 Media Type
   * consumes="application/json"
   * consumes="!application/json"
   * consumes="application/*"
   * consumes="*\/*"
   * MediaType.APPLICATION_JSON_VALUE
   * Request 헤더에 Content-Type이 매핑이되어야 요청된다.
   */
  @PostMapping(value = "/mapping-consume", consumes = MediaType.TEXT_PLAIN_VALUE)
  public String mappingConsumes() {
      log.info("mappingConsumes");
      return "ok";
  }
  ```
  - HTTP 요청의 Content-Type 헤더를 기반으로 미디어 타입으로 매핑한다.
  - 만약 맞지 않으면 HTTP 415 상태코드(Unsupported Media Type)을 반환한다.
  ```
  consumes = "text/plain"
  consumes = {"text/plain", "application/*"}
  consumes = MediaType.TEXT_PLAIN_VALUE
  ```

- 미디어 타입 조건 매핑 - HTTP Request Header에 Accept: produce

  ```java
  /** * Accept 헤더 기반 Media Type
  * produces = "text/html"
  * produces = "!text/html"
  * produces = "text/*"
  * produces = "*\/*"
  */
  @PostMapping(value = "/mapping-produce", produces = MediaType.TEXT_PLAIN_VALUE)
  public String mappingProduces() {
      log.info("mappingProduces");
      return "ok";
  }
  ```
  - HTTP 요청의 Accept 헤더를 기반으로 미디어 타입으로 매핑한다.
  - 만약 맞지 않으면 HTTP 406 상태코드(Not Acceptable)을 반환한다.

  ```
  produces = "text/plain"
  produces = {"text/plain", "application/*"}
  produces = MediaType.TEXT_PLAIN_VALUE
  produces = "text/plain;charset=UTF-8"
  ```

#### @PathVariable

```java
/** * PathVariable 사용
 * 변수명이 같으면 생략 가능
 * @PathVariable("userId") String userId -> @PathVariable userId
 */
@GetMapping("/mapping/{userId}")
public String mappingPath(@PathVariable("userId") String data) {
    log.info("mappingPath userId={}", data);
    return "ok";
}
```

- HTTP API에서 선호하는 스타일로 리소스 경로에 식별자를 넣는다.
- @PathVariable("variableName") 을 사용하면 URL에 {variableName}과 매핑되는 부분을 편리하게 조회할 수 있다.
- @PathVariable 애노테이션이 붙은 변수 이름과 파라미터 이름이 같으면 생략할 수 있다.

#### HTTP 요청 - 기본, 헤더 

```java
@Slf4j
@RestController
public class RequestHeaderController {
  
    @RequestMapping("/headers")
    public String headers(HttpServletRequest request,
                HttpServletResponse response,
                HttpMethod httpMethod,
                Locale locale,
                @RequestHeader MultiValueMap<String, String> headerMap,
                @RequestHeader("host") String host,
                @CookieValue(value = "myCookie", required = false) String cookie) {

        log.info("request={}", request);
        log.info("response={}", response);
        log.info("httpMethod={}", httpMethod);
        log.info("locale={}", locale);
        log.info("headerMap={}", headerMap);
        log.info("header host={}", host);
        log.info("myCookie={}", cookie);

        return "ok";
    }
}
```

- 애노테이션 기반의 스프링 컨트롤러는 다양한 파라미터를 지원한다.
    
  - HttpServletRequest
    - HTTP 요청
  - HttpServletResponse
    - HTTP 응답
  - (Enum)HttpMethod
    - HTTP 메서드를 조회한다. org.springframework.http.HttpMethod
  - Locale|Locale(지역 정보)를 조회한다.|
  - @RequestHeader MultiValueMap<String, String> headerMapper
    - 모든 HTTP 헤더를 MultiValueMap 형식으로 조회
    - MultiValueMap 이란?
      - Map과 유사하나, 하나의 Key에 여러 value를 저장할 수 있다.
      - HTTP header, HTTP Query Parametre와 같이 하나의 키에 여러 값을 받을 때 사용
        ```java
        MultiValueMap<String, String> map = new LinkedMultiValueMap();
        map.add("keyA", "value1");
        map.add("keyA", "value2");

        List<String> values = map.get("keyA");
        ```
  - @RequestHeader("host") String host
    - 특정 HTTP 헤더 조회한다. 
    - 속성: required(필수 값 여부), defaultValue(기본값)
  - @CookieValue(value="myCookie", required=false) String cookie
    - 특정 쿠키를 조회한다. 
    - 속성: required(필수 값 여부), defaultValue(기본값)

  - 참고
    - 파라미터 목록: https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-arguments
    - 응답 값 목록: https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-return-types

#### HTTP 요청 파라미터

- Client에게 요청 데이터를 전달할 때 사용하는 3가지 방법
  - GET - 쿼리 파라미터
    - /url?username=hello&age=20
    - 메시지 바디 없이, URL의 쿼리 파라미터에 데이터를 포함해서 전달
    - 예) 검색, 필터, 페이징등에서 많이 사용
  - POST - HTML Form
    - content-type: application/x-www-form-urlencoded
    - 메시지 바디에 쿼리 파라미터 형식으로 전달 username=hello&age=20
    - 예) 회원 가입, 상품 주문, HTML Form 사용
  - HTTP message body에 데이터를 직접 담아서 요청
    - HTTP API에서 주로 사용, JSON, XML, TEXT
    - 데이터 형식은 주로 JSON 사용
    - POST, PUT, PATCH

#### 쿼리 파라미터 전송 및 HTML Form 전송

- GET, 쿼리 파라미터 전송
  ```
  http://localhost:8080/request-param?username=hello&age=20
  ```

- POST, HTML Form 전송
  ```
  POST /request-param ...
  content-type: application/x-www-form-urlencoded
  username=hello&age=20
  ```

- GET 쿼리 파리미터 전송 방식이든, POST HTML Form 전송 방식이든 둘다 형식이 같으므로 구분없이 조회할 수 있다.
- 요청 파라미터(request parameter) 조회라 한다.

- @RequestParam
  
  ```java
  @RequestMapping("/request-param-v1")
	public void requestParamV1(HttpServletRequest request, HttpServletResponse response) throws IOException {
      String username = request.getParameter("username");
      int age = Integer.parseInt(request.getParameter("age"));
      log.info("username={}, age={}", username, age);		
      response.getWriter().write("ok");
	}

	@ResponseBody
	@RequestMapping("/request-param-v2")
	public String requestParamV2(@RequestParam("username") String memberName, @RequestParam("age") int memberAge) {
      log.info("username={}, age={}", memberName, memberAge);		
      return "ok";
	}
	
	@ResponseBody
	@RequestMapping("/request-param-v3")
	public String requestParamV3(@RequestParam String username, @RequestParam int age) {
      log.info("username={}, age={}", username, age);		
      return "ok";
	}
	
	@ResponseBody
	@RequestMapping("/request-param-v4")
	public String requestParamV4(String username, int age) {
      log.info("username={}, age={}", username, age);		
      return "ok";
	}

  @ResponseBody
  @RequestMapping("/request-param-map")
  public String requestParamMap(@RequestParam Map<String, Object> paramMap) {
      log.info("username={}, age={}", paramMap.get("username"),
      paramMap.get("age"));
      return "ok";
  }
  ```
  - V2
    - @RequestParam : 파라미터 이름으로 바인딩
    - @ResponseBody : View 조회를 무시하고, HTTP message body에 직접 해당 내용 입력
    - @RequestParam의 name(value) 속성이 파라미터 이름으로 사용
      - @RequestParam("username") String memberName -> request.getParameter("username")
  
  - V3
    - HTTP 파라미터 이름이 변수 이름과 같으면 @RequestParam(name="xx") 생략 가능

  - V4
    - String , int , Integer 등의 단순 타입이면 @RequestParam 도 생략 가능
  
  - required
    - 파라미터 필수 여부, default로 true 이다.
    - required가 false인 경우, NULL이 입력된다.
      - primitive type에 변수를 사용하면 null이 입력되지 않기 때문에 500 예외 발생
    - 파라미터 이름만 사용
      ```
      /request-param?username=&age=10
      ```
      - 빈문자가 들어가게 된다.

  - default 
    - 파라미터에 값이 없는 경우 defaultValue 를 사용하면 기본 값을 적용할 수 있다.
    - 이미 기본 값이 있기 때문에 required 는 의미가 없다.
    - defaultValue 는 빈 문자의 경우에도 설정한 기본 값이 적용된다.
      ```
      /request-param?username=&age=10
      ```
  
  - Map 또는 MultiValueMap
    - 파라미터의 값이 1개가 확실하다면 Map 을 사용해도 되지만, 그렇지 않다면 MultiValueMap 을 사용하자.
    - @RequestParam Map
      - Map(key=value)
    - @RequestParam MultiValueMap
      - MultiValueMap(key=[value1, value2, ...] ex) (key=userIds, value=[id1, id2])

- @ModelAttribute
  
  - 요청 파라미터에 대한 프로퍼티를 갖고 있는 객체로 변환이 가능하다.
  - @ModelAttribute 실행 순서
    - HelloData 객체를 생성한다 
    - 요청 파라미터의 이름으로 HelloData 객체의 프로퍼티를 찾고, setter를 호출하여 값을 바인딩한다.
    - ex) 파라미터: username -> setter: setUsername() 메서드 호출
    - 프로퍼티
      - 객체에 getUsername() , setUsername() 메서드(getter/setter)가 있으면, 이 객체는 username 이라는 프로퍼티를 가지고 있다.
      - username 프로퍼티의 값을 변경하면 setUsername() 이 호출되고, 조회하면 getUsername() 이 호출된다
      
  - username, age를 갖는 객체 생성
    ```
    @Getter @Setter @ToString
    public class HelloData {
        private String username;
        private int age;
    }
    ```
  
  - @ModelAttribute 사용
    
    ```java
    @ResponseBody
    @RequestMapping("/model-attribute-v1")
    public String modelAttributeV1(@ModelAttribute HelloData helloData) {
        log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());
        return "ok";
    }

    @ResponseBody
    @RequestMapping("/model-attribute-v2")
    public String modelAttributeV2(HelloData helloData) {
        log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());
        return "ok";
    }
    ```
    - V1
      - Spring은 username, age의 setter/getter를 갖고 있는 HelloData 객체를 생성하고 setter를 통해 해당 값을 바인딩한다.
      - 바인딩 오류
        - int형인 age에 abc와 같은 문자열이 들어가면 BindException이 발생한다.
    
    - V2
      - @ModelAttribute 생략이 가능하다.

- 스프링은 @RequestParam, @ModelAttribute 생략시 다음과 같은 규칙이 적용된다.
  - String, int, Integer와 같이 primitive type과 그에대한 wrapper class -> @RequestParam
  - 나머지 -> @ModelAttribute (argument resolver로 지정해둔 타입 외)


** 참고: 스프링 MVC - 백엔드 웹 개발 핵심 기술
