---
layout: post
title: Spring MVC - Redirect
summary: 김영한님 - 스프링 MVC - 백엔드 웹 개발 핵심 기술
author: devhtak
date: '2021-04-18 21:41:00 +0900'
category: Spring
---

#### Post - Redirect - Get

- Post 요청 핸들러에서 HTML을 바로 return한다면 생기는 문제
  - 새로 고침을 하게 되면 같은 Post 요청이 반복된다.
  - Post 요청이므로 계속 새로운 Data가 입력된다.
    - HTTP POST Method는 멱등성이 보장되지 않기 때문에 요청마다 새로운 데이터를 입력하게 된다.
  - 해결 방법으로 PRG 패턴이 나온 것(Post - Redirect - Get)

- PRG 패턴
  - redirect를 활용하여 마지막 Post 호출에 대한 내용을 Get에 대한 호출로 상세화면 등을 보여주는 URL로 리턴한다.
  - 이렇게 GET 요청으로 끝나면, 새로고침을 하고 같은 요청을 계속 보내도 데이터의 변경이 없다.
    - HTTP POST Method 이 외에 다른 Method는 멱등성을 보장한다.

#### RedirectAttributes

- RedirectAttributes를 사용하면 Get 요청에 대하여 attribute를 만들어서 사용할 수 있다.
  ```java
  @PostMapping("/add")
  public String addItem(Item item, RedirectAttributes redirectAttributes) {
      Item savedItem = itemRepository.save(item);
      
      redirectAttributes.addAttribute("itemId", savedItem.getId());
      redirectAttributes.addAttribute("status", true);
      return "redirect:/basic/items/{itemId}";
  }
  ```
  - 요청 URL
    ```
    http://localhost:8080/basic/items/3?status=true
    ```
  - 파라미터
    - 요청 URL에 status라는 파라메터를 넘겨준 것을 확인할 수 있다.
  - URL 인코딩
    - itemId가 PathVariable로 들어간 것을 확인할 수 있다.
    - URL 인코딩도 해주고, PathVariable, 쿼리 파라미터까지 처리한다.



** 참고: 스프링 MVC - 백엔드 웹 개발 핵심 기술
