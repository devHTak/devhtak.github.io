---
layout: post
title: Blocking과 NonBlocking
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

#### 출처

- 토비의 봄 TV 8회 스프링 리액티프 프로그래밍(4) - 자바와 스프링의 비동기 기술
