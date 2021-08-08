---
layout: post
title: WebFlux, Operator
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
        iter.forEach(i -> subscriber.onNext(i));
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


#### 출처

- 토비의 봄 TV 6회 스프링 리액티브 프로그래밍 (2) - Reactive Stream - Operator
