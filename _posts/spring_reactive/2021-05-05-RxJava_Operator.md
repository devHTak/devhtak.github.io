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

#### 데이터 필터링 연산자

- filter
  - 전달받은 데이터가 조건에 맞는지 확인한 후 결과가 true인 데이터만 통지한다.
  - filter라는 단어의 산전적인 의미가 무언가를 걸러낸다는 의미
  - 파라미터로 받는 Predicate 함수형 인터페이스에서 조건을 확인

  ```java
  Observable.fromIterable(SampleData.carList)
      .filter(car -> car.getCarMaker() == CarMaker.CHEVOLET)
      .subscribe(System.out::println);
  ```
  
- distinct
  - 이미 통지된 동일한 데이터가 있다면 이후의 동일한 데이터는 통지하지 않는다.
  - distinct의 사전적 의미는 명확하게 구별되는 이라는 뜻을 포함하고 있다.

  ```java
  Observable.fromArray(SimpleData.carMakersDuplicated)
      .distinct(Car::getCarMaker)
      .subscribe(System.out::println);
  ```
    - distinct 내에 람다를 입력할 수 있는데, 입력한 값에 중복을 확인한다.

- take
  - 파라미터로 지정한 개수나 기간이 될 때까지 데이터를 통지한다.
  - 지정한 범위가 통지 데이터보다 클 경우 데이터를 모두 통지하고 완료한다.
  ```java
  Observable.just("a", "b", "c", "d")
      .take(2)
      .subscribe(System.out::println);
      
  Observable.interval(1000L, TimeUnit.MILLISECONDS)
      .take(3500L, TimeUnit.MILLISECONDS)
      .subscribe(System.out::println);
  Thread.sleep(3500L);
  ```
    - a, b 만 전달한다
    - 시간을 지정할 수 있다.

- takeUntil 첫번째 유형
  - 파라미터로 지정한 조건이 true가 될 때까지 데이터를 계속 통지한다.
  - true가 된 데이터까지 전송한 후에 완료한다.
  ```java
  Observable.fromIterable(SampleData.carList)
      .takeUntil(car -> car.getName().equals("트랙스"))
      .subscribe(System.out::println);
  ```
    - 데이터 중 트랙스 데이터가 있을 때까지 데이터가 전송된다.
    - 말리부, K3, Q3, 트랙스 출력

- takeUntil 두번째 유형

  ![image](https://user-images.githubusercontent.com/42403023/117156331-859bb180-adf8-11eb-9d9a-b9adf221ae5c.png)

  - 파라미터로 지정한 Observable이 최초 데이터를 통지할 때까지 데이터를 계속 통지한다.
  ```java
  Observable.interval(1000L, TimeUnit.MILLISECONDS)
      .takeUntil(Observable.timer(5500L, TimeUnit.MILLISECONDS)
      .subscribe(System.out::println);
  Thread.sleep(5500L);
  ```
    - 새로운 Observable에 timer를 5500로 세팅했기 때문에 5초까지 데이터가 전송된다.
    - 5.5초에 파라미터로 넘긴 Observable에 데이터가 전송되기 때문에 완료된다.

- skip 첫번째 유형
  - 파라미터로 지정한 숫자만큼 데이터를 건너뛴 후 나머지 데이터를 통지한다.
  ```java
  Observable.range(1, 15)
      .skip(3)
      .subscribe(System.out::println);
  ```
  - 1, 2, 3은 skip하고 4부터 출력된다.

- skip 두번째 유형
  - 파라미터로 지정한 시간만큼 데이터를 건너뛴 후 지정한 시간 이 후에 나머지 데이터를 통지한다.
  ```java
  Observable.interval(300L, TimeUnit.MILLISECONDS)
      .skip(1000L, TimeUnit.MILLISECONDS)
      .subscribe(System.out::println);
  Thread.sleep(5500L);
  ```
  - 0.3, 0.6, 0.9 초까지는 건너뛴 후, 1.2초부터 3 데이터가 출력된다.

- 추가: takeWhile, skipUntil, skipLast

#### 데이터 변환 연산자

- map
  - 원본 Observable 에서 통지하는 데이터를 원하는 값으로 변환 후 통지한다.
  - 변환 전, 후 데이터 타입은 달라도 상관없다.
  - null을 반환하면 NullpointException이 발생하므로 데이터 하나를 반드시 반환해야 한다.
  - 예제
    ```java
    List<Integer> oddList = Arrays.asList(1, 3, 5, 7);
    Observable.fromIterable(oddList)
	.map(data -> data + 1)
	.subscribe(System.out::println);
    ```

- flatMap 첫번째 유형
  - 원본 데이터를 원하는 값으로 변환 후 통지하는 것은 map과 같다
  - map이 1대 1 변환인 것과 달리 flatMap은 1대 다 변환이므로 데이터 한개로 여러 데이터를 통지할 수 있다.
  - map은 변환된 데이터를 반환하지만 flatMap은 변환된 여러개의 데이터를 담고 있는 새로운 Observable을 반환
  - 예제
    ```java
    Observable.just("Hello")
	.flatMap(hello -> Observable.just("Java", "JPA", "RxJava", "Spring").map(lang -> hello + " " + lang))
	.subscribe(System.out::println);
    ```

- flatMap 두번째 유형
  - 원본 데이터와 변환된 데이터를 조합하여 새로운 데이터를 통지
  - 즉, Observable에 원본 + 변환 = 최종 데이터를 실어서 반환
  - 예제
    ```java
    Observable.range(2, 1)
        .flatMap(
            data -> Observable.range(1, 9),
            (source, transform) -> {
	        return source + " * " + transform + " = " + (source*transform);
            }
    )
    .subscribe(System.out::println);
    ```
    
- concatMap
  - flatMap과 마찬가지로 받은 데이터를 변환하여 새로운 Observable로 반환
  - 반환된 새로운 Observable을 하나씩 순서대로 실행하는 것이 flatMap과 다르다.
  - 즉, 데이터의 처리 순서는 보장하지만 처리중인 Observable의 처리가 끝나야 다음 Observable이 실행되므로 처리 성능에 영향을 줄 수 있다.
  - 예제
    ```java
    Observable.interval(100L, TimeUnit.MILLISECONDS)
        .take(4)
        .skip(2)
        .concatMap(num -> Observable.interval(200L, TimeUnit.MILLISECONDS).take(10).skip(1)
            .map(row -> num + " * " + row + " = " + (row*num)))
        .subscribe(System.out::println, error->{}, ()-> System.out.println("OnComplete"));

    Thread.sleep(5000L);
    ```
    - concatMap을 사용하면 concatMap 안에 Observable에서 보내는 순서를 보장한다.
    - flatMap을 사용하면 flatMap 안에 Observable에서 보내는 순서를 보장하지 않는다.

- switchMap
  - concatMap과 마찬가지로 받은 데이터를 변환하여 새로운 Observable로 반환
  - concatMap과 다른 점은 switchMap은 순서를 보장하지만 새로운 데이터가 통지되면 현재 처리중이던 작업을 바로 중단한다.
  - 예제
    ```java
    Observable.interval(100L, TimeUnit.MILLISECONDS)
        .take(4)
        .skip(2)
        .switchMap(num -> Observable.interval(200L, TimeUnit.MILLISECONDS).take(10).skip(1)
            .map(row -> num + " * " + row + " = " + (row*num)))
        .subscribe(System.out::println, error->{}, ()-> System.out.println("OnComplete"));

    Thread.sleep(5000L);
    ```
    - 2단은 출력되지 않고, 3단은 출력된다.
    - 2단 출력될 시점에 3단관련 데이터를 전송하여 중단되었다.

- groupBy
  - 하나의 Observable을 여러개의 새로운 GroupedByObservable로 만든다.
  - 원본 Observable의 데이터를 그룹별로 묶는다기보다는 각각의 데이터들이 그룹에 해당하는 Key를 가지게 된다.
  - GropuedByObsrevable은 getKey()를 통해 구분된 그룹을 알 수 있게 해준다.
  - 예제
    ```java
    Observable<GroupedObservable<CarMaker, Car>> observable = 
        Observable.fromIterable(SampleData.carList).groupBy(car -> car.getCarMaker());

    observable.subscribe(groupedObservable -> gropuedObservable
        .filter(data -> gropuedObservable.getKey().equals(CarMaker.CHEVOLET)
        .subscribe(data -> {
            System.out.println(groupedObservable.getKey() + " " + data); // key가 Carmaker()가 된다
    ));
    ```

- toList
  - 통지되는 데이터를 모두 List에 담아 통지한다.
  - 원본 Observable에서 완료 통지를 받는 즉시 리스트를 통지한다.
  - 통지되는 데이터는 원본 데이터를 담은 리스트 하나이므로 Single로 반환된다.
  - 예시
    ```java
    Single<List<Integer>> single = Observable.just(1, 3, 5, 7, 9).toList();
    
    single.subscribe(System.out::println);
    ```
    ```java
    Observable.fromIterable(SampleData.carList)
        .toList()
	.subscribe(System.out::println);
    ```
    - 1, 3, 5, 7, 9가 리스트로 한번만 전송된다.
    - carList가 하나씩 전송되는 것이 아닌 리스트로 한번만 전송된다.

- toMap
  - 통지되는 데이터를 모두 Map에 담아 통지한다.
  - 원본 Observable에서 완료 통지를 받는 즉시 Map을 통지한다.
  - 이미 사용중인 key(키)를 또 생성하면 기존에 있던 key(키)와 value(값)을 덮어쓴다.
  - 통지되는 데이터는 원본 데이터를 담은 Map하나이므로 Single로 반환된다.
  - 예시
    ```java
    Single<Map<String, String>> single = 
        Observable.just("a-Alpha", "b-Beta", "c-Charlie", "e-Echo")
            .toMap(
	        data -> data.split("-")[0],  // 반환값이 key가 된다.
		data -> data.split("-")[1]   // 반환값이 value가 된다.
            );
    single.subscribe(System.out::println);    
    ```
    - a: Alpha .. 식의 key, value 형태의 Map을 한번에 전송한다.
    - toMap의 key만 파라미터로 넣을 수 있다. (key or key, value) 

#### 데이터 결합 연산자

- merge
  - 다수의 Observable에서 통지된 데이터를 받아 다시 하나의 Observable로 통지한다.
  - 통지 시점이 빠른 Observable의 데이터부터 순차적으로 통지되고 통지 시점이 같을 경우에는 merge() 함수의 파라미터로 먼저 지정된 Observable 데이터로부터 통지된다.
  - 예시
  
    |-|0.2s|0.4s|0.6s|0.8s|1s|1.2s|1.4s|1.6s|1.8s|2s|
    |---|---|---|---|---|---|---|---|---|---|---|
    |Observable1|0|1|2|3|4|-|-|-|-|-|
    |Observable2|-|1000|-|1001|-|1002|-|1003|-|1004|
    |Observable|0|1,1000|2,|3,1001|4|1002|-|1003|-|1004|
    
    ```java
    Observable<Long> observable1 = Observable.interval(200L, TimeUnit.MILLISECONDS)
        .take(5);
    Observable<Long> observable2 = Observable.interval(400L, TimeUnit.MILLISECONDS)
        .take(5)
        .map(data -> data + 1000);
    Observable.merge(observable1, observable2)
        .subscribe(System.out::println);
    Thread.sleep(4000);
    ````
    
- concat
  - 다수의 Observable에서 통지된 데이터를 받아 다시 하나의 Observable로 통지
  - 하나의 Observable에서 통지가 끝나면 다음 Observable에서 연이어 통지가 된다.
  - 각 Observable의 통지시점과 상관없이 concat() 함수의 파라미터로 먼저 입력된 Observable의 데이터부터 모두 통지된 후, 다음 Observable의 데이터가 통지된다.
  - 예시
    ```java
    Observable<Long> observable1 = Observable.interval(500L, TimeUnit.MILLISECONDS)
        .take(4);
    Observable<Long> observable2 = Observable.interval(300L, TimeUnit.MILLISECONDS)
        .take(5)
        .map(data -> data + 1000);
    Observable.concat(observable1, observable2)
        .subscribe(System.out::println);
    Thread.sleep(3500L);	
    ```
    - observable2가 먼저 통지되지만 observable1이 먼저 파라미터로 입력되었기 때문에 observable1이 먼저 통지된다.
    - 0, 1, 2, 3, 1000, 1001, 1002, 1003 순으로 입력된다.
    - concat에 파라미터로 List와 같은 Collection도 가능하다

- zip
  - 다수의 Observable에서 통지된 데이터를 받아 다시 하나의 Observable로 통지
  - 각 Observable에서 통지된 데이터가 모두 모이면 각 Observable에서 동일한 index의 데이터로 새로운 데이터를 생성한 후 통지한다.
  - 통지하는 데이터 개수가 가장 적은 Observable의 통지 시점에 완료 통지 시점을 맞춘다.
  - 예시
    ```java
    Observable<Long> observable1 = Observable.interval(200L, TimeUnit.MILLISECONDS)
        .take(4);

    Observable<Long> observable2 = Observable.interval(400L, TimeUnit.MILLISECONDS)
        .take(6)
        .map(data -> data + 1000);

    Observable.zip(observable1, observable2, (data1, data2) -> data1 + data2)
        .subscribe(System.out::println);

    Thread.sleep(3500L);
    ```
    - 4개만 출력된다. observable1에 개수가 적기 때문에
    - 1000, 1002, 1004, 1006 이 출력된다.
    
- combineLatest
  - 다수의 Observable에서 통지된 데이터를 받아 다시 하나의 Observable로 통지
  - 각 Observable에서 데이터를 통지할 때마다 모든 Observable에서 마지막으로 통지한 각 데이터를 함수형 인터페이스에 전달하고, 새로운 데이터를 생성해 통지한다.
  - 예제
    ```java
    Observable<Long> observable1 = Observable.interval(500L, TimeUnit.MILLISECONDS)
        .take(4);
    Observable<Long> observable2 = Observable.interval(700L, TimeUnit.MILLISECONDS)
        .take(4);
    Observable.combineLatest(observable1, observable2,
            (data1, data2) -> "data1: " + data1 +", data2: " + data2)
        .subscribe(System.out::println);
    Thread.sleep(3500L);
    ```
    ```
    // 출력
    data1: 0, data2: 0
    data1: 1, data2: 0
    data1: 1, data2: 1
    data1: 2, data2: 1
    data1: 3, data2: 1
    data1: 3, data2: 2
    data1: 3, data2: 3
    ```
    - 0.7s에 observable2에는 0이 마지막으로 도착하고,  observable1에는 0이 마지막으로 도착했다.
    - 1.0s에 observable1에는 1이 마지막으로 도착하고,  observable2에는 0이 마지막으로 도착했다.
    - 1.4s에 observable2에는 1이 마지막으로 도착하고,  observable1에는 1이 마지막으로 도착했다.
    - ... 이런 방식으로 계속 출력되는 것

#### 에러 처리 연산자

- RxJava에서는 Try-Catch를 할 수 있다.
  ```java
  try {
      Observable.just(2)
          .map(num -> num / 0)
	  .subscribe(System.out::println);
  } catch(Exception e) {
      System.out.println("에러 처리 필요: " + e.getMessage()); // 출력되지 않는다.
  }
  ```
  - divide zero로 오류가 발생할 수 있지만, catch 문을 타지 않는다.
  - console에 찍힌 오류문을 확인해보면 subscribe에 onError 문을 작성하지 않았다는 내용이 적혀 있다.

- RxJava에서는 onError를 작성하여야 한다.
  ```java
  Observable.just(5)
      .flatMap(num -> {
          return Observable.interval(200L, TimeUnit.MILLISECONDS)
              .doOnNext(data-> System.out.println("do on next: " + data))
              .take(5)
              .map(i -> num / i);
      }) 
      .subscribe( 
              data -> System.out.println("on next: " + data),
              error -> System.out.println("error: " + error),
              () -> System.out.println("on complete!"));
  Thread.sleep(1000L);
  ```
  - do on next에서 0이 통지한다. map에서 divide zero exception이 발생하고, subscribe에서 error를 타고 종료한다.

- onErrorReturn
  - 에러가 발생했을 때 에러를 의미하는 데이터로 대체할 수 있다.
  - onErrorReturn()을 호출하면 onError 이벤트는 발생하지 않는다.
  - 예제
    ```java
    Observable.just(5)
    .flatMap(num -> {
        return Observable.interval(200L, TimeUnit.MILLISECONDS)
            .take(5)
            .map(i -> num / i)
            .onErrorReturn(exception -> {
                if(exception instanceof ArithmeticException)
                    System.out.println("ArithmeticException: " + exception.getMessage());
                return -1L;
            });
    }) 
    .subscribe( 
            data -> System.out.println("on next: " + data),
            error -> System.out.println("error: " + error),
            () -> System.out.println("on complete!"));
    Thread.sleep(1000L);
    ```
    ```
    // 출력
    ArithmeticException: / by zero
    on next: -1
    on complete!
    ```

- onErrorResumeNext
  - 에러가 발생했을 때 에러를 의미하는 Observable로 대체할 수 있다.
  - Observable로 대체할 수 있으므로 데이터 교체와 더불어 에러 처리를 위한 추가 작업을 할 수 있다.
  - 예제
    ```java
    Observable.just(5)
        .flatMap(num -> {
            return Observable.interval(200L, TimeUnit.MILLISECONDS)
                .take(5)
                .map(i -> num / i)
                .onErrorResumeNext(exception -> {
                    System.out.println("EXCEPTION: " + exception.getMessage());
                    return Observable.interval(200L, TimeUnit.MILLISECONDS)
                        .take(5).skip(1).map(i -> num / i);
                });
        }) 
        .subscribe(data -> System.out.println("on next: " + data));
    Thread.sleep(2000L);
    ```
    ```
    // 출력
    EXCEPTION: / by zero
    on next: 5
    on next: 2
    on next: 1
    on next: 1
    ```
    - Exception이 발생하여도 onErrorResumeNext에서 전달한 Observable에 데이터가 전송된다.

- retry
  - 데이터 통지 중 에러가 발생했을 때, 데이터 통지를 재시도한다.
  - 즉, onError 이벤트가 발생하면 subscribe()를 다시 호출하여 재구독한다.
  - 에러가 발생한 시점에 통지에 실패한 데이터만 다시 통지되는 것이 아니라 처음부터 다시 통지된다.
  - 예제
    ```java
    Observable.just(10, 12, 15, 16)
        .zipWith(Observable.just(1, 2, 0, 4), (a, b) -> {
            int result;
            try {
                result = a / b;
                return result;
            } catch(ArithmeticException e) {
                System.out.println(e.getMessage());
                throw e;
            }
        })
        .retry(3)
        .onErrorReturn(throwable -> -1)
        .subscribe(
            data -> System.out.println("ON_NEXT: " + data),
            error -> System.out.println("ON_ERROR: " + error),
            ()-> System.out.println("ON_COMPLETE")
    );
    ```
    ```
    // 출력
    ON_NEXT: 10
    ON_NEXT: 6
    / by zero
    ...
    // 3번 반복
    ...
    ON_NEXT: -1
    ON_COMPLETE
    ```
    - EXCEPTION이 발생한 후 retry를 3번 반복하여 발생한다.
    - 시도한 후에 onErrorReturn을 통해 -1을 전송한다.
    - Exception이 발생한 시점부터 다시 값을 보내는 것이 아닌, 처음부터 값을 전송한다.

#### 유틸리티 연산자

- delay 첫번째 유형
  - 생산자가 데이터를 생성 및 통지를 하지만 설정한 시간만큼 소비자쪽으로의 데이터 전달을 지연시킨다.
  - 예시
    ```java
    Observable.just(1, 3, 4, 5)
        .doOnNext(data -> System.out.println("DO ON NEXT: " + data))
        .delay(2000L, TimeUnit.MILLISECONDS)
        .subscribe(data -> System.out.println("ON NEXT: " + data));

    Thread.sleep(2500L);
    ```
    - 2초 정도 delay가 지난 후에 ON NEXT에 수신된다.

- delay 두번째 유형
  - 파라미터로 생성되는 Observable이 데이터를 통지할 때까지 각각의 원본 데이터의 통지를 지연시킨다.
  - 예시
    ```java
    Observable.just(1, 3, 5, 7)
        .delay(item -> {
            Thread.sleep(1000L);
            return Observable.just(item);
        }).subscribe(data-> System.out.println("ON NEXT: " + data));
    ```

- delaySubscription
  - 생산자가 데이터의 생성 및 동지 자체를 설정한 시간만큼 지연시킨다.
  - 즉, 소비자가 구독을 해도 구독 시점 자체가 지연된다.
  - delay에 경우 구독 즉시 호출되지만, 구독 시점은 delay 시간 후에 진행된다.
  - delaySubscription에 경우 delay 시간 후에 호출되고 바로 구독된다.
  - 예시
    ```java
    Observable.just(1, 3, 4, 5)
        .doOnNext(data -> System.out.println("DO ON NEXT: " + data))
        .delaySubscription(2000L, TimeUnit.MILLISECONDS)
        .subscribe(data -> System.out.println("ON NEXT: " + data));

    Thread.sleep(2500L);
    ```

- timeout
  - 각각의 데이터 통지 시, 지정된 시간안에 통지가 되지 않으면 에러를 통지한다.
    - Timeout 시간 안에 1번에 호출이 발생해야 한다.
  - 에러 통지 시 전달되는 에러 객체는 TimeoutException이다.
  - 예시
    ```java
    Observable.range(1, 5)
        .map(num -> {
            long time = 1000L;
            if(num == 4) {
                time = 1500L;
            }
            Thread.sleep(time);
            return num;
        })
        .timeout(1200L, TimeUnit.MILLISECONDS)
        .subscribe(
            data -> System.out.println("ON_NEXT: " + data),
            error -> System.out.println("ON_ERROR: " + error)
        );
    ```
    ```
    // 출력
    ON_NEXT: 1
    ON_NEXT: 2
    ON_NEXT: 3
    ON_ERROR: java.util.concurrent.TimeoutException:
    ```
    - 1, 2, 3 을 진행할 때에는 1초 간격으로 값을 던졌기 때문에 1.2초 내에 timeout 시간 안에 동작되었다.
    - 4를 전송할 차례에서는 1.2초 이내에 진행되지 않아 오류가 발생된다.

- timeInterval
  - 각각의 데이터가 통지되는 데 걸린 시간을 통지한다.
  - 통지된 데이터와 데이터가 통지되는 데 걸린 시간을 소비자쪽에서 모두 처리할 수 있다.
  - 예시
    ```java
    Observable.just(1, 3, 5, 7, 9)
        .delay(item -> {
            Thread.sleep((int)(Math.random()*1000));
            return Observable.just(item);
        })
        .timeInterval()
        .subscribe(timed -> System.out.println("통지 걸린 시간:" + timed.time() +", 데이터: " + timed));
    ```
    - 보내준 객체로 value, interval time 등을 알 수 있다.
    - 출력
      ```
      통지 걸린 시간:148, 데이터: Timed[time=148, unit=MILLISECONDS, value=1]
      통지 걸린 시간:11, 데이터: Timed[time=11, unit=MILLISECONDS, value=3]
      통지 걸린 시간:197, 데이터: Timed[time=197, unit=MILLISECONDS, value=5]
      통지 걸린 시간:222, 데이터: Timed[time=222, unit=MILLISECONDS, value=7]
      통지 걸린 시간:109, 데이터: Timed[time=109, unit=MILLISECONDS, value=9]
      ```      

- materialize / dematerialize
  - materialize: 통지된 데이터와 통지된 데이터의 통지 타입 자체를 Notification 객체에 담고 이 Notification 객체를 통지한다.
    - 즉, 통지 데이터의 메타 데이터를 포함하여 통지한다고 볼 수있다.
  - dematerialize: 통지된 Notification 객체를 원래의 통지 데이터로 변환하여 통지한다.
  - materialize 예시
    ```java
    Observable.just(1, 2, 3)
        .materialize()
        .subscribe(notification-> {
            String notificationType = 
                notification.isOnNext() ? "onNext()" : 
                    (notification.isOnError() ? "onError()" : "onComplete()");
            System.out.println("notification 타입: " + notificationType);
            System.out.println("on next: " + notification.getValue());
        });
    ```
    ```
    // 출력
    notification 타입: onNext()
    on next: 1
    notification 타입: onNext()
    on next: 2
    notification 타입: onNext()
    on next: 3
    notification 타입: onNext()
    notification 타입: onComplete()
    on next: null
    ```
    
#### 조건과 불린 연산자

- all
  - 통지되는 모든 데이터가 설정한 조건에 맞는지를 판단한다.
  - 결과 값을 한번만 토지하면 되기때문에 true/false 값을 Single로 반환한다.
  - 통지된 데이터가 조건에 맞지 않는다면 이후 데이터는 구독 해지되어 통지되지 않는다.
  - 예시
    ```java
    Observable.just(1, 3, 5, 7, 8, 10)
        .doOnNext(num -> System.out.println("Do on next: " + num))
        .all(num -> num < 6)
        .subscribe(data -> System.out.println("Do on next: " + data));
    ```
    ```
    Do on next: 1
    Do on next: 3
    Do on next: 5
    Do on next: 7
    on next: false
    ```
    - false 조건이 나올 때까지는 정상 전달된다.
    - false가 되면 Single<Boolean> 타입으로 전송된다. 

- amb
  - 여러 개의 Observable 중 최초 통지 시점이 가장 빠른 Observable의 데이터만 통지되고, 나머비 Observable은 무시된다.
  - 즉, 가장 먼저 통지를 시작한 Observable의 데이터만 통지된다.
  - 예시
    ```java
    List<Observable<Integer>> observables = Arrays.asList(
        Observable.just(1, 3, 5, 7)
            .delay(100L, TimeUnit.MILLISECONDS).doOnComplete(()->System.out.println("completeA")),
        Observable.just(2, 4, 6, 8)
            .delay(200L, TimeUnit.MILLISECONDS).doOnComplete(()->System.out.println("completeB")),
        Observable.just(10, 11, 12, 13)
            .delay(300L, TimeUnit.MILLISECONDS).doOnComplete(()->System.out.println("completeC"))	
    );

    Observable.amb(observables)
        .doOnComplete(()-> System.out.println("COMPLETE"))
        .subscribe(data->System.out.println("NEXT: " + data));
    Thread.sleep(1000L);
    ```
    ```
    // 출력
    NEXT: 1
    NEXT: 3
    NEXT: 5
    NEXT: 7
    completeA
    COMPLETE
    ```
    - 가장 먼저 도착한 홀수 데이터를 갖는 Observable만 통지된다.	

- contains
  - 파라미터의 데이터가 Observable에 포함되어 있는지를 판단한다.
  - 결과값을 한번만 통지하면 되기 때문에 t/f 값은 Single로 반환한다.
  - 결과 통지 시점은 Observable에 포함된 데이터를 통지하거나 완료를 통지할 때이다
  - 예시
    ```java
    Observable.just(1, 3, 5, 7)
        .doOnNext(data -> System.out.println("next: " + data))
        .contains(3)
        .subscribe(data -> System.out.println("ON NEXT: " + data))
    ```
    ```
    //출력
    next: 1
    next: 3
    ON NEXT: true
    ```
    - contains에 부합되는 3이 통지된 후에 true를 리턴하고 종료된다.
    
    ```java
    Observable.just(1, 2, 5, 7)
        .doOnNext(data -> System.out.println("next: " + data))
        .contains(3)
        .subscribe(data -> System.out.println("ON NEXT: " + data));
    ```
    
    ```
    // 출력
    next: 1
    next: 2
    next: 5
    next: 7
    ON NEXT: false
    ```
    
    - 전체를 통지 받은 후 포함되어 있지 않다면 false를 Single 타입으로 보낸다.

- defaultIfEmpty
  - 통지할 데이터가 없을 경우 파라미터로 입력된 값을 통지한다.
  - 즉, 연산자 이름 의미 그대로 Observable에 통지할 데이터가 없이 비어 있는 상태일 때 디폴트 값을 통지한다.
  - 예시
    ```java
    Observable.just(1, 2, 3, 4, 5) 
        .filter(num -> num > 10)
        .defaultIfEmpty(10)
        .subscribe(System.out::println);
    ```
    - 10이 출력된다.

- sequenceEqual
  - 두 Observable이 동일한 순서로 동일한 갯수의 같은 데이터를 통지하는지 판단한다.
  - 통지 시점과 무관하게 데이터의 정합성만 판단하므로 통지 시점이 다르더라도 조건이 맞는다면 true를 통지한다.
  - 예시
    ```java
    Observable<Integer> observable1 = Observable.just(1, 3, 5, 7, 9)
        .subscribeOn(Schedulers.computation())
        .delay(num -> {
            Thread.sleep(500L);
            return Observable.just(num);
        }).doOnNext(num -> System.out.println("observable1: " + num));

    Observable<Integer> observable2 = Observable.just(1, 3, 5, 6, 9)
        .subscribeOn(Schedulers.computation())
        .delay(num -> {
            Thread.sleep(700L);
            return Observable.just(num);
        }).doOnNext(num -> System.out.println("observable2: " + num));

    Observable.sequenceEqual(observable1, observable2)
        .subscribe(data-> System.out.println("ON NEXT: " + data));
    Thread.sleep(6000L);
    ```
    - 7과 6이 다르기 때문에 false를 반환한다.


#### 데이터 집계 연산자

- count
  - Observable이 통지한 데이터의 총 개수를 통지한다.
  - 총 개수만 통지하면 되므로 결과값은 Single로 반환한다.
  - 데이터의 총 개수를 통지하는 시점은 완료 통지를 받은 시점이다.
  - 여러 Observable을 사용하면 모든 Observable의 데이터 개수를 확인한다. 
  - 예시
    ```java
    Observable.concat(Arrays.asList(
            Observable.just(1, 3, 5, 7, 9),
            Observable.just(2, 4, 6, 8),
            Observable.just(10, 11, 12)
        ))
        .count()
        .subscribe(data-> System.out.println("ON NEXT: " + data));
    ```
    - 12가 출력된다.
        
- reduce
  - Observable이 통지한 데이터를 이용하여 어떤 결과를 일정한 방식으로 합성한 후, 최종 결과 반환
  - Observable이 통지한 데이터가 숫자일 경우 파라미터로 지정한 함수형 인터페이스에 정의된 계산 방식으로 값을 집계할 수 있다.
  - 예시
    ```java
    Observable.range(1, 10)
        .doOnNext(System.out::println)
        .reduce((x, y) -> x + y)
        .subscribe(data -> System.out.println("누적 합계: " + data));
    ```
    - x가 totalNum으로 누적합을 나타내며, y는 현재 받는 데이터의 값을 나타낸다.
    ```java
    Observable.range(1, 10)
        .doOnNext(System.out::println)
        .reduce(10, (x, y) -> x + y)
        .subscribe(data -> System.out.println("누적 합계: " + data));
    ```
    - 10이 초기값으로 입력되며 65가 출력된다.
    ```java
    Observable.just("a", "b", "c")
        .doOnNext(System.out::println)
        .reduce((x, y) -> "(" + x + "," + y + ")" )
        .subscribe(data -> System.out.println("ON NEXT: " + data));
    ```
    - 문자열에 경우 string 또한 중첩되어 출력된다.
    - ON NEXT: ((a,b),c)이 출력된다.
    
- scan
  - reduce와 같은 방식으로 생성하지만 차이점이 있다.
  - 차이점은 reduce는 마지막 최종 값만 보여주지만, scan에 경우 과정을 확인할 수 있다.
  - 예시
    ```java
    Observable.just("a", "b", "c")
        .scan((x, y) -> "(" + x + "," + y + ")" )
        .subscribe(data -> System.out.println("ON NEXT: " + data));
    ```
    ```
    // 출력
    ON NEXT: a
    ON NEXT: (a,b)
    ON NEXT: ((a,b),c)
    ```
    - 문자열에 경우 string 또한 중첩되어 출력된다.
    - doOnNext를 출력하지 않았지만 출력되는 결과를 계속 확인할 수 있다.


#### 출처

- Kevin의 알기쉬운 RxJava 1부
