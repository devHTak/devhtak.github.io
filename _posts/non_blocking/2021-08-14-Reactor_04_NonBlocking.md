---
layout: post
title: Blocking과 NonBlocking
summary: Reactive Programming
author: devhtak
date: '2021-08-09 21:41:00 +0900'
category: Reactive
---

#### java.util.concurrent.Future

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

- java.util.concurrent.FutureTask

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

#### 출처

- 토비의 봄 TV 8회 스프링 리액티프 프로그래밍(4) - 자바와 스프링의 비동기 기술
