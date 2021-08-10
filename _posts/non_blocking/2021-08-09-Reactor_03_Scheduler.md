---
layout: post
title: Reactor, Operator
summary: Reactive Programming
author: devhtak
date: '2021-08-09 21:41:00 +0900'
category: Reactive
---

#### Thread Scheduling

- Reactor는 비동기 실행을 강행하지 않는다.
  ```java
  Flux<Integer> flux = Flux.range(0, 5);
  flux.subscribe(item -> log.info("onNext: " + item),
    throwable -> log.info("onError: " + throwable.getMessage()),
    () -> log.info("onComplete"));
  ```
  ```
  22:02:38.940 [main] INFO com.example.demo.iterable.SchedulerController - onNext: 0
  22:02:38.941 [main] INFO com.example.demo.iterable.SchedulerController - onNext: 1
  22:02:38.941 [main] INFO com.example.demo.iterable.SchedulerController - onNext: 2
  22:02:38.941 [main] INFO com.example.demo.iterable.SchedulerController - onNext: 3
  22:02:38.941 [main] INFO com.example.demo.iterable.SchedulerController - onNext: 4
  22:02:38.941 [main] INFO com.example.demo.iterable.SchedulerController - onComplete
  ```
  - 모두 main 메서드에서 실행한다.
  - Publisher(subscribe)가 Subscribe(onSubscribe)를 호출하고, Subscribe는 Subscription을 통해 onNext, onError, onComplete가 실행되는 일련의 동작이 하나의 main 메서드를 통해 실행된다.
  - 실제로는 Pub과 Sub이 직렬적으로 돌아가는 구현하지 않는다.
    - Scheduler의 publishOn, subscribeOn을 통해 새로운 쓰레드를 구성한다.

- subscribeOn

  ![image](https://user-images.githubusercontent.com/42403023/128712347-52ce37fe-81dd-4563-81a1-1604879f7658.png)

  ** 이미지 출처: https://tech.kakao.com/2018/05/29/reactor-programming/
  
  - Subscriber에서 request(Subscription) 신호를 별도 Scheduler로 처리한다 
  - 전형적으로 blocking IO와 같이 publisher가 느리고 consumer가 빠른 경우에 사용
  - 예시
    - Flux 활용
      ```java
      flux.subscribeOn(Scehdulers.single()).subscribe();
      ```
    - Publisher 구현체 활용
      ```java
      Publisher<Integer> publisher = subscriber -> {
        subscriber.onSubscribe(new Subscription() {
          @Override
          public void request(long n) { 
            Arrays.asList(1, 2, 3, 4, 5).forEach(i -> subscriber.onNext(i));
            subscriber.onComplete();
          }
          @Override
          public void cancel() {}
        });
      };
      Publisher<Integer> subOnPub = subscriber -> {
        ExecutorService executorService = Executors.newSingleThreadExecutor();
        executorService.execute(() -> publisher.subscribe(subscriber));
      };
      Subscriber<Integer> subscriber = new Subscriber<Integer>() {
        public void onSubscribe(Subscription subscription) {log.info("onSubscribe"); subscription.request(Long.MAX_VALUE);};
        public void onNext(Integer item) {log.info("onNext: " + item);};
        public void onError(Throwable throwable) {log.info("onError: " + throwable.getMessage());};
        public void onComplete() {log.info("onComplete");};
      };
      subOnPub.subscribe(subscriber);
      ```
      ```
      22:37:35.742 [pool-1-thread-1] INFO com.example.demo.iterable.SchedulerController - onSubscribe
      22:37:35.748 [pool-1-thread-1] INFO com.example.demo.iterable.SchedulerController - onNext: 1
      22:37:35.748 [pool-1-thread-1] INFO com.example.demo.iterable.SchedulerController - onNext: 2
      22:37:35.748 [pool-1-thread-1] INFO com.example.demo.iterable.SchedulerController - onNext: 3
      22:37:35.748 [pool-1-thread-1] INFO com.example.demo.iterable.SchedulerController - onNext: 4
      22:37:35.748 [pool-1-thread-1] INFO com.example.demo.iterable.SchedulerController - onNext: 5
      22:37:35.748 [pool-1-thread-1] INFO com.example.demo.iterable.SchedulerController - onComplete

      ```
      - single()을 사용했기 때문에 동일한 Thread를 사용하였지만 Main method를 사용하지는 않는다.

- publishOn
  
  ![image](https://user-images.githubusercontent.com/42403023/128712424-90d978df-60c5-4094-8bc1-d6c5aa1206c4.png)

  ** 이미지 출처: https://tech.kakao.com/2018/05/29/reactor-programming/
  
  - publishOn은 subscribe 이후 onNext, onComplete, onError을 호출할 때 별도 쓰레드를 생성한다.
  - publishOn() 메서드를 이용하면 next, complete, error신호를 별도 쓰레드로 처리할 수 있다. 
  - map(), flatMap() 등의 변환도 publishOn()이 지정한 쓰레드를 이용해서 처리한다.
  - Publisher는 빠르게 진행되며 Subscriber가 상대적으로 느린 경우 사용한다.
    ```java
    flux.publishOn(Schedulers.single()).subscribe()
    ```
    ```java
    Publisher<Integer> publisher = subscriber -> {
      subscriber.onSubscribe(new Subscription() {
        @Override
        public void request(long n) { 
          Arrays.asList(1, 2, 3, 4, 5).forEach(item -> subscriber.onNext(item));
          subscriber.onComplete();
        }
        @Override
        public void cancel() {}
      });
    };		
    Publisher<Integer> pubOnPub = sub -> {
      publisher.subscribe(new Subscriber<Integer>() {
        ExecutorService executorService = Executors.newSingleThreadExecutor();
        public void onSubscribe(Subscription subscription) {sub.onSubscribe(subscription);}
        public void onNext(Integer item) { executorService.execute(()-> sub.onNext(item)); }
        public void onError(Throwable throwable) {executorService.execute( () -> sub.onError(throwable)); }
        public void onComplete() { executorService.execute(()-> sub.onComplete());}
      });
    };
    Subscriber<Integer> subscriber = new Subscriber<Integer>() {
      public void onSubscribe(Subscription subscription) {log.info("onSubscribe"); subscription.request(Long.MAX_VALUE);};
      public void onNext(Integer item) {log.info("onNext: " + item);};
      public void onError(Throwable throwable) {log.info("onError: " + throwable.getMessage());};
      public void onComplete() {log.info("onComplete");};
    };
    pubOnPub.subscribe(subscriber);
    ```
    ```
    11:16:26.826 [main] INFO com.example.demo.iterable.SchedulerController - onSubscribe
    11:16:26.839 [pool-1-thread-1] INFO com.example.demo.iterable.SchedulerController - onNext: 1
    11:16:26.840 [pool-1-thread-1] INFO com.example.demo.iterable.SchedulerController - onNext: 2
    11:16:26.840 [pool-1-thread-1] INFO com.example.demo.iterable.SchedulerController - onNext: 3
    11:16:26.841 [pool-1-thread-1] INFO com.example.demo.iterable.SchedulerController - onNext: 4
    11:16:26.841 [pool-1-thread-1] INFO com.example.demo.iterable.SchedulerController - onNext: 5
    11:16:26.841 [pool-1-thread-1] INFO com.example.demo.iterable.SchedulerController - onComplete
    ```
    
- publishOn과 subscribeOn 모두 적용 가능하다.
  ```java
  Publisher<Integer> publisher = subscriber -> {
    subscriber.onSubscribe(new Subscription() {
        @Override
        public void request(long n) { 
          Arrays.asList(1, 2, 3, 4, 5).forEach(item -> subscriber.onNext(item));
          subscriber.onComplete();
        }				
        @Override
        public void cancel() {}
      });
    };
    Publisher<Integer> subOnPub = sub -> {
      ExecutorService executorService = Executors.newSingleThreadExecutor(new CustomizableThreadFactory() {
        @Override
        public String getThreadNamePrefix() { return "subon-"; }
      });
      executorService.execute(() -> publisher.subscribe(sub));
    };
    Publisher<Integer> pubOnPub = sub -> {
      subOnPub.subscribe(new Subscriber<Integer>() {
        ExecutorService executorService = Executors.newSingleThreadExecutor(new CustomizableThreadFactory() {
          @Override
          public String getThreadNamePrefix() { return "pubon-"; }
        });
        public void onSubscribe(Subscription subscription) {sub.onSubscribe(subscription);}
        public void onNext(Integer item) { executorService.execute(()-> sub.onNext(item)); }
        public void onError(Throwable throwable) {executorService.execute( () -> sub.onError(throwable)); }
        public void onComplete() { executorService.execute(()-> sub.onComplete());}
      });
    };
    Subscriber<Integer> subscriber = new Subscriber<Integer>() {
      public void onSubscribe(Subscription subscription) {log.info("onSubscribe"); subscription.request(Long.MAX_VALUE);};
      public void onNext(Integer item) {log.info("onNext: " + item);};
      public void onError(Throwable throwable) {log.info("onError: " + throwable.getMessage());};
      public void onComplete() {log.info("onComplete");};
    };
    pubOnPub.subscribe(subscriber);
    ```
    ```java
    Flux.range(1, 5)
      .log()
      .subscribeOn(Schedulers.newSingle("subOn-"))
      .publishOn(Schedulers.newSingle("pubOn-"))
      .subscribe(item -> log.info("onNext: " + item),
        throwable -> log.info("onComplete: " + throwable.getMessage()),
        () -> log.info("onComplete"));
    ```
    ```
    11:25:48.183 [subon-1] INFO com.example.demo.iterable.SchedulerController - onSubscribe
    11:25:48.190 [pubon-1] INFO com.example.demo.iterable.SchedulerController - onNext: 1
    11:25:48.190 [pubon-1] INFO com.example.demo.iterable.SchedulerController - onNext: 2
    11:25:48.190 [pubon-1] INFO com.example.demo.iterable.SchedulerController - onNext: 3
    11:25:48.190 [pubon-1] INFO com.example.demo.iterable.SchedulerController - onNext: 4
    11:25:48.190 [pubon-1] INFO com.example.demo.iterable.SchedulerController - onNext: 5
    11:25:48.194 [pubon-1] INFO com.example.demo.iterable.SchedulerController - onComplete
    ```
    - onSubscribe, request에서는 subscribeOn에서 생성한 스레드, onNext, onError, onComplete는 publishOn에서 생성한 스레드로 실행된다.

#### interval

```java
Flux.interval(Duration.ofMillis(200L))
  .take(5)
  .subscribe(item -> log.info("onNext: " + item));
		
Thread.sleep(1200);
```
```
13:21:43.819 [main] DEBUG reactor.util.Loggers - Using Slf4j logging framework
13:21:44.066 [parallel-1] INFO com.example.demo.iterable.SchedulerController - onNext: 0
13:21:44.267 [parallel-1] INFO com.example.demo.iterable.SchedulerController - onNext: 1
13:21:44.454 [parallel-1] INFO com.example.demo.iterable.SchedulerController - onNext: 2
13:21:44.656 [parallel-1] INFO com.example.demo.iterable.SchedulerController - onNext: 3
13:21:44.858 [parallel-1] INFO com.example.demo.iterable.SchedulerController - onNext: 4
```
- interval에 설정한 주기마다 데이터를 전송한다.
- 주의할 점은 메인메서드가 해당 주기 이상 살아있어야 한다.
- 쓰레드를 살펴보면 메인쓰레드가 아닌 다른 쓰레드에서 실행되는 것을 확인할 수 있다.
- Publisher와 Subscriber 로 구현해보기
  ```java
  // ExecutorService es = Executors.newSingleTrheadExecutor();
  // es.execute(() -> sub.onNex(i);
  ScheduledExecutorService exec = Executors.newSingleThreadScheduledExecutor();
  exec.scheduleAtFixedRate( () -> {
    sub.onNext(i);
  }, 0, 300, TimeUnit.MILLISECONDS);
  ```
  - Subscription에서 request 메서드를 수정하였다.
    - ExecutorService 대신 ScheduledExecutorService를 사용했다.
    - scheduleAtFixedRate를 사용하여 지정된 기간만큼 대기 후에 전송한다.

#### Scheduler의 메서드 정리
    
#### 출처

- 토비의 봄 TV 7회 스프링 리액티브 프로그래밍(3) - Reactive Streams - Scheduler
