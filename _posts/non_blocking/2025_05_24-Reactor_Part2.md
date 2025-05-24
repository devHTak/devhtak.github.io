### Project Reactor

#### 개요
- Reactor 는 Spring Framework 주도하에 개발된 리액티브 스트림즈 구현체
- spring-boot-starter-webflux 내부에 Reactor Core 라이브러리가 포함되어 있다.
- 특징
  - Reactive Streams
  - Non Blocking
  - Java's Functional API
  - Flux/Mono
  - Well suited for MSA - Reactor는 마이크로서비스에서 수맣은 서비스들 간에 지속적으로 발생하는 I/O 처리에 적합
  - Backpressure-ready Network
- Hello Reactor
  ```java
  Flux<String> sequence = Flux.just("Hello", "Reactor");
  sequence.map(data -> data.toLowCase())
        .subscribe(System.out::println);
  ```
  - Flux 는 Reactor에서 Publisher 역할, subscribe에 들어간 인자가 Subscriber 역할을 한다.
  - just, map은 operator 메소드로 just는 데이터를 생성하여 제공, map은 전달받은 데이터를 가공한다.
  - Reactor는 데이터를 생성하여 제공(1단계), 가공(2단계), 전달받은 데이터 처리(3단계)로 구성
- 마블다이어그램
  - 비동기적인 데이터 흐름을 시간의 흐름에 따라 시각적으로 표시한 다이어그램
  - Publisher의 타임라인과 Subscriber 타임라인 중간에 operator에 대해 표시하며 marvel(구슬)은 데이터를 의미

#### Cold Sequence, Hot Sequence
- Cold란 새로 시작한다는 의미, Hot은 새로 시작하지 않는다.
- Cold Sequence
  ```java
  Flux<String> coldFlux = Flux.fromIterable(Arrays.asList("KOREA", "JAPAN", "CHINESE"))
                            .map(String::toLowCase);
  coldFlux.subscribe(System.out::println); // KOREA, JAPAN, CHINESE 출려
  System.out.println("--------------");
  Thread.sleep(2000L);
  coldFlux.subscribe(System.out::println); // KOREA, JAPAN, CHINESE 출려
  ```
  - Subscriber가 구독할 떄마다 데이터 흐름이 처음부터 다시 시작
    - 구독이 발생할 때마다 emit된 데이터를 처음부터 다시 전달
  - 결과적으로 Publisher Sequence 타임라인이 구독할때마다 하나씩 더 생긴다.
- Hot Sequence
  ```java
  String[] singers = {"A", "B", "C", "D", "E"};
  Flux<String> concertFlux = Flux.fromArray(singers)
      .delayElements(Duration.ofSeconds(1)) // 각 데이터의 emit을 일정 시간 지연시킨다.
      .share(); // cold sequence를 hot sequence로 변환

  concertFlux.subscribe(System.out.println); // A ~ E 출력
  Thread.sleep(2500);
  concertFlux.subscribe(System.out.println); // C ~ E 출력
  Thread.sleep(3000);
  ```
  - 구독이 발생한 시점 이전에 emit된 데이터는 Subscriber가 전달받지 못하고 구독이 발생한 이후 emit된 데이터만 전달 받는다.
  - 구독이 여러번 발생해도 타임라인은 하나밖에 생성되지 않는다.
- HTTP 요청/응답에서 Cold, Hot Sequence 동작 흐름

#### Backpressure

