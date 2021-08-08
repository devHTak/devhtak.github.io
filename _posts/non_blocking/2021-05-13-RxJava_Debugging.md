---
layout: post
title: RxJava, Debugging
summary: Reactive Programming
author: devhtak
date: '2021-05-13 21:41:00 +0900'
category: Reactive
---

#### RxJava의 디버깅 문제점과 대안

- 문제점
  - RxJava는 비동기 프로그래밍 방식과 선언전 프로그래밍 방식으로 개발한다.
    - 기존의 동기 프로그래밍 방식과 명령형 방식의 개발 방식과는 다르게 디버깅하기 까다롭다.

  - RxJava 프로그래밍은 데이터를 생성 및 통지하고 이를 구독하여 처리하는 과정이 하나의 문장으로 되어 있다.
  - 즉, RxJava 프로그래밍은 선언적 프로그래밍 방식이기 때문에 데이터의 상태 변화를 확인하기 위한 디버깅이 쉽지 않다.
  - RxJava 프로그래밍은 여러 쓰레드가 동시에 실행되는 비동기 프로그래밍이기 때문에 실행 시, 항상 같은 결과가 나온다는 보장이 없다.

- 대안
  - RxJava에서는 do 로 시작하는 함수를 통해 생산자나 소비자쪽에서 이벤트 발생 시, 로그를 기록할 수 있는 방법을 제공한다.
  - 함수형 프로그래밍의 특성상 부수효과는 소비쪽에서 처리하는 것이 맞지만 do 함수는 예외다.
    - 어떤 함수에 파라미터를 전달하고 그 함수를 호출 했을 때 결과값을 리턴하는 경우에는 부수효과가 없다고 하며, 결과값을 리턴하지 않고 처리하는 경우에는 부수효과가 있다고 이야기 한다.
  - 소비자가 전달 받은 데이터를 처리하기 전 원본 데이터의 상태나 변환 및 필터링 등으로 가공되는 시점의 데이터 상태를 do 함수를 통해 쉽게 파악할 수 있다.

#### 디버깅을 위한 do 함수

- doOnSubscribe
  - 구독 시작 시, 지정된 작업을 처리할 수 있다.
  - onSubscribe 이벤트가 발생하기 직전에 실행된다.
  - 예시
    ```java
    Observable.just(1, 2, 3, 4, 5, 6)
        .doOnSubscribe(disposable -> System.out.println("Do On Subscribe: 생산자, 구독 처리 준비 완료"))
        .subscribe(
            data -> System.out.println("ON NEXT: " + data),
            error -> System.out.println("ON ERROR: " + error),
            () -> System.out.println("ON COMPLETE"),
            dispose -> System.out.println("ON SUBSCRIBE: 소비자, 구독 처리 준비 완료 알림 받음")
        );
    ```
    ```
    // 출력
    Do On Subscribe: 생산자, 구독 처리 준비 완료
    ON SUBSCRIBE: 소비자, 구독 처리 준비 완료 알림 받음
    ON NEXT: 1
    ON NEXT: 2
    ON NEXT: 3
    ON NEXT: 4
    ON NEXT: 5
    ON NEXT: 6
    ON COMPLETE
    ```
    - subscribe하기 직전 doOnSubscribe에서 설정한 람다가 실행된다.

- doOnNext
  - 생산자가 데이터를 통지하는 시점에 지정된 작업을 처리할 수 있다.
  - onNext 이벤트가 발생하기 직전에 실행된다.
  - 통지된 데이터가 함수형 인터페이스의 파라미터로 전달되므로 통지 시점마다 데이터의 상태를 확인할 수 있다.
  - 예시
    ```java
    Observable.just(1, 3, 5, 7, 9)
        .doOnNext(data -> System.out.println("원본: " + data))
        .filter(data -> data < 5)
        .doOnNext(data -> System.out.println("필터: " + data))
        .map(data -> "##" + data + "##")
        .doOnNext(data -> System.out.println("맵: " + data))
        .subscribe(data -> System.out.println("최종: " + data));		
    ```
    ```
    // 출력
    원본: 1
    필터: 1
    맵: ##1##
    최종: ##1##
    원본: 3
    필터: 3
    맵: ##3##
    최종: ##3##
    원본: 5
    원본: 7
    원본: 9
    ```
    - filter, map함수를 거치기 전에 데이터도 확인할 수 있다.

- doOnComplete
  - 생산자가 완료를 통지하는 시점에 지정된 작업을 처리할 수 있다.
  - onComplete 이벤트가 발생하기 직전에 실행된다.
  - 예제
    ```java
    Observable.range(1, 3)
        .doOnComplete(() -> System.out.println("DO ON COMPLETE: #생산자 데이터 통지 완료"))
        .subscribe(
              data -> System.out.println("ON NEXT: " + data),
              error -> System.out.println("ON ERROR: " + error),
              () -> System.out.println("ON COMPLETE")
        );
    ```
    ```
    // 출력
    ON NEXT: 1
    ON NEXT: 2
    ON NEXT: 3
    DO ON COMPLETE: #생산자 데이터 통지 완료
    ON COMPLETE
    ```
    
- doOnError
  - 생산자가 에러를 통지하는 시점에, 지정된 작업을 처리할 수 있다.
  - onError 이벤트가 발생하기 직전에 실행된다.
  - 통지된 에러 객체가 함수형 인터페이스의 파라미터로 전달되므로 에러 상태를 확인할 수 있다.
  - 예시
    ```java
    Observable.just(1, 6, 9, 12, 15, 20)
        .zipWith(Observable.just(1, 2, 3, 4, 0, 5), (a, b) -> a/b)
        .doOnError(error -> System.out.println("DO ON Error: " + error.getMessage()))
        .subscribe(
            data -> System.out.println("ON NEXT: " + data),
            error -> System.out.println("ON ERROR: " + error)
        );
    ```
    ```
    // 출력
    ON NEXT: 1
    ON NEXT: 3
    ON NEXT: 3
    ON NEXT: 3
    DO ON Error: / by zero
    ON ERROR: java.lang.ArithmeticException: / by zero
    ```

- doOnEach
  - doOnNext, doOnComplete, doOnError를 한번에 처리할 수 있다.
  - Notification 객체를 함수형 인터페이스의 파라미터로 전달 받아 처리
  - 예시
    ```java
    Observable.range(1, 3)
        .doOnEach(notification -> {
            if(notification.isOnComplete()) {
                System.out.println("IS ON COMPLETE");
            } else if(notification.isOnError()) {
                System.out.println("IS ON ERROR: " + notification.getError());
            } else {
                System.out.println("IS ON NEXT: " + notification.getValue());
            }
        })
        .subscribe(
            data -> System.out.println("on next: " + data),
            error -> System.out.println("on error: " + error),
            () -> System.out.println("on complete")
        );
    ```
    ```
    // 출력
    IS ON NEXT: 1
    on next: 1
    IS ON NEXT: 2
    on next: 2
    IS ON NEXT: 3
    on next: 3
    IS ON COMPLETE
    on complete
    ```

- doOnCancel / doOnDispose
  - 소비자가 구독을 해지하는 시점에 지정된 작업을 처리할 수 있다.
  - 완료나 에러로 종료될 경우에는 실행되지 않는다.
  - 예시
    ```java
    Observable.range(1, 10)
        .zipWith(Observable.interval(300L, TimeUnit.MILLISECONDS), (num1, num2) -> num1 * num2)
        .doOnDispose(() -> System.out.println("DO ON DISPOSE: 구독해지완료"))
        .subscribe(new Observer() {
            private Disposable disposable;
            private long startTime;
            @Override
            public void onSubscribe(Disposable disposable) {
                // TODO Auto-generated method stub
                this.disposable = disposable;
                startTime = System.currentTimeMillis();
            }
            @Override
            public void onNext(Object t) {
                // TODO Auto-generated method stub
                System.out.println("ON NEXT: " + t);
                if(System.currentTimeMillis() - startTime > 1000L) {
                    System.out.println("구독 해지");
                    disposable.dispose();
                }
            }

            @Override
            public void onError(Throwable error) { System.out.println("ON ERROR: " + error);}

            @Override
            public void onComplete() { System.out.println("ON COMPLETE");}				
        });	

    Thread.sleep(2000L);
    ```
    ```
    // 출력
    ON NEXT: 0
    ON NEXT: 2
    ON NEXT: 6
    ON NEXT: 12
    구독 해지
    DO ON DISPOSE: 구독해지완료
    ```
    - 구독이 해지된 후에 실행된다.\

- 추가(확장버전)
  - doAfterNext: 생산자가 통지한 데이터가 소비자에 전달된 직후 호출되는 함수
  - doOnTerminate: 완료 또는 에러가 통지될 때 호출되는 함수
    - doOnError + doOnComplete
  - doAfterTerminate: 완료 또는 에러가 통지된 후 호출되는 함수
    - after doOnComplete + doOnError
  - doFinally: 구독이 취소된 후, 완료 또는 에러가 통지된 후 호출되는 함수
    - doOnDispose/doOnCancel + doOnComplete + doOnError
  - doOnLifecycle: 소비자가 구독할 때 또는 구독 해지할 때 호출되는 함수
    - doOnSubscribe + doOnDispose/doOnCancel
    
#### 출처

Kevin의 알기 쉬운 RxJava 2부
