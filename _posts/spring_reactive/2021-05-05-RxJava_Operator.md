---
layout: post
title: RxJava, 연산자
summary: The Java
author: devhtak
date: '2021-05-05 21:41:00 +0900'
category: RxJava
---

#### RxJava의 Operator

- RxJava에서의 연산자는 메서드다.
- 연산자를 이용하여 데이터를 생성하고 통지하는 Flowable이나 Observable 등의 생산자를 생성할 수 있다.
- Flowable이나 Observable에서 통지한 데이터를 다양한 연산자를 사용하여 가공 처리하여 결과값을 만들어 낸다.
- 목차
  - Flowable/Observable 생성 연산자
  - 통지된 데이터를 필터링 해주는 연산자
  - 통지된 데이터를 변환해주는 연산자
  - 여러 개의 Flowable/Observable을 결합하는 연산자
  - 에러 처리 연산자
  - 유틸리티 연산자
  - 조건과 불린 연산자
  - 통지된 데이터를 집계해주는 연산자

#### Flowable/Observable 생성 연산자

- interval(Long initialDelay, Long period, TimeUnit timeUnit)
  ```java
  Observable.interval(0L, 1000L, TimeUnit.MILLISECONDS)
			.map(num -> num + " count")
			.subscribe(System.out::println);
		
  Thread.sleep(3000);
  ```
    - 독립된 스레드를 실행하기 때문에 main thread에 sleep을 사용
    
  - time interval 시간만큼 대기한 후에 반복적으로 계속 통지한다.
  - 완료없이 계속 통지한다.
  - 지정한 시간 간격마다 0부터 시작하는 숫자(Long)을 통지
  - initialDelay 파라미터를 이용하여 최초 통지에 대한 대기 시간을 지정할 수 있다.
  - 완료 없이 계속 통지한다.
  - 호출한 스레드와는 별도의 스레드에서 실행(독립된 스레드)
  - polling 용도의 작업을 수행할 때 활용할 수 있다.

- range(int start, int count)
  ```java
  Observable.range(0, 5)
			.subscribe(System.out::println);
  ```
  
  - 지정한 start 부터 count 개의 숫자를 통지한다.
  - for, while 등 반복문을 대체할 수 있다.

- timer(long time, TimeUnit timeUnit)
  ```java
  Observable.timer(2000, TimeUnit.MILLISECONDS)
			.map(count -> "Do work!")
			.subscribe(System.out::println);
		
  Thread.sleep(3000L);
  ```
    - 독립된 스레드를 실행하기 때문에 main thread에 sleep을 사용
    
  - 지정한 시간이 지나면 0을 통지한다.
  - 0을 통지하고 onComplete() 이벤트가 발생하여 종료한다.
  - 호출한 스레드와는 별도의 스레드에서 실행된다.
  - 특정 시간을 대기한 후에 어떤 처리를 하고자 할 때 활용할 수 있다.

- defer
  
  ```java
  Observable<LocalTime> observable = Observable.defer(() -> {
      LocalTime currentTime = LocalTime.now();
      return Observable.just(currentTime);
  });

  Observable<LocalTime> observableJust = Observable.just(LocalTime.now());

  observable.subscribe(time -> {
      System.out.println("# defer() 구독1의 구독 시간: " + time);
  });
  observableJust.subscribe(time -> {
      System.out.println("# just() 구독1의 구독 시간: " + time);
  });

  Thread.sleep(3000L);
  
  observable.subscribe(time -> {
      System.out.println("# defer() 구독2의 구독 시간: " + time);
  });
  observableJust.subscribe(time -> {
      System.out.println("# just() 구독2의 구독 시간: " + time);
  });
  ```
    - 2번째 구독 시간을 확인하면 defer에 경우 3초 지연 시간 이후에 찍히고, just는 처음 데이터를 전송한 후에 바로 전송한 것
    - 즉, just는 바로 데이터를 전송한 것이고, defer는 구독할 때 전송한 것이다.
  
  - 구독이 발생할 때마자 즉, subscribe()가 호출될 때마다 새로운 Observable을 생성한다.
  - 선언한 시점의 데이터를 통지하는 것이 아닌 호출 시점의 데이터를 통지한다.
  - 데이터 생성을 미루는 효과가 있기 때문에 최신 데이터를 얻고자할 때 활용할 수 있다.

- fromIterable
  ```java
  List<String> countries = Arrays.asList("Korea", "Canada", "Italy");
		
  Observable.fromIterable(countries)
			.subscribe(System.out::println);
  ```
  
  - Iterable 구현 객체를 파라미터로 입력 받아서 순서대로 통지하는 연산자이다.
  - Iterable에 담긴 데이터를 순서대로 통지한다.

- fromFuture()
  ```java
  public static void main(String[] args) {
      System.out.println("#start time");

      // 긴 처리가 걸리는 작업
      Future<Double> future = longTimeWork();

      // 짧은 처리 작업
      shortTimeWork();

      Observable.fromFuture(future)
          .subscribe(System.out::println);
	}
	
	public static Future<Double> longTimeWork() {
		  return CompletableFuture.supplyAsync(()-> calculate());
	}
	
	public static double calculate() {
      System.out.println("#긴 처리 시간이 걸리는 작업");
      try {
          Thread.sleep(3000L);
      } catch (InterruptedException e) {
          // TODO Auto-generated catch block
          e.printStackTrace();
      }

      return 10000000000L;
	}
	
	public static void shortTimeWork() {
		  System.out.println("#짧은 처리 시간이 걸리는 작업");
	}
  ```
  
  - Future 인터페이스는 자바 5에서 비동기 처리를 위해 추가된 동시성 API이다.
  - 시간이 오래 걸리는 작업은 Future를 반환하는 ExcutorService에게 맡기고 비동기로 다른 작업을 수행할 수 있다.
  - Java 8에서는 CompletableFuture 클래스를 통해 구현이 간결해졌다.
  - Future를 사용하면 여러 작업들을 비동기로 수행할 수 있다.

  - Future 학습

#### 출처

- Kevin의 알기쉬운 RxJava 1부
