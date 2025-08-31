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
- Context 특징
  - Context는 구독이 발생할 때마다 하나의 Context가 해당 구독에 연결
  - Context는 Operator chain의 아래에서 위로 전파
  - 동일한 키에 대한 값을 중복해 저장하면 Operator 체인에서 가장 위쪽에 위치한 contextWrite()로 저장한 값으로 덮어쓴다.
  - Inner Sequence 내부에서는 외부 Context에 저장한 데이터를 읽을 수 있지만 Inner Sequence 외부에서는 Inner Sequence 내부 Context에 저장된 데이터를 읽을 수 없다.

#### Debugging
- Dubug Mode를 사용한 디버깅
  ```java
	Hooks.onOperationDebug();
	```
  - 실행 결과에 출력된 에러 메세지 이외에 stacktrace 내부에는 의미있는 내용을 확인하기 어렵기 때문에 디버그모드를 실행하면 Operator 체인상에 에러가 발생한 지점을 정확히 가리키고 있다.
  - 디버그 모드를 활성화하는 경우 애플리케이션 내부에 비용이 많이 드는 동작 과정을 거친다.
  - 애플리케이션 내에 있는 모든 Operator의 Stacktrace를 캡처한다.
  - 에러가 발생하면 캡처한 정보를 기반으로 에러가 발생한 Assembly의 Stacktrace를 원본 Stacktrace 중간에 끼어 넣는다.
    - Assembly란 Operator 연산으로 반환되는 Flux/Mono를 말한다.
    - 에러가 발생한 operator의 stacktrace를 캡처한 assembly 정보를 traceback 이라 한다.
- checkpoint() operator를 사용한 디버깅
  - checkpoint operator를 사용하면 operator 채인 내의 스택트레이스만 캡처
  - Traceback 출력
    ```java
    Flux.just(2, 4, 6, 8)
        .zipWith(Flux.just(1, 2, 3, 0), (x, y) -> x/y)
        .map(num -> num + 2)
        .checkpoint()
        .subscribe(
            data -> log.info("# onNext: {}", data),
            error -> log.error("# onError: ", error);
        );
    ```
    - map 다음에 추가한 checkpoint 지점까지 에러 전파 예상 가능
  - traceback 출력 없이 식별자를 포함한 Description 을 출력하여 에러 발생 지점 예상하는 방법
    ```java
    Flux.just(2, 4, 6, 8)
        .zipWith(Flux.just(1, 2, 3, 0), (x, y) -> x/y)
        .checkpoint("zipwith checkpoint")
        .map(num -> num + 2)
        .checkpoint("map checkpoint")
        .subscribe(
            data -> log.info("# onNext: {}", data),
            error -> log.error("# onError: ", error);
        );
    ```
    - 파라미터로 입력한 description 출력되는 것 확인 가능
  - Traceback과 Description 모두 출력하는 방법
    ```java
    Flux.just(2, 4, 6, 8)
        .zipWith(Flux.just(1, 2, 3, 0), (x, y) -> x/y)
        .checkpoint("zipwith checkpoint", true)
        .map(num -> num + 2)
        .checkpoint("map checkpoint", true)
        .subscribe(
            data -> log.info("# onNext: {}", data),
            error -> log.error("# onError: ", error);
        );
    ```
    - checkpoint와 description ahen cnffur
  - operator 체인이 조금 복잡해지면 에러 발생 지점을 찾는 것이 쉽지 않다.
- log() operator를 사용한 디버깅
  ```java
  Flux.fromArray(new String[]{ "BANANAS", "APPLES", "PEARS", "MELONS"))
      .map(String::toLowCase)
      .map(fruit -> fruit.substring(0, fruit.length() - 1))
      .log()
      .map(fruitMap::get)
      .subscribe(
          data -> log.info("# onNext: {}", data),
          error -> log.error("# onError: ", error);
      );
  ```
  - log는 reactor sequence의 동작을 출력하는 데, 이 로그를 통해 디버깅 가능
  - log operator를 추가하면 onSubscribe, request, onNext 같은 singal 출력
  - log operator에 파라미터 추가 가능
    - log("Fruit.Substring", Level.FINE)
    - Fruit.Substring -> 카테고리 표시, Level.FINE -> 로그 레벨 지정

#### Testing
- StepVerifier를 사용한 테스팅
  ```java
  StepVerifier.create(Mono.just("Hello Reactor")) // 테스트할 대상 sequence 생성
              .expectNext("Hello Reactor") // expectXX를 통해 기댓값 평가
              .as("# expect next") // 이전 기댓값 평가 단계에 대한 설명 추가 가능
              .expectComplete()
              .verify(); // 전체 operator chain의 테스트를 트리거
  ```
  - 가장 일반적인 테스트 방식은 Flux, Mono를 Reactor sequence로 정의한 후, 구독 시점에 해당 operator chain이 시나리오 대로 동작하는 지 테스트
  - 다음에 발생할 signal에 대해 데이터 emit, 특정 시간동안 emit된 데이터가 있는 지 등을 단계적으로 테스트할 수 있다.
  
  |메소드|설명|
  |---|---|
  |expectSubscription|구독이 이뤄짐을 기대|
  |expectNext|onNext 시그널을 통해 전달된 값이 파라미터로 전달된 값과 같음을 기대|
  |expectCopmlete|onComplete 시그널 전송되기를 기대|
  |expectError|onError 시그널 기대|
  |expectNextCount|구독 시점 또는 이전 expectNext 를 통해 기댓값이 평가된 데이터 이후부터 emit된 수를 기대|
  |expectNoEvent|주어진 시간동안 signal 이벤트가 발생하지 않았음을 기대|
  |expectAccessibleContext|구독 시점 이후 context가 전파되었음을 기대|
  |expectNextSequence|emit된 데이터들이 파라미터로 전달되 iterable 요소와 매치됨을 기대|

  |메소드|설명|
  |---|---|
  |verify| 검증을 트리거|
  |verifyComplete| 검증 트리거 및 onComplete signal 기대|
  |verifyError| 검증 트리거 및 onError signal 기대|
  |verifyTimeout|검증 트리거 및 주어진 시간이 초과되어도 publisher 가 종료되지 않음을 기대|

  - withVirtualTime(() -> method()).then(() -> VirtualTimeScheduler.get().advanceTimeBy(Duration.ofHours(1)))
    - VirtualTimeScheduler 라는 가상 스케줄러의 제어를 받아 특정 시간에 대한 테스트가 가능하다.
  - Backpressure 테스트
    - verifyThenAssertThat().hasDroppedElements()
    - 검증을 트리거한 후, 추가적인 Assertion이 가능한데, hasDroppedElements()를 사용하면 drop된 데이터가 있음을 판단
  - Context 테스트
    - expectAccessibleContext.hasKey(key1).hasKey(key2).then()
    - expectAccessibleContext 를 통해 Context 전파를 확인하고 hasKey를 통해 key가 있는 지 확인할 수 있다.
    - then 메소드를 통해 다음 signal의 기대값 평가를 할 수 있다.
  - record 기반 테스트
    - recordWith를 사용하여 단순 기댓값 평가를 넘어 좀 더 구체적인 조건으로 테스트 가능
- TestPublisher를 사용한 테스팅
  - 정상 동작(Well-behaved)하는 TestPublisher
    ```java
    TestPublisher<Integer> source = TestPublisher.create();
    StepVerifier.create(method(source.flux())) // 테스트 대상 클래스에 파라미터로 전달하기 위해 변환
                .expectSubscription()
                .then(() -> source.emit(2, 4, 6, 8, 10)) // 필요한 데이터를 emit
                .expectNext(1, 2, 3, 4)
                .expectError()
                .verify();
    ```
    - TestPublisher를 사용하여 복잡한 로직이 포함된 대상 메서드를 테스트하거나 조건에 따라 signal을 변경해야 되는 등의 특정 상황을 테스트하기 용이하게 한다.
    - 오동작(Misbehaving)하는 TestPublisher
      ```java
      // 데이터가 null이어도 동작하는 TestPublisher 생성
      TestPublisher<Integer> source = TestPublisher.createNoncompliant(TestPublisher.Violation.ALLOW_NULL);
      StepVerifier.create(method(source.flux())) // 테스트 대상 클래스에 파라미터로 전달하기 위해 변환
                  .expectSubscription()
                  .then(() -> {
                                getDataSource().stream().forEach(data -> source.next(data)));
                                source.complete();
                  }).expectNext(1, 2, 3, 4)
                .expectError()
                .verify();
    ```
- PublisherProbe를 사용한 테스팅
  ```java
  PublisherProbe<String> probe = PublisherProbe.of(method());
  StepVerifier.create(method(probe.mono))
              .expectNextCount(1)
              .verifyComplete();
  probe.assertWasSubscribed(); // 해당 파라미터가 구독이 되었고, 요청을 했는지, 중간에 취소되지 않았는 지 확인할 수 있다.
  probe.assertWasRequested();
  probe.assertWasNotCanceled();
  ```
  - PublisherProbe를 통해 Sequence의 실행 경로를 테스트할 수 있다.

#### Operators
- Sequence를 생성하기 위한 operator
  - justOrEmpty
    - just의 확장 operator로 emit할 데이터가 null일 경우 NullPointerException이 발생하지 않고 onComplete signal 전송
  - fromIterable
    - iterable에 포함된 데이터를 emit하는 Flux 생성
  - fromStream
    - stream에 포함된 데이터를 emit하는 flux를 생성
    - java stream 특성 상 Stream 은 재사용할 수 없으며 cancel, error, complete 시 자동으로 닫히게 된다.
  - range
    - n부터 1씩 증가한 연속된 수를 m개 emit 하는 flux를 생성
    - for문처럼 특정 횟수만큼 어떤 작업을 처리하고자 할 경우에 주로 사용
  - defer
    ```java
    System.out.println(LocalDateTime.now()); // 2025.06.01 20:54:00
    Mono<LocalDateTime> justMono = Muno.just(LocalDateTime.now());
    Mono<LocalDateTime> deferMono = Muno.defer(() -> Mono.just(LocalDateTime.now()));

    Thread.sleep(2000L);

    justMono.subscribe(System.out::println); // 2025.06.01 20:54:00
    deferMono.subscribe(System.out::println); // 2025.06.01 20:56:00

    Thread.sleep(2000L);

    justMono.subscribe(System.out::println); // 2025.06.01 20:54:00
    deferMono.subscribe(System.out::println); // 2025.06.01 20:58:00
    ```
    - 구독하는 시점에 데이터를 emit 하는 flux, mono 생성
    - emit을 지연시키기 때문에 꼭 필요한 시점에 데이터를 emit하여 불필요한 프로세스를 줄일 수 있다.
    - just는 hot publisher 이고, defer는 cold publisher 이다.
  - using
    ```java
    Path path = Paths.get("./example.txt");
    Flux.using(() -> Files.lines(path), Flux::fromStream, Stream::close)
        .subscribe(log::info)
    ```
    - 파라미터로 전달받은 resource를 emit하는 flux 생성
    - 첫번째 파라미터는 읽어 올 resource이고, 두번째 파라미터는 읽어 온 resource를 emit하는 flux, 세번째는 종료 signal이 발생할 경우 resources를 해제하는 등의 후처리를 할 수 있게 한다.
  - generate
    ```java
    Flux.generate(() -> 0, (state, sink) -> {
        sink.next(state);
        if(state == 10) sink.complete();
        return ++state;
    }).subscribe(log::info)
    ```
    - 프로그래밍 방식으로 signal 이벤트를 발생시키며 특히 동기적으로 데이터를 하나씩 순차적으로 emit할 때 사용
    - 첫번째 파라미터는 emit할 숫자의 초깃값 지정, 두번째는 상태를 지정한다.
  - create
    - generate와 동일하게 프로그래밍 방식으로 signal 이벤트를 발생시키지만 차이가 있다.
    - generate는 동기적으로 한번에 한건씩 발생하지만 create는 한번에 여러건을 비동기적으로 emit할 수 있다.
- Sequence 필터링 Operator
  - filter
    ```java
    Flux.range(1, 30)
        .filter(num -> num % 2 != 0)
        .subscribe(log::info)
    ```
    - Upstream에서 emit된 데이터 중에서 조건에 일치하는 데이터만 Downstream으로 emit
  - filterWhen
    ```java
    Flux.fromIterable(method())
        .filterWhen(vaccine -> Mono.just(vaccineMap.get(vacchine).getT2() >= 23_000_000)
                                .publishOn(Scheduelrs.parallel()))
        // filterWhen operator 의 inner sequence를 통해 백신명에 해당하는 수량이 3_000_000개 이상이라면 해당 백신명을 Subscriber에게 emit
        .subscribe(log::info)
    ```
    - 비동기적으로 필터링을 수행한다.
    - filterWhen() operator는 내부에서 Inner Sequence를 통해 조건에 맞는 데이터인지를 비동기적으로 테스트한 후, 테스트 결과가 true 라면 downstream으로 emit
  - skip
    ```java
    Flux.interval(Duration.ofSeconds(1)) // 5500ms 동안 0 ~ 5 까지 emit
        .skip(2) // 0, 1 skip
        .subscribe(log::info); // 2, 3, 4 출력
    Threa.sleep(5500L);
    ```
    ```java
    Flux.interval(Duration.ofMIllis(300)) // 2000ms 동안 0 ~ 5 까지 emit
        .skip(Duration.ofSeconds(1)) // 1000ms 동안(0, 1, 2) emit skip
        .subscribe(log::info); // 3, 4, 5 출력
    Threa.sleep(2000L);
    ```
    - skip operator는 upstream에서 emit된 데이터 중 파라미터로 입력받은 숫자만큼 건너뛴 후 나머지를 emit
  - take
    ```java
    Flux.interval(Duration.ofSeconds(1)) // 4000ms 동안 0 ~ 3 까지 emit
        .take(3) // 0 ~ 2 까지 take
        // take(Duration.ofMillis(2500) - 2500ms 동안 emit된 데이터 take
        .subscibe(log::info); // 0, 1, 2 출력
    Threa.sleep(4000L);
    ```
    - upstream에서 emit되는 데이터 중 파라미터로 입력받은 숫자만큼 downstream으로 emit
  - takeXXX
    ```java
    Flux.fromIterable(method())
        .takeLast(2)
        // .takeUntil(tuple -> tuple.getT2() > 20_000_000)
        // .takeWhile(tuple -> tuple.gett2() < 20_000_000)
        .subscribe(log::info);
    ```
    - takeLast: upstream에서 emit된 데이터 중 가장 마지막에 emit된 데이터를 downstream으로 emit
    - takeUntil: 파라미터로 입력한 람다표현식을 true가 될때까지 upstream에서 emit된 데이터를 downstream으로 emit
    - takeWhile: takeUntil과 반대로 표현식이 true일 때까지 upstream에서 emit된 데이터를 downstream으로 emit
  - next
    ```java
    Flux.fromIterable(method())
        .next()
        .subscribe(log::info);
    ```
    - upstream에서 emit된 데이터 중 첫번째 데이터만 downstream으로 emit
- Sequence 변환 Operator
  - map
    - upstream에서 emit된 데이터를 mapper function 을 사용하여 변환한 후 downstream으로 emit
  - flatMap
    ```java
    Flux.just("Good", "Bad")
        .flatMap(feel -> Flux.just("Morning", "Afternoon", "Evening")
                        .map(time -> feeling + " " + time))
        .subscribe(log::info); //Good Morning, Good Afternoon, Good Evening, Bad ~ 출력
    ```
    - upstream에서 emit 한 데이터는 flatMap 내부 inner sequence를 생성하여 1개 이상의 변환된 데이터를 emit
    - flatMap 내부 sequence에서 Scheduler를 설정하여 비동기적으로 데이터를 emit할 수 있다.
  - concat
    ```java
    Flux.concat(Flux.just(1, 2, 3), Flux.just(4, 5, 6))
        .subscribe(log::info); // 1, 2, 3, 4, 5, 6 출력
    ```
    - 파라미터로 입력되는 publisher sequence를 연결하여 데이터를 순차적으로 emit
    - 먼저 입력되는 publisher가 종료될 때까지 나머지 publisher가 대기하는 특성을 갖는다.
  - merge
    ```java
    Flux.merge(
      Flux.just(1, 2, 3, 4).delayElements(Duration.ofMillis(300L))
      Flux.just(5, 6, 7).delayElements(Duration.ofMillis(500L))
    ).subscribe(log::info); // 1(300ms), 5(500ms), 2(600ms), 3(900ms), 6(1000ms), 4(1200ms), 7(1500ms) 출력
    Thread.sleep(2000L);
    ```
    - merge operator의 동작을 이해하기 위해선 emit되는 시간 주기를 다르게 설정하는 것이 좋다.
    - publisher가 emit하는 시간이 빠른 데이터부터 차례대로 emit한다.
  - zip
    ```java
    Flux.zip(
      Flux.just(1, 2, 3).delayElements(Duration.ofMillis(300L))
      Flux.just(4, 5, 6).delayElements(Duration.ofMillis(500L))
      // (n1, n2) -> n1 * n2
    ).subscribe(log::info); // [1,4], [2,5], [3,6] // 4, 10, 18 출력
    Thread.sleep(2000L);
    ```
    - emit된 데이터를 결합하는 데, 하나씩 emit을 기다렸다가 결합, Tuple로 묶어서 전달
    - 세번째 파라미터로 tuple이 아닌 최종 변환된 데이터를 전달할 수 있도록 한다.
  - and
    ```java
    Mono.just("Task 1")
    .delayElements(Duration.ofSeconds(1))
    .doOnNext(data -> log.info("mono: {}", data))
    .and(Flux.just("Task 2", "Task 3")
            .delayElements(Duration.ofMillis(600))
            .doOnNext(data -> log.info("flux: {}", data))
    .subscribe(log::info); // flux: Task 2, Mono: Task 1, Flux: Task 3 출력
    Thread.sleep(5000);
    ```
    - Mono의 complete signal과 파라미터로 입력된 publisher 의 complete signal을 결합하여 새로운 Mono<Void> 반환
  - collectList
    - Flux에서 데이터를 모아 List로 변환하여 Mono<List<T>> 로 반환
  - collectMap
    ```java
    Flux.range(0, 26)
        .collectMap(key -> methodKey(key), value -> methodValue(value))
        .subscribe(log::info);
    ```
    - Flux에서 emit된 데이터를 기반으로 key, value를 생성하여 Mono<Map<T, V>>로 반환
- Sequence 내부 동작 확인하기 위한 Operator
  |operator|description|
  |---|---|
  |doOnSubscribe()|Publisher가 구독중일 때 트리거되는 동작을 추가할 수 있다.|
  |doOnRequest()|Publisher가 요청을 수신할 때 트리거되는 동작을 추가할 수 있다.|
  |doOnNext()|Publisher가 데이터를 emit할 때 트리거되는 동작을 추가할 수 있다.|
  |doOnComplete()|Publisher가 성공적으로 완료되었을 때 트리거되는 동작을 추가할 수 있다.|
  |doOnError()|Publisher가 에러가 발생한 상태로 종료되었을 때 트리거되는 동작을 추가할 수 있다.|
  |doOnCancel()|Publisher가 취소되었을 때 트리거되는 동작을 추가할 수 있다.|
  |doOnTerminate()|Publisher가 성공적으로 완료되었을 떄 또는 에러가 발생한 상태로 종료되었을 때 트리거되는 동작을 추가할 수 있다.|
  |doOnEach()|Publisher가 데이털르 emit할 때, 성공적으로 완료되었을 때, 에러가 발생한 상태로 종료되었을 때 트리거되는 동작을 추가할 수 있다.|
  |doOnDiscard()|upstream에 있는 전체 operator 체인의 동작 중 operator에 의해 폐기되는 요소를 조건부로 정리할 수 있다.|
  |doAfterTerminate()|downstream을 성공적으로 완료한 직후 또는 에러가 발생하여 publisher가 종료된 직후에 트리거되는 동작을 추가할 수 있다.|
  |doFirst()|publisher가 구독되기 전에 트리거되는 동작을 추가할 수 있다.|
  |doFinally()|에러를 포함해 어떤 이유든 publisher가 종료된 후 트리거되는 동작을 추가할 수 있다.|

- 예외처리를 위한 Operator
  - error
    ```java
    Flux.range(1, 5)
        .flatMap(num -> {
          if(num % 2 == 0)
            return Flux.error(IllegalArgumentException::new);
          return Mono.just(num * 2);
        }).subscribe(data -> log.info("next: {}", data)); // next: 1, onError 발생
    ```
    - error operator는 파라미터로 지정된 에러로 종료하는 flux 생성
    - throw 키워드로 예외를 의도적으로 던지는 것과 같은 역할을 한다.
  - onErrorReturn
    ```java
    getBooks()
    	.map(book -> book.getName().toUpperCase())
    	.onErrorReturn(NullPointerException.class, "No pen name") // onErrorReturn("No pen name") 가능
    	.onErrorReturn(IllegalFormatException.class, "Illegal pen name")
    	.subscribe(log::info);
    	// NPE가 발생하면 No pen name 출력, IllegalFormatException 발생 시 Illegal pen name 출력
    ```
    - 에러 이벤트가 발생했을 때 Downstream으로 전파하지 않고 대체값을 emit
  - onErrorResume
    ```java
    getBooksFromCache()
    	.onErrorResume(error -> getBooksFromDb())
    	.subscribe(log::info, log::error);
    ```
    - 에러 이벤트가 발생했을 때 Downstream으로 전파하지 않고 대체 publisher emit
    - try catch문의 catch 블록에서 예외가 발새앟여 또 다른 메서드를 호출하는 형태
  - onErrorContinue
    ```java
    Flux.just(1, 2, 4, 0, 12)
    	.map(num -> 12 / num)
    	.onErrorContinue((error, num) -> log.error("e: {}, num: {}", e.getMessage(), num))
    	.subscribe(log::info); // 12, 6, 3, 1 출력
    ```
    - 에러가 발생했을 때, 에러 영역 내에 있는 데이터는 제거하고, upstream에서 후속 데이터를 emit하는 방식으로 에러 복구 
  - retry
    - 에러가 발생하면 파라미터로 입력한 횟수만큼 원본 Flux의 Sequence를 다시 구독, 만약 Long.MAX_VALUE를 입력하면 무한 반복된다.
- Sequence의 동작 시간 측정을 위한 Operator
  - elasped
    ```java
    Flux.range(1, 3)
    	.delayElements(Duration.ofSeconds(1))
    	.elasped()
    	.subscribe(data -> log.info("onNext: {}, time: {}", data.getT2(), data.getT1());
    // onNext: 1, time: 1029, // onNext: 2, time: 1005, // onNext: 3, time: 1001
    ```
    - emit된 데이터 사이의 경과 시간을 측정하여 Tuple<Long, T> 형태로 Downstream에 emit
- Flux Sequence 분할을 위한 Operator
  - window
    ```java
    Flux.range(1, 7)
    	.window(3)
        .flatMap(flux -> {
	    log.info("=============");
    	    return flux;
        }).subscribe(new BaseSubscriber<>()) {
    	    @Override
    	    protected void hookOnSubscirbe(Subscription subscription) {
	        subscription.request(2);
            }
    	    @Override
            protected void hookOnNext(Integer value) {
                log.info("# onNext: {}", value);
	        request(2);
            }
        });
	// ======= 1, 2, 3 ======= 4, 5, 6 ======= 7
    ```
    - Upstream에서 emit되는 첫번째 데이터부터 maxSize 숫자만큼의 데이터를 포함한 새로운 Flux로 분할
  - buffer
    ```java
    Flux.range(1, 7)
    	.buffer(3)
    	.subscribe(buffer -> log.info("onNext: {}", buffer);
    	// onNext 1, 2, 3 // onNext: 4, 5, 6 // onNext: 7
    ```
    - upstream에서 emit되는 첫번째 데이터부터 maxSize 숫자만큼 데이터를 list 버퍼로 한번에 emit
  - bufferTimeout
    ```java
    Flux.range(1, 7)
    	.map(num -> {
	    try {
                if(num < 5) Thread.sleep(100L);
                else Thread.sleep(300L);
    	    } catch(InterruptedException e) {}
    	    return num;
    	})
    	.bufferTimeout(3, Duration.ofMillis(450L))
    	.subscribe(buffer -> log.info("onNext: {}", buffer);
    	// onNext: 1, 2, 3 // onNext: 4, 5 // onNext: 6 // onNext: 7
    ```
    - upstream에서 emit되는 첫번째 데이터부터 maxSize 숫자만큼 데이터 또는 maxTime 내에 emit되는 데이터를 list 버퍼로 한번에 emit 
  - groupBy
    ```java
    Flux.fromIterable(SampleData.books)
    	.groupBy(book -> book.getAuthorName())
    	.flatMap(groupedFlux -> groupedFlux.map(book -> book.getName() + " " + book.getAuthorName()).collectList())
    	.subscribe(log::info);
    ```
    - emit되는 keyMapper로 생성한 key를 기준으로 그룹화한 GroupedFlux를 리턴하며 이 GroupedFlux를 통해서 그룹별로 작업을 수행할 수 있다.
- 다수의 Subscriber에게 Flux를 멀티캐스팅(Multicasting)
  - Subscriber가 구독하면 Upstream에서 emit된 데이터가 구독중인 모든 Subscriber에게 멀티캐스팅된다.
  - 해당 Operator는 Cold Publisher는 Hot Publisher로 동작하게 하는 특징이 있다.
  - publish
    ```java
    ConnectableFlux<Integer> flux = Flux.range(1, 5)
				    .delayElements(Duration.ofMillis(300L)
				    .publish();
    Thread.sleep(500L);
    flux.subscribe(log::info); // 0.5초 뒤 첫번째 구독 발생

    Thread.sleep(200L);
    flux.subscribe(log::info); // 0.2초 뒤 두번째 구독 발생

    flux.connect(); // 해당 시점부터 0.3초에 한번씩 emit
    
    Thread.sleep(1000L); 
    flux.subscribe(log::info); // 0.2초 뒤 3번째 구독 발생, (1, 2) emit은 이미 시간이 지난 뒤 subscibe 했기 때문에 받지 못한다. (hot publisher)

    Thread.sleep(20000L);
    ```
    - publish operator 는 구독을 하더라도 구독 시점에 즉시 데이터를 emit하지 않고 connect 호출하는 시점에 데이터를 emit    
  - autoConnect
    ```java
    ConnectableFlux<Integer> flux = Flux.range(1, 5)
				    .delayElements(Duration.ofMillis(300L)
				    .publish()
    			 	    .autoConnect(2);
    Thread.sleep(500L);
    flux.subscribe(log::info); // 0.5초 뒤 첫번째 구독 발생

    Thread.sleep(200L);
    flux.subscribe(log::info); // 0.2초 뒤 두번째 구독 발생, 2번째 구독이 실행되어 upstream에서 데이터 emit

    Thread.sleep(1000L); 
    flux.subscribe(log::info); // 0.2초 뒤 3번째 구독 발생, (1, 2) emit은 이미 시간이 지난 뒤 subscibe 했기 때문에 받지 못한다. (hot publisher)

    Thread.sleep(20000L);
    ```
    - autoConnect는 지정된 숫자만큼 구독이 발생하는 시점에 자동으로 연결되기 때문에 별도의 connect 호출이 필요 없다.
  - refCount
    ```java
    Flux<Long> publisher = Flux.interval(Duration.ofMillis(500))
    			.publish()
    			// .autoConnect(1);
    			.refConnect(1); // refConnect 를 이용해 1개의 구독이 발생하는 시점에 Upstream 소스 연결

    Disposable disposable = publisher.subscibe(log::info); // 연결 0, 1, 2, 3 구독

    Thread.sleep(2100L);
    disposable.dispose(); // 2.1초 후 구독 해제

    publisher.subscribe(log::info); // 두번째 구독에 대해 첫번째 구독이 취소되었기 때문에 Upstream에 다시 연결
    // refCount 는 0, 1, 2, 3 구독, autoConnect 는 4, 5, 6, 7, 8 구독
    // autoConnect에 경우 첫번째 구독이 취소되지만 upstream 소스로의 연결이 해제된 것이 아니기 때문에 4부터 전달받는다.

    Thread.sleep(2500L);
    ```
    - 파라미터로 입력된 숫자만큼의 구독이 발생하는 시점에 Upstream 소스에 연결되며 모든 구독이 취소되거나 Upstream의 데이터 emit이 종료되면 연결이 해제
    - 주로 무한 스트림 상황에서 모든 구독이 취소될 경우 연결을 해제하는 데 사용할 수 있다.

#### 출처
- 황적식 저자의 스프링으로 시작하는 리액티브 프로그래밍. 
