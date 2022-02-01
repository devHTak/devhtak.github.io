---
layout: post
title: Reactor, Flux와 Mono
summary: Reactive Programming
author: devhtak
date: '2021-08-08 21:41:00 +0900'
category: Reactive
---

#### Reactor core

- Reactor core는 Reactor3의 핵심 모듈
- Java 7 버전 이하에서는 reactor 1, 2 사용이 가능하나 Java 8 버전, reactor 3 사용 권장
- dependency
  ```
  <!-- https://mvnrepository.com/artifact/io.projectreactor/reactor-core -->
  <dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-core</artifactId>
    <version>3.3.18.RELEASE</version>
  </dependency>
  ```
  - 만약 프로젝트에 spring-webflux가 추가되어 있으면 reactor-core를 추가할 필요가 없다.

#### Flux와 Mono

- Flux와 Mono는 Publisher 구현체며 전송하는 데이터의 갯수의 차이가 존재한다.
- Flux: 0~N개의 데이터 전달
  - public abstract class Flux<T> implements CorePublisher<T>
  - public abstract void subscribe(CoreSubscriber<? super T> actual);
    - subscribe 메소드를 추상 메소드로 지정하고 있다.
- Mono: 0~1개의 데이터 전달
  - public abstract class Mono<T> implements CorePublisher<T>
  - public abstract void subscribe(CoreSubscriber<? super T> actual);

#### Flux

![image](https://user-images.githubusercontent.com/42403023/128662109-340c04be-e4bc-465f-9f42-6582a535f122.png)

** 이미지 출처: https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html

- Flux는 Publisher의 구현체로 0~N개의 데이터를 전달할 수 있다.
  - 데이터를 전달할 때마다 onNext 이벤트가 발생
  - 모든 데이터의 전달 처리가 완료되면 onComplete 이벤트가 발생
  - 데이터 전달 과정에서 오류가 발생하면 onError 이벤트가 발생

- Flux를 생성하는 방법
  - just
    ```java
    Flux<String> flux = Flux.jst("Hello", "Reactor", "Flux", "Mono");
    ```
    - just 내에 들어가는 값들로 Flux를 생성한다.
  
  - range
    ```java
    Flux<Integer> flux = Flux.range(0, 10);
    ```
    - int 범위를 지정하여 순차적인 데이터를 생성한다.
  
  - fromArray, fromIterable, fromStream
    - fromArray: 이미 생성되어 있는 Array의 데이터를 사용하여 Flux를 생성한다.
    - fromIterable: 이미 생성되어 있는 Collections 구현체와 같은 Iterable의 데이터를 사용하여 Flux를 생성한다.
    - fromStream: stream을 통하여 Flux를 생성할 수 있다.

  - empty
    - 아무값도 전달하지 않는 빈데이터의 Flux를 만들 수 있다.

  - generate
    ```java
    Flux<Integer> flux = Flux.generate(sink -> { // SynchronousSink
      sink.onRequest(request -> {
        // request는 subscriber가 요청한 갯수
        int data = rand.nextInt(100) + 1;
        sink.next(data);
        if(emitCount == request) {
          sink.complete();
        }
      });
    });
    ```
    - 동기 방식으로 한번에 1개의 데이터를 생성할 때 사용한다.
    - subscriber로부터 요청이 왔을 때 신호를 생성하며, 실행할 때 인자로 SynchronousSink로 전달한다.
    - next(), complete, error 신호를 발생하여 1번에 1개의 next() 신호만을 발생할 수 있다.

  - create
    ```java
    Flux<Integer> flux = Flux.create(sink -> { // FluxSink
      sink.onRequest(request -> {
        // request는 subscriber가 요청한 갯수
        for(int i = 0; i < request; i++) {
          sink.next(i);
        }
        if(emitCount == request) {
          sink.complete();
        }
      });
    });
    ```
    - create를 사용하면 Subscriber의 요청과 상관없이 비동기로 데이터를 발생할 수 있다.
    - Consumer<? super FluxSink<T>> emitter를 파라미터로 받는 데, Consumer는 람다이고, FluxSink를 살펴보자.
      - FluxSink: 0/1개의 onError/onComplete 이벤트 다음에 임의의 수의 다음 신호를 방출하기 위한 다운스트림 가입자
        
      - generate와 달리 한번에 한 개 이상의 next() 신호를 발생할 수 있다.
      - 해당 차이는 배압(backpressure)를 일으키며 이에 대한 전략을 설정해주어야 한다.
      - 배압전략 1. IGNORE : Subscriber의 요청 무시하고 발생(Subscriber의 큐가 다 차면 IllegalStateException 발생)
      - 배압전략 2. ERROR : 익셉션(IllegalStateException) 발생
      - 배압전략 3. DROP : Subscriber가 데이터를 받을 준비가 안 되어 있으면 데이터 발생 누락
      - 배압전략 4. LATEST : 마지막 신호만 Subscriber에 전달
      - 배압전략 5. BUFFER : 버퍼에 저장했다가 Subscriber 요청시 전달. 버퍼 제한이 없으므로 OutOfMemoryError 발생 가능

- Flux 실행 순서

```java
Flux<Integer> flux = Flux.range(0, 10).filter(i -> i % 3 == 0);
flux.subscribe(item->System.out.println("onNext: " + item),
  e -> System.out.println("onError: " + e.getMessage()),
  () -> System.out.println("onComplete")
);
```
- 출력
  ```
  onNext: 0
  onNext: 3
  onNext: 6
  onNext: 9
  onComplete
  ```
- 실행 순서
  - range를 통해 Flux를 생성한다. filter를 통해 3의 배수만 전송할 수 있도록 한다.
  - subscribe를 하기 전까지는 데이터를 전달하지 않는다.
  - subscribe() 메서드를 실행하면, Subscriber가 등록되고 데이터를 전송해달라는 요청을 보낸다.
    - Publisher의 subscribe를 호출할 때 인자로 받은 Subscriber객체의 onSubscribe 메서드가 실행되며, 이 때 인자로 받은 Subscription을 활용한다(request, cancel)
  - 데이터를 전달하면서 onNext() 메서드가 실행되고 만약 오류가 발생하면 onError() 메서드가 실행, 정상 완료되면 onComplete() 메서드가 실행된다.

#### Flux 특징과 활용방식

- Mono : 데이터를 한개만 보낼 때 사용
- Flux: 여러개의 데이터를 보낼 때 사용한다.
  ```java
  @Data 
  @AllArgsConstructor
  public class Event {
  	private long id;
	private String value;
  }
  ```
  ```java
  @RestController
  @Slf4j
  public class MyController {
  	@GetMapping("/event/{id}")
  	public Mono<Event> helloMono(@PathVariable Long id) {
  		return Mono.just(new Event(id, "event: " + id));
  	}
  	@GetMapping("/events")
  	public Flux<Event> helloFlux() {
  		return Flux.just(new Event(1, "event: 1"), new Event(2, "event: 2"));
  	}
  }
  ```

- Mono에 단일 데이터를 Collection으로 넣어서 여러개의 데이터를 전달할 수 있지만, Flux의 fromIterable을 사용하면, Stream 형태에 데이터를 가공할 수 있다.
  - Mono로 Collection을 보내면 한번에 보낸다, Flux는 Iterable하게 데이터를 순차적으로 보낼 수 있다.
  ```java
  @RestController
  @Slf4j
  public class MyController {
  	@GetMapping("/event/{id}")
  	public Mono<List<Event>> helloMono(@PathVariable Long id) {
  		List<Event> list = Arrays.asList(new Event(1, "event: 1"), new Event(2, "event: 2"));
  		return Mono.just(list);
  	}
  	@GetMapping(value = "/events", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
  	public Flux<Event> helloFlux() {
  		List<Event> list = Arrays.asList(new Event(1, "event: 1"), new Event(2, "event: 2"));
  		return Flux.fromIterable(list).filter(event -> event.getId() % 2 == 0);
  	}
  }
  ```
  - text/event-stream: event stream 타입으로 "data: " 형식의 prefix가 붙는다. SSE(Server Sent Event) 참고
  - fromIterable

    ![image](https://user-images.githubusercontent.com/42403023/132988690-21f3e0e6-2dee-438f-9eb2-9a9c89c1ec5f.png)
    
    - 이미지 출처: https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html

  - fromStream, take
    ```java
    @GetMapping(value = "/events", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<Event> helloFlux() {
    	return Flux
    		.fromStream(Stream.generate(() -> new Event(System.currentTimeMillis(), "events")))
    		.delayElements(Duration.ofSeconds(1))
    		.take(10);
    }
    ```
    - take: 지정된 숫자만큼 데이터를 보낸 후 cancel()을 호출한다.
    - delayElements: 지정된 시간만큼 대기 후 데이터 전달
      - 메서드를 생성하는 쓰레드가 10초를 대기하는 것이 아닌 delay하는 쓰레드가 새롭게 생성되어 delay한다.

  - 두개의 Flux 묶기
    ```java
    @GetMapping(value = "/events", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<Event> helloFlux() {
    	Flux<Event> f = Flux.<Event, Long>generate(() -> 1L, (id, sink) -> {
    	sink.next(new Event(id, "events: " + id));
    		return id + 1;
    	});
    	Flux<Long> interval = Flux.interval(Duration.ofSeconds(1));
    	return Flux.zip(f, interval).map(tu -> tu.getT1());
    }
    ```
    - interval은 정해진 시간동안 대기 후 데이터를 전달한다.

#### Mono

![image](https://user-images.githubusercontent.com/42403023/128662145-06e5d567-87a8-4ef9-adaf-9ceca39aa77a.png)

** 이미지 출처: https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html

- Mono는 Publisher의 구현체로 0~1개의 데이터를 전달할 수 있다.
  - 데이터를 전달할 때마다 onNext 이벤트가 발생
  - 모든 데이터의 전달 처리가 완료되면 onComplete 이벤트가 발생
  - 데이터 전달 과정에서 오류가 발생하면 onError 이벤트가 발생

- Mono를 생성하는 방법
  - just
    ```java
    Mono<Integer> mono = Mono.just(10).filter(i -> i % 3 != 0);
    mono.subscribe(item -> System.out.println("onNext: " + item),
      throwable -> System.out.println("onError: " + throwable.getMessage()),
      () -> System.out.println("onComplete")
    );
    ```
    - just 내에 들어가는 값으로 Mono를 생성한다.
  - empty
    - 아무값도 전달하지 않는 빈데이터의 Mono를 만들 수 있다.

#### Mono의 동작방식과 block()

- NonBlocking과 Sync
  ```java
  @GetMapping("/")
  public Mono<String> hello() {
      log.info("pos0");
      // Publisher -> (Publisher) -> (Publisher) -> Subscriber
      Mono m = Mono.just("Hello WebFlux").log(); 
      log.info("pos1");
      return m;
  }
  ```
  ```
  2021-09-08 20:45:10.081  INFO 38476 --- [ctor-http-nio-2] com.example.demo.mono.MyController       : pos0
  2021-09-08 20:45:10.082  INFO 38476 --- [ctor-http-nio-2] com.example.demo.mono.MyController       : pos1
  2021-09-08 20:45:10.094  INFO 38476 --- [ctor-http-nio-2] reactor.Mono.Just.1                      : | onSubscribe([Synchronous Fuseable] Operators.ScalarSubscription)
  2021-09-08 20:45:10.096  INFO 38476 --- [ctor-http-nio-2] reactor.Mono.Just.1                      : | request(unbounded)
  2021-09-08 20:45:10.097  INFO 38476 --- [ctor-http-nio-2] reactor.Mono.Just.1                      : | onNext(Hello WebFlux)
  2021-09-08 20:45:10.100  INFO 38476 --- [ctor-http-nio-2] reactor.Mono.Just.1                      : | onComplete()
  ```
  - Subscriber는 Spring이 자동으로 걸어주게 되어 있다.
  - pos0 다음에 Mono에 대한 로그, pos1 로그가 찍히는 것이 아닌, pos0, pos1 후에 Mono에 대한 로그가 찍혔다.
    - Spring에서 Subscriber가 subscribe 를 할 때 Mono가 실행되기 때문이다.
  - just에 service를 호출한다면 service에 메소드가 먼저 실행된 후 결과값이 just에 들어간다.
    ```java
    @GetMapping("/")
    public Mono<String> hello() {
        log.info("pos0");
        // Publisher -> (Publisher) -> (Publisher) -> Subscriber
        Mono m = Mono.just(myService.findById(1L)).log(); 
        log.info("pos1");
        return m;
    }
    ```
    ```
    2021-09-08 20:51:47.814  INFO 18036 --- [ctor-http-nio-2] com.example.demo.mono.MyController       : pos0
    2021-09-08 20:51:47.814  INFO 18036 --- [ctor-http-nio-2] com.example.demo.mono.MyService          : service: 1
    2021-09-08 20:51:47.817  INFO 18036 --- [ctor-http-nio-2] com.example.demo.mono.MyController       : pos1
    2021-09-08 20:51:47.829  INFO 18036 --- [ctor-http-nio-2] reactor.Mono.Just.1                      : | onSubscribe([Synchronous Fuseable] Operators.ScalarSubscription)
    2021-09-08 20:51:47.830  INFO 18036 --- [ctor-http-nio-2] reactor.Mono.Just.1                      : | request(unbounded)
    2021-09-08 20:51:47.831  INFO 18036 --- [ctor-http-nio-2] reactor.Mono.Just.1                      : | onNext(1)
    2021-09-08 20:51:47.834  INFO 18036 --- [ctor-http-nio-2] reactor.Mono.Just.1                      : | onComplete()
    ```
    - just 내부에 있는 myService에 메소드도 미리 실행된다.


- NonBlocking과 Async
  ```java
  @GetMapping("/")
  public Mono<String> hello() {
  	log.info("pos0");
  	// Publisher -> (Publisher) -> (Publisher) -> Subscriber
  	Mono<String> m = Mono.fromSupplier(() -> myService.findById(1L)).log(); 
  	log.info("pos1");
  	return m;
  }
  ```
  ```
  2021-09-08 20:55:18.650  INFO 20448 --- [ctor-http-nio-3] com.example.demo.mono.MyController       : pos0
  2021-09-08 20:55:18.653  INFO 20448 --- [ctor-http-nio-3] com.example.demo.mono.MyController       : pos1
  2021-09-08 20:55:18.664  INFO 20448 --- [ctor-http-nio-3] reactor.Mono.Supplier.1                  : | onSubscribe([Fuseable] Operators.MonoSubscriber)
  2021-09-08 20:55:18.667  INFO 20448 --- [ctor-http-nio-3] reactor.Mono.Supplier.1                  : | request(unbounded)
  2021-09-08 20:55:18.668  INFO 20448 --- [ctor-http-nio-3] com.example.demo.mono.MyService          : service: 1
  2021-09-08 20:55:18.668  INFO 20448 --- [ctor-http-nio-3] reactor.Mono.Supplier.1                  : | onNext(1)
  2021-09-08 20:55:18.671  INFO 20448 --- [ctor-http-nio-3] reactor.Mono.Supplier.1                  : | onComplete()
  ```
  - fromSupplier를 통해 callback 형식으로 작성해 주었다.
  - 동작 순서를 보면 Subscribe된 상태에서 service가 실행되는 것을 확인할 수 있다.

- return하기 전에 subscribe() 호출
  ```java
  @GetMapping("/")
  public Mono<String> hello() {
  	log.info("pos0");
	// Publisher -> (Publisher) -> (Publisher) -> Subscriber
	Mono<String> m = Mono.fromSupplier(() -> myService.findById(1L))
	    		.doOnNext(c -> log.info(c))
	    		.log(); 
	    
	m.subscribe(); // 먼저 subscribe
	log.info("pos1");
	return m;
  }
  ```
  ```
  2021-09-08 21:01:59.063  INFO 31860 --- [ctor-http-nio-3] com.example.demo.mono.MyController       : pos0
  2021-09-08 21:01:59.067  INFO 31860 --- [ctor-http-nio-3] reactor.Mono.PeekFuseable.1              : | onSubscribe([Fuseable] FluxPeekFuseable.PeekFuseableSubscriber)
  2021-09-08 21:01:59.070  INFO 31860 --- [ctor-http-nio-3] reactor.Mono.PeekFuseable.1              : | request(unbounded)
  2021-09-08 21:01:59.070  INFO 31860 --- [ctor-http-nio-3] com.example.demo.mono.MyService          : service: 1
  2021-09-08 21:01:59.070  INFO 31860 --- [ctor-http-nio-3] com.example.demo.mono.MyController       : 1
  2021-09-08 21:01:59.070  INFO 31860 --- [ctor-http-nio-3] reactor.Mono.PeekFuseable.1              : | onNext(1)
  2021-09-08 21:01:59.071  INFO 31860 --- [ctor-http-nio-3] reactor.Mono.PeekFuseable.1              : | onComplete()
  2021-09-08 21:01:59.071  INFO 31860 --- [ctor-http-nio-3] com.example.demo.mono.MyController       : pos1
  2021-09-08 21:01:59.081  INFO 31860 --- [ctor-http-nio-3] reactor.Mono.PeekFuseable.1              : | onSubscribe([Fuseable] FluxPeekFuseable.PeekFuseableSubscriber)
  2021-09-08 21:01:59.081  INFO 31860 --- [ctor-http-nio-3] reactor.Mono.PeekFuseable.1              : | request(unbounded)
  2021-09-08 21:01:59.081  INFO 31860 --- [ctor-http-nio-3] com.example.demo.mono.MyService          : service: 1
  2021-09-08 21:01:59.081  INFO 31860 --- [ctor-http-nio-3] com.example.demo.mono.MyController       : 1
  2021-09-08 21:01:59.081  INFO 31860 --- [ctor-http-nio-3] reactor.Mono.PeekFuseable.1              : | onNext(1)
  2021-09-08 21:01:59.086  INFO 31860 --- [ctor-http-nio-3] reactor.Mono.PeekFuseable.1              : | onComplete()
  ```
  - pos0로그 후 subscribe()를 하기 Mono에 대한 로그가 나오고 pos1이 찍힌다. 그 다음 return으로 다시 Mono에 대한 subscribe가 진행된다.
  - Publisher는 1개이지만 여러개의 subscriber를 둘 수 있다.
  
- Hot 과 Cold
  - Cold 방식은 subscribe할 때마다 매번 독립적인 데이터를 발행한다.
  - Hot 방식은 subscribe 할때 마다, 새로운 데이터를 발행이나 동작하지 않는 방식이다. 
    - subscribe를 매번 할때마다 미리 생성해 둔 데이터로 동작을 한다. 그리고 subscribe 와 무관하게 즉시 데이터 발행과 동작도 가능한 방식이다.

- block()
  - publisher가 제공하는 결과값을 꺼내서 Mono나 Flux 같은 컨테이너를 제거하고 값을 넘겨주는 것이 목적.
  - block으로 값을 빼왔으면, return에서 다시 Mono를 호출해서 처음부터 끝까지 재 생성 작업을 거치지 말고, 결과값을 Mono.just() 로 감싸서 전달하는 것이 훨씬 효율적
  - Mono 작업들이 DB 조회, Api 요청 등 고 비용 작업인 경우를 생각해보면 명확함

#### 출처

- https://brunch.co.kr/@springboot/154
- https://projectreactor.io/docs/core/release/api/overview-summary.html
- https://javacan.tistory.com/tag/Flux.fromStream
- https://projectreactor.io/docs/core/release/api/reactor/core/publisher/FluxSink.html
- 토비의 봄 TV 13회 스프링 리액티프 프로그래밍(9) Mono의 동작방식과 block()
- 토비의 봄 TV 14회 스프링 리액티프 프로그래밍(10) Flux의 특징과 활용방식
