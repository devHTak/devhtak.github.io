---
layout: post
title: API Version 관리
summary: RESTful API
author: devhtak
date: '2020-12-24 22:41:00 +0900'
category: RESTful API
---

### Version 관리

- 공개 API를 관리할 때에는 버전 관리가 필요할 수 있다.
- URI 활용, Parameter 활용, 특정 Header 값 또는 기존 Header 값을 활용하는 방법 등이 있다.

- DAO
  ```java
  @Data
  @NoArgsConstructor @AllArgsConstructor
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
  ```java
  @Data
  @AllArgsConstructor @NoArgsConstructor
  public class UserV2 extends User {
      private String grade;
  }  
  ```
    - 버전별 보내주는 객체를 따로 생성했다.

- URI 활용
  - 기초적인 방법으로 URI에 version을 추가하여 관리하는 방법이다.
  - URI에 정보를 녹여 활용할 수는 있으나 URI가 길어진다는 단점이 있다.
    ```java
    @GetMapping("/v1/users/{id}")
    public User retrieveUserV1(@PathVariable Integer id) {
        return userService.findOne(id).orElseThrow(() -> {
          throw new UserNotFoundException(String.format("User[%s] not found", id));
        });
    }
    @GetMapping("/v2/users/{id}")
    public UserV2 retrieveUserV2(@PathVariable Integer id) {
        User byId = userService.findOne(id).orElseThrow(() -> {
          throw new UserNotFoundException(String.format("User[%s] not found", id));
        });

        UserV2 userV2 = new UserV2();
        BeanUtils.copyProperties(byId, userV2); // byId에 값을 userV2에 복사
        userV2.setGrade("VIP");
        return userV2;
    }
    ```
    - @GetMapping value에 버전 정보를 입력하여 사용
    
- 파라미터 사용

  ```java
  @GetMapping(value="/users/{id}", params="version=1")
  public User retrieveUserV1(@PathVariable Integer id) {
      return userService.findOne(id).orElseThrow(() -> {
        throw new UserNotFoundException(String.format("User[%s] not found", id));
      });
  }
  @GetMapping(value="/users/{id}", params="version=2")
  public UserV2 retrieveUserV2(@PathVariable Integer id) {
      User byId = userService.findOne(id).orElseThrow(() -> {
        throw new UserNotFoundException(String.format("User[%s] not found", id));
      });

      UserV2 userV2 = new UserV2();
      BeanUtils.copyProperties(byId, userV2); // byId에 값을 userV2에 복사
      userV2.setGrade("VIP");
      return userV2;
  }
  ```
    - query string을 통해 parameter에 version을 넘겨준다.
      - http://localhost:8180/users/1?version=1
      - http://localhost:8180/users/1?version=2
      
- 신규 헤더 생성

  ```java
  @GetMapping(value="/users/{id}", headers="X-API-VERSION=1")
  public User retrieveUserV1(@PathVariable Integer id) {
      return userService.findOne(id).orElseThrow(() -> {
        throw new UserNotFoundException(String.format("User[%s] not found", id));
      });
  }
  @GetMapping(value="/users/{id}", headers="X-API-VERSION=2")
  public UserV2 retrieveUserV2(@PathVariable Integer id) {
      User byId = userService.findOne(id).orElseThrow(() -> {
        throw new UserNotFoundException(String.format("User[%s] not found", id));
      });

      UserV2 userV2 = new UserV2();
      BeanUtils.copyProperties(byId, userV2); // byId에 값을 userV2에 복사
      userV2.setGrade("VIP");
      return userV2;
  }
  ```
    - 테스트 방법: Postman을 활용하여 Header 값 정보를 세팅하여 넘겨줄 수 있다.
      - key: X-API-VERSION, Value: 1 또는 2
      
- 기존 헤더 생성

  ```java
  @GetMapping(value="/users/{id}", produces= "application/vnd.company.appv1+json")
  public User retrieveUserV1(@PathVariable Integer id) {
      return userService.findOne(id).orElseThrow(() -> {
        throw new UserNotFoundException(String.format("User[%s] not found", id));
      });
  }
  @GetMapping(value="/users/{id}", produces= "application/vnd.company.appv2+json")
  public UserV2 retrieveUserV2(@PathVariable Integer id) {
      User byId = userService.findOne(id).orElseThrow(() -> {
        throw new UserNotFoundException(String.format("User[%s] not found", id));
      });

      UserV2 userV2 = new UserV2();
      BeanUtils.copyProperties(byId, userV2); // byId에 값을 userV2에 복사
      userV2.setGrade("VIP");
      return userV2;
  }
  ```
    - produces 와 consumes
      - consumes: 소비가능한 미디어 타입의 목록을 지정해서 주요한 매핑을 제한할 수 있다. consumes에 지정한 미디어 타입과 Content-Type 요청 헤더가 일치할 때만 요청이 매핑된다.
      - produces: 생산 가능한 미디어 타입의 목록을 지정해서 주요 매핑을 제한할 수 있다. Accept 요청 헤더에 이러한 값 중 하나와 일치할 때만 요청이 매칭된다.
    - Accpet Header에 해당하는 값에 버전을 입력하면 된다.
      - key: Accept, value: application/vnd.company.appv1+json 또는 application/vnd.company.appv2+json
