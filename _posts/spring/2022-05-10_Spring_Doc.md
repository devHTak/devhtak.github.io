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


#### 참고

- Springfox와 Springdoc: https://junho85.pe.kr/1583
- https://oingdaddy.tistory.com/271?category=824422
- Springdoc 설정: https://blog.jiniworld.me/83 [hello jiniworld]
