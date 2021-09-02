---
layout: post
title: Validation
summary: Spring MVC Basic
author: devhtak
date: '2021-09-02 21:41:00 +0900'
category: Spring
---

#### validation 필요

- Controller의 중요한 역할 중 하나는 HTTP 요청이 정상인지 검증하는 것
- 클라이언트 검증 + 서버 검증
  - 클라이언트 검증은 조작이 가능하므로 보안에 취약하다.
  - 서버만으로 검증하면 즉각적인 고객 사용성이 부족해진다.
  - 둘이 적절히 섞어서 사용하되, 최종적으로 서버 검증은 빌수이다.
  - API 방식에 경우 API 스펙에 맞게 검증오류를 겨롸에 담아 잘 넘겨주어야 한다.
- process
  - GET /items/add <-> 상품 등록 화면 전달
  - POST /items.add <-> 상품 저장 후 redirect /items/{itemId}
  - GET / items/{id} <-> 상품 상세
  - 상품 저장 당시 validation을 진행한 후 validation에 실패하면 다시 상품 등록 화면을 전달해야 한다.

#### V1. Spring에서 제공하는 기능을 사용하지 않는 경우

```java
Map<String, String> errors = new HashMap<>();

if(StringUtils.hasText(item.getName()) {
    errors.put("itemName", "상품 이름은 필수입니다.");
}
if(item.getPrice() < 100 && item.getPrice() > 100000) {
    errors.put("price", "상품 가격은 100원 이상 100000원 미만입니다..");
}

if(!errors.isEmpty()) {
    model.addAttribute("errors", errors);
    return "/items/add";
}
```
- 문제점 1. 타입 바인딩.
  - price를 Integer로 구현했을 때, 고객 입력 값이 String이면 바인딩 오류가 발생한다.

- 문제점 2. 복잡한 소스.
  
#### V2. BindingResult 사용

- Spring이 제공하는 검증 오류 처리 방법

```java
@GetMapping("/items/add")
public String addItem(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirect) {
    if(StringUtils.hasText(item.getName()) {
        bindingResult.addError(new FieldError("item", "itemName", "상품 이름은 필수 입니다."));
    }
    if(item.getPrice() < 100 && item.getPrice() > 100000) {
        bindingResult.addError(new FieldError("item", "price", "상품 가격은 100원 이상 100000원 미만입니다."));
    }

    if(bindingResult.hasErrors() {
        log.info("errors: {}", bindingResult);
        return "items/add";
    }

    Item savedItem = itemRepository.save(item);
    redirect.addAttribute("itemId", savedItem.getId());      
    return "redirect:/items/{itemId}";
}
```
  - FieldError: 필드에 오류가 있는 경우 객체를 생성하여 BindingResult 객체에 넣어주면 된다.
  - ObjectError: 특정 필드를 넘어서는 오류가 있으면 ObjectError 객체를 생성하여 BindingResult 객체에 담아두면 된다.
  - 생성자 파라미터
    - objectName: 오류가 발생한 객체 이름
    - field: 오류 필드
    - rejectedValue: 사용자가 입력한 값(거절 이유)
    - bindingFailure: 타입 오류 같은 비인딩 실패인지, 검증 실패인지 구분 값
    - codes: 메시지 코드
    - arguments: 메시지에서 사용하는 인자
    - defaultMessage: 기본 오류 메시지
  - 바인딩 시점에서 오류가 발생하면 모델 객체에서 사용자가 입력한 값을 유지하기 어렵지만 FieldError, ObjectError를 사용하면 저장하는 기능을 제공한다.
    - rejectedValue가 오류 발생시 저장하는 필드

- BindingResult가 없으면 400오류가 발생하여 Controller가 호출되지 않고, 오류 페이지로 이동한다.
- BindingResult가 있으면 오류 정보를 BindingResult에 담아서 컨트롤러를 정상 호출한다.
  
- BindingResult에 검증 오류를 적용하는 3가지 방법
  - @ModelAttribute 의 객체에 타입 오류 등으로 바인딩이 실패하는 경우 스프링이 FieldError 생성해서 BindingResult 에 넣어준다.
  - 개발자가 직접 넣어준다.
  - Validator 사용

- 타입 오류 확인
  - 숫자가 입력되어야 할 곳에 문자를 입력해서 타입을 다르게 해서 BindingResult 를 호출하고 bindingResult 의 값을 확인해보자.

- 주의
  - BindingResult 는 검증할 대상 바로 다음에 와야한다. 순서가 중요하다. 예를 들어서 @ModelAttribute Item item , 바로 다음에 BindingResult 가 와야 한다. 
  - BindingResult 는 Model에 자동으로 포함된다.

- BindingResult와 Errors
  - org.springframework.validation.Errors
  - org.springframework.validation.BindingResult
  - BindingResult 는 인터페이스이고, Errors 인터페이스를 상속받고 있다.
  - BeanPropertyBindingResult(구현체)는 BindingResult, Errors 를 구현하고 있기 때문에 둘 다 사용할 수 있다.
    - Errors 인터페이스는 단순한 오류 저장과 조회 기능을 제공한다. 
    - BindingResult 는 여기에 더해서 추가적인 기능들을 제공한다. addError() 도 BindingResult 가 제공하므로 여기서는 BindingResult 를 많이 사용한다.
    - 주로 관례상 BindingResult 를 많이 사용한다.

#### 출처

- 김영한님의 Spring MVC 2 편 - 백엔드 웹 개발 활용 기술
