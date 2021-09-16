---
layout: post
title: Converter와 Formatter
summary: Spring MVC Basic
author: devhtak
date: '2021-09-16 21:41:00 +0900'
category: Spring
---

#### Converter

- 타입 변환
  - 문자를 숫자로 변환하거나, 반대로 숫자를 문자로 변환해야 하는 것처럼 타입 변환을 해야 하는 경우가 많다.
    ```java
    @GetMapping("/hello-v1")
    public String helloV1(HttpServletRequest request) {
        String data = request.getParameter("data"); //문자 타입 조회
        Integer intValue = Integer.valueOf(data); //숫자 타입으로 변경
        System.out.println("intValue = " + intValue);
        return "ok";
    }
    ```
    - Parameter, PathVariable 등 넘어오는 데이터는 기본적으로 String 타입이기 때문에, Integer 타입으로 변환해주어야 한다.
  - 하지만 Spring Boot에서는 바로 Integer로 받을 수 있게 한다.
    ```java
    @GetMapping("/hello-v2")
    public String helloV2(@RequestParam Integer data) {
        System.out.println("data = " + data);
        return "ok";
    }
    ```
    - 해당 역할을 Converter가 해준다.
    - @RequestParam, @ModelAttribute, @PathVariable 등에서도 확인할 수 있다.
    - 스프링의 타입 변환 적용 예
      - 스프링 MVC 요청 파라미터 (@RequestParam , @ModelAttribute , @PathVariable)
      - @Value 등으로 YML 정보 읽기
      - XML에 넣은 스프링 빈 정보를 변환
      - 뷰를 렌더링 할 때

- Converter 
  - Converter Interface
    ```java
    public interface Converter<S, T> {
        T convert(S source);
    }
    ```
    - Type Converter를 설정하기 위해 Converter Interface를 구현하면 된다.

  - 예시
    - IpPort.java
      ```java
      @Getter
      @EqualsAndHashCode
      public class IpPort {
          private String ip;
          private int port;
          public IpPort(String ip, int port) {
              this.ip = ip;
              this.port = port;
          }
      }
      ```
    - StringToIpPortConverter.java
      ```java
      @Slf4j
      public class StringToIpPortConverter implements Converter<String, IpPort> {
          @Override
          public IpPort convert(String source) {
              log.info("convert source={}", source);
              String[] split = source.split(":");
              String ip = split[0];
              int port = Integer.parseInt(split[1]);
              return new IpPort(ip, port);
          }
      }
      ```
    - IpPortToStringConver.java
      ```java
      @Slf4j
      public class IpPortToStringConverter implements Converter<IpPort, String> {
          @Override
          public String convert(IpPort source) {
              log.info("convert source={}", source);
              return source.getIp() + ":" + source.getPort();
          }
      }
      ```
    - Test
      ```java
      @Test
      void stringToIpPort() {
          StringToIpPortConverter converter = new StringToIpPortConverter();
          String source = "127.0.0.1:8080";
          IpPort result = converter.convert(source);
          assertThat(result).isEqualTo(new IpPort("127.0.0.1", 8080));
      }
      @Test
      void ipPortToString() {
          IpPortToStringConverter converter = new IpPortToStringConverter();
          IpPort source = new IpPort("127.0.0.1", 8080);
          String result = converter.convert(source);
          assertThat(result).isEqualTo("127.0.0.1:8080");
      }
      ```
  - 다양한 Converter
    - Converter 기본 타입 컨버터
    - ConverterFactory 전체 클래스 계층 구조가 필요할 때
    - GenericConverter 정교한 구현, 대상 필드의 애노테이션 정보 사용 가능
    - ConditionalGenericConverter 특정 조건이 참인 경우에만 실행
  
- ConversionService
  ```java
  public interface ConversionService {
      boolean canConvert(@Nullable Class<?> sourceType, Class<?> targetType);
      boolean canConvert(@Nullable TypeDescriptor sourceType, TypeDescriptor targetType);
      <T> T convert(@Nullable Object source, Class<T> targetType);
      Object convert(@Nullable Object source, @Nullable TypeDescriptor sourceType, TypeDescriptor targetType);
  }
  ```
  - 컨버팅 가능 여부, 컨버팅 기능을 제공
  - 테스트
    ```java
    @Test
    void conversionService() {
        //등록
        DefaultConversionService conversionService = new DefaultConversionService();
        conversionService.addConverter(new StringToIpPortConverter());
        conversionService.addConverter(new IpPortToStringConverter());

        //사용
        assertThat(conversionService.convert("10", Integer.class)).isEqualTo(10);
        assertThat(conversionService.convert(10, String.class)).isEqualTo("10");
        IpPort ipPort = conversionService.convert("127.0.0.1:8080", IpPort.class);
        assertThat(ipPort).isEqualTo(new IpPort("127.0.0.1", 8080));
        String ipPortString = conversionService.convert(new IpPort("127.0.0.1", 8080), String.class);
        assertThat(ipPortString).isEqualTo("127.0.0.1:8080");
     }
     ```
     - DefaultConversionService 는 ConversionService 인터페이스를 구현했는데, 추가로 컨버터를 등록하는 기능도 제공한다.
     - ISP (Interface Segregation Principal)
       ```
       등록 기능과 사용 기능 분리에 필요
       : Converter를 등록할 때에는 구현된 Converter를 알아야된다.
       : 하지만 Converter는 내부에서 제공하기 떄문에 Converter를 사용하는 입장에서는 컨버전 서비스 인터페이스에만 의존하면 된다.
       ```
       - DefaultConversionService 는 다음 두 인터페이스를 구현했다.
         - ConversionService : 컨버터 사용에 초점
         - ConverterRegistry : 컨버터 등록에 초점

- Spring Boot에 Converter 등록
  - WebConfig.java 
    ```java
    @Configuration
    public class WebConfig implements WebMvcConfigurer {
        @Override
        public void addFormatters(FormatterRegistry registry) {
            registry.addConverter(new StringToIpPortConverter());
            registry.addConverter(new IpPortToStringConverter());
        }
    }
    ```
    - WebMvcConfigurer 가 제공하는 addFormatters() 를 사용해서 추가하고 싶은 컨버터를 등록하면 된다. 
    - 스프링은 내부에서 ConversionService 를 사용하여 컨버터를 추가해준다.
  
  - @RequestParam ... Type Converter 처리 과정
    - @RequestParam 은 @RequestParam 을 처리하는 ArgumentResolver 인 RequestParamMethodArgumentResolver 에서 ConversionService 를 사용해서 타입을 변환한다. 
    
- Thymeleaf 에서 converter 사용하기
  - View로 전송한 데이터 사용
    - Controller
      ```java
      @GetMapping("/converter-view")
      public String converterView(Model model) {
          model.addAttribute("ipPort", new IpPort("127.0.0.1", 8080));
          return "converter-view";
      }
      ```
    - View
      ```html
      <ul>
          <li>${ipPort}: <span th:text="${ipPort}" ></span></li>
          <li>${{ipPort}}: <span th:text="${{ipPort}}" ></span></li>
      </ul>
      ```
    - 타임리프는 ${{...}} 를 사용하면 자동으로 컨버전 서비스를 사용해서 변환된 결과를 출력해준다. 
    - 스프링이 제공하는 컨버전 서비스를 사용하므로, 우리가 등록한 컨버터들을 사용할 수 있다.
    - 변수 표현식 : ${...}
    - 컨버전 서비스 적용 : ${{...}}

  - Form에서 사용
    - View
      ```html
      <form th:object="${form}" th:method="post">
          th:field <input type="text" th:field="*{ipPort}"><br/>
          th:value <input type="text" th:value="*{ipPort}">(보여주기 용도)<br/>
          <input type="submit"/>
      </form>
      ```

#### 

#### 출처

- 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술 \[인프런 김영한님 강의]
