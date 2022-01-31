---
layout: post
title: Reactor, Operator
summary: Reactive Programming
author: devhtak
date: '2021-08-08 21:41:00 +0900'
category: Reactive
---

#### Publisher, Subscriber, Subscription 관계

```java
Publisher<Integer> publisher = new Publisher<Integer>() {
  Iterable<Integer> iter = Arrays.asList(1, 2, 3, 4, 5);
  @Override
  public void subscribe(Subscriber<? super Integer> subscriber) {
    // TODO Auto-generated method stub
    subscriber.onSubscribe(new Subscription() {
      @Override
      public void request(long n) {
        try {
          iter.forEach(i -> subscriber.onNext(i));
          subscriber.onComplete();
        } catch(Exception e) {
          subscriber.onError(e);
        }
      }
      @Override
      public void cancel() {}
    });
  }
};

Subscriber<Integer> subscriber = new Subscriber<Integer>() {
  @Override
  public void onSubscribe(Subscription s) { s.request(Long.MAX_VALUE); }
  @Override
  public void onNext(Integer integer) { System.out.println("onNext: " + integer); }
  @Override
  public void onComplete() { System.out.println("onComplete");}
  @Override
  public void onError(Throwable t) { System.out.println("onError: " + t.getMessage()); }
};

publisher.subscribe(subscriber)
```

-  Publisher는 Sbuscriber를 인자로 subscribe 메소드 실행. subscribe 메소드에서 Subscriber에 onSubscribe 메소드를 실행하여 구독을 맺는다.
-  Subscription을 통해 onNext 메소드로 값을 보내고, 완료되면 onComplete가 오류가 발생하면 onError가 발생한다.


#### Operator 구조

- Sequence를 생성, 변형, 필터링 등을 하기 위해 사용
- Publisher -> \[Data1] -> Operation1 -> \[Data2] -> Operation2 -> \[Data3] -> Subscriber

- 예시1: map
  - 1:1 구조로 기존에 있던 sequence를 변형하여 전달하는 역할을 한다.
    ```java
    public static void main(String[] args) {
      Publisher<Integer> publisher = iterPub(Stream.iterate(1, a->a+1).limit(10).collect(Collectors.toList()));
	  Publisher<Integer> mapPublisher = mapPub(publisher, s -> s * 10);
	  mapPublisher.subscribe(logSub());
    }	
    private Publisher<Integer> mapPub(Publisher<Integer> pub, Function<Integer, Integer> function) {
      return new Publisher<Integer>() {
        @Override
        public void subscribe(Subscriber<? super Integer> subscriber) {
          pub.subscribe(new Subscriber<Integer>() {
            @Override
            public void onSubscribe(Subscription subscription) { subscriber.onSubscribe(subscription); }
            @Override
            public void onNext(Integer item) { subscriber.onNext(function.apply(item)); }
            @Override
            public void onError(Throwable throwable) { subscriber.onError(throwable); }
            @Override
            public void onComplete() { subscriber.onComplete(); }
          });
        }
      };
    }
    private Publisher<Integer> iterPub(Iterable<Integer> iter) {
      return new Publisher<Integer>() {
        @Override
        public void subscribe(Subscriber<? super Integer> subscriber) {
          subscriber.onSubscribe(new Subscription() {
            @Override
            public void request(long n) {
              try { iter.forEach(i -> subscriber.onNext(i)); subscriber.onComplete();} 
              catch(Exception e) { subscriber.onError(e); }
            }
            @Override
            public void cancel() {}
          });
        }
      };
    }
    private Subscriber<Integer> logSub() {
      return new Subscriber<Integer>() {
        @Override
        public void onSubscribe(Subscription s) { s.request(Long.MAX_VALUE); }
        @Override
        public void onNext(Integer integer) { System.out.println("onNext: " + integer); }
        @Override
        public void onComplete() { System.out.println("onComplete");}
        @Override
        public void onError(Throwable t) { System.out.println("onError: " + t.getMessage()); }
      };
    }
    ```
    - mapPub 메소드에서 새로운 Subscriber를 생성하여 최종 Subscriber와 연결하였다.
    - mapPub 인자로 넘겨준 function을 통해 데이터를 재정의하여 넘겨줌으로써 map의 역할을 한다.
    - Subscriber를 구현하는 추상 클래스를 만들고 onNext는 추상메소드로 하여 해당 메소드만 정의하도록 리팩토링이 가능하다.

- 예시 2. reduce
  ```java
  private Publisher<Integer> reducePub(Publisher<Integer> pub, int init, BiFunction<Integer, Integer, Integer> function ){
    return new Publisher<Integer>() {
      @Override
      public void subscribe(Subscriber<? super Integer> subscriber) {
        pub.subscribe(new Subscriber<Integer>() {
          int result = init;
          @Override
          public void onSubscribe(Subscription subscription) {subscriber.onSubscribe(subscription);}
          @Override
          public void onNext(Integer item) {function.apply(result, item);}
          @Override
          public void onError(Throwable throwable) {subscriber.onError(throwable);}
          @Override
          public void onComplete() {subscriber.onNext(result); subscriber.onComplete();}
        });
      }
    };
  }
  ```
  - 인자로 넘겨준 publisher를 통해서 넘겨주는 데이터에 대하여 onNext 메서드를 통해 값을 계산하고, 마지막 onComplete를 통해 계산된 값을 넘겨주고 완료처리를 한다. 

- Generic을 활용하여 Subscriber 구현
  - 위 예시 1, 2에서 중복되는 onSubscribe, onError 메서드의 코드를 줄이기 위하여 Subscriber 구현
  - 필요시 오버라이딩 하여 사용
    ```java
    public class DelegateSubscriber<T, R> implements Subscriber<T> {
      Subscriber subscriber;
      public DelegateSubscriber(Subscriber<? super R> subscriber) { this.subscriber = subscriber; }
      @Override
      public void onSubscribe(Subscription subscription) { subscriber.onSubscribe(subscription); }
      public void onNext(T item) { subscriber.onNext(item); }
      @Override
      public void onComplete() { subscriber.onComplete(); }
      @Override
      public void onError(Throwable throwable) { subscriber.onError(throwable); }
    }
    ```
    
#### 다양한 Operator

- map
  - 1:1 변환
  - Stream의 map과 유사하다
  - 예시
    ```java
    Flux.just("a", "ab", "abc", "abcd")
      .map(str -> str.length())
      .subscribe(System.out::println);
    ```
    ```
    1
    2
    3
    4
    ```

- flatMap
  - 1:N 변환
  - 1개의 데이터에서 시퀀스를 생성할 때 사용하기 때문에 1:N 방식으로 변환을 처리한다.
  - 예시
    ```java
    Flux.just("a", "ab", "abc")
      .flatMap(str -> Flux.range(1, str.length()))
      .subscribe(System.out::println);    
    ```
    ```
    1 // "a"
    1 // "ab"
    2
    1 // "abc"
    2
    3
    ```

- filter
  - boolean타입 조건으로 true일 떄만 통과시킨다.
  - 예시
    ```java
    Flux.range(1, 10)
      .filter(data -> data % 3 == 0)
      .subscribe(System.out::println);      
    ```
    ```
    3
    6
    9
    ```
    
- defaultEmpty
  - 빈 시퀀스인 경우 기본 값 사용
  - 시퀀스에 데이터가 없을 때 특정 값을 기본으로 사용할 수 있다.
  - Mono, Flux 모두 제공한다.
  - 예시
    ```java
    Flux<String> popularCar = getPopularCar("BENZ").defaultEmpty("Empty");
    popularCar.subscribe(System.out::println);
    ```
    ```
    Empty
    ```
    - getPopularCar가 brand를 입력받아 해당 브랜드에 인기 차를 Flux타입으로 리턴할 때 defaultEmpty로 빈값일 경우에 기본값을 설정할 수 있다.
  
- switchIfEmpty
  - 빈 시퀀스인 경우 다른 시퀀스 사용
  - Flux와 Mono 모두 제공
  - 예시
    ```java
    Flux<String> emptyCar = Flux.just("Empty");
    Flux<String> popularCar = getPopularCar("BENZ").switchIfEmpty(emptyCar);
    popularCar.subscribe(System.out::println);
    ```
    ```
    Empty
    ```

- startWith
  - 특정 값으로 시작하는 시퀀스로 변환
  - 데이터나 시퀀스를 넣어줄 수 있다
  - 예제
    ```java
    Flux.range(3,3)
      .startWith(2)
      .startWith(Flux.range(0, 2)
      .subscribe(System.out::println);
    ```
    ```
    0
    1
    2
    3
    4
    5
    ```

- concatWithValues
  - 특정값으로 끝나도록 시퀀스를 변환한다.
  - 데이터를 넣어줄 수 있다.
  - 예제
    ```java
    Flux.range(1,3)
      .concatWithValues(4, 5)
      .subscribe(System.out::print);
    ```
    ```
    12345
    ```

- concatWith
  - 특정값으로 끝나도록 시퀀스를 변환한다.
  - 시퀀스를 넣어줄 수 있다.
  - 예제
    ```java
    Flux.range(1,3)
      .concatWith(Flux.range(4, 2))
      .subscribe(System.out::print);
    ```
    ```
    12345
    ```
    
- mergeWith
  - 시퀀스를 넣어준 순서가 아닌 시퀀스 발생하는 데이터 순서로 데이터를 섞는다.
  - 예제
    ```java
    Flux<String> first = Flux.interval(Duration.ofMillis(150)).take(3).map(item -> item + " first");
    Flux<String> second = Flux.interval(Duration.ofMillis(100)).take(3).map(item -> item + " second");
    
    first.mergeWith(second)
      .subscribe(System.out::println);
    Thread.sleep(500);
    ```
    ```
    0 second // 100mi
    0 first  // 150mi
    1 second // 200mi
    2 second // 300mi
    1 first  // 300mi    
    2 first  // 450mi
    ``` 
    
- zipWith

  ![image](https://user-images.githubusercontent.com/42403023/129035998-ddaf56f2-15ef-4868-bd62-dd9aa884d7a1.png)

  ** 이미지 출처: https://javacan.tistory.com/entry/Reactor-Start-4-tbasic-ransformation?category=699082
  
  - 시퀀스를 묶어서 사용할 수 있다.
  - 두 시퀀스의 값을 묶은 값 쌍을 생성하는 시퀀스를 생성할 수 있다.
  - 발생한 개수를 맞춰서 쌍을 만든다.
  - 예시
    ```java
    Flux<String> first = Flux.interval(Duration.ofMillis(150)).take(3).map(item -> item + " first");
    Flux<String> second = Flux.interval(Duration.ofMillis(100)).take(3).map(item -> item + " second");    
    first.zipWith(second)
      .subscribe(System.out::println);    
    Thread.sleep(500);
    ```
    ```
    [0 first,0 second]
    [1 first,1 second]
    [2 first,2 second]
    ```

- combineLatest
  
  ![image](https://user-images.githubusercontent.com/42403023/129036093-8a4e393c-5fdc-4c4b-878d-0fb491d65393.png)

  ** 이미지 출처: https://javacan.tistory.com/entry/Reactor-Start-4-tbasic-ransformation?category=699082
  - 두 시퀀스의 값을 묶은 값 쌍을 생성하는 시퀀스를 생성할 수 있다.
  - 가장 최근 데이터를 쌍으로 만든다.
  - 예시
    ```java
    Flux<String> first = Flux.interval(Duration.ofMillis(150)).take(10).map(item -> item + " first");
    Flux<String> second = Flux.interval(Duration.ofMillis(100)).take(10).map(item -> item + " second");    
    Flux.combineLatest(first, second, (a, b) -> a + ", " + b)
      .subscribe(System.out::println);
    Thread.sleep(1500);    
    ```
    ```
    0 first, 0 second
    0 first, 1 second
    0 first, 2 second
    1 first, 2 second
    1 first, 3 second
    2 first, 3 second
    2 first, 4 second
    2 first, 5 second
    3 first, 5 second
    3 first, 6 second
    4 first, 6 second
    4 first, 7 second
    5 first, 7 second
    5 first, 8 second
    5 first, 9 second
    6 first, 9 second
    7 first, 9 second
    8 first, 9 second
    9 first, 9 second
    ```

- take / takeLast
  - take(long): 시퀀스에서 처음 n개의 데이터만 유지할 경우 사용
  - take(Duration): 지정한 시간동안 발생한 데이터만 유지하고 싶을 때 사용
  - takeLast(long):  마지막 n개의 데이터만 유지하고 싶을 때 사용
  - takeWhile() / takeUntile(): Predicate를 인자로 받아 takeWhile은 true를 리턴하는 동안 데이터를 포함하고, takeUntil()은 true를 리턴할 때까지 데이터를 포함한다.

- skip / skipLast
  - skip(long): 시퀀스에서 처음 n개의 데이터를 거르고 싶을 때 사용한다. 
  - skip(Duration): 지정한 처음 시간 동안 발생한 데이터를 거르고 싶을 때 사용한다.
  - skipLast(long): 마지막 n개의 데이터를 거르고 싶을 때 사용한다.
  - skipWhile() / skipUntil(): Predicate를 인자로 받아 skipWhile은 true를 리턴하는 동안 데이터를 거르고, skipUntil()은 true를 리턴할 때까지 데이터를 거른다.
    
#### 출처

- 토비의 봄 TV 6회 스프링 리액티브 프로그래밍 (2) - Reactive Stream - Operator
- https://javacan.tistory.com/entry/Reactor-Start-4-tbasic-ransformation?category=699082
