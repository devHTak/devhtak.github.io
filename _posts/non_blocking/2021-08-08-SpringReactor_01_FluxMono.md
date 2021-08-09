---
layout: post
title: Spring Reactor, Flux와 Mono
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
    Flux<Integer> flux = Flux.jst("Hello", "Reactor", "Flux", "Mono");
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

#### 출처

- https://brunch.co.kr/@springboot/154
- https://projectreactor.io/docs/core/release/api/overview-summary.html
- https://javacan.tistory.com/tag/Flux.fromStream
- https://projectreactor.io/docs/core/release/api/reactor/core/publisher/FluxSink.html
