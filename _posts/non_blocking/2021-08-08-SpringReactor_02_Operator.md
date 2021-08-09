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

Subscriber<Integer> sub = new Subscriber<Integer>() {
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
- Publischer -> \[Data1] -> Operation1 -> \[Data2] -> Operation2 -> \[Dara3] -> Subscriber

- 예시1: map
  - 1:1 구조로 기존에 있던 sequence를 변형하여 전달하는 역할을 한다.
    ```java
    public static void main(String[] args) {
      Publisher<Integer> publisher = iterPub(Stream.iterate(1, a->a+1).limit(10).collect(Collectors.toList()));
	    Publisher<Integer> mapPublisher = mapPub(publisher, s -> s * 10);
	    mapPublisher.subscribe(logSub());
	    return "ok";
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
    public class DelegateSubscriber<T, ㄲ> implements Subscriber<T> {
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
    
#### 출처

- 토비의 봄 TV 6회 스프링 리액티브 프로그래밍 (2) - Reactive Stream - Operator
