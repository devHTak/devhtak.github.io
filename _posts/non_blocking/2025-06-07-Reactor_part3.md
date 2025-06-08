#### Spring Webflux
- Spring Webflux 탄생 배경
  - Spring MBC의 한계
    - Servlet 기반의 Blocking IO 방식
    - 하나의 요청을 사용하기 위해 하나의 스레드를 사용하고, 해당 스레드의 작업이 끝나기 전까지 스레드는 block 된다.
  - 대량의 요청을 안정적으로 처리하기 위해 Non-Blocking 형태의 Spring Webflux 탄생
- Webflux 기술 스택
  ```
  Servlet Containers <-> Netty Servlet 3.1+ Containers
  Servlet API <-> Reactive Streams Adapter
  Spring Security <-> Spring Security Reactive
  Spring MVC <-> Spring Webflux
  Spring Data Reposiotries(JDBC, JPA, NoSQL) <-> Spring Data Reactive Repository(R2DBC, NoSQL)
  ```
  - 서버
    - Spring MVC는 Tomcat과 같이 Servlet Container에서 Blocking IO 방식으로 동작
    - Webflux는 Non Blocking을 동작하는 Netty 등의 서버 엔진 사용
  - 서버 API
    - Webflux는 기본 서버 엔진이 Netty 지만, Jetty나 Undertow 같은 서버 엔진에서 지원하는 리액티브 스트림즈 어댑터를 통해 리액티브 스트림즈 지원
  - 보안
    - WebFilter를 이용해 Spring Security를 Webflux에서 사용할 수 있다.
  - 데이터 액세스
    - Non-Blocking IO를 지원할 수 있도록 R2DBC 및 NoSQL 모듈 사용
- 요청 처리 흐름
1. 최초에 요청이 들어오면 Netty 등의 서버 엔진을 거쳐 HttpHandler가 전달 받는다. Handler는 다양한 서버 엔진을 지원할 수 있도록 추상화해주는 역할
  a. 각 서버엔진마다 주어지는 ServerHttpRequest, ServerHttpResponse를 포함하는 ServerWebExchange를 생성한 후 WebFilter 전달
2. ServerWebExchange는 WebFilter 체인에서 전처리 과정을 거친 후, WebHandler 인터페이스 구현체인 DispatcherHandler에게 전달
3. DispatcherServlet과 유사한 역할을 하는 Dispatcher Handler에서는 HandlerMapping List를 원본 Flux의 소스로 전달 받는다.
4. ServerWebExchange를 처리할 핸들러를 조회
5. 조회한 핸들러의 호출을 HandlerAdapter에게 위임
6. HandlerAdapter는 ServerWebExchange를 처리할 핸들러 호출
7. Controller 또는 HandlerFunction 형태의 핸들러에서 요청을 처리한 후 응답 데이터를 리턴
8. 핸들러로부터 리턴받은 응답 데이터를 처리할 HandlerResultHandler를 조회
9. 조회한 HandlerResultHandler가 응답 데이터를 처리한 후 response 리턴
- 핵심 컴포넌트
  - HttpHandler
    ```java
    public interface HttpHandler {
        Mono<Void> handle(ServerHttpRequest request, ServerHttpResponse response);
    }
    public class HttpWebHandlerAdapter extends WebHandlerDecorator implements HttpHandler {
        Mono<Void> handle(ServerHttpRequest request, ServerHttpResponse response) {
            // ...
            ServerWebExchane exchange = createExchange(request, response);
            // ...
        }
    }
    ```
  - WebFilter
    ```java
    public interface WebFilter {
        Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain);
    }
    @Component
    public class BookLogFilter implements WebFilter {
        Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {
            String path = exchange.getRequest().getURI().getPath();
            return chain.filter(exchange).doAfterTerminate(() -> {
                if(path.contains("books") {
                  System.out.println("path: " +  path);
              }
            });
        }
    }
    }
    ``` 
    - Servlet Filter 처럼 핸들러가 요청을 처리하기 전에 전처리 작업을 할 수 있도록 해준다.
    - WebFilterChain을 통해 필터 체인을 형성하여 원하는 만큼의 필터를 추가할 수 있다.
  - HandlerFilterFunction
    ```java
    @FuntionalInterface
    public interface HandlerFilterFunction<T extends ServerResquest, R extends ServerResponse> {
        Mono<R> filter(ServerWebRequest request, HandlerFunction<T> next);
        ///
    }
    public class BookRouterFunctionFilter implements HandlerFilterFunction {
        @Override
        Mono<R> filter(ServerWebRequest request, HandlerFunction<T> next) {
            String path = exchange.getRequest().getURI().getPath();
            return next.handler(exchange).doAfterTerminate(() -> {
                if(path.contains("books") {
                  System.out.println("path: " +  path);
              }
            });
        }
    }
    @Configuration
    public class BookRouterFunction {
        return RouterFunctions
            .route(GET("v1/router/books/{bookId}), (ServerRequest) request -> this.getBook(request))
            .filter(new BookRouterFunctionFilter());
    }
    ``` 
    - 함수형 기반의 요청 핸들러에 적용할 수 있는 Filter
    - WebFilter 구현체는 Spring Bean으로 등록되는 반면 HandlerFilterFunction 구현체는 애너테이션 기반의 핸들러가 아닌 함수형 기반의 요청 핸들러에서 함수 형태로 사용되기 때문에 bean으로 동작하지 않는다.
  - DispatcherHandler
    - WebHandler 구현체로 Front Controller 패턴이 적용되어 먼저 요청을 전달받은 후 다른 컴포넌트에 요청 처리 위임
    - DispatchHandler 자체가 Bean으로 등록되어 동작하며, HandlerMapping, HandlerAdapter, HandlerResultHandler 등의 요청 처리를 위한 위임 컴포넌트 검색
  - HandlerMapping
    ```java
    public interface HandlerMapping {
        Mono<Object> getHandler(ServerWebExchange exchange);
    }
    ```
    - MVC와 동일하게 request와 handler object에 대한 매핑을 정의하는 인터페이스이며, HandlerMapping 인터페이스를 구현하는 구현클래스로 RequestMappingHandlerMapping, RouterFunctionMapping 등이 있다.
  - HandlerAdapter
    ```java
    public interface HandlerAdapter {
        boolean supports(Object handler); // 파라미터로 전달받은 handler object를 지원하는지 체크
        Mono<HandlerResult> handle(ServerWebExchange exchange, Object handler); // 파라미터로 전달받은 handler object를 통해 핸들러 메서드 호출
    }
    ```
    - HandlerMapping을 통해 얻은 핸들러를 직접 호출하는 역할을 하며 HandlerResult를 리턴받는다.
    - RequestMappingHandlerAdapter, HandlerFunctionAdapter, SimpleHandlerAdapter, WebSocketHandlerAdapter 존재
- Non-Blocking 프로세스 구조
  - Blocking IO 방식의 MVC는 요청을 처리하는 스레드가 차단될 수 있기 떄문에 기본적으로 대용량의 스레드 풀을 사용해 하나의 요청을 하나의 스레드가 처리
  - NonBlocking IO 방식은 WebFlux는 스레드가 차단되지 않기 때문에 적은 수의 고정된 스레드 풀을 사용해 더 많은 요청을 처리
  - 스레드 차단 없이 더 많은 요청을 처리할 수 있는 이유는 Event Loop 방식 때문
    - 요청을 요청 핸들러가 전달받는다.
    - 전달받은 요청을 이벤트 루프에 푸시하면 이벤트 루프는 네트워크, DB 연결 작업 등 비용이 드는 작업에 대한 콜백 등록
    - 작업이 완료되면 완료 이벤트를 이벤트 루프에 푸쉬
    - 등록한 콜백을 호출해 처리 결과 전달
  - 요청 처리, DB 연결과 같은 IO 작업을 이벤트로 처리하기 때문에 이벤트에 대한 콜백을 등록하고 동시에 다음 이벤트 처리로 넘어간다.
- 스레드 모델
  - NonBlocking IO를 지원하는 Netty등의 서버 엔진에서 작은 수의 고정된 크기의 스레드를 생성하여 대량의 요청 처리
  - Reactor Netty에서는 CPU 코어의 개수가 4보다 작은 경우 최소 4개의 워커 스레드를 생성하고, 4보다 더 많다면 코어 개수만큼 스레드 생성
    - 적은 수의 스레드로 대량의 요청을 효과적으로 처리할 수 있다.
  - 클라이언트 요청부터 응답 처리과정 안에서 Blocking되는 지점이 존재한다면 오히려 성능이 저하된다.
    - Scheduler 를 통해 서버 엔진에서 제공하는 스레드 풀이 아닌 다른 스레드 풀을 사용한다.

#### Annotation 기반의 Controller
- Spring Webflux 는 두가지 프로그래밍 모델 지원. 하나는 애노테이션 기반 프로그래밍, 다른 하나는 함수형 기반 프로그래밍 모델
- Annotation 기반 Controller
  ```java
  @RestController
  @RequiredArgsConstructor
  @RequestMapping("/v1/books")
  public class BookController {
      private final BookService bookService;

      @PostMapping
      @ResponseStatu(HttpStatus.CREATED)
      public Mono createBook(@RequestBody Mono<BookDto.Post> requestBody) {
          Mono<Book> book = bookService.createBook(requestBody.toDto());
          Mono<BookDto.Response> response = book.toResponse();
          return this.toResponse(book);
      }

      @PatchMapping("/{bookId}")
      public Mono updateBook(@PathVariable long bookId, @RequestBody Mono<BookDto.Post> requestBody) {
          requestBody.setBookId(bookId)
          Mono<Book> book = bookService.updateBook(requestBody.toDto());
          return this.toResponse(book);
      }
      @GetMapping("/{bookId}")
      public Mono getBook(@PathVariable long bookId) {
          Mono<Book> book = bookService.getBook(bookId);
          return this.toResponse(book);
      }

      private Mono<BookDto.Response> toResponse(Mono<Book> book) {
          return mono.flatMap(book -> Mono.just(book.toResponse()));
      }
  }
  ```
  - MVC 기반과 차이점은 리턴타입과 아규먼트가 Mono<Book>이라는 것을 제외하고 거의 없다.
  - Webflux는 핸들러 메서드의 아규먼트를 리액티브 타입을 지원하기 때문에 method argument로 전달 받은 후 서비스 레이어에 전달할 수 있다.

#### 함수형 엔드포인트
- Webflux는 기존의 애노테이션 기반 프로그래밍 모델과 함께 함수형 엔드포인트 기반 새로운 프로그래밍 모델 지원
  - 들어오는 요청을 라우팅하고, 라우팅된 요청을 처리하며 결과 값을 응답으로 리턴하는등의 모든 작업을 하나의 함수 체인에서 처리
- HandlerFunction을 사용한 request 처리
  ```java
  @FunctionalInterface
  public interface HandlerFunction<T extends ServerResponse> {
      Mono<T> handle(ServerRequest request);
  }
  ```
  - 함수형 엔드포인트는 들어오는 요청을 처리하기 위해 HandlerFunction이라는 함수형 기반의 핻를러 사용
  - HandlerFunction 은 요청처리를 위한 ServerRequest하나만 handle() 아규먼트로 전달받으며, 응답은 Mono<ServerResponse> 형태로 리턴된다.
  - ServerRequest
    - Http Headers, method, URI, query parameters에 접근할 수 있는 메서드를 제공하며, Http Body 정보에 접근하기 위한 별도의 메서드 제공
  - ServerResponse
    - HandlerFunction or HandlerFilterFunction에서 리턴되는 Http Response를 표현
    - Response는 BodyBuilder와 HeadersBuilder를 통해 HTTP response body, header 정보 추가 가능
- Request 라우팅을 위한 RouterFunction
  ```java
  @FunctionalInterface
  public interface RouterFunction<T extends ServerResponse> {
      Optional<HandlerFunction<T>> route(ServerRequest request);
      // ...
  }
  @Configuration("bookRouterV1")
  public class BookRouter {
      @Bean
      public RouterFunction<?> routeBook(BookHandler handler) {
          return route()
              .POST("/v1/books", handerl::createBook(handler::createBook)
              .PATHCH("/v1/books/{bookId}", handler::updateBook)
              .GET("/v1/books", hnadler::getBooks)
              .GET("/v1/books/{bookId}", handler::getBook)
              .build();
      }
  }
  @Component("bookHandlerV1")
  @RequiredArgsConstructor
  public class BookHandler {
      private final BookMapper mapper;
      public Mono<ServerResponse> createBook(ServerRequest request) {
          return request.bodyToMono(BookDto.Post.class)
                  .map(post -> bookService.createBook(post))
                  .flatMap(book -> ServerResponse
                      .created(URI.create("/v1/books" + book.getId())).build());
      }
      // ...
  }
  ```
  - RouterFunction은 요청을 해당 HandlerFunction으로 라우팅해주는 역할을 한다.
  - route() 메서드에서 파라미터로 전달받은 request에 매치되는 HandlerFunction을 리턴
- 함수형 엔드포인트에서의 request body 유효성 검증
  ```java
  @Component("bookValidatorV2")
  public class BookValidator implements Validator {
      @Override
      public boolean supports(Class<?> clazz) {
          return BookDto.Post.class.isAssignableFrom(clazz);
      }
      @Override
      public void validate(Object target, Errors errors) {
          BookDto.Post post = (BookDto.Post)target;
          ValidationUtils.rejectIfEmptyOrWhitespace(errors, "titleKorean", "field,required");
          // ...
      }
  }
  @Component("bookHandlerV1")
  @RequiredArgsConstructor
  public class BookHandler {
      private final BookMapper mapper;
      private final BookValidator validator
      public Mono<ServerResponse> createBook(ServerRequest request) {
          return request.bodyToMono(BookDto.Post.class)
                  .doOnNext(post -> this.validate(post))
                  .map(post -> bookService.createBook(post))
                  .flatMap(book -> ServerResponse
                      .created(URI.create("/v1/books" + book.getId())).build());
      }
      // ...
      private void validate(BookDto.Post post) {
          Error error = new BeanPropertyBindingResult(post, BookDto.class.getName());
          validator.validate(post, errors);
          if(errors.hasErrors()) {
              log.error(errors.getAllErrors().toString());
              throw new ServerWebInputException(errors.toString());
          }
      }
  }
  ```
  - 함수형 엔드포인트는 spring validator 인터페이스 구현한 Custom Validator를 이용해 request body 유효성 검증을 적용할 수 있다.
  - 이 외에도 Spring Validator 인터페이스 사용, javax 표준 Validator 인터페이스 사용 가능하다
#### R2DBC

#### 예외처리

#### WebClient
