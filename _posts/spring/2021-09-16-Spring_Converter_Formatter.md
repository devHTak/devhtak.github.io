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

#### Formatter

- Formatter는 Converter 기능에 Locale 정보를 추가하여 사용할 수 있다.
- Locale
  - 날짜 숫자의 표현 방법은 Locale 현지화 정보가 사용될 수 있다.
  - 객체를 특정한 포멧에 맞추어 문자로 출력하거나 또는 그 반대의 역할을 하는 것에 특화된 기능이 바로 포맷터( Formatter )이다.

- Formatter interface
  ```java
  public interface Printer<T> {
      String print(T object, Locale locale);
  }
  public interface Parser<T> {
      T parse(String text, Locale locale) throws ParseException;
  }
  public interface Formatter<T> extends Printer<T>, Parser<T> {}
  ```

  - 예시 : Price같이 Number 타입을 Locale 정보에 따른 Formatter 생성
    ```java
    @Slf4j
    public class MyNumberFormatter implements Formatter<Number> {
        @Override
        public Number parse(String text, Locale locale) throws ParseException {
            log.info("text={}, locale={}", text, locale);
            NumberFormat format = NumberFormat.getInstance(locale);
            return format.parse(text);
        }
        
        @Override
        public String print(Number object, Locale locale) {
            log.info("object={}, locale={}", object, locale);
            return NumberFormat.getInstance(locale).print(object);
        }
    }
    ```
    - 테스트
      ```java
      @Test
      void parse() throws ParseException {
          Number result = formatter.parse("1,000", Locale.KOREA);
          assertThat(result).isEqualTo(1000L); //Long 타입 주의
      }
      @Test
      void print() {
          String result = formatter.print(1000, Locale.KOREA);
          assertThat(result).isEqualTo("1,000");
      }
      ```

- ConversionService
  - FormattingConversionService는 Formatter를 지원하는 컨버전 서비스
  - DefaultFormattingConversionService 는 FormattingConversionService에 기본적인 통화, 숫자관련 몇가지 기본 포맷터를 추가하여 제공
    ```java
    @Test
    void formattingConversionService() {
        DefaultFormattingConversionService conversionService = new DefaultFormattingConversionService();
        
        //컨버터 등록
        conversionService.addConverter(new StringToIpPortConverter());
        conversionService.addConverter(new IpPortToStringConverter());
        
        //포맷터 등록
        conversionService.addFormatter(new MyNumberFormatter());
        
        //컨버터 사용
        IpPort ipPort = conversionService.convert("127.0.0.1:8080", IpPort.class);
        assertThat(ipPort).isEqualTo(new IpPort("127.0.0.1", 8080));
        
        //포맷터 사용
        assertThat(conversionService.convert(1000, String.class)).isEqualTo("1,000");
        assertThat(conversionService.convert("1,000", Long.class)).isEqualTo(1000L);
    }
    ```
  - DefaultFormattingConversionService 상속 관계
    - FormattingConversionService 는 ConversionService 관련 기능을 상속받기 때문에 결과적으로 컨버터도 포맷터도 모두 등록할 수 있다.
    - 사용할 때는 ConversionService 가 제공하는 convert 를 사용하면 된다.
    - 스프링 부트는 DefaultFormattingConversionService 를 상속 받은 WebConversionService 를 내부에서 사용한다.

- 포맷터 적용하기
  - Converter와 동일하게 등록하면 된다.
  - FormatterRegistry 에 addFormatter를 사용하면 된다.
  ```java
  @Configuration
  public class WebConfig implements WebMvcConfigurer {
      @Override
      public void addFormatters(FormatterRegistry registry) {
          registry.addConverter(new StringToIpPortConverter());
          registry.addConverter(new IpPortToStringConverter());
          
          //추가
          registry.addFormatter(new MyNumberFormatter());
      }
  }
  ```
  - 우선순위: 컨버터가 우선하므로 포맷터가 적용되지 않고, 컨버터가 적용된다.

- Spring 기본 제공 Formatter
  - @NumberFormat : 숫자 관련 형식 지정 포맷터 사용, NumberFormatAnnotationFormatterFactory
  - @DateTimeFormat : 날짜 관련 형식 지정 포맷터 사용, Jsr310DateTimeFormatAnnotationFormatterFactory
  ```java
  @Data
  class Form {
      @NumberFormat(pattern = "###,###")
      private Integer number;
      @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
      private LocalDateTime localDateTime;
  }
  ```
  
- 주의
  ```
  메시지 컨버터( HttpMessageConverter )에는 컨버전 서비스가 적용되지 않는다.
  특히 객체를 JSON으로 변환할 때 메시지 컨버터를 사용하면서 이 부분을 많이 오해하는데, HttpMessageConverter 의 역할은 HTTP 메시지 바디의 내용을 객체로 변환하거나 객체를 HTTP 메시지 바디에 입력하는 것이다. 
  예를 들어서 JSON을 객체로 변환하는 메시지 컨버터는 내부에서 Jackson 같은 라이브러리를 사용한다. 
  객체를 JSON으로 변환한다면 그 결과는 이 라이브러리에 달린 것이다. 
  따라서 JSON 결과로 만들어지는 숫자나 날짜 포맷을 변경하고 싶으면 해당 라이브러리가 제공하는 설정을 통해서 포맷을 지정해야 한다. 
  결과적으로 이것은 컨버전 서비스와 전혀 관계가 없다.
  컨버전 서비스는 @RequestParam , @ModelAttribute , @PathVariable , 뷰 템플릿 등에서 사용할 수 있다
  ```

#### 출처

- 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술 \[인프런 김영한님 강의]
