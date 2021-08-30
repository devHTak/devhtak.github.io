---
layout: post
title: CompletableFuture
summary: Reactive Programming
author: devhtak
date: '2021-08-30 21:41:00 +0900'
category: Reactive
---

#### Future

- Future (java)
  - Future는 비동기 처리 결과를 표현하기 위해서 사용된다. 비동기 처리가 완료되었는지 확인하고, 처리 완료를 기다리고, 처리 결과를 리턴하는 메소드를 제공한다. 
  - get() 메서드를 통해 결과를 가져온다. 동기식으로 결과가 끝날때까지 대기한다.
  - Future 구현체로 FutureTask가 있다.

- ListenableFuture (spring)
  - 비동기 task 처리가 가능하며 callback method를 통해 task가 성공했을 때, 실패했을 때의 작업을 미리 지정해놓을 수 있다.
  - completable() 메서드를 통해서 CompletableFuture로 변환할 수 있다.

- CompletableFuture
  - Java 8에서 나왔으며 간단하게 비동기적인 결과를 담고 있고, 가져올 수 있다.
  - CompletableFuture 리스트의 모든값이 완료될 때까지 기다릴지 아니면 하나의 값만 완료되길 기다릴지 선택할 수 있다는 것이 장점
  - 병렬성과(Parallelism)과 동시성(Concurrency)에서 CompletableFuture가 중요한데, 여러개의 cpu core 사이에 지연 실행이나 예외를 callable하게 처리할 수 있어서 명시적인 처리가 가능해진다.

#### CompletableFuture 사용

- 예시
  ```java
  CompletableFuture<String> cf1 = new CompletableFuture<>();
  cf1.complete("Hello");
  log.info(cf1.get()); // Hello 출력
  
  CompletableFuture<String> cf2 = new CompletableFuture<>();
  cf2.completeExceptionally(new RuntimeException());
  log.error(cf2.get()); // 예외 발생
  ```
  
  - CompletableFuture에는 비동기 Task에 대한 결과가 담기며, get() 메서드를 호출해서 가져올 수 있다.
  - complete()를 통해 완료와 함께 결과값을 지정할 수 있고, completeExceptionally() 를 통해 완료와 함께 예외를 지정할 수 있다.

- runAsync & thenRun
  ```java
  CompletableFuture
      .runAsync(() -> log.info("runAsync"))
      .thenRun(() -> log.info("thenRun"))
      .thenRun(() -> log.info("thenRun"));
  log.info("exit");
  ForkJoinPool.commonPool().shutdown();
  ForkJoinPool.commonPool().awaitTermination(10, TimeUnit.SECONDS);
  ```
  
  ```
  22:10:31.377 [main] INFO com.example.demo.completable.Example - exit
  22:10:31.377 [ForkJoinPool.commonPool-worker-3] INFO com.example.demo.completable.Example - runAsync
  22:10:31.381 [ForkJoinPool.commonPool-worker-3] INFO com.example.demo.completable.Example - thenRun
  22:10:31.381 [ForkJoinPool.commonPool-worker-3] INFO com.example.demo.completable.Example - thenRun
  ```
  
  - 의존적인 비동기작업을 간단하게 구현할 수 있다.
  - 두개 이상의 비동기작업을 실행하여 끝나면 결과를 사용하도록도 구현할 수 있다.
  - ForkJoinPool
    - Java 7에서 새로 지원하는 fork-join 풀은 기본적으로 큰 업무를 작은 업무로 나누어 배분해서 , 일을 한 후에 일을 취합하는 형태입니다.
    - Fork 를 통해서 업무를 분담하고 Join 을 통해서 업무를 취합합니다.
    - 쓰레드별 큐를 추가하여 업무를 분담하고 나누어 처리하여 효율을 높일 때 사용

     ![image](https://user-images.githubusercontent.com/42403023/131344084-88f64d4b-e28c-440d-a14b-633c11edf73c.png)
     
     - 이미지 출처: https://hamait.tistory.com/612 [HAMA 블로그]

- supplyAsync, thenApply, thenAccept
  ```java
  CompletableFuture
      .supplyAsync(() -> { 
          log.info("supplyAsync");
          return 1;
      }).thenApply(s -> { 
          log.info("thenApply {}", s); return s + 1; 
      }).thenAccept(s -> log.info("thenAccept {}", s)); 
  log.info("exit");
  ForkJoinPool.commonPool().shutdown();
  ForkJoinPool.commonPool().awaitTermination(10, TimeUnit.SECONDS);
  ```
  
  ```
  22:15:38.440 [main] INFO com.example.demo.completable.Example - exit
  22:15:38.440 [ForkJoinPool.commonPool-worker-3] INFO com.example.demo.completable.Example - supplyAsync
  22:15:38.445 [ForkJoinPool.commonPool-worker-3] INFO com.example.demo.completable.Example - thenApply 1
  22:15:38.447 [ForkJoinPool.commonPool-worker-3] INFO com.example.demo.completable.Example - thenAccept 2
  ```
  
  - thenApply(Function<T, R>) : 앞의 비동기 작업의 결과를 받아 사용해 새로운 값을 return.
  - thenAccept(Consumer<T>) : 앞의 비동기 작업의 결과를 받아 사용하며 return이 없다.
  
- thenCompose
  ```java
  CompletableFuture
      .supplyAsync(() -> {
          log.info("supplyAsync");
          return 1;
      }).thenCompose(s -> { 
          log.info("thenApply {}", s);
          return CompletableFuture.completedFuture(s + 1);
      }).thenApply(s -> {
          log.info("thenApply {}", s);
          return s + 1;
      }).thenAccept(s -> log.info("thenAccept {}", s));
  log.info("exit");
  ForkJoinPool.commonPool().shutdown();
  ForkJoinPool.commonPool().awaitTermination(10, TimeUnit.SECONDS);
  ```

  ```
  22:56:16.613 [main] INFO com.example.demo.completable.Example - exit
  22:56:16.613 [ForkJoinPool.commonPool-worker-3] INFO com.example.demo.completable.Example - supplyAsync
  22:56:16.618 [ForkJoinPool.commonPool-worker-3] INFO com.example.demo.completable.Example - thenApply 1
  22:56:16.621 [ForkJoinPool.commonPool-worker-3] INFO com.example.demo.completable.Example - thenApply 2
  22:56:16.621 [ForkJoinPool.commonPool-worker-3] INFO com.example.demo.completable.Example - thenAccept 3
  ```

  - thenCompose(Function<T, R>): return이 CompletableFuture인 경우 thenCompose를 사용한다.

- exceptionally
  ```java
  CompletableFuture
      .supplyAsync(() -> {
          log.info("supplyAsync");
          return 1;
      }).thenCompose(s -> { 
          log.info("thenApply {}", s);
          if (1 == 1) throw new RuntimeException();
          return CompletableFuture.completedFuture(s + 1);
      }).exceptionally(e -> {
          log.info("exceptionally");
          return -10;
      }).thenApply(s -> {
          log.info("thenApply {}", s);
          return s + 1;
      }).thenAccept(s -> log.info("thenAccept {}", s));
  log.info("exit");
  ForkJoinPool.commonPool().shutdown();
  ForkJoinPool.commonPool().awaitTermination(10, TimeUnit.SECONDS);
  ```

  ```
  23:00:17.344 [main] INFO com.example.demo.completable.Example - exit
  23:00:17.344 [ForkJoinPool.commonPool-worker-3] INFO com.example.demo.completable.Example - supplyAsync
  23:00:17.350 [ForkJoinPool.commonPool-worker-3] INFO com.example.demo.completable.Example - thenApply 1
  23:00:17.352 [ForkJoinPool.commonPool-worker-3] INFO com.example.demo.completable.Example - exceptionally
  23:00:17.352 [ForkJoinPool.commonPool-worker-3] INFO com.example.demo.completable.Example - thenAccept -10
  ```
  - Exception이 발생하면 바로 exceptionally를 타게 된다.

- thenApplyAsync
  ```java
  ExecutorService es = Executors.newFixedThreadPool(10);
  CompletableFuture
      .supplyAsync(() -> {
          log.info("supplyAsync");
          return 1;
      }, es).thenCompose(s -> {
          log.info("thenApply {}", s);
          return CompletableFuture.completedFuture(s + 1);
      }).thenApply(s -> {
          log.info("thenApply {}", s);
          return s + 2;
      }).thenApplyAsync(s -> {
          log.info("thenApply {}", s);
          return s + 3;
      }, es).exceptionally(e -> {
          log.info("exceptionally");
          return -10;
      }).thenAcceptAsync(s -> log.info("thenAccept {}", s), es);
  log.info("exit");
  ForkJoinPool.commonPool().shutdown();
  ForkJoinPool.commonPool().awaitTermination(10, TimeUnit.SECONDS);
  ```
	
  ```
  23:03:34.965 [main] INFO com.example.demo.completable.Example - exit
  23:03:34.964 [pool-1-thread-1] INFO com.example.demo.completable.Example - supplyAsync
  23:03:34.970 [pool-1-thread-1] INFO com.example.demo.completable.Example - thenApply 1
  23:03:34.972 [pool-1-thread-1] INFO com.example.demo.completable.Example - thenApply 2
  23:03:34.973 [pool-1-thread-2] INFO com.example.demo.completable.Example - thenApply 4
  23:03:34.974 [pool-1-thread-3] INFO com.example.demo.completable.Example - thenAccept 7
  ```
  - 이 작업은 다른 스레드에서 처리를 하려고 할 때, thenApplyAsync를 사용한다.
  - 스레드의 사용을 더 효율적으로 하고 자원을 더 효율적으로 사용한다.
  - 현재 스레드 풀의 정책에 따라서 새로운 스레드를 할당하거나 대기중인 스레드를 사용한다. (스레드 풀 전략에 따라 다르다.)
  
#### 

#### 출처

- 토비의 봄 TV 11회 스프링 리액티프 프로그래밍(7) - CompletableFuture
- https://gunju-ko.github.io/java/2018/07/05/Future.html
- https://hamait.tistory.com/612 [HAMA 블로그]
