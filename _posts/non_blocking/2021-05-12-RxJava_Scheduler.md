---
layout: post
title: RxJava, Scheduler
summary: RxJava
author: devhtak
date: '2021-05-12 21:41:00 +0900'
category: RxJava
---

#### 물리적인 쓰레드와 논리적인 쓰레드의 이해

- 물리적인 쓰레드는 하드웨어와 관련이 있고, 논리적인 쓰레드는 소프트웨어와 관련이 있다.
- 물리적 쓰레드
  - 물리적인 쓰레드를 이해하기 위해서는 CPU의 코어를 먼저 알아야 한다.
  - 코어란?    
    - CPU의 명령어를 처리하는 반도체 유닛
    - 코어의 갯수가 많으면 명령어를 병렬로(parallel) 더 많이 더 빠르게 처리할 수 있다. 
  - 물리적인 쓰레드는 물리적인 코어를 논리적으로 쪼객 논리적 코어다.

- 논리적 쓰레드
  - 논리적인 쓰레드는 프로세스 내에서 실행되는 세부 작업의 단위이다. 
  - 프로세스는 컴퓨터에서 실행할 수 있는 실행 파일(프로그램)을 실행하면 생기는 인스턴스이다. 
  - 논리적인 쓰레드의 생성 개수는 이론적으로는 제한이 없지만 실제로는 물리적인 쓰레드의 가용 범위내에서 생성할 수 있다. 

![image](https://user-images.githubusercontent.com/42403023/117760816-47bbe480-b261-11eb-8c1c-fa1ccb98bd02.png)

- 물리적인 쓰레드는 CPU가 가지고 있는 코어에 개수 등을 말하며 논리적인 쓰레드는 애플리케이션 단에서 실행되는 스레드이다.
- 물리적인 쓰레드: 병렬성
  - 코어 개수와 스레드 개수만큼 할당되어 있기 때문에 병렬적으로 처리한다.
- 논리적인 쓰레드: 동시성
  - 병렬적으로 작업하는 것으로 보이지만, 실제 물리적인 쓰레드에서 할당되어 실행되며 병렬적으로 실행되는 모습을 보이지만 동시에 진행한다.

#### 스케쥴러(Scheduler)

- RxJava에서의 스케쥴러는 RxJava 비동기 프로그래밍을 위한 쓰레드(Thread) 관리자이다.
- 즉, 스케쥴러를 이용해서 어떤 쓰레드에서 무엇을 처리할 지에 대해서 제어할 수 있다.
- 스케쥴러를 이용해서 데이터를 통지하는 쪽과 데이터를 처리하는 쪽 쓰레드를 별도로 지정해서 분리할 수 있다.
- RxJava의 스케쥴러를 통해 쓰레드를 위한 코드의 간결성 및 쓰레드 관리의 복잡함을 줄일 수 있다.
- RxJava에서 스케쥴러를 지정하기 위해서 subscribeOn( ), observeOn( ) 유틸리티 연산자를 사용한다.
- 생산자쪽의 데이터 흐름을 제어하기 위해서는 subscribeOn( ) 연산자를 사용한다.
- 소비자쪽에서 전달받은 데이터 처리를 제어하기 위해서는 observeOn( ) 연산자를 사용한다.
- subscribeOn( ), observeOn( ) 연산자는 각각 파라미터로 Scheduler를 지정해야 한다.

#### 스케줄러의 종류

- Schedulers.io()
  - I/O 처리 작업을 할 때 사용하는 스케쥴러
  - 네트워크 요청 처리, 각종 입/출력 작업, 데이터베이스 쿼리 등에 사용
  - 쓰레드 풀에서 쓰레드를 가져오거나 가져올 쓰레드가 없으면 새로운 쓰레드를 생성한다. 
  - 예시
    ```java
    File[] files = new File("src/main/java/com/example").listFiles();
		
		Observable.fromArray(files)
        .doOnNext(data-> System.out.println(Thread.currentThread().getName() + ", ON NEXT: " + data))
        .filter(data -> data.isDirectory())
        .map(dir -> dir.getName())
        .subscribeOn(Schedulers.io())
        .subscribe(data -> System.out.println(Thread.currentThread().getName() + ", SUBSCRIBE: " + data));

		Thread.sleep(1000L);
    ```
    ```
    // 출력
    RxCachedThreadScheduler-1, ON NEXT: src\main\java\com\example\RxJava02Application.java
    RxCachedThreadScheduler-1, ON NEXT: src\main\java\com\example\scheduler
    RxCachedThreadScheduler-1, SUBSCRIBE: scheduler
    RxCachedThreadScheduler-1, ON NEXT: src\main\java\com\example\subject
    RxCachedThreadScheduler-1, SUBSCRIBE: subject
    ```
    - 같은 스레드로 진행하였다.
    - file io가 있기 때문에 해당 스케줄러를 사용했다.
    
- Schedulers.computation()
  - 논리적인 연산 처리 시, 사용하는 스케쥴러
  - CPU 코어의 물리적 쓰레드 수를 넘지 않는 범위에서 쓰레드를 생성한다.
  - 대기 시간 없이 빠르게 계산 작업을 수행하기 위해 사용한다.
  - 예시
    ```java
    File[] files = new File("src/main/java/com/example").listFiles();
		
		Observable.fromArray(files)
        .doOnNext(data-> System.out.println(Thread.currentThread().getName() + ", ON NEXT: " + data))
        .subscribeOn(Schedulers.io())
        .observeOn(Schedulers.computation())
        .filter(data -> data.isDirectory())
        .map(dir -> dir.getName())
        .subscribe(data -> System.out.println(Thread.currentThread().getName() + ", SUBSCRIBE: " + data));
		
		Thread.sleep(1000L);
    ```
    ```
    // 출력
    RxCachedThreadScheduler-1, ON NEXT: src\main\java\com\example\RxJava02Application.java
    RxCachedThreadScheduler-1, ON NEXT: src\main\java\com\example\scheduler
    RxCachedThreadScheduler-1, ON NEXT: src\main\java\com\example\subject
    RxComputationThreadPool-1, SUBSCRIBE: scheduler
    RxComputationThreadPool-1, SUBSCRIBE: subject
    ```
    - subscribe에는 IO로 observe에 경우 computation으로 지정하였다.
    - 다른 Thread가 생성되어 사용된 것을 확인할 수 있다.

  - 예시 2
    ```java
    File[] files = new File("src/main/java/com/example").listFiles();
		
		Observable.fromArray(files)
        .doOnNext(data-> System.out.println(Thread.currentThread().getName() + ", #데이터 통지"))
        .subscribeOn(Schedulers.io())
        .observeOn(Schedulers.computation())
        .filter(data -> data.isDirectory())
        .doOnNext(data-> System.out.println(Thread.currentThread().getName() + ", #필터 거침"))
        .map(dir -> dir.getName())
        .doOnNext(data-> System.out.println(Thread.currentThread().getName() + ", #map 거침"))
        .observeOn(Schedulers.computation())
        .subscribe(data -> System.out.println(Thread.currentThread().getName() + ", SUBSCRIBE: " + data));

		Thread.sleep(1000L);
    ```
    ```
    // 출력
    RxCachedThreadScheduler-1, #데이터 통지
    RxCachedThreadScheduler-1, #데이터 통지
    RxCachedThreadScheduler-1, #데이터 통지
    RxComputationThreadPool-1, #필터 거침
    RxComputationThreadPool-1, #map 거침
    RxComputationThreadPool-1, #필터 거침
    RxComputationThreadPool-2, SUBSCRIBE: scheduler
    RxComputationThreadPool-1, #map 거침
    RxComputationThreadPool-2, SUBSCRIBE: subject
    ```
    - map을 거친 후 다시 observeOn으로 새로운 Scheduler를 설정하였고, computation은 물리적 스레드(코어, 스래드 개수)에 맞게 새로 생성한다.
    - subscribe 지점에서 2번째 스레드가 생성된 것을 확인할 수 있다.
  
  - 예시
    ```java
    Observable<Integer> observable1 = Observable.just(1, 3, 5, 7);
		Observable<Integer> observable2 = Observable.just(2, 4, 6, 8);
		Observable<Integer> observable3 = Observable.just(10, 20, 30, 40);
		Observable<Integer> observable4 = Observable.range(1, 24);
		
		Observable source = Observable.zip(observable1, observable2, observable3, observable4,
				  (data1, data2, data3, hour) -> hour +"시: " + Collections.max(Arrays.asList(data1, data2, data3)));
		
		source.subscribeOn(Schedulers.computation())
			  .subscribe(data-> System.out.println(Thread.currentThread().getName() + ", #First: " + data));
			
			
		source.subscribeOn(Schedulers.computation())
			  .subscribe(data-> System.out.println(Thread.currentThread().getName() + ", #Second: " + data));
		
		Thread.sleep(1000L);
    ```
    ```
    // 출력
    RxComputationThreadPool-1, #First: 1시: 10
    RxComputationThreadPool-1, #First: 2시: 20
    RxComputationThreadPool-2, #Second: 1시: 10
    RxComputationThreadPool-2, #Second: 2시: 20
    RxComputationThreadPool-1, #First: 3시: 30
    RxComputationThreadPool-2, #Second: 3시: 30
    RxComputationThreadPool-2, #Second: 4시: 40
    RxComputationThreadPool-1, #First: 4시: 40
    ```
    - First와 Second에 다른 Thread가 실행된 것을 확인할 수 있다.

- Schedulers.newThread()
  - 요청시마다 매번 새로운 쓰레드를 생성한다.
  - 매번 생성되면 쓰레드 비용도 많이 들고, 재사용도 되지 않는다.
  - 예시
    ```java
    Observable<String> observable = Observable.just("1", "2", "3", "4", "5");
		
		observable.subscribeOn(Schedulers.newThread())
        .map(data -> "## " + data + " ##")
        .subscribe(data -> System.out.println(Thread.currentThread().getName() + " : " + data));
		
		observable.subscribeOn(Schedulers.newThread())
        .map(data -> "## " + data + " ##")
        .subscribe(data -> System.out.println(Thread.currentThread().getName() + " : " + data));
		
		Thread.sleep(300L);
    ```
    ```
    // 출력
    RxNewThreadScheduler-2 : ## 1 ##
    RxNewThreadScheduler-2 : ## 2 ##
    RxNewThreadScheduler-1 : ## 1 ##
    RxNewThreadScheduler-2 : ## 3 ##
    RxNewThreadScheduler-1 : ## 2 ##
    RxNewThreadScheduler-2 : ## 4 ##
    RxNewThreadScheduler-1 : ## 3 ##
    RxNewThreadScheduler-2 : ## 5 ##
    RxNewThreadScheduler-1 : ## 4 ##
    RxNewThreadScheduler-1 : ## 5 ##
    ```
- Schedulers.trampoline()
  - 현재 실행되고 있는 쓰레드에 큐(Queue)를 생성하여 처리할 작업들을 큐에 넣고 순서대로 처리한다.
    - FIFO

- Schedulers.single()
  - 단일 쓰레드를 생성하여 처리 작업을 진행한다
  - 여러번 구독해도 공통으로 사용한다.

- Schedulers.from(executor)
  - Executor를 사용해서 생성한 쓰레드를 사용한다.
  - RxJava의 Scheduler와 Executor의 동작 방식이 다르므로 자주 사용되지 않음.

#### 출처

Kevin의 알기 쉬운 RxJava 2부
