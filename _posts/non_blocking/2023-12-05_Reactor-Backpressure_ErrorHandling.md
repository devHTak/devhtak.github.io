---
layout: post
title: WebClient와 WebTestClient
summary: Reactive Programming
author: devhtak
date: '2021-08-30 21:41:00 +0900'
category: Reactive
---

#### Exception Handling과 Backparessure

- Blocking, Synchronous한 방식으로 try-catch를 하여 예외처리를 진행했다.
- 반면 Reactive 환경에서는 오류 처리와 백프레셔 관리가 안정적이고 반응성있는 애플리케이션을 보장하는데 중요한 역할을 한다.
  - Error Handling: 스트림을 처리하는 과정에서 오류가 발생하면 전체 스트림이 실패할 수 있기 때문에 효과적으로 처리하는 것이 중요하다.
  - Backpressure: 일반적으로 Subscriber가 Publisher에 속도를 따라간다면 문제가 없지만, Consumer에 처리가 느려진다면 Backpressure에 대한 전략을 세워야 한다.

#### Exception Handler Operator

- onErrorReturn
  ```java
  Flux.just("A", "B", "C", "D")
      .map(s -> {
          if(s.equals("C")) {
            throws RuntimeException("C");
          }
          return s;
      }).onErrorReturn("ERROR")
      .subscribe(System.out::println);
  ```
  - stream에서 오류가 발생하면 대체 값을 제공한다
  - stream을 중단하고 fallback 값을 반환
- onErrorResume
  ```java
  Flux.just("A", "B", "C", "D")
      .map(s -> {
          if(s.equals("C")) {
            throws RuntimeException("C");
          }
          return s;
      }).onErrorResume(e -> Flux.just(e.getMessage());
      .subscribe(System.out::println);
  ```
  - onErroReturn과 유사하게 onErrorReume는 연산자를 사용하여 오류가 발생할 때 fallback stream을 제공할 수 있다.
  - error는 fallback stream 내에 추가로 전파되거나 해결될 수 있다.
- Retry
  ```java
  Flux.just("A", "B", "C", "D")
      .map(s -> {
          if(s.equals("C")) {
            throws RuntimeException("C");
          }
          return s;
      }).retry(3)
      .subscribe(System.out::println);
  ```
  - retry 연산자를 사용하면 error 발생 시 stream을 다시 시도할 수 있다.
  - retry 횟수를 지정하고 delay도 넣을 수 있다.
 
#### Backpressure

- Webflux 에서 TCP를 활용한 Backpressure
  - WebFlux 프레임워크는 이벤트를 바이트로 변환하여 TCP를 통해 전송 / 수신
    - TCP 흐름제어를 사용하여 백프레셔를 바이트로 제어
  - Consumer는 다음 logical element를 요청하기 전 이전 작업 수행 가능
  - Receiver가 이벤트를 처리하는 동안 WebFlux는 요청이 없으므로 확인 없이 바이트를 queue에 넣는다.
  - TCP 프로토콜의 특성으로 인해 새 이벤트가 있는 경우 publisher는 네트워크로 이벤트를 전송
- Backpressure를 사용하여 Systemic Failures 예방하는 전략  
  - 즉시 처리하지 못하는 데이터를 Buffering하기
    ```java
    Flux.range(1, 100)
        .onBackpressureBuffer(10) // 버퍼 크기를 10으로 지정
        .subscribe(s -> System.out.println("Received: " + s));
    ```
    - consumer는 이벤트를 처리할 수 있을 때까지 나머지 이벤트를 임시 저장
    - 무제한 버퍼는 메모리 문제를 일으킬 수 있다
    - project reactor는 이 전략을 구현하기 위한 버퍼 연산자를 제공
  - 즉시 처리하지 못하는 데이터는 삭제하고 Tracking하지 않는 것
    - onBackpressureLatest
      - Consumer가 처리할 수 없는 과도한 이벤트를 삭제하고 가장 최근의 이벤트만 유지합니다.
      - 이를 통해 Consumer가 데이터에 압도당하는 것을 방지하는 동시에 최신 정보를 수신할 수 있습니다.
    - onBackpressureDrop
      - 추가 이벤트 삭제

- Publisher의 Event 제어
  - Publisher 측 Data Stream 제어하기
    ```java
    Flux.interval(Duration.ofMillis(10)) // 10ms 마다 데이터 전송
        .sample(Duration.ofMillis(100)) // 100ms 마다 sampling하여 최신 항목을 전송하는데 사용
        .subscribe(s-> System.out.println("Received: " + s));
    ```
    - interval: 지정한 시간동안 대기했다가 데이터를 전송한다.
    - sample: 모든 항목을 전달하는 것이 아닌 sampling한 시점에 가장 최근 항목만 전달하여 데이터가 전송되는 속도를 제어하는 데 도움이 된다.
  - Subscriber에서 request 요청 시 데이터 전달
    - Publisher는 Subscriber에 새 이벤트를 요청할 떄까지 기다리며, 수요에 따라 이벤트 처리
  - LimitRate
    ```java
    Flux.range(1, 25)
        .limitRate(10) // 만약 15개를 요청해도 10개 전송하고, 다음 스텝에 나머지 5개 전송 
        .subscribe(System.out::println);
    ```
    - Subscriber에 요청에 대한 최대값과 상관없이 데이터 전송할 양을 정해둔다.
  - Cancel
    ```java
    Flux.range(1, 25)
        .subscribe(new BaseSubscriber<Integer>() {
            @Override
            protected void hookOnNext(Integer value) {
                request(3);
                cancel();
            }
        });
    ```
    - Publisher에서 subscribe 하는 Subscriber를 구현하거나 BaseSubscriber를 상속받아 오버라이딩 할 수 있기 때문에 해당 클래스를 재정의하여 Subscription을 취소하는 것을 구현할 수 있다.

#### 출처

- https://www.baeldung.com/spring-webflux-backpressure
