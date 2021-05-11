---
layout: post
title: RxJava, Processor와 Subject
summary: RxJava
author: devhtak
date: '2021-05-11 21:41:00 +0900'
category: RxJava
---

#### Processor와 Subject란?

- Processor는 Reactive Streams에서 정의한 Publisher 인터페이스와 Subscriber 인터페이스를 둘 다 상속한 확장 인터페이스이다.
  ```java
  public interface Processor<T, R> extends Subscriber<T>, Publisher<R> {}
  ```
- 즉, Processor는 Publisher(생산자)의 기능과 Subscriber(소비자)의 기능을 모두 가지고 있다.
- Processor는 Hot Publisher(뜨거운 생산자)이다.
- Subject는 Reactive Streams의 Processor와 동일한 기능을 하나 배압 기능이 없는 추상 클래스
  - cf) Flowable vs Observable 차이점도 배압 기능이었다. 
- Processor와 Subject의 구현 클래스에는 다음과 같은 클래스가 있다
  - PublishProcessor / PublishSubject
  - AsyncProcessor / AsyncSubject
  - BehaviorProcessor / BehaviorSubject
  - ReplayProcessor / ReplaySubject

#### PublishSubject

- 구독 전에 통지된 데이터는 받을 수 없고, 구독한 이후에 통지된 데이터만 받을 수 있다.
- 데이터 통지가 완료 된 이후에 소비자가 구독하면 완료 또는 에러 통지를 받는다.
- 예제
  ```java
  PublishSubject<Integer> subject = PublishSubject.create();
		
  subject.subscribe(price -> System.out.println("# 소비자 1: " + price));
  subject.onNext(3500);

  subject.subscribe(price -> System.out.println("# 소비자 2: " + price));
  subject.onNext(3400);

  subject.subscribe(price -> System.out.println("# 소비자 3: " + price));
  subject.onNext(3300);

  subject.subscribe(
      price -> System.out.println("# 소비자 4: " + price),
      error -> System.out.println("error: " + error),
      () -> System.out.println("On Complete")
  );
  
  subject.onComplete();
  ```
  ```
  // 출력
  # 소비자 1: 3500
  # 소비자 1: 3400
  # 소비자 2: 3400
  # 소비자 1: 3300
  # 소비자 2: 3300
  # 소비자 3: 3300
  On Complete
  ```
  -  전형적인 Hot Publisher에 모습을 보여주고 있다.
  

#### AsyncSubject

- 완료 전까지 아무것도 통지하지 않고 있다가 완료했을 때 마지막으로 통지한 데이터와 완료만 통지한다.
- 모든 소비자는 구독 시점에 상관없이 마지막으로 통지된 데이터와 완료 통지만 받는다.
- 완료 후에 구독한 소비자라도 마지막으로 통지된 데이터와 완료 통지를 받는다.

- 예시
  ```java
  AsyncSubject<Integer> subject = AsyncSubject.create();
		
  subject.onNext(1000);
  subject.doOnNext(price -> System.out.println("#doOnNext.소비자1: " + price))
      .subscribe(price -> System.out.println("#subscribe.소비자1: " + price));

  subject.onNext(2000);
  subject.doOnNext(price -> System.out.println("#doOnNext.소비자2: " + price))
      .subscribe(price -> System.out.println("#subscribe.소비자2: " + price));

  subject.onNext(3000);
  subject.doOnNext(price -> System.out.println("#doOnNext.소비자3: " + price))
      .subscribe(price -> System.out.println("#subscribe.소비자3: " + price));

  subject.onNext(4000);
  subject.onComplete();

  subject.doOnNext(price -> System.out.println("#doOnNext.소비자4: " + price))
      .subscribe(price -> System.out.println("#subscribe.소비자4: " + price));
  ```
  ```
  // 출력
  #doOnNext.소비자1: 4000
  #subscribe.소비자1: 4000
  #doOnNext.소비자2: 4000
  #subscribe.소비자2: 4000
  #doOnNext.소비자3: 4000
  #subscribe.소비자3: 4000
  #doOnNext.소비자4: 4000
  #subscribe.소비자4: 4000
  ```
  - 소비자 1, 2, 3, 4 에게 맨 마지막 데이터인 4000만 전송된다.
  - onComplete가 호출된 후에 구독한 소비자 4에게도 데이터가 전송된다.

#### Behavior Subject

- 구독 시점에 이미 통지된 데이터가 있다면 이미 통지된 데이터의 마지막 데이터를 전달 받은 후, 구독 이후에 통지된 데이터를 전달받는다.
- 처리가 완료된 이후에 구독하면 완료나 에러 통지만 전달 받는다.
- 예시
  ```java
  BehaviorSubject<Integer> subject = BehaviorSubject.createDefault(3000);		
  subject.subscribe(price -> System.out.println("#소비자 1: " + price));
  subject.onNext(3500);

  subject.subscribe(price -> System.out.println("#소비자 2: " + price));
  subject.onNext(3300);

  subject.subscribe(price -> System.out.println("#소비자 3: " + price));
  subject.onNext(3400);
  ```
  ```
  // 출력
  #소비자 1: 3000
  #소비자 1: 3500
  #소비자 2: 3500
  #소비자 1: 3300
  #소비자 2: 3300
  #소비자 3: 3300
  #소비자 1: 3400
  #소비자 2: 3400
  #소비자 3: 3400
  ```
  - 소비자2는 구독하기 이전인 3500을 구독했고, 소비자3은 구독하기 이전인 3400을 구독했다.
  
#### ReplaySubject

- 구독 시점에 이미 통지된 데이터가 있다면 이미 통지된 데이터 중에서 최근 통지된 데이터를 지정한 개수만크 전달 받은 후, 구독 이 후에 통지된 데이터를 전달 받는다.
- 이미 처리가 완료된 이후에 구독하더라도 지정한 개수 만큼의 최근 통지된 데이터를 전달 받는다.

#### 출처

Kevin의 알기 쉬운 RxJava 2부
