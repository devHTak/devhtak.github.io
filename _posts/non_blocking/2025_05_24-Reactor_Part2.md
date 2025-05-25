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
  ```java
  public static main(String[] args) throws InterruptedException {
      // Cold Sequence
      Mono<String> coldMono = getWordTime(uri);
      coldMono.subscribe(dateTime -> log.info("# dateTime1: {}". dateTime);
      Thread.sleep(2000);
      coldMono.subscribe(dateTime -> log.info("# dateTime2: {}". dateTime);
      Thread.sleep(2000);

      // Hot Sequence
      Mono<String> hotMono = getWordTime(uri).cache();
      hotMono.subscribe(dateTime -> log.info("# dateTime1: {}". dateTime);
      Thread.sleep(2000);
      hotMono.subscribe(dateTime -> log.info("# dateTime2: {}". dateTime);
      Thread.sleep(2000);
  }
  private static Mono<String> getWordTime(URI wordTimeUri) {
      return webClient.create()
                .get().uri(worldTimeUri).retrieve()
                .bodyToMono(String.class)
                .ma(response -> {
                    DocumentContext jsonContext = JsonPath.parse(response);
                    return jsonContext.read("$.datetime");
                })
  }
  ```
  - Cold Sequence 동작 방식
    - 구독이 발생할 때마다 데이터의 emit 과정이 처음부터 새로 시작되는 Cold Sequence 특징으로 인해 두번의 구독이 발생했기 때문에 두번의 HTTP 요청이 이뤄지고 2초 정도 차이나는 응답을 확인할 수 있다.
  - Hot Sequence 동작 방식
    - cache operator
      - hot source(sequence)로 병환하며 마지막 emit된 데이터에 대해 캐시한뒤, 구독이 발생할 때마다 캐시된 데이터를 전달
    - Subscriber의 최초 구독이 발생해야 Publisher가 데이터를 emit 하는 warm-up 과 subscriber의 구독 여부와 상관없이 데이터를 emit 하는 Hot으로 구분할 수 있다.
   
#### Backpressure
- Backpressure란
  - 배압이란 뜻으로 Publisher가 끊임없이 emit 하는 무수한 데이터를 적절하게 제어하여 데이터 처리에 과부하가 발생하지 않도록 제어 하는 것
- Reactor에서 Backpressure 처리 방식
  - 데이터 개수 제어: subscriber가 처리할 수 있는 수준의 적절한 데이터 개수 요청
    ```java
    Flux.range(1, 5)
        .doOnRequest(data -> log.info("# doOnRequest: {}", data))
        .subscribe(new BaseSubscriver<Integer>() {
            @Override
            protected void hookOnSubscribe(Subscription subscription) {
                request(1);
            }
            @SneakyThrows
            @Override
            protected void hookOnNext(Integer value) {
                Thread.sleep(2000L);
                log.info("# hookOnNext: {}", value);
                request(1);
            }
        });
    ```
    - hookOnSubscribe: onSubscribe() 대신하여 구독 시점에 request() 호출하여 최초 데이터 요청 개수 제어
    - hookOnNext: onNext 메서드를 대신하여 emit 한 데이터를 처리한 후 Publisher에게 다시 데이터를 요청하는 역할로 request() 로 데이터 요청 개수 제어
  - Backpressure 전략 사용
    - IGNORE 전략: Backpressure 적용하지 않는다. (IllegalStateException 발생)
    - ERROR 전략: Downstream으로 전달할 데이터가 버퍼에 가득 찰 경우 예외 발생 (IllegalStateException 발생)
    - DROP 전략: Downstream으로 전달할 데이터가 버퍼에 가득 찰 경우 버퍼 밖에서 대기하는 먼저 emit된 데이터부터 drop
    - LATEST 전략: Downstream으로 전달할 데이터가 버퍼에 가득 찰 경우 버퍼 밖에서 대기하는 가장 최근에 emit된 데이터부터 버퍼에 채우는 전략
    - BUFFER 전략: Downstream으로 전달할 데이터가 버퍼에 가득찰 경우 버퍼 안에 있는 데이터부터 drop 시키는 전략
      - onBackpressureBuffer(int buffer_size, Consume<T> con, BufferOverflowStrategy strategy)
      - DROP_LATEST: 가장 최근에 버퍼안에 채워진 데이터를 drop하여 확보된 공간에 emit된 데이터를 채우는 전략
      - DROP_OLDEST: 버퍼 안에 채워진 데이터 중 가장 오래된 데이터를 drop 한 후, 확보된 공간에 emit된 데이터를 새우는 정략
     
#### Sinks
- Sinks 란?
  - Reactor에선 Processor 인터페이스를 구현한 구현 클래스인 FluxProcessor, MonoProcessor, EmitterProcessor 등을 지원
  - Sinks가 Reactor 3.4.0에 등장하였으며 Processor는 Reactor 3.5.0부터 완전히 제거될 예정
  - Sinks는 리액티브 스트림즈의 Signal을 프로그래밍 방식으로 푸쉬할 수 있는 구조이며 Flux, Mono의 의미 체계를 갖는다.
    - Flux, Mono는 onNext같은 Signal을 내부적으로 전송해주는 방식이었는데, Sinks를 사용하면 프로그래밍 코드를 통해 명시적으로 Signal을 전송할 수 있다.
    - generate, create operator는 싱글스레드 기반에서 Signal을 전송하는 반면 Sinks는 멀티 스레드 방식으로 Signal을 전송해도 스레드 안정성을 보장하기 때문에 예기치 않은 동작으로 이어지는 것을 방지
  ```java
  Sinks.Many<String> unicastSink = Sinks.many().unicast().onBacpressureBuffer();
  Flux<String> flux = unicastSink.asFlux();
  IntStream.range(1, 10)
  .forEach(n -> {
      try {
          new Thread(() -> {
              unicastSink.emitNext(doTask(n), Sinks.EmitFailureHandler.FAIL_FAST);
              log.info("# emitted: {}", n);
          }).start();
      } catch(InterruptedException e) {
          log.error(e.getMessage();
      }
  });
  flux.publishOn(Schedulers.parallel())
      .map(result -> result + " success")
      .doOnNext(n -> log.info("# map: {}", n)
      .publishOn(Schedulers.parallel())
      .subscribe(data -> log.info("# onNext: {}", data);

  Thread.sleep(200L);
  ```
  - doTask() 메서드가 루프를 돌때마다 새로운 스레드에서 실행
- Sink 종류 및 특징
  - Sinks.One, Sinks.Many를 사용하여 전송할 수 있다.
  - Sinks.One은 한건의 데이터를 프로그래밍 방식으로 emit하는 역할을 하기도 하고, Mono 방식으로 Subscriber가 데이터를 소비할 수 있도록 해주는 Sinks 클래스 내부에서 인터페이스로 정의된 Sinks의 스펙 또는 사양으로 볼 수 있다.
    - EmitFailureHandler객체를 통해서 emit 도중 발생한 에러를 실패 처리하며 에러가 발생했을 때 재시도를 하지 않고 즉시 실패 처리 한다.
    ```java
    public interface EmitFailuerHandler {
        EmitFailureHandler FAIL_FAST = (signalType, emission) -> false;
        boolean onEmitFailure(SignalType signalType, EmitResult emitResult);
    }
    ```
  - Sinks.Many
    - Sinks.Many는 ManySpec을 리턴하며 UnicatSpec, MulticastSpec, MulticastRelpaySpec 를 리턴하는 형태의 추상메소드를 갖는 인터페이스이다.
    - UnicastSpec
      - onBackpressureBuffer 메서드를 호출하여 사용할 수 있다.
      - unicast는 하나의 특정 시스템만 정보를 전달받는 방식으로 단 하나의 Subscriber에게만 데이터를 emit
      - 두번째 subscribe를 호출하면 IllegalStateException 발생
    - MulticastSpec
      - onBackpressureBuffer 메서드를 호출하여 사용할 수 있다.
      - 하나 이상의 Subscriber에게 데이터를 emit
    - MulticaspReplaySpec
      - emit된 데이터 중에서 특정 시점으로 되돌린 데이터부터 emit

#### Scheduler
- 스레드의 개념 이해
- Scheduler란?
- Scheduler를 위한 전용 Operator
- publishOn, subscribeOn의 동작 이해
- Scheduler 종류
