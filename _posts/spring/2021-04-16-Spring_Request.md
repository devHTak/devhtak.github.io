---
layout: post
title: Spring MVC - 요청
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
  - Locale
    - Locale(지역 정보)를 조회한다.
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

#### HTTP 요청 메시지 - 쿼리 파라미터 전송 및 HTML Form 전송

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
      - MultiValueMap(key=\[value1, value2, ...] ex) (key=userIds, value=\[id1, id2])

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
    ```java
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

#### HTTP 요청 메시지 - HTTP Message Body

- HTTP message body에 데이터를 직접 담아서 요청
  - HTTP API에서 주로 사용, JSON, XML, TEXT
  - 데이터 형식은 주로 JSON 사용
  - POST, PUT, PATCH

- 요청 파라미터와 다르게, HTTP 메시지 바디를 통해 데이터가 직접 데이터가 넘어오는 경우는 @RequestParam , @ModelAttribute 를 사용할 수 없다. (물론 HTML Form 형식으로 전달되는 경우는요청 파라미터로 인정된다.)

- 일반 텍스트
```java
@Slf4j
@Controller
public class RequestBodyStringController {
	
	@PostMapping("/request-body-string-v1")
	public void requestBodyStringV1(HttpServletRequest request, HttpServletResponse response) throws IOException {
		ServletInputStream inputStream = request.getInputStream();
		String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);		
		log.info("messageBody: {}", messageBody);		
		response.getWriter().write("ok");
	}
	
	@PostMapping("/request-body-string-v2")
	public void requestBodyStringV2(InputStream inputStream, Writer responseWriter) throws IOException {
		String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
		log.info("messageBody: {}", messageBody);
		responseWriter.write("ok");
	}
	
	@PostMapping("/request-body-string-v3")
	public HttpEntity<String> requestBodyStringV3(HttpEntity<String> httpEntity){
		String messageBody = httpEntity.getBody();
		log.info("messageBody: {}", messageBody);		
		return new HttpEntity<>("ok");
	}
	
	@PostMapping("/request-body-string-v3-1")
	public ResponseEntity<String> requestBodyStringV3_1(RequestEntity<String> requestEntity){
		String messageBody = requestEntity.getBody();
		log.info("messageBody: {}", messageBody);		
		return ResponseEntity.ok().body("ok");
	}
	
	@PostMapping("/request-body-string-v4")
	@ResponseBody
	public String requestBodyStringV4(@RequestBody String messageBody){
		log.info("messageBody: {}", messageBody);		
		return "ok";
	}
}
```
  - V1
    - request 객체에서 InputStream을 받아서 body를 얻을 수 있다.
  
  - V2
    - parameter로 바로 InputStream과 OutputStream을 받아서 사용할 수 있다.
    - InputStream(Reader): HTTP 요청 메시지 바디의 내용을 직접 조회
    - OutputStream(Writer): HTTP 응답 메시지의 바디에 직접 결과 출력

  - V3
    - HttpEntity 객체를 파라미터, 리턴 타입으로 지정할 수 있다.
    - HttpEntity: HTTP header, body 정보를 편리하게 조회
      - 메시지 바디 정보를 직접 조회
      - 요청 파라미터를 조회하는 기능과 관계 없음 @RequestParam X, @ModelAttribute X
    - HttpEntity는 응답에도 사용 가능
      - 메시지 바디 정보 직접 반환헤더 정보 포함 가능
      - view 조회X
  
  - V3_1
    - HttpEntity 를 상속받은 다음 객체들도 같은 기능을 제공한다.
    - RequestEntity
      - HttpMethod, url 정보가 추가, 요청에서 사용
    - ResponseEntity
      - HTTP 상태 코드 설정 가능, 응답에서 사용
        ```java
      	return new ResponseEntity<String>("Hello World", responseHeaders, HttpStatus.CREATED)
	```

  - V4
    - @RequestBody 로 HttpEntity 객체 없이 바로 객체를 받을 수 있다.
      - @RequestBody 를 사용하면 HTTP 메시지 바디 정보를 편리하게 조회할 수 있다. 
      - 참고로 헤더 정보가 필요하다면 HttpEntity 를 사용하거나 @RequestHeader 를 사용하면 된다.
      - 이렇게 메시지 바디를 직접 조회하는 기능은 요청 파라미터를 조회하는 @RequestParam ,@ModelAttribute 와는 전혀 관계가 없다.
    - @ResponseBody 로 바로 객체를 리턴할 수 있다.
      - @ResponseBody 를 사용하면 응답 결과를 HTTP 메시지 바디에 직접 담아서 전달할 수 있다.
      - 물론 이 경우에도 view를 사용하지 않는다.

- JSON 요청
```java
@Slf4j
@Controller
@RequiredArgsConstructor
public class RequestBodyJsonController {
	
	private final ObjectMapper objectMapper;
	
	@PostMapping("/request-body-json-v1")
	public void requestBodyJsonV1(HttpServletRequest request, HttpServletResponse response) throws IOException {
		ServletInputStream inputStream = request.getInputStream();
		String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
		
		log.info("messageBody: {}", messageBody);
		HelloData helloData = objectMapper.readValue(messageBody, HelloData.class);
		log.info("username: {}, age: {}", helloData.getUsername(), helloData.getAge());
		response.getWriter().write("ok");
	}

	@PostMapping("/request-body-json-v2")
	@ResponseBody
	public String requestBodyJsonV2(@RequestBody String messageBody) throws IOException {
		log.info("messageBody: {}", messageBody);
		HelloData helloData = objectMapper.readValue(messageBody, HelloData.class);
		log.info("username: {}, age: {}", helloData.getUsername(), helloData.getAge());
		return "ok";
	}
	
	@PostMapping("/request-body-json-v3")
	@ResponseBody
	public String requestBodyJsonV3(@RequestBody HelloData helloData) {
		log.info("username: {}, age: {}", helloData.getUsername(), helloData.getAge());
		return "ok";
	}
	
	@PostMapping("/request-body-json-v4")
	@ResponseBody
	public String requestBodyJsonV4(HttpEntity<HelloData> httpEntity) {
		HelloData helloData = httpEntity.getBody();
		log.info("username: {}, age: {}", helloData.getUsername(), helloData.getAge());
		return "ok";
	}
	
	@PostMapping("/request-body-json-v5")
	@ResponseBody
	public HelloData requestBodyJsonV5(@RequestBody HelloData helloData) {
		log.info("username: {}, age: {}", helloData.getUsername(), helloData.getAge());
		return helloData;
	}
}
```
  - V1
    - HttpServletRequest를 사용해서 직접 HTTP 메시지 바디에서 데이터를 읽어와서, 문자로 변환한다.
    - 문자로 된 JSON 데이터를 Jackson 라이브러리인 objectMapper 를 사용해서 자바 객체로 변환한다

  - V2
    - 이전에 학습했던 @RequestBody 를 사용해서 HTTP 메시지에서 데이터를 꺼내고 messageBody에 저장한다.
    - 문자로 된 JSON 데이터인 messageBody 를 objectMapper 를 통해서 자바 객체로 변환한다.

  - V3
    - @RequestBody 객체 파라미터
      - @RequestBody HelloData data
      - @RequestBody 에 직접 만든 객체를 지정할 수 있다.
      - HttpEntity , @RequestBody 를 사용하면 HTTP 메시지 컨버터가 HTTP 메시지 바디의 내용을 우리가 원하는 문자나 객체 등으로 변환해준다.
      - HTTP 메시지 컨버터는 문자 뿐만 아니라 JSON도 객체로 변환해주는데, 우리가 방금 V2에서 했던 작업을 대신 처리해준다.
    - @RequestBody는 생략 불가능
      - @ModelAttribute 에서 학습한 내용을 떠올려보자.
      - 스프링은 @ModelAttribute , @RequestParam 해당 생략시 다음과 같은 규칙을 적용한다.
      - String , int , Integer 같은 단순 타입 = @RequestParam, 나머지 = @ModelAttribute (argument resolver 로 지정해둔 타입 외)
    
  - V4
    - @ResponseBody
      - 응답의 경우에도 @ResponseBody 를 사용하면 해당 객체를 HTTP 메시지 바디에 직접 넣어줄 수 있다.
      - 물론 이 경우에도 HttpEntity 를 사용해도 된다.
    - @RequestBody 요청
      - JSON 요청 HTTP 메시지 컨버터 객체
    - @ResponseBody 응답
      - 객체 HTTP 메시지 컨버터 JSON 응답
      
** 참고: 스프링 MVC - 백엔드 웹 개발 핵심 기술
