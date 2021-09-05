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
- FieldError, ObjectError
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
  - codes, arguments 로 Message 처리가 가능하다. 
    - 또는 bindingResult.rejectValue()에 errorCode를 넣어준다. errorCode는 required만 설정해도 관련한 메시지 코드를 생성한다.(MessageCodeResolver)
    - ex) errorCode: required -> required.item.itemName, required.itemName, required.java.lang.String, required 순으로 생성

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

#### V3. Validator 분리

- Controller에 validation, controller 소스가 복잡하게 섞여있기 때문에 분리해주는 것이 좋다.
- ItemValidator.java
  ```java
  @Component
  public class ItemValidator extends Validator {
      @Override
      public boolean supports(Class<?> clazz) {
          return Item.class.isAssingalbeFrom(clazz);
      }
      
      @Override
      public void validate(Objec target, Errors errors) {
          Item item = (Item) target;
          
          if(StringUtils.hasText(item.getName()) {
              errors.rejectValue("itemName", "상품 이름은 필수 입니다.");
          }
          if(item.getPrice() < 100 && item.getPrice() > 100000) {
              errors.rejectValue("price", "상품 가격은 100원 이상 100000원 미만입니다.");
          }
      }
  }
  ```
  - supports: 해당 검증기를 지원하는 여부 확인
  - validate: 검증 대상 객체와 bindingResult

- ItemController.java
  ```java
  @Slf4j
  @Controller
  public class ItemController {
      @Autowired private ItemValidator itemValidator;
      
      @PostMapping("/items/add")
      public String addItem(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirect) {
          itemValidator.validate(item, bindingResult);
          if(bindingResult.hasErrors() {
              log.info("errors: {}", bindingResult);
              return "items/add";
          }

          Item savedItem = itemRepository.save(item);
          redirect.addAttribute("itemId", savedItem.getId());      
          return "redirect:/items/{itemId}";
      }
  }
  ```

- WebDataBinder 사용
  - WebDataBinder를 사용하면 스프링의 파라미터 바인딩의 역할 및 검증 기능도 내부에 포함된다.
  - WebDataBinder 에 여러개의 validator를 등록할 때 내부에서 Validator를 구분짖기 위해 support를 정의해주어야 한다.
  
  - ItemController에 추가
    ```java
    @Slf4j
    @Controller
    public class ItemController {
        @Autowired private ItemRepository itemRepository;
        @Autowired private ItemValidator itemValidator;
        
        @InitBinder
        public void init(WebDataBinder binder) {
            log.info("init binder: {}", binder);
            dataBinder.addValidators(itemValidator);
        }
      
        @PostMapping("/items/add")
        public String addItem(@Validated @ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirect) {
            if(bindingResult.hasErrors() {
                log.info("errors: {}", bindingResult);
                return "items/add";
            }
            
            Item savedItem = itemRepository.save(item);
            redirect.addAttribute("itemId", savedItem.getId());      
            return "redirect:/items/{itemId}";
        }
    }
    ```

#### Bean Validation

- 검증 기능을 매번 코드로 작성하기 번거롭기 때문에 일반적인 로직은 annotation으로 사용할 수 있으며 표준화한 것을 Bean Validation이라고 한다.
- Bean Validation 2.0(JSR-380) 이라는 기술 표준과 Hibernate Validator라는 구현체가 있다(ORM 과는 관련이 없다.)
  - Jakarta Bean Validation: Bean Validation 인터페이스로 hibernate-validator 구현체
- dependency
  ```
  // Gradle
  implementation 'org.springframework.boot:spring-boot-starter-validation'
  // Maven
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-validation</artifactId>
  </dependency>
  ```
  
- 검증 annotation
  ```java
  public class Item {
      private Long id;
      
      @NotBlank(message = "공백X")
      private String itemName;
      
      @NotNull
      @Range(min = 1000, max = 100000)
      private Integer price;
      
      @NotNull
      @Max(9999)
      private Integer quantity;
  }
  ```
  - @NotBlank: 빈값 + 공백만 있는 경우 허용하지 않는다.
  - @NotNull: Null값을 허용하지 않는다.
  - @Range(min = 1000, max = 10000): 범위 안의 값이 있어야 한다.
  - @Max(9999) : 최대 9999까지만 허용한다.
  - message 속성을 통해 validation에 위배되는 경우 나타낼 메시지를 설정해 줄 수 있다.
    - 에러 코드로 속성에 설정된 메시지를 보여주는 방법 가능

- Bean Validation 적용
  - @InitBinder + WebDataBinder에 validator를 넣는 소스 제거 필요
    - 오류 검증기가 중복 적용
  
  - Spring 적용 원리
    - dependency를 추가하면 자동으로 Bean Validator를 인지하고 스프링에 통합한다. 
    - 자동으로 Validator로 등록한다.
      - LocalValidatorFactoryBean을 글로벌 Validator로 등록한다.
      - 해당 Validator는 @NotNull 같은 애노테이션을 보고 검증을 수행
    - 검증이 필요할 때 파라미터에 @Valid(javax 제공) 또는 @Validation(spring 제공)을 적용하면 된다.
  
  - 검증 순서
    - @ModelAttribute + @Valid or @Validation 각각의 필드에 타입 변환 시도
      - 실패할 경우 typeMismatch로 FieldError 추가
    - Validator 적용

- 에러 코드
  - validation 애노테이션 명 오류 코드를 기반으로 MessageCodesResolver를 통해 다양한 메시지 코드가 순서대로 생성된다.
    - 예시1. @NotBlank
      - NotBlank.item.itemName -> NotBlank.itemName -> NotBlank.java.lang.String -> NotBlank
    - 예시2. @Range
      - Range.item.price -> Range.price -> Range.java.lang.Integer -> Range
  - 메시지 찾는 순서
    - 생성된 메시지 코드 순서대로 messageSource 에서 메시지 찾기
    - 애노테이션의 message 속성 사용 @NotBlank(message = "공백! {0}")
    - 라이브러리가 제공하는 기본 값 사용 공백일 수 없습니다

- BeanValidation - groups
  - 한계: ID에 @NotNull 적용이 필요하다. 하지만 등록할 때에는 Id가 바로 채번되지 않기 때문에 form에서 넘어올 때 validation 오류가 발생한다.
    - 해결 방법 1. BeanValidation의 groups의 기능 사용
    - 해결 방법 2. Item을 직접 사용하지 않고, ItemSaveForm, ItemUpdateFrom 같이 폼전송을 위한 별도의 모델 객체를 만들어 사용
  - groups 적용
    - @Validated에 적용되는 분류 별 interface 를 정의하고, Item에 groups를 정의하면 된다.
      - @Valid에는 적용이 불가능하다
    - SaveCheck interface
      ```java
      public interface SaveCheck { }
      ```
    - UpdateCheck interface
      ```java
      public interface UpdateCheck { }
      ```
    - Item.java에 groups 속성 정의
      ```java
      public class Item {
          private Long id;      
          
          @NotBlank(message = "공백X", groups=UpdateCheck.class)
          private String itemName;
          
          @NotNull
          @Range(min = 1000, max = 100000, groups= {UpdateCheck.class, SvaeCheck.class})
          private Integer price;
          
          @NotNull
          @Max(9999 , groups=SaveCheck.class)
          private Integer quantity;
      }
      ```
    - handler 파라미터에 @Validated에 value 설정
      ```java
      // ...
      public String addItem(@Validated(SaveCheck.class) @ModelAttribute Item item, BindingResult result) {
          // ...
      }
      
      public String editItem(@Validated(UpdateCheck.class) @ModelAttribute Item item, BindingResult result) {
          // ...
      }
      // ...
      ```
  - Form 별로 전달하고자 하는 데이터가 다르기 때문에 groups 를 적용하기 보다는 Form 별로 요청 객체를 분리하여 사용한다.

- Form 객체 분리
  - Item.java 원복
    ```java
    public class Item {
        private Long id;      
        private String itemName;
        private Integer price;
        private Integer quantity;
    }
    ```
  - ItemSaveForm.java
    ```java
    public class ItemSvaeForm {
        @NotBlank(message = "공백X")
        private String itemName;
        
        @NotNull
        @Range(min = 1000, max = 100000)
        private Integer price;
        
        @NotNull
        @Max(9999)
        private Integer quantity;
    }
    ```
  - ItemUpdateForm.java
    ```java
    public class ItemUpdteForm {
        @NotBlank
        private Long id;
        
        @NotBlank(message = "공백X")
        private String itemName;
        
        @NotNull
        @Range(min = 1000, max = 100000)
        private Integer price;
        
        @NotNull
        @Max(9999)
        private Integer quantity;
    }
    ```

#### 출처

- 김영한님의 Spring MVC 2 편 - 백엔드 웹 개발 활용 기술
