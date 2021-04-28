---
layout: post
title: Spring MVC
summary: 김영한님 - 스프링 MVC - 백엔드 웹 개발 핵심 기술
author: devhtak
date: '2021-04-15 21:41:00 +0900'
category: Spring
---

#### Spring MVC 전체 구조

- Spring MVC 구조
  
  ![image](https://user-images.githubusercontent.com/42403023/114796789-b88bf000-9dcc-11eb-9281-8f25cc28a7be.png)
  
  ** 이미지 출처: https://www.egovframe.go.kr/wiki/doku.php?id=egovframework:rte2:ptl:spring_mvc_architecture
  - Client 에 HTTP 요청 -> Dispatcher Servlet
  - DispatcherServlet 에서 Handler Mapping을 통해 핸들러 조회
    - 핸들러 조회: 핸들러 매핑을 통해 요청 URL에 매핑된 핸들러(컨트롤러) 조회
  - DispatcherServlet 에서 Handler Adapter 목록을 통해 핸들러를 처리할 수 있는 Handler Adapter 조회
    - 핸들러 어댑터 조회: 핸들러를 실행할 수 있는 핸들러 어댑터 조회
  - DispatcherServlet 에서 Handler Adapter 호출(호출 메서드: handle(handler))
    - 핸들러 어댑터 실행: 핸들러 어댑터를 실행
  - Handler Adapter 에서 Handler 호출
    - 핸들러 실행: 핸들러 어댑터가 실제 핸들러 실행
  - Handler Adapter 에서 Dispatcher Servlet 에게 ModelAndView 반환
    - ModelAndView 반환: 핸들러 어댑터는 핸들러가 반환하는 정보를 ModelAndView에 담아서 반환한다.
  - Dispatcher Servlet에서 ViewResolver 호출
    - ViewReolsver 호출: ViewResolver를 찾고 실행한다.
  - ViewResolver가 Dispatcher Servlet에게 View 반환
    - View 반환: ViewResolver는 뷰의 논리 이름을 물리 이름으로 바꾸고, 렌더링 역할을 담당하는 View 객체를 반환한다.
  - Dispatcher Servlet이 View 호출 (호출 메서드: render(model))
    - View Rendering: 뷰를 통해서 뷰를 렌더링한다.

- Dispatcher Servlet
  - org.springframework.web.servlet.DispatcherServlet
  - Spring MVC에서 Front Controller 역할을 한다.
  - Dispatcher Servlet 서블릿 등록
    - DispatcherServlet 도 HttpServlet을 상속받아 사용하고, 서블릿으로 동작한다.
      - DispatcherServlet -> FrameworkServlet -> HttpServletBean -> HttpServlet
    - Spring Boot에서는 DispatcherServlet을 서블릿으로 자동 등록하며, 모든 경로에 매핑
      - 더 자세한 경로가 우선순위가 높다. 새로운 서블릿을 등록하면 함께 동작한다.
  - 요청 흐름
    - Servlet이 호출되면 HttpServlet이 제공하는 service()가 호출된다.
    - Spring MVC는 DispatcherServlet의 부모인 FrameworkServlet에서 service()를 오버라이드 해두었다.
    - FrameworkServlet.service()를 시작으로 여러 메서드가 호출되면서 DispatcherServlet.doDispatch()가 호출된다.
  - DispatcherServlet의 doDispatch()
    ```java
    protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
        HttpServletRequest processedRequest = request;
        HandlerExecutionChain mappedHandler = null;
        boolean multipartRequestParsed = false;

        WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

        try {
          ModelAndView mv = null;
          Exception dispatchException = null;

          try {
              processedRequest = checkMultipart(request);
              multipartRequestParsed = (processedRequest != request);

              // Determine handler for the current request.
              mappedHandler = getHandler(processedRequest);
              if (mappedHandler == null) {
                  noHandlerFound(processedRequest, response);
                  return;
              }

              // Determine handler adapter for the current request.
              HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

              // Process last-modified header, if supported by the handler.
              String method = request.getMethod();
              boolean isGet = "GET".equals(method);
              if (isGet || "HEAD".equals(method)) {
                  long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
                  if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
                      return;
                  }
              }

              if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                  return;
              }

              // Actually invoke the handler.
              mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

              if (asyncManager.isConcurrentHandlingStarted()) {
                  return;
              }

              applyDefaultViewName(processedRequest, mv);
              mappedHandler.applyPostHandle(processedRequest, response, mv);
          }
          catch (Exception ex) {
              dispatchException = ex;
          }
          catch (Throwable err) {
              // As of 4.3, we're processing Errors thrown from handler methods as well,
              // making them available for @ExceptionHandler methods and other scenarios.
              dispatchException = new NestedServletException("Handler dispatch failed", err);
          }
            processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
        }
        catch (Exception ex) {
            triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
        }
        catch (Throwable err) {
            triggerAfterCompletion(processedRequest, response, mappedHandler,
                  new NestedServletException("Handler processing failed", err));
        }
        finally {
            if (asyncManager.isConcurrentHandlingStarted()) {
                // Instead of postHandle and afterCompletion
                if (mappedHandler != null) {
                   mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
                }
            }
            else {
                // Clean up any resources used by a multipart request.
                if (multipartRequestParsed) {
                    cleanupMultipart(processedRequest);
                }
            }
        }
    }
    ```

- 주요 인터페이스
  - 핸들러 매핑: org.springframework.web.servlet.HandlerMapping
  - 핸들러 어댑터: org.springframework.web.servlet.HandlerAdapter
  - 뷰 리졸버: org.springframework.web.servlet.ViewResolver
  - 뷰: org.springframework.web.servlet.View
  - 인터페이스 사용 장점
    - 스프링 MVC의 큰 강점은 DispatcherServlet 코드의 변경 없이, 원하는 기능을 변경하거나 확장할 수 있다는 점이다.
    - 지금까지 설명한 대부분을 확장 가능할 수 있게 인터페이스로 제공한다.
    - 이 인터페이스들만 구현해서 DispatcherServlet 에 등록하면 여러분만의 컨트롤러를 만들 수도 있다.
  

#### Handler Mapping과 Handler Adapter

- Handler Mapping
  - 요청 정보를 기준으로 어떤 핸들러를 사용할 것 인가를 결정하는 인터페이스로, url, header 정보 등으로 해당 컨트롤러(핸들러)를 선택하는 기준이 되는 인터페이스이다
  - 우선순위 0: Request Mapping Handler Mapping
    - 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용
  - 우선순위 1: BeanName Url Handler Mapping
    - 스프링 빈의 이름으로 핸들러를 찾는다.

- Handler Adapter
  - HandlerMapping에서 결정된 핸들러 정보로 해당 메서드를 직접 호출해 주는 스펙
  - 우선순위 0: Request Mapping Handler Adapter
    - 애노테이션 기반의 컨트롤러인 @RequestMapping 에서 사용
  - 우선순위 1: Http Request Handler Adapter
    - HttpRequestHandler 처리 
  - 우선순위 2: Simple Controller Handler Adapter
    - Controller 인터페이스(애노테이션 아님, 과거에 사용) 처리

- 우선순위에 맞게 실행되며, 없는 경우 다음 우선순위에 객체를 활용하여 Handler, Handler Adapter를 찾는다.

#### ViewResolver

- ViewResolver
  - 우선순위 1. BeanNameViewResolver : 빈 이름으로 뷰를 찾아서 반환한다. (예: 엑셀 파일 생성 기능에 사용)
  - 우선순위 2. InternalResourceViewResolver : JSP를 처리할 수 있는 뷰를 반환한다.

- InternalResourceViewResolver
  - 등록 방법
    - application.properties
      ```
      spring.mvc.view.prefix=/WEB-INF/views/
      spring.mvc.view.suffix=.jsp
      ```
      - 스프링 부트는 InternalResourceViewResolver 라는 뷰 리졸버를 자동으로 등록하는데, 이때 application.properties 에 등록한 spring.mvc.view.prefix , spring.mvc.view.suffix 설정 정보를 사용해서 등록한다

    - Bean 등록
      ```java
      @Bean
      ViewResolver internalResourceViewResolver() {
          return new InternalResourceViewResolver("/WEB-INF/views/", ".jsp");
      }
      ```
  - InternalResourceViewResolver 는 만약 JSTL 라이브러리가 있으면 InternalResourceView 를 상속받은 JstlView 를 반환한다. JstlView 는 JSTL 태그 사용시 약간의 부가 기능이 추가된다.
  - 다른 뷰는 실제 뷰를 렌더링하지만, JSP의 경우 forward() 통해서 해당 JSP로 이동(실행)해야 렌더링이 된다. JSP를 제외한 나머지 뷰 템플릿들은 forward() 과정 없이 바로 렌더링 된다.

- Thymeleaf 뷰 템플릿을 사용하면 ThymeleafViewResolver 를 등록해야 한다. 최근에는 라이브러리만 추가하면 스프링 부트가 이런 작업도 모두 자동화해준다

#### Spring MVC 예제

```java
@Controller
@RequestMapping("/springmvc/v2/members")
public class SpringMemberControllerV2 {	
    private final MemberRepository memberRepository = MemberRepository.getInstance();

    @RequestMapping("/new-form")	
    public ModelAndView newForm() {
        return new ModelAndView("new-form");
    }

    @RequestMapping("/save")
    public ModelAndView save(HttpServletRequest request, HttpServletResponse response) {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));
        Member member = new Member();
        member.setUsername(username); member.setAge(age);

        Member returnMember = memberRepository.save(member);

        ModelAndView mv = new ModelAndView("save-result");
        mv.addObject("member", returnMember);
        return mv;
    }

    @RequestMapping
    public ModelAndView allMembers() {
        List<Member> members = memberRepository.findAll();

        ModelAndView mv = new ModelAndView("members");
        mv.addObject("members", members);
        return mv;
    }
}
```

- @RequestMapping
  - 스프링은 애노테이션을 활용한 매우 유연하고, 실용적인 컨트롤러를 만들었는데 이것이 바로 @RequestMapping 애노테이션을 사용하는 컨트롤러이다.
  - Handler Mapping: RequestMappingHandlerMapping, Handler Adapter: RequestMappingHandlerAdapter 사용, 우선순위가 제일 높다.
  - 요청 정보를 매핑한다. 해당 URL이 호출되면 이 메서드가 호출된다. 애노테이션을 기반으로 동작하기 때문에, 메서드의 이름은 임의로 지으면 된다.

- @Controller
  - 스프링이 자동으로 스프링 빈으로 등록한다. (내부에 @Component 애노테이션이 있어서 컴포넌트 스캔의 대상이 됨)
  - 스프링 MVC에서 애노테이션 기반 컨트롤러로 인식한다.
  
- ModelAndView 
  - 모델과 뷰 정보를 담아서 반환하면 된다.
  - mv.addObject("member", member)
    - 스프링이 제공하는 ModelAndView 를 통해 Model 데이터를 추가할 때는 addObject() 를 사용하면 된다. 
    - 이 데이터는 이후 뷰를 렌더링 할 때 사용된다.

- 실용적인 방식
  ```java
  @Controller
  @RequestMapping("/springmvc/v3/members")
  public class SpringMemberControllerV3 {

      private final MemberRepository memberRepository = MemberRepository.getInstance();

      @GetMapping("/new-form")	
      public String newForm() {
          return "new-form";
      }

      @PostMapping("/save")
      public String save(@RequestParam("username") String username, @RequestParam("age") int age, Model model) {
          Member member = new Member();
          member.setUsername(username); member.setAge(age);
          Member returnMember = memberRepository.save(member);

          model.addAttribute("member", returnMember);
          return "save-result";
      }

      @GetMapping
      public String allMembers(Model model) {
          List<Member> members = memberRepository.findAll();

          model.addAttribute("members", members);
          return "members";
      }
  }
  ```
  
  - Model 파라미터
    - save() , members() 를 보면 Model을 파라미터로 받는 것을 확인할 수 있다. 스프링 MVC도 이런 편의 기능을 제공한다.

  - ViewName 직접 반환
    - 뷰의 논리 이름을 반환할 수 있다.
    
  - @RequestParam 사용
    - 스프링은 HTTP 요청 파라미터를 @RequestParam 으로 받을 수 있다.
    - @RequestParam("username") 은 request.getParameter("username") 와 거의 같은 코드라 생각하면 된다.
    - 물론 GET 쿼리 파라미터, POST Form 방식을 모두 지원한다

  - @RequestMapping -> @GetMapping, @PostMapping
    - @RequestMapping 은 URL만 매칭하는 것이 아니라, HTTP Method도 함께 구분할 수 있다.
      - @RequestMapping(value = "/new-form", method = RequestMethod.GET)
    - @GetMapping , @PostMapping 으로 더 편리하게 사용할 수 있다.
      - 참고로 Get, Post, Put, Delete, Patch 모두 애노테이션이 준비되어 있다.
      - @GetMapping 
        ```
        @Target(ElementType.METHOD)
        @Retention(RetentionPolicy.RUNTIME)
        @Documented
        @RequestMapping(method = RequestMethod.GET)
        ```
        - RequestMapping에 method를 Get으로 사용하는 것
      
** 출처: 스프링 MVC - 백엔드 웹 개발 핵심 기술
