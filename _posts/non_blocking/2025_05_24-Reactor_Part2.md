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
  - Scheduler 는 Reactor Sequence에서 사용되는 스레드 관리자 역할
  - 물리적인 스레드는 병렬성과 관련이 있으며 논리적인 스레드는 동시성과 관련이 있다.
- Scheduler란?
  - 운영체제 레벨에서 Scheduler는 실행되는 프로세스를 선택하고 실행하는 등 프로세스의 라이프사이클을 관리
  - Reactor에서도 Scheduler는 비동기 프로그래밍을 위해 사용되는 스레드를 관리해주는 역할
    - Ract Condition 등 생길 수 있는 문제를 최소화하여 간결한 코드, 스레드 제어에 대한 부담을 적게 해준다.
- Scheduler를 위한 전용 Operator
  - subscribeOn
    ```java
    Flux.fromArray(new Integer[] {1, 3, 5, 7})
        .subscribeOn(Schedulers.boundedElastic())
        .doOnNext(data -> log.info("# doOnNext: {}", data)
        .doOnSubscribe(subscription -> log.info("# doOnSubscription"))
        .subscribe(data -> log.info("#next: {}", data);
    ```
    - 구독이 발생한 직후에 원본 Publisher의 동작을 처리하기 위해 스레드 할당
    - doOnNext, doOnSubscribe는 데이터 emit, 구독 발생 시점에 처리 동작이며 스레드 확인 가능
    - doOnSubscribe는 main 스레드 동작, 그 외에는 Scheduler에 의해 새로운 스레드로 동작한다.
  - publishOn
    ```java
    Flux.fromArray(new Integer[] {1, 3, 5, 7})
        .doOnNext(data -> log.info("# doOnNext: {}", data)
        .doOnSubscribe(subscription -> log.info("# doOnSubscription"))
        .publishOn(Schedulers.parallel())
        .subscribe(data -> log.info("#next: {}", data);
    ```
    - Signal을 전송할 때 실행되는 스레드를 제어하는 역할로 Downstream의 실행 스레드 변경
    - doOnSubscribe, doOnNext는 메인 스레드에서 동작하며 onNext는 새로운 스레드에서 실행된다.
  - parallel()
    ```java
    Flux.fromArray(new Integer[] {1, 3, 5, 7})
        .parallel(4)
        .runOn(Schedulers.parallel())
        .subscribe(data -> log.info("#next: {}", data);
    ```
    - subscribeOn, publishOn은 동시성을 가지는 논리적인 스레드이지만 parallel 은 병렬성을 가지는 물리적인 스레드
      - parallel 의 인자로 실행될 스레드의 개수를 지정할 수 있다. (CPU 코어 개수만큼 병렬 처리)
    - 실제로 병렬 작업을 수행할 스레드 할당은 runOn() 에서 담당
- publishOn, subscribeOn의 동작 이해
  ```java
   Flux.fromArray(new Integer[] {1, 3, 5, 7})
        .publishOn(Schedulers.parallel()) // 하위에 새로운 스레드 생성
        .doOnNext(data -> log.info("# doOnNext: {}", data) 
        .filter(data -> data > 3) // A thread
        .doOnNext(data -> log.info("# doOnNext: {}", data)
        .publishOn(Schedulers.parallel()) // 하위에 새로운 스레드 생성
        .map(data -> data * 10);  // B thread
        .subscribe(data -> log.info("#next: {}", data); // B thread
  ```
  - 여러 operator 가 있을 때 publishOn을 호출하면 하위 downstream에 대해 새로운 스레드로 처리되며 Operator 체인상에 한개 이상 사용 가능하며 실행 목적에 맞게 적절하게 분리할 수 있다.
  - subscribe와 publishOn을 함께 사용하여 원본 Publisher에서 데이터를 emit하는 스레드와 emit된 데이터를 가공 처리하는 스레드를 적절하게 분리할 수 있다.
  - subscribeOn 체이상에 위치와 상관없이 구독 시점 직후, 즉 Publisher가 데이터를 emit하기 전에 실행 스레드를 변경
- Scheduler 종류
  - Schedulers.immediate()
    - 별도 스레드 생성없이 현재 스레드에서 작업을 처리하고자 할 때 사용
  - Scedulers.single()
    - 스레드 하나만 생성하여 Scheduler가 제거되기 전까지 재사용
    - 여러번 호출해도 하나만 생성된다(single-1)
  - Schedulers.newSingle()
    - 호출할 때마다 매번 새로운 스레드 하나 생성
    - 파라미터로는 스레드 명과 데몬 스레드 동작 여부 설정
    - 데몬 스레드는 보조 스레드라고도 불리며 주 스레드가 종료되면 자동으로 종료된다.
  - Schedulers.boundedElastic()
    - ExecutorService 기반의 스레드 풀을 생성한 후, 그 안에서 정해진 수만큼의 스레드를 사용하여 작업을 처리하고 작업이 종료된 스레드는 바납하여 재사용하는 방식으로 Blocking-IO에 최적화 되어 있다.
  - Schedulers.parallel()
    - Non-Blocking IO 에 최적화되어 있는 Scheduler로서 CPU 코어 수만큼 스레드 생성
  - Schedulers.fromExcutorService()
    - 기존에 사용하고 있는 ExecutorService가 있다면 그로부터 Scheduler를 생성하는 방식, Reactor에서는 이 방식이 권장되지 않는다.
  - Schedulers.newXXX()
    - single(), boundedElastic(), parallel()은 Reactor에서 제공하는 Scheduler 인스턴스를 사용하지만 newXXX를 사용하면 새로운 Scheduler 인스턴스를 생성할 수 있다.
    - 스레드 이름, 디폴트 스레드 개수, 유휴시간, 데몬스레드 여부등을 직접 지정하여 커스텀 스레드 풀로 새로 생성할 수 있다.

#### Context
- Context란
  ```
  the situation, events or information that are related to something and that help you to understand it
  ```
  - Context는 어떤 상황에서 그 상황을 처리하기 위해 필요한 정보
  ```
  Reactor에서 Context 정의는 아래와 같다.
  A key/value stroe that is propagated between components such as operators via the context protocol
  ```
  - propagate: downstream, upstream으로 Context가 전파되어 Operator 체인상의 각 Operator가 해당 Context 정보를 동일하게 이용할 수 있다.
  - ThreadLocal과 다소 유사한 면이 있지만 실행 스레드와 매핑하는 것이 아닌 Scheduler와 매핑한다.
    - 구독이 발생할 때마다 해당 구독과 연결된 하나의 Context가 생기는 것
  ```java
  Mono.deferContextual(ctx -> Mono.just("Hello " + ctx.get("firstName"))
      .subscribeOn(Schedulers.boundedElastic())
      .publishOn(Schedulers.parallel())
      .transformDefferedContextual((mono, ctx) -> mono.map(data -> data + " " + ctx.get("lastName:)))
      .contextWrite(context -> context.put("lastName", "Jobs"))
      .contextWrite(context -> context.put("firstName", "Steve"))
      .subscribe(data -> log.info("# onNext: {}", data));
  ```
  - Context에 데이터 쓰기
    - contextWrite을 통해 Context에 데이터를 쓰고 있다.
    - contextWrite 내부 구현을 보면 Function 으로 람다 타입의 Context 파라미터, 리턴으로 사용되고 있다.
  - Context에 쓰인 데이터 읽기
    - Context에서 데이터를 읽는 방식
      - 원본 데이터 소스 레벨에서 읽는 방식
      - Operator 체인의 중간에서 읽는 방식
      - contextWriter 로 쓰기 작업 가능
    - deferContextual operator 원본 데이터 소스 레벨에서 Context 데이터를 읽을 수 있다.
    - transformDeferredContextual 을 사용하면 체인 중간에서 데이터를 읽을 수 있다.
    - 저장된 데이터를 읽을 때는 ContextView를 사용하며 Reactor에서 Operator 체인상의 서로 다른 스레드들이 Context의 저장된 데이터에 손쉽게 접근할 수 있으며 context.put 은 매번 불변객체로 스레드 안정성을 보장
- 자주 사용되는 Context API
  - 쓰기
    - put(key,value)
      - key/value 형태로 context 값을 쓴다.
    - of(key1, value1, key2, value2 ...)
      - key/value 형태로 여러개의 값을 쓴다.
    - putAll(ContextView)
      - 현재 Context와 입력받은 ContextVlue merge
    - delete(key)
      - Context에서 key에 해당하는 value를 지운다.
  - 읽기
    - get(key)
    - getOrEmpty(key)
    - getOrDefault(key, default value)
    - hasKey(key)
    - isEmpty()
    - size()
- Context의 특징
