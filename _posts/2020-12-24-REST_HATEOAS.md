---
layout: post
title: HATEOAS
summary: RESTful API
author: devhtak
date: '2020-12-24 22:41:00 +0900'
category: RESTful API
---

### HATEOAS

- Hypermedia As the Engine Of Application State
- 현재 리소스와 연관된 호출가능한 자원 상태 정보를 제공
  - Level 0: The Swarmp of POX
  - Level 1: Resources
  - Level 2: HTTP Verbs
  - Level 3: Hypermedia controls
- 즉, User 정보를 조회하는 API에 User 삭제, User 수정 등에 대한 link를 추가하여 Client 측에서 해당 정보를 사용할 수 있도록 제공해주는 

- 의존성 추가
  
  ```java
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-hateoas</artifactId>
  </dependency>
  ```

- Spring boot 버전마다 구현이 다르다
  - Spring 2.1.8.RELEASE일 경우
    - Resource와 ControllerLinkBuilder 사용
      ```java
      @GetMapping("/users/{id}")
      public Resource<User> retrieveUser(@PathVariable int id) {
          User user = userService.findOne(id);
          Resource<User> resource = new Resource<>(user);
          ControllerLinkBuilder linkTo = linkTo(methodOn(this.getClass()).retrieveAllUsers());
          resource.add(linkTo.withRel("all-users");
          
          return resource;
      }
      ```
  - Spring 2.2일 경우
    - Resource -> EntityModel, ControllerLinkBuilder -> WebMvcLinkBuilder 사용
      ```java
      @GetMapping("/users/{id}")
      public EntityModel<User> retrieveUser(@PathVariable int id) {
          User user = userService.findOne(id);
          EntityModel<User> model = new EntityModel<>(user);
          WebMvcLinkBuilder linkTo = linkTo(methodOn(this.getClass()).retrieveAllUsers());
          model.add(linkTo.withRel("all-users");
          
          return model;
      }
      ```
        - import static을 사용하면 임포트한 클래스에 static method를 클래스.정적메서드 형태로 호출하지 않아도 된다.
