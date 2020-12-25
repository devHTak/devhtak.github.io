---
layout: post
title: Swagger Documentation
summary: RESTful API
author: devhtak
date: '2020-12-25 22:41:00 +0900'
category: RESTful API
---

### Swagger Documentation

- Swagger는 개발자가 REST API 서비스를 설계, 빌드, 문서화할 수 있도록한다.
- 다음과 같은 경우 유용하게 사용된다.
  - 다른 개발팀과 협업을 진행할 경우
  - 이미 구축되어있는 프로젝트에 대한 유지보수를 진행할 경우
  - 백엔드의 API를 호출하는 프론트앤드 프로그램을 제작할 경우

- dependency
  ```java
  <dependency>
      <groupId>io.springfox</groupId>
      <artifactId>springfox-swagger2</artifactId>
      <version>2.9.2</version>
  </dependency>
  <dependency>
      <groupId>io.springfox</groupId>
      <artifactId>springfox-swagger-ui</artifactId>
      <version>2.9.2</version>
  </dependency>
  ```

- URL
  - localhost:8180/v2/api-docs
    - Swagger 문서에 기본이 되는 정보를 JSON 형태로 제공한다 
  - localhost:8180/swagger-ui.html
    - HTML 형식으로 배포되고 있는 Api Documentation을 볼 수 있다.

- Customizing
  - config 파일 등록
    - api info, contact 정보, produce(accept), consume(content-type) 등을 지정한다.
      ```java
      @Configuration
      @EnableSwagger2
      public class SwaggerConfig {
          private static final Contact DEFAULT_CONTACT = new Contact("devhtak", "https://devhtak.github.io", "devhtak@gmail.com");
          private static final ApiInfo DEFAULT_API_INFO = new ApiInfo("API Title", "My User management API service", "1.0", "urn:tos", 
              DEFAULT_CONTACT, "Apache 2.0", "http://www.apache.org/license/LICENSE-2.0", new ArrayList<>());
          private static final Set<String> DEFAULT_PRODUCES_AND_CONSUMES = new HashSet<>(Arrays.asList("application/json", "application/xml"));

          @Bean
          public Docket api() {
              return new Docket(DocumentationType.SWAGGER_2)
                  .apiInfo(DEFAULT_API_INFO)
                  .produces(DEFAULT_PRODUCES_AND_CONSUMES)
                  .consumes(DEFAULT_PRODUCES_AND_CONSUMES);
          }
      }  
      ```
    - HATEOAS와 같이 사용할 때 예외가 발생할 수 있다.
      - 아래와 같이 configuration에 빈으로 등록하면 사용할 수 있다.
        ```java
        @Bean
        public LinkDiscoverers discoverers() {
            List<LinkDiscoverer> plugins = new ArrayList<>();
            plugins.add(new CollectionJsonLinkDiscoverer());
            return new LinkDiscoverers(SimplePluginRegistry.create(plugins));
        }
        ```
        
  - model 파일
    - 사용되는 객체에 description등을 사용할 수 있다.
      ```java
      @Data
      @NoArgsConstructor @AllArgsConstructor
      @Getter @Setter
      @ApiModel(description = "사용자 상세 정보를 위한 도메인 객체")
      public class User {
          private Integer id;
          @Size(min = 2, message= "이름은 2글자 이상 입력해주세요.")
          @ApiModelProperty(notes = "사용자 이름을 입력해 주세요.")
          private String name;

          @Past
          @ApiModelProperty(notes = "사용자 등록일을 입력해 주세요.")
          private LocalDateTime joinDate;

          //@JsonIgnore
          @ApiModelProperty(notes = "사용자 패스워드를 입력해 주세요.")
          private String password;

          //@JsonIgnore
          @ApiModelProperty(notes = "사용자 주민번호를 입력해 주세요.")
          private String ssn;
      }
      ```
