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

#### 출처

- Kevin의 알기쉬운 RxJava 1부
