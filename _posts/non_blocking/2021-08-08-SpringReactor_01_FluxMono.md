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
    - just 내에 들어가는 값들로 Flux를 생성한다.
  - range
    - int 범위를 지정하여 순차적인 데이터를 생성한다.
  - fromArray, fromIterable, fromStream
    - fromArray: 이미 생성되어 있는 Array의 데이터를 사용하여 Flux를 생성한다.
    - fromIterable: 이미 생성되어 있는 Collections 구현체와 같은 Iterable의 데이터를 사용하여 Flux를 생성한다.
    - fromStream: stream을 통하여 Flux를 생성할 수 있다.
  - empty
    - 아무값도 전달하지 않는 빈데이터의 Flux를 만들 수 있다.
  - create
    - create를 사용하면 Subscriber의 요청과 상관없이 비동기로 데이터를 발생할 수 있다.
    - Consumer<? super FluxSink<T>> emitter를 파라미터로 받는 데, Consumer는 람다이고, FluxSink를 살펴보자.
      - FluxSink: 0/1개의 onError/onComplete 이벤트 다음에 임의의 수의 다음 신호를 방출하기 위한 다운스트림 가입자
        ```java

        ```


- 


#### Mono

![image](https://user-images.githubusercontent.com/42403023/128662145-06e5d567-87a8-4ef9-adaf-9ceca39aa77a.png)

** 이미지 출처: https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html

- Mono는 Publisher의 구현체로 0~1개의 데이터를 전달할 수 있다.
  - 데이터를 전달할 때마다 onNext 이벤트가 발생
  - 모든 데이터의 전달 처리가 완료되면 onComplete 이벤트가 발생
  - 데이터 전달 과정에서 오류가 발생하면 onError 이벤트가 발생

- Mono를 생성하는 방법
  - just
    - just 내에 들어가는 값들로 Mono를 생성한다.
  - empty
    - 아무값도 전달하지 않는 빈데이터의 Mono를 만들 수 있다.

#### Lifecycle hooks

#### 출처

- https://brunch.co.kr/@springboot/154
- https://projectreactor.io/docs/core/release/api/overview-summary.html
- https://javacan.tistory.com/tag/Flux.fromStream
- https://projectreactor.io/docs/core/release/api/reactor/core/publisher/FluxSink.html
