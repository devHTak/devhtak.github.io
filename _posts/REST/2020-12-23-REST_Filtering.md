---
layout: post
title: JsonFilter
summary: RESTful API
author: devhtak
date: '2020-12-20 22:41:00 +0900'
category: RESTful API
---

### Filtering

- API에서 패스워드, 주민등록번호 등 객체에서 보여주지 않을 데이터를 지정하는 방법이다.
- 의존성
  ```
  <!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-annotations -->
  <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-annotations</artifactId>
  </dependency>
  ```
  
- @JsonIgnore 또는 @JsonIgnoreProperties 활용
  - @JsonIgnore
    ```java
    @Data
    @NoArgsConstructor @AllArgsConstructor
    public class User {
        private Integer id;

        @Size(min = 2, message= "이름은 2글자 이상 입력해주세요.")
        private String name;

        @Past
        private LocalDateTime joinDate;

        @JsonIgnore
        private String password;

        @JsonIgnore
        private String ssn;
    }
    ```
    - 사용하는 객체에 컬럼마다 @JsonIgnore을 붙여주면 해당 객체를 넘겨줄 때 컬럼에 해당하는 값을 넘겨주지 않는다.
  - @JsonIgnoreProperties
    ```java
    @Data
    @NoArgsConstructor @AllArgsConstructor
    @JsonIgnoreProperties(value= {"password", "ssn"})
    public class User {
        private Integer id;

        @Size(min = 2, message= "이름은 2글자 이상 입력해주세요.")
        private String name;

        @Past
        private LocalDateTime joinDate;

        private String password;

        private String ssn;
    }
    ```
    - 보내주지 않는 컬럼을 @JsonIgnoreProperties를 활용하여 배열 형태로 작성하면 된다.
  
  - 결과
    ```
    {
        "id": 1,
        "name": "Kenneth",
        "joinDate": "2020-12-24T15:41:28.961"
    }
    ```

- @JsonFilter 활용하기
  - @JsonIgnore 또는 @JsonIgnoreProperties와 다르게 핸들러마다 숨기고자 하는 컬럼을 지정하여 사용할 수 있다.
  - 핸들러에 Filter를 만들어 저장한 후 MappingJacksonView 를 리턴하는 형태
  
    ```java
    @Data
    @NoArgsConstructor @AllArgsConstructor
    @JsonFilter("UserInfo")
    public class User {
        private Integer id;

        @Size(min = 2, message= "이름은 2글자 이상 입력해주세요.")
        private String name;

        @Past
        private LocalDateTime joinDate;

        private String password;

        private String ssn;
    }
    ```
      - 객체에 사용하고자 하는 @JsonFilter에 Filter Id를 지정한다.
    
    ```java
    @GetMapping(value = "/users/{id}", produces = "application/vnd.company.appv1+json")
    public MappingJacksonValue retrieveUserV1(@PathVariable Integer id) {
        User byId = userService.findOne(id).orElseThrow(() -> {
          throw new UserNotFoundException(String.format("User[%s] not found", id));
        });

        SimpleBeanPropertyFilter filter = SimpleBeanPropertyFilter.filterOutAllExcept("id", "name", "joinDate");

        FilterProvider filters = new SimpleFilterProvider().addFilter("UserInfo", filter);
        MappingJacksonValue mapping = new MappingJacksonValue(byId);
        mapping.setFilters(filters);
        return mapping;
    }
    ```
      - 핸들러에 SimpleBeanPropertyFilter.filterOutAllExcept 를 활용하여 보내고자 하는 컬럼을 지정
      - FilterProvider에서 사용하고자 하는 Filter Id와 filter를 지정한 후 MappingJacksonValue에 보내고자 하는 객체를 지정한후 filter를 세팅하여 return해주면 된다.
      
    - 결과
      ```
      {
          "id": 1,
          "name": "Kenneth",
          "joinDate": "2020-12-24T15:41:28.961"
      }
      ```
  
