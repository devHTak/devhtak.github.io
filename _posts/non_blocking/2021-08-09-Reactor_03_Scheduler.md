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
- Schedulers.parallel()를 사용해서 신호를 주기적으로 발생한다. 다른 스케줄러를 사용하고 싶다면 interval(Duration, Scheduler) 메서드를 사용하면 된다.
- 주의할 점은 메인메서드가 해당 주기 이상 살아있어야 한다.
- 쓰레드를 살펴보면 메인쓰레드가 아닌 다른 쓰레드에서 실행되는 것을 확인할 수 있다.
- Publisher와 Subscriber 로 구현해보기
  - subscription
    ```java
    new Subscription() {
      int no = 0;
      volatile boolean cancelled = false;
      public void request(long n) {
        // ExecutorService es = Executors.newSingleTrheadExecutor();
        // es.execute(() -> sub.onNex(i);
        ScheduledExecutorService exec = Executors.newSingleThreadScheduledExecutor();
        exec.scheduleAtFixedRate( () -> {
          if(cancelled) {
            exec.shutdown();
            return;
          }
          sub.onNext(i);
        }, 0, 300, TimeUnit.MILLISECONDS);
      }
      
      public void cancel() {
        cancelled = true;
      }
    }  
    ```
    - ExecutorService 대신 ScheduledExecutorService를 사용했다.
    - scheduleAtFixedRate를 사용하여 지정된 기간만큼 대기 후에 전송한다.
  - Subscriber 수정
    ```java
    Subscriber<Integer> subscriber = new Subscriber<Integer>() {
      int count = 0;
      Subscription subsc;
      public void onSubscribe(Subscription subscription) {
        log.info("onSubscribe"); 
        subsc = subscription;
        subscription.request(Long.MAX_VALUE);
      };
      public void onNext(Integer item) {
        log.info("onNext: " + item);
        if(++count >= 5) {
          subscription.cancel();
        }
      };
      public void onError(Throwable throwable) {log.info("onError: " + throwable.getMessage());};
      public void onComplete() {log.info("onComplete");};
    };
    ```
    - subscription의 cancel을 호출하여 중단해준다

#### Schedulers의 메서드 정리

- immediate()
  - 현재 쓰레드에서 실행한다.

- single()
  - 쓰레드가 한 개인 쓰레드 풀을 이용해서 실행한다. 즉 한 쓰레드를 공유한다.
    
- parallel()
  - 고정 크기 쓰레드 풀을 이용해서 실행한다. 병렬 작업에 적합하다.
  - ExecutorService기반으로 단일 스레드 고정 크기(Fixed) 스레드 풀을 사용하여 병렬 작업에 적합함.

- elastic()
  - 쓰레드 풀을 이용해서 실행한다. 블로킹 IO를 리액터로 처리할 때 적합하다. 쓰레드가 필요하면 새로 생성하고 일정 시간(기본 60초) 이상 유휴 상태인 쓰레드는 제거한다. 데몬 쓰레드를 생성한다.
  - 스레드 갯수는 무한정으로 증가할 수 있고 수행시간이 오래걸리는 블로킹 작업에 대한 대안으로 사용할 수 있게 최적화 되어있다.

- single, parallel, elastic
  - 매번 새로운 쓰레드 풀을 만들지 않고 동일한 쓰레드 풀을 리턴한다.
  - 해당 메서드가 생성하는 쓰레드는 데몬 쓰레드로서 main 쓰레드가 종료되면 함께 종료된다.

- newSingle(), newParallel(), newElastic()
  - 같은 종류의 쓰레드 풀인데 새로 생성하고 싶다면 해당 메서드를 사용하면 된다.
  - 기본으로 데몬 쓰레드가 아니기 때문에 어플리케이션 종료시에는 다음과 같이 dispose() 메서드를 호출해서 쓰레드를 종료시켜 주어야 한다. 그렇지 않으면 어플리케이션이 종료되지 않는 문제가 발생할 수 있다.
  - 파라미터
    - name : 쓰레드 이름으로 사용할 접두사이다.
    - daemon : 데몬 쓰레드 여부를 지정한다. 지정하지 않으면 false이다. 데몬 쓰레드가 아닌 경우 JVM 종료시에 생성한 스케줄러의 dispose()를 호출해서 풀에 있는 쓰레드를 종료해야 한다.
    - ttlSeconds : elastic 쓰레드 풀의 쓰레드 유휴 시간을 지정한다. 지정하지 않으면 60(초)이다.
    - parallelism : 작업 쓰레드 개수를 지정한다. 지정하지 않으면 Runtime.getRuntime().availableProcessors()이 리턴한 값을 사용한다.

- 예시
  ```java
  Flux.range(0, 5)
    .subscribeOn(Schedulers.newParallel("SUB1"))
    .log()
    .map(item -> item * 10)
    .publishOn(Schedulers.newParallel("PUB1"))
    .log()
    .subscribe(item -> log.info("first onNext:" + item));
  ```
  ```
  20:41:30.064 [main] DEBUG reactor.util.Loggers - Using Slf4j logging framework
  20:41:30.124 [main] INFO reactor.Flux.SubscribeOn.1 - onSubscribe(FluxSubscribeOn.SubscribeOnSubscriber)
  20:41:30.130 [main] INFO reactor.Flux.PublishOn.2 - | onSubscribe([Fuseable] FluxPublishOn.PublishOnSubscriber)
  20:41:30.131 [main] INFO reactor.Flux.PublishOn.2 - | request(unbounded)
  20:41:30.135 [main] INFO reactor.Flux.SubscribeOn.1 - request(256)
  20:41:30.141 [SUB1-2] INFO reactor.Flux.SubscribeOn.1 - onNext(0)
  20:41:30.142 [SUB1-2] INFO reactor.Flux.SubscribeOn.1 - onNext(1)
  20:41:30.143 [SUB1-2] INFO reactor.Flux.SubscribeOn.1 - onNext(2)
  20:41:30.143 [SUB1-2] INFO reactor.Flux.SubscribeOn.1 - onNext(3)
  20:41:30.143 [SUB1-2] INFO reactor.Flux.SubscribeOn.1 - onNext(4)
  20:41:30.148 [PUB1-1] INFO reactor.Flux.PublishOn.2 - | onNext(0)
  20:41:30.148 [PUB1-1] INFO com.example.demo.iterable.SchedulerController - first onNext:0
  20:41:30.148 [PUB1-1] INFO reactor.Flux.PublishOn.2 - | onNext(10)
  20:41:30.148 [PUB1-1] INFO com.example.demo.iterable.SchedulerController - first onNext:10
  20:41:30.149 [PUB1-1] INFO reactor.Flux.PublishOn.2 - | onNext(20)
  20:41:30.149 [PUB1-1] INFO com.example.demo.iterable.SchedulerController - first onNext:20
  20:41:30.149 [PUB1-1] INFO reactor.Flux.PublishOn.2 - | onNext(30)
  20:41:30.149 [PUB1-1] INFO com.example.demo.iterable.SchedulerController - first onNext:30
  20:41:30.149 [PUB1-1] INFO reactor.Flux.PublishOn.2 - | onNext(40)
  20:41:30.149 [PUB1-1] INFO com.example.demo.iterable.SchedulerController - first onNext:40
  20:41:30.153 [SUB1-2] INFO reactor.Flux.SubscribeOn.1 - onComplete()
  20:41:30.154 [PUB1-1] INFO reactor.Flux.PublishOn.2 - | onComplete()
  ``` 

#### 출처

- 토비의 봄 TV 7회 스프링 리액티브 프로그래밍(3) - Reactive Streams - Scheduler
- https://javacan.tistory.com/entry/Reactor-Start-6-Thread-Scheduling
