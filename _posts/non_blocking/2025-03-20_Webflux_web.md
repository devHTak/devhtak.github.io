
#### 리액티브 웹 - Servlet 구조와 유사한 인터페이스 구현
- 서블릿 API와 유사한 인터페이스를 가져야 한다.
  - javax.servlet.Servlet#service 메서드 대체
  ```java
  // 요청 및 응답
  interface ServerHttpRequest { Flux<DataBuffer> getBody(); }
  interface ServerHttpResponse { Mono<Void> writeWith(Publisher<? extends DataBuffer> body); }
  interface ServerWebExchange {
      ServerHttpRequest getRequest();
      ServerHttpResponse getResponse();
      Mono<WebSession> getSession();
  }
  // 핸들러 및 필터
  interface WebHandler { Mono<Void> handle(ServerWebExchange exchange); }
  interface WebFilterChain { Mono<Void> filter(ServerWebExchange exchange); }
  interface WebFilter { Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain); }
  ```
- Reactive Web 스택
  - 요청 -> 서버 엔진(Netty, Undertow...) -(ServerHttpRequest)-> HttpHandler -(ServerWebExchange)-> WebFilterChain -(ServerWebExchange)-> DispatcherHandler -> RouterFunctionMapping -> HandleFunctionAdapter or RequestMappingHandlerMapping
    - 기본 서버 엔진에서 처리하는 요청 입력
    - HttpHandler 계층을 통해 , request, response, 세션 및 관련 정보를 exchange로 결합
    - WebFilterChain 계층을 통해 정의된 WebFilter를 체인으로 구성
    - 모든 필터 적용이 완료되면 WebHandler 인스턴스 호출(DispatcherHandler)
    - HandlerMapping의 인스턴스를 찾고 적합한 핸들러를 호출

#### 웹플럭스로 구현하는 순수한 함수형 웹
- 새로운 함수적 접근 방식을 사용해 복잡한 라우팅을 작성하는 방법
  - 새로운 함수적 접근 방식이란? AWS 람다와 같이 간단한 응용 프로그램을 쉽게 만들 수 있게 하는 것으로 Vert.x, Ratpack과 같은 다른 프레임워크에서 지원하는 것처럼 함수적인 라우팅 매핑과 복잡한 요청 라우팅 로직을 내장 API를 이용하여 경량 애플리케이션을 개발할 수 있게 하였다.
```java
@Bean
public RouterFunction<ServerResponse> routes(OrderHandler handler) {
    return nest(path("/orders"),
            nest(accept(APPLICATION_JSON),
                route(GET("/{id}"), handler::get)
                  .andRoute(method(HttpMethod.GET), handler::list)
            ).andNest(contentType(APPLICATION_JSON), route(POST("/"), handler::create))
        );      
}
```
```java
public class OrderHandler {
    final OrderRepository repository;

    public Mono<ServerResponse> create(ServerRequest request) {
        return request.bodyToMono(Order.class)
                  .flatMap(repository::save)
                  .flatMap(o -> ServerResponse.create(URI.create("/order/"+o.id)).build());
    }
}
```
  - nest, route: RouterFunctions 내부 팩토리 메서드로 RouterFuntion 인터페이스를 다른 동작으로 반환
  - 함수적 방법을 통해 핸들러 선언을 할 수 있도록 해주며 모든 경로 매핑을 한군데에서 명시적으로 관리할 수 있게 한다.
- 
