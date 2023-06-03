---
layout: post
title: Spring Boot. Config
summary: Spring Boot
author: devhtak
date: '2023-06-03 21:41:00 +0900'
category: Spring
---

#### Spring Boot Auto Configuration
- Spring Boot 는 자동 구성(Auto Configuration)이라는 기능을 제공하여 일반적으로 자주 사용하는 많은 빈을 자동으로 등록해준다.
- spring-boot-starter.jar > spring-boot-autoconfigure.jar
- 예시, JdbcTemplateAutoConfiguration
  ```java
  @AutoConfiguration(after = DataSourceAutoConfiguration.class)
  @ConditionalOnClass({ DataSource.class, JdbcTemplate.class })
  @ConditionalOnSingleCandidate(DataSource.class)
  @EnableConfigurationProperties(JdbcProperties.class)
  @Import({ DatabaseInitializationDependencyConfigurer.class, JdbcTemplateConfiguration.class,
      NamedParameterJdbcTemplateConfiguration.class })
  public class JdbcTemplateAutoConfiguration {}
  ```
  - @AutoConfiguration
    - 자동구성을 사용하려면 이 애노테이션을 등록해야 한다.
    - 자동구성도 내부에 @Configuration 이 있어서 빈을 등록하는 자바 설정 파일로 사용할 수 있다.
    - after = DataSourceAutoConfiguration.class
      - 자동구성이 실행되는 순서를 지정
      - JdbcTemplate은 DataSource가 필요하기 때문에 DataSourceAutoConfiguration 다음 실행하도록 지정
  - @ConditionalOnClass({ DataSource.class, JdbcTemplate.class })
    - 이런 클래스가 있는 경우 설정 동작. 만약 없으면 여기 있는 설정들이 무효화
  - @ConditionalXxx
    - JdbcTemplate은 DataSource, JdbcTempalte라는 클래스가 있어야 동작할 수 있다.
  - @Import
    - 스프링에서 자바 설정을 추가할 때 사용한다.
- JdbcTemplateConfiguration
  ```java
  @Configuration(proxyBeanMethods = false)
  @ConditionalOnMissingBean(JdbcOperations.class)
  class JdbcTemplateConfiguration {
    @Bean
    @Primary
    JdbcTemplate jdbcTemplate(DataSource dataSource, JdbcProperties properties) {
      JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
      JdbcProperties.Template template = properties.getTemplate();
      jdbcTemplate.setFetchSize(template.getFetchSize());
      jdbcTemplate.setMaxRows(template.getMaxRows());
      if (template.getQueryTimeout() != null) {
        jdbcTemplate.setQueryTimeout((int) template.getQueryTimeout().getSeconds());
      }
      return jdbcTemplate;
    }
  }
  ```
  - @ConditionalOnMissingBean(JdbcOperations.class)
    - JdbcOperations 빈이 없으면 동작
- 자동구성 기능을 위한 개념
  - @Conditional: 특정 조건에 맞을 때 설정이 동작하도록 한다
  - @AutoConfiguration: 자동 구성 등록
- 스프링 부트 동작
  - @SpringBootApplication -> @EnableAutoConfiguration -> @Import(AutoConfigurationImportSelector.class)
  - META-INF/spring/org.springframework.boot.autoconfigure.Autoconfiguration.import에 정의된 Configuration들을 읽어서 등록
    - 2.7 이전까지는 spring.factories 
  - @ComponentScan으로 먼저 빈을 읽고 난후 import에 정의된 Configuration들을 읽는다.
  - 해당 메타파일에 등록되어있는 클래스들도 전부 @Configuration이 붙어있으며, 이 클래스들에 특정조건(@Conditional) 가능

#### 외부 설정
- 외부설정 방식
  - OS 환경변수
  - 자바 시스템 속성(-Dprofile=develop)
  - 자바 커맨드라인 인수
  - 외부파일(설정 데이터)
- 스프링에서 외부 설정
  - 스프링은 Environment, PropertySource를 추상화하여 환경변수 종류에 상관없이 유연하게 사용할 수 있다.
  - Environment: 특정 외부 설정에 종속되지 않고, key=value 형식의 외부설정에 접근할 수 있다.
    ```java
    String url = environment.getProperty("spring.datasource.url");
    ```
    - @Value
      - 필드, 파라미터에 사용해 주입할 수 있다.
      - 주입 받아야하는 여러개인 경우 @ConfigurationProperties를 활용할 수 있다.
    - @ConfigurationProperties
      ```java
      @ConfigurationProperties(prefix = "spring.jdbc")
      public class JdbcProperties { //... }
      ```
      - 외부 설정의 묶음 정보를 객체로 변환하는 기능 제공, 지정해둔 타입 이외의 타입이 들어오면 예외 발생하며 객체로 관리하기 때문에 type safety
      - @ConfigurationPropertiesScan을 사용하여 한번 더 설정해야 하는 번거러움을 피할 수 있다.
      - setter 가 필요하지 않으며 java validation으로 검증도 가능
    - @EnableConfigurationProperties
      ```java
      @AutoConfiguration(after = DataSourceAutoConfiguration.class)
      @ConditionalOnClass({ DataSource.class, JdbcTemplate.class })
      @ConditionalOnSingleCandidate(DataSource.class)
      @EnableConfigurationProperties(JdbcProperties.class)
      @Import({ DatabaseInitializationDependencyConfigurer.class, JdbcTemplateConfiguration.class,
          NamedParameterJdbcTemplateConfiguration.class })
      public class JdbcTemplateAutoConfiguration {

      }
      ```
  - PropertySource: 로딩 시점에 필요한 PropertySource 를 생성하여 Environment를 사용할 수 있게 한다
  
- 설정 우선순위
  - 설정 데이터(application.yml)
    - jar 내부
      - application.yml 파일
      - 프로필 적용 파일(application-develop.yml)
    - jar 외부
      - application.yml 파일
      - 프로필 적용 파일(application-develop.yml)
  - OS 환경변수
  - 자바 시스템 속성
  - 커맨드라인 옵션인수
  - @TestPropertySource(테스트 실행 시)



