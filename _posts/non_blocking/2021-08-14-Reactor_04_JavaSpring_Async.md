---
layout: post
title: Java, Spring Async
summary: Reactive Programming
author: devhtak
date: '2021-08-09 21:41:00 +0900'
category: Reactive
---

#### Java에서 쓰레드 결과를 가져오는 방법

- java.util.concurrent.Future

  - 비동기적인 연산, 작업에 대한 결과를 갖는다.
  - 쓰레드가 같은 경우에는 return을 받으면 되지만, 다른 쓰레드에 결과를 받기위한 인터페이스
  - Thread Pool
    - Thread를 새로 만드는 것은 리소스가 많이 들기 때문에 Pool에 여러 쓰레드를 생성해 두고, 필요할 때 가져다 쓰고 반납하도록 하여 자원 낭비를 최소화하는 방법
    ```java
    public static void main(String[] args) throws InterruptedException, ExecutionException {
      ExecutorService executorService = Executors.newCachedThreadPool();
      Future<String> future = executorService.submit(()->{
        Thread.sleep(200);
        log.info("Async");
        return "Hello";
      });	
      log.info("Exit");
      log.info(future.get());
    }
    ```
    ```
    11:21:05.148 [main] INFO com.example.demo.java.SpringReactorController - Exit
    11:21:05.351 [pool-1-thread-1] INFO com.example.demo.java.SpringReactorController - Async
    11:21:05.352 [main] INFO com.example.demo.java.SpringReactorController - Hello
    ```
      - ExecutorService의 submit을 통해 Callable 이나 Runnable을 받을 수 있다.
      - Thread는 Callable이나 Runnable의 구현된 메서드를 수행한다는 공통점이 있지만, 아래와 같은 차이점이 있다.
        - interface Callable: V call() throws Exception: 리턴값이 존재하며 Exception을 던질 수 있다.
        - interface Runnable: void run() : 인자, 결과값 리턴이 없다.
      - Blocking : 메인 쓰레드는 pool-1-thread-1이 끝난후에 리턴되는 값을 기다린 후에 결과값을 찍는다. 즉, 다른 쓰레드의 결과를 기다리고 있다.
      - Future에서 get()메서드를 통해 결과 값을 가져올 수 있고 isDone()을 통해 자식 쓰레드가 완료되었는지 확인할 수 있다.

- 비동기 결과를 가져오는 방법
  - java.util.concurrent.FutureTask
    - Callback을 사용하여 가져오는 방법
  - FutureTask 에 done이라는 메서드를 오버라이딩 하여 get()을 호출할 수 있다.
    - done()은 완료될 때 실행되는 메서드로 hook의 일종
  ```java
  @FunctionalInterface
  interface SuccessCallback {
    void onSuccess(String result);
  }
  
  @FunctionalInterface
  interface ExceeptionCallback {
    void onError(Throwable t);
  }
  
  public static class CallbackFutureTask extends FutureTask<String> {
    private SuccessCallback sc;
    private ExceltionCallback ec;
    public CallbackFutureTask(Callable<String> callable, SuccessCallback sc, ExceptionCallback ec) {
      super(callable);
      this.sc = Objects.requireNonNull(sc);
      this.ec = Objects.requireNonNull(ec);
    }
    @Override
    protected void done() {
      // TODO Auto-generated method stub
      try {
        sc.onSuccess(get());
      } catch (InterruptedException | ExecutionException e) {
        ec.onError(e);
      }
    }
  }
  
  public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    FutureTask<String> futureTask = new CallbackFutureTask (()-> {
        Thread.sleep(200);
        log.info("Async");
        return "Hello";
      }, log::info, throable -> log.error(throable.getMessage()) );
    executorService.execute(futureTask);
    executorService.shutdown();
  }
  ```
  ```
  11:58:56.779 [pool-1-thread-1] INFO com.example.demo.java.SpringReactorController - Async
  11:58:56.787 [pool-1-thread-1] INFO com.example.demo.java.SpringReactorController - Hello
  ```
  - SuccessCallback과 ExceptionCallback을 활용하여 callback 을 사용했다.
  - 안타까운 부분은 설정 + 비즈니스 로직이 함께 작성되어 있어서 분리하는 것이 필요하다.

#### Spring에서 쓰레드 결과를 가져오는 방법

- @EnableAsync + @Async
  ```java
  @SpringBootApplication
  @EnableAsync
  @Slf4j
  public class WebFluxExampleApplication {
    public static void main(String[] args) {
      SpringApplication.run(WebFluxExampleApplication.class, args);
    }
    @Component
    public static class MyService {
      @Async
      public Future<String> hello() throws InterruptedException {
        log.info("hello()");
        Thread.sleep(1000);
        return new AsyncResult<String>("Hello");
      }
    }	
    @Bean
    ThreadPoolTaskExecutor tp() {
      // @Async 사용하기 위한 Thread 설정
      ThreadPoolTaskExecutor te = new ThreadPoolTaskExecutor();
      te.setCorePoolSize(10);
      te.setMaxPoolSize(100);
      te.setQueueCapacity(200);
      te.setThreadNamePrefix("mythread");
      te.initialize();
      return te;
    }
    @Autowired MyService myService;
    @Bean
    ApplicationRunner run() {
      return args -> {
        log.info("Run()");
        Future<String> result = myService.hello();
        log.info("Exit: " + result.isDone());
        log.info("Result: " + result.get());
      };
    }
  }
  ```
  ```
  2021-08-15 17:37:17.531  INFO 29808 --- [           main] c.e.demo.WebFluxExampleApplication       : Run()
  2021-08-15 17:37:17.534  INFO 29808 --- [           main] c.e.demo.WebFluxExampleApplication       : Exit: false
  2021-08-15 17:37:17.542  INFO 29808 --- [         task-1] c.e.demo.WebFluxExampleApplication       : hello()
  2021-08-15 17:37:18.545  INFO 29808 --- [           main] c.e.demo.WebFluxExampleApplication       : Result: Hello
  ```
  - Exit이 찍힌 시점이 MyService에 비동기에 대한 결과를 기다리지 않았다.
  - result.get()을 할 때에 비동기 작업이 끝난 후에 가져온다.

- ListenableFuture를 통해 callback을 지정해줄 수 있다
  
  ```java
  ListenableFuture<String> result = myService.hello();
  result.addCallback(item -> log.info(item), e-> log.error(e.getMessage()));
  ```

#### Spring WebMVC

- Servlet을 직접 개발하는 일은 없지만, 3.0 이전에는 Blocking 방식으로 처리하였다.
  - IO 작업에 대해 쓰레드를 하나를 생성하여 처리하였다. 
  - 커넥션당 하나의 쓰레드 할당
  - HttpServletRequest, HttpServletResponse는 InputStream/OuptutStream을 통해 구현되었는데 기본적으로 Blocking 방식이다.

- 하지만 외부에 API를 대기하는 작업을 블로킹 방식으로 진행한다면 비효율적이다.
  - Req1  --> ServletThread01 - req -> Blocking(DB, API..) - WorkThread -> res(html or json <- AsyncContext) 
  
- 3.0 부터 비동기적으로 Servlet 요청을 처리하는 기능이 추가
  - 요청에 대하여  Pool에서 할당받은 Servlet Thread가 Work Thread로 요청을 보내고 반납한다.
  - Work Thread가 처리완료 후 Pool에서 할당받은 new Servlet Thread에게 응답을 보내고 Servlet Thread는 NIO Connector에게 응답을 보낸 후 반납한다.
  - ServletThread가 작업이 끝날 때까지 대기하지 않고 바로 반납하기 때문에 많은 요청/응답을 처리할 수 있다.
    
- 3.1 Servlet Non Blocking IO
  - Callback
  
- Callable을 활용한 예시
  - Client <-> NIO Connector <-> Servlet Thread(<-> Thread Pool) <-> 작업 쓰레드(<-> Thread Pool)
  ```java
  @GetMapping("/callable")
  public Callable<String> async() {
    log.info("callable");
    return () -> {
      log.info("async");
      Thread.sleep(2000);
      return "ok";
    };
  }
  ```
  ```
  2021-08-16 12:06:34.056  INFO 8796 --- [nio-8080-exec-1] com.example.demo.async.AsyncController   : callable
  2021-08-16 12:06:34.064  INFO 8796 --- [         task-2] com.example.demo.async.AsyncController   : async
  ```
  - Callable을 만들어 새로운 쓰레드로 ok를 리턴
  - callable을 호출한 시간 측정
  ```java
  public static void main(String[] args) throws InterruptedException {
    ExecutorService executorService = Executors.newFixedThreadPool(200);
    RestTemplate restTemplate = new RestTemplate();
    String uri = "http://localhost:8080/callable";
    StopWatch mainWatch = new StopWatch();
    mainWatch.start();
    for(int i = 0; i < 100; i++) {
      executorService.execute(() -> {
        StopWatch subWatch = new StopWatch();
        subWatch.start();
        restTemplate.getForObject(uri, String.class);
        subWatch.stop();
        log.info("Elapsed -> " + subWatch.getTotalTimeSeconds());
      });
    }
    executorService.shutdown();
    executorService.awaitTermination(100, TimeUnit.SECONDS);

    mainWatch.stop();		
    log.info("Main Watch -> " + mainWatch.getTotalTimeSeconds());
  }
  ```
  - 톰캣에 Thread Pool사이즈를 1개로 줄여도 모든 요청이 2초만에 끝난다.
  - NIO Connector 1개로 서블릿 쓰레드만 1개로 사용하고 작업 쓰레드는 100개가 실행되어 2초안에 끝나는 것

- DeferredResult Queue
  - Client <-> NIO Connector <-> Servlet Thread(<-> Thread Pool) <-> DeferredResult Queue (<- Event)
  ```java
  Queue<DeferredResult<String>> results = new ConcurrentLinkedQueue<>();
  
  @GetMapping("/deferred-result")
  public DeferredResult<String> deferredResult() {
    log.info("deferred-result");
    DeferredResult<String> dr = new DeferredResult<>();
    results.add(dr);
    return dr;
  }
  @GetMapping("/deferred-result/count")
  public String deferredResultCount() {
    return String.valueOf(results.size());
  }
  @GetMapping("/deferred-result/event")
  public String deferredResultEvent(String message) {
    for(DeferredResult<String> dr: results) {
      dr.setResult("Hello: " + message);
      results.remove(dr);
    }
    return "OK";
  }
  ```
  - /deferred-result로 요청을 보내면 바로 응답이 오지 않는다. /deferred-result/event를 요청하면 해당 deferredResult가 값이 세팅되어 리턴된다.
    - Servlet Thread가 요청을 받고 DeferredResult를 Queue에 저장한 후 풀에 다시 반납한다.
    - DeferredResult Queue에 있는 값은 event 요청이 올때까지 대기한다(따로 Thread를 생성하지 않는다.)
    - /event 요청이 오면 Queue에 있는 값들을 전부 제거하고 응답을 전송한다.
  - 채팅방에서 메세지를 보내면 내부에 돌고 있는 DeferredResult가 단체인원에게 같은 메시지를 보내주는 등에서 사용한다.

- Emitter
  ```java
  @GetMapping("/emitter")
  public ResponseBodyEmitter emitter() {
    ResponseBodyEmitter emitter = new ResponseBodyEmitter();
    Executors.newSingleThreadExecutor().submit(() -> {
      for(int i = 0; i < 50; i++) {
        try {
          emitter.send("<p>Stream " + i + " </p>");
          Thread.sleep(100);
        } catch (Exception e) {}
      }
    });
    return emitter;
  }
  ```
  - spring-webmvc에서 HTTP Streaming을 하는 가장 간단한 방법은 SSE(Server-Sent Event)를 사용하는 것이다.
  - Spring에서 지원하는 Async 응답 방식인 DeferredResult, Callable과 같다
    - 반환형이 Async 응답을 필요로하는 경우, Spring MVC는 request.startAsync()를 호출하여 AsyncContext를 획득하여 저장해둔다.
    - AsyncContext는 비동기 처리를 제어할 수 있도록 해준다.
      - ex) Sevrlet container thread에서 요청 처리를 재개시키며, 기존의 forward와 같은 동작을 하는 dispatch 메서드를 제공한다.
    - 이로 인해 응답을 별도의 thread에서 비동기로 처리할 수 있다.
    - 요청 thread를 종료시키되, response는 열어둔다
    - 다른 thread에서 AsyncContext를 사용해서 response를 완성시킨다.
    - Spring MVC는 Servlet container로 다시 요청을 보내고(dispatch), client로 응답한다.

#### 출처

- 토비의 봄 TV 8회 스프링 리액티프 프로그래밍(4) - 자바와 스프링의 비동기 기술
- https://supawer0728.github.io/2018/03/15/spring-http-stereamings/
