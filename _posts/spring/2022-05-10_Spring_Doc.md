---
layout: post
title: Spring Document
summary: Spring Document
author: devhtak
date: '2022-05-10 21:41:00 +0900'
category: Spring
---

#### Spring API Documentation

- REST API 문서 자동화에 대한 정리
- SpringDoc OpenApi+swagger 와 Spring Rest Docs 정리

#### Springfox Swagger와 Springdoc OpenAPI 3.0

- Swagger
  - RESTful Web Service를 만들 때 API의 문서를 자동으로 만들어 주고, API를 직접 테스트해 볼 수 있는 UI 툴을 제공

- Springfox Swagger
  - Swagger를 이용해 API 문서를 쉽게 쓸수있도록 도와주는 라이브러리로 Springfox가 있다.

- Springdoc
  - Springfox는 2018년 6월을 마지막으로 업데이트가 중단되었는 데(2년 후 다시 업데이트) 이 때 Springdoc이 나타남
  - Springdoc도 Springfox와 마찬가지로 swagger 문서를 쉽게 작성할 수 있도록 도와주는 라이브러리
  
  <img width="652" alt="image" src="https://user-images.githubusercontent.com/42403023/167617267-370f331a-366f-4df5-a474-3dab7185f02d.png">
    - 이미지 출처: https://oingdaddy.tistory.com/271?category=824422  
  
##### Springdoc OpenAPI 3.0

- dependency
  ```
  // https://mvnrepository.com/artifact/org.springdoc/springdoc-openapi-ui
  implementation group: 'org.springdoc', name: 'springdoc-openapi-ui', version: '1.6.4'
  ```

- 설정(application.yml)
  ```yml
  springdoc:
    version: '@project.version@'
    api-docs:
      path: /api-docs
    default-consumes-media-type: application/json
    default-produces-media-type: application/json
    swagger-ui:
      operations-sorter: alpha
      tags-sorter: alpha
      path: /swagger-ui.html
      disable-swagger-default-url: true
      display-query-params-without-oauth2: true
      doc-expansion: none
    paths-to-match:
      - /api/**
  ```
  - springdoc.api-docs.path
    - 기본값: /v3/api-docs
    - spring boot web application의 api를 OpenAPI 3.0을 이용하여 JSON 형식화한 것의 경로
  - springdoc.default-consumes-media-type
    - 기본값 : application/json
    - request media type 의 기본 값
  - springdoc.default-produces-media-type
    - 기본값 : */*
    - response media type 의 기본 값
  - springdoc.swagger-ui.operations-sorter
    - 기본값 : 컨트롤러 내에서 정의한 api 메서드 순
    - 태그 내 각 api의 정렬 기준
    - alpha(알파벳 오름차순), method(http method 순)
  - springdoc.swagger-ui.tags-sorter
    - 태그 정렬 기준
  - springdoc.swagger-ui.path
    - 기본 값 : /swagger-ui.html
    - Swagger HTML 문서 경로
  - springdoc.swagger-ui.disable-swagger-default-url
    - swagger-ui default url인 petstore html 문서 비활성화 여부
    - v1.4.1 이상 버전부터 지원
  - springdoc.swagger-ui.display-query-params-without-oauth2
    - 기본 값 : false
    - json화 된 config파일 대신 파라미터를 이용하여 swagger-ui에 접근하도록 한다
    - api-docs(/api-docs) 및 swagger-ui.configUrl(/api-docs/swagger-config)를 두번씩 호출 방지
    - v1.4.1 이상 버전부터 지원
  - springdoc.swaggerui.doc-expansion
    - 기본 값: list
    - tag와 operation을 펼치는 방식에 대한 설정
    - ["list", "full", "none"]
    - none으로 설정할 경우, tag 및 operation이 모두 닫힌채로 문서가 열립니다.
  - springdoc.paths-to-match
    - OpenAPI 3 로 문서화할 api path 리스트

- sample controller
  ```java
  @GetMapping("/api/accounts/{accountId}")
  public ResponseEntity<AccountResponse> getAccountByAccountId(@PathVariable String accountId) {
      return ResponseEntity.ok(accountService.findAccountByAccountId(accountId));
  }

  @PostMapping("/api/accounts")
  public ResponseEntity<AccountResponse> saveAccount(@RequestBody AccountResource accountResource) {
      return ResponseEntity.status(HttpStatus.CREATED)
              .body(accountService.saveAccount(accountResource));
  }

  @PutMapping("/api/accounts/{accountId}")
  public ResponseEntity<AccountResponse> updateAccount(@PathVariable String accountId, @RequestBody AccountResource accountResource) {
      return ResponseEntity.ok(accountService.updateAccount(accountId, accountResource));
  }
  ```
  
- document 접속
  - http://localhost:{port}/api-docs
    ```
    {
      "openapi":"3.0.1",
      "info":{
        "title":"OpenAPI definition",
        "version":"v0"
      },
      "servers":[
        {
          "url":"http://localhost:8000",
          "description":"Generated server url"
        }
      ],
      "paths": {
        "/api/accounts/{accountId}":{
          "get":{
            "tags":["account-controller"],
            "operationId":"getAccountByAccountId",
            "parameters":[
              {
                "name":"accountId",
                "in":"path",
                "required":true,
                "schema":{
                  "type":"string"
                }
              }
            ],
            "responses":{
              "200":{
                "description":"OK",
                "content":{
                  "application/json":{
                    "schema":{
                      "$ref":"#/components/schemas/AccountResponse"
                    }}}}}},
          ...
        }
      }
      "components":{
        "schemas":{
          "AccountResource":{
            "type":"object",
            "properties":{
              "name":{
                "type":"string"
              },
          ...
        }
      }   
    ```
    - 화면에 뿌려지기 위한 원본 데이터를 가지고 오는것을 확인할 수 있다.
  - http://localhost:{port}/swagger-ui.html
    
    <img width="672" alt="image" src="https://user-images.githubusercontent.com/42403023/167624649-d8c6cb62-6b28-4407-b98d-e37930a7c339.png">
    
    - /api-docs에서 가지고 온 데이터를 swagger-ui 화면에 뿌려준다.
    - 내가 지정한 path에 대한 정보를 확인할 수 있다.

- Annotation
  |swagger 3 annotations|swagger 2 annotations|description|
  |---|---|---|
  |@Tag|@Api|클래스를 Swagger 리소스로 표시|
  |@Parameter(hidden = true) or @Operation(hidden = true) or @Hidden|@ApiIgnore| API 작업에서 단일 매개 변수를 나타냄|
  |@Parameter|@ApiImplicitParam|API 작업에서 단일 매개 변수를 나타냄|
  |@Parameters|@ApiImplicitParams|API 작업에서 복수 매개 변수를 나타냄|
  |@Schema|@ApiModel|Swagger 모델에 대한 추가 정보를 제공|
  |@Schema(accessMode = READ_ONLY)|@ApiModelProperty(hidden = true)|모델 속성의 데이터를 추가하고 조작|
  |@Schema|@ApiModelProperty|Swagger 모델에 대한 추가 정보를 제공|
  |@Operation(summary = "foo", description = "bar")|@ApiOperation(value = "foo", notes = "bar")|특정 경로에 대한 작업 또는 일반적으로 HTTP 메서드를 설명|
  |@Parameter|@ApiParam|작업 매개 변수에 대한 추가 메타 데이터를 추가| 
  |@ApiResponse(responseCode = "404", description = "foo")|@ApiResponse(code = 404, message = "foo")|작업의 가능한 응답을 설명|
  
  - 적용 예
    ```java
    @RestController
    @RequestMapping("/api")
    @Tag(name = "Account controller", description = "Account controller desc")
    public class AccountController {

        private final AccountService accountService;

        @Autowired
        public AccountController(AccountService accountService) {
            this.accountService = accountService;
        }

        @Operation(
                summary = "사용자 정보 조회"
                , description = "사용자의 ID를 통해 사용자의 정보를 조회한다."
                , tags = { "contact" })
        @ApiResponses(value = {
                @ApiResponse(responseCode = "200", description = "successful operation",
                        content = @Content(array = @ArraySchema(schema = @Schema(implementation = Contact.class)))) })
        @GetMapping("/accounts/{accountId}")
        public ResponseEntity<AccountResponse> getAccountByAccountId(@PathVariable String accountId) {
            return ResponseEntity.ok(accountService.findAccountByAccountId(accountId));
        }
    ```
    
#### Spring REST Docs

- Swagger와 차이점
  - 테스트가 성공해야 문서가 작성 된다.
    - REST Docs로 문서를 작성하는 것은 API의 신뢰도를 높이고 테스트 코드의 검증을 강제로 하게 만든다
  - 실제 코드에 추가되는 코드가 없다.
    - 프로덕션 코드와 분리되어 있기 때문에 Swagger같이 Config 설정 코드나 어노테이션이 프로덕션 코드를 더럽히지 않는다.

- dependency 추가
  ```
  plugins { 
    id "org.asciidoctor.convert" version "1.5.9.2"
  }
  dependencies {
    asciidoctor 'org.springframework.restdocs:spring-restdocs-asciidoctor' 
    testImplementation 'org.springframework.restdocs:spring-restdocs-mockmvc' 
  }
  ext { 
    snippetsDir = file('build/generated-snippets')
  }
  test { 
    outputs.dir snippetsDir
  }
  asciidoctor { 
    inputs.dir snippetsDir 
    dependsOn test 
  }
  // JAR 파일에 패키징하는 설정
  bootJar {
    dependsOn asciidoctor 
    from ("${asciidoctor.outputDir}/html5") { 
      into 'static/docs'
    }
  }
  ```

- 테스트 코드 작성
  ```java
  @AutoConfigureMockMvc
  @AutoConfigureRestDocs
  @SpringBootTest
  class AccountControllerTest {

      @Autowired MockMvc mockMvc;
      @MockBean AccountService accountService;
      @Autowired ObjectMapper objectMapper;

      @Test
      void saveAccountTest() throws Exception {
          final AccountResponse accountResponse = new AccountResponse("TEST", "1234", LocalDateTime.of(2022, 5, 10, 22, 00));
          final AccountResource accountResource = new AccountResource("TEST", "TEST1234");
          when(accountService.saveAccount(any())).thenReturn(accountResponse);


          this.mockMvc.perform(post("/api/accounts")
                          .content(objectMapper.writeValueAsString(accountResource))
                          .contentType(MediaType.APPLICATION_JSON))
                  .andExpect(status().isCreated()) // 4
                  .andDo(MockMvcRestDocumentation.document("account-create",
                          PayloadDocumentation.requestFields(
                                PayloadDocumentation.fieldWithPath("name").description("Account Name").optional(),
                                PayloadDocumentation.fieldWithPath("password").description("AccountPassword")
                        ),
                        PayloadDocumentation.responseFields(
                                PayloadDocumentation.fieldWithPath("name").description("Account Save Name"),
                                PayloadDocumentation.fieldWithPath("accountId").description("Account Id"),
                                PayloadDocumentation.fieldWithPath("createdAt").description("Account created time")
                          )
                  ));
      }
  }
  ```
  - @AutoConfigureRestDocs
    - Spring REST Docs에 대해 auto-configuration 할 수 있도록 class 파일에 적용하는 어노테이션
  - Test 코드
    - mockMvc를 통해 API를 호출한 후, andDo에 다큐먼트를 작성한다.
    - MockMvcRestDocumentation.document
      - parameter1: document 이름
      - parameter2: request 설명
      - parameter3: response 설명
        - fieldWithPath는 key 값 description는 fieldWithPath 설명

- 문서화
  <img width="368" alt="image" src="https://user-images.githubusercontent.com/42403023/167635010-ae5d47ae-b5a4-4fa0-9a27-209348a3665e.png">

  - 테스트 코드를 작성한 후 빌드하면 문서가 생성되는 것을 확인할 수 있다.
  - /src/docs/asciidoc 폴더를 생성하고, \*.adoc 파일을 생성하자
    ```
    = Spring REST Docs
    :toc: left
    :toclevels: 2
    :sectlinks:

    [[resources-account]]
    == Account

    [[resources-account-create]]
    === Account 생성

    ==== HTTP request

    include::{snippets}/account-create/http-request.adoc[]

    ==== HTTP response

    include::{snippets}/account-create/http-response.adoc[]
    ```
    - Asciidoctor는 일반 텍스트를 처리하고 필요에 맞게 스타일 및 레이아웃 된 HTML을 생성한다.

#### 참고

- Springfox와 Springdoc: https://junho85.pe.kr/1583
- https://oingdaddy.tistory.com/271?category=824422
- https://oingdaddy.tistory.com/272?category=824422
- Springdoc 설정: https://blog.jiniworld.me/83 [hello jiniworld]
- https://tecoble.techcourse.co.kr/post/2020-08-18-spring-rest-docs/
