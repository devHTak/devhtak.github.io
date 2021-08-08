---
layout: post
title: RxJava, Test
summary: Reactive Programming
author: Reactive Programming
date: '2021-05-14 21:41:00 +0900'
category: Reactive
---

#### 테스트를 위한 blocking 함수

- 비동기 처리 결과를 테스트하려면 현재 쓰레드에서 호출 대상 쓰레드의 실행 결과를 반환 받을때까지 대기할 수 있어야 한다.
- RxJava에서는 현재 쓰레드에서 호출 대상 쓰레드의 처리 결과를 받을 수 있는 blocking 함수를 제공한다.
- Observable에서 통지되고 가공 처리된 결과 데이터를 현재 쓰레드에 반환하므로, 반환된 결과 값과 예상되는 기대값을 비교하여 단위 테스트를 수행할 수 있다.
- 일반적으로 테스트하는 경우
  ```java
  @Test
	void noBlockingTest() {
      List<Integer> list = new ArrayList<>();

      Observable.range(0, 10)
          .subscribeOn(Schedulers.computation())
          .subscribe(data -> list.add(data));
      
      // int size = Observable.range(0, 10).subscribeOn(Schedulers.computation()).reduce(total, data -> total + 1).blockingGet();
      assertEquals(10, list.size());
	}
  ```
  - 테스트에 실패한다.
  - 새로운 스레드에서 list에 값을 넣은 것이기 때문에 실제 메인 스레드에서의 list.size()는 0이다.

#### blockingFirst

- 생산자가 통지한 첫번째 데이터를 반환한다.
- 통지된 데이터가 없을 경우 NoSuchElementExamption을 발생시킨다.
- 예시
  ```java
  @Test
	void getIntegerStreamFirstTest() {
      Integer first = Observable.range(0, 10)
          .subscribeOn(Schedulers.computation()).blockingFirst();

      assertEquals(0, first);
	}
  ```
  
#### blockingLast

- 생산자가 통지한 마지막 데이터를 반환
- 통지된 데이터가 없을 경우 NoSuclElementException을 발생시킨다.
- 결과를 반환하는 시점이 완료를 통지하는 시점이므로 완료 통지가 없는 데이터 통지일 경우 사용할 수 없다.
- 예시
  ```java
  @Test
	void getIntegerStreamLastTest() {
      Integer last = Observable.range(0, 10)
          .subscribeOn(Schedulers.computation()).blockingLast();

      assertEquals(9, last);
	}
  ```
  
#### blockingSingle

- 생산자가 한개의 데이터를 통지하고 완료되면 해당 데이터를 반환한다.
- 2개 이상의 데이터를 통지할 경우 IllegalArgumentException을 발생시킨다.
- 예시
  ```java
  @Test
	void getIntegerStreamSingleTest() {
      Integer odd = Observable.just(2, 4, 5, 6, 8)
          .filter(data -> data % 2 != 0)
          .subscribeOn(Schedulers.computation()).blockingSingle();

      assertEquals(5, odd);
	}
  ```

#### blockingGet

- 생산자가 0개 또는 1개의 데이터를 통지하고 완료되면 해당 데이터를 반환한다.
- 즉, 생산자가 Maybe일 경우 사용할 수 있다.
- 예시 1: 빈값에 대하여 Maybe 생산자 확인
  ```java
  @Test
	void getNullTest() {
		  assertNull(Observable.empty().firstElement().blockingGet());
	}
  ```
  
- 예시 2: 전체 합에 대한 값 확인
  ```java
  @Test
	void getSumTest() {
      int sum = Observable.range(0, 10)
            .subscribeOn(Schedulers.computation())
            .reduce((total, data) -> total + data)
            .blockingGet();

      assertEquals(45, sum);
	}
  ```
- 예시 3: zip을 활용하여 여러 Observable의 데이터를 통합한 값 확인
  ```java
  @Test
	void getZipSumTest() {
      int sum = Observable.zip(
            Observable.range(0, 10).filter(data -> data % 2 == 0),
            Observable.range(0, 10).filter(data -> data % 2 != 0),
            (a, b) -> a + b
        )
        .subscribeOn(Schedulers.computation())
        .reduce((total, num) -> total + num )
        .blockingGet();

      assertEquals(45, sum);
	}
  ```
  
#### blockingIterable

- 생산자가 통지한 모든 데이터를 받을 수 있는 Iterable을 얻게 한다.
- 구독 후, Iterator의 next() 메서드를 호출하는 시점부터 처리한다.
- 예시
  ```java
  @Test
	void getIterableTest() {
      Iterable<Integer> iterable = Observable.range(0, 10).blockingIterable();
      Iterator<Integer> it = iterable.iterator();

      int i = 0;
      while(it.hasNext()) {
          assertEquals(i, it.next());
          i++;
      }
	}
  ```

#### blockingForEach

- 생산자가 통지한 데이터를 순차적으로 모두 통지한다.
- 통지된 각각의 데이터가 모두 조건에 맞아야 true를 반환
- 예시
  ```java
  @Test
	void getForEachTest() {
      Observable.range(0, 10)
          .subscribeOn(Schedulers.computation())
          .filter(data -> data %2 == 0)
          .blockingForEach(data -> assertTrue(data % 2 == 0));
	}
  ```

#### blockingSubscribe

- 통지된 원본 데이터를 호출한 원본 쓰레드에서 부수적인 처리를 할 수 있도록 해준다.
- 소비자가 전달 받은 데이터로 어떤 부수적인 처리할 때 이 처리 결과를 테스트할 수 있다.
- 예시
  ```java
  @Test
	void getSubscribeTest() {
      Calculator calculator = new Calculator();

      Observable.range(0, 10)
          .subscribeOn(Schedulers.computation())
          .filter(data -> data %2 == 0)
          .blockingSubscribe(data -> calculator.setSum(data));

      assertEquals(20, calculator.getSum());
	}
	
	class Calculator {
      private int sum = 0;

      public void setSum(int data) {
          this.sum += data;
      }

      public int getSum() {
          return this.sum;
      }
	}
  ```

#### 테스트를 위한 TestSubscriber / TestObserver

- 테스트 용도로 사용되는 소비자 클래스
- assert 함수를 이용해 통지된 데이터를 검증할 수 있다.
- await 함수를 이용해 지정된 시간 동안 대기하거나 완료 또는 에러 이벤트가 발생할 때까지 대기할 수 있다.
- 완료, 에러, 구독 해지 등의 이벤트 발생 결과 값을 이용하여 데이터를 검증할 수 있다.

#### assertEmpty

- 테스트 시점까지 통지받은 데이터가 없다면 테스트에 성공한다.
- Observable.empty()로 생성 시, 완료를 통지하기 때문에 테스트가 실패한다.
- 즉, 통지 이벤트 자체(완료, 에러 통지 포함)가 없는지를 테스트할 수 있다.
- 예시
  ```java
  // 테스트 실패
  @Test
  void getIntegerStreamEmptyFailTest() {
      // when
      Observable<Integer> observable = Observable.range(0, 10);
      TestObserver<Integer> observer = observable.test();
		
      // then
      observer.awaitDone(1000L, TimeUnit.MILLISECONDS).assertEmpty();
  }
	
  // 테스트 성공
  @Test
  void getIntegerStreamEmptySuccessTest() {
      // when
      Observable<Integer> observable = Observable.range(0, 10);
      TestObserver<Integer> observer = observable
          .delay(1000L, TimeUnit.MILLISECONDS).test();
		
      // then
      observer.awaitDone(100L, TimeUnit.MILLISECONDS).assertEmpty();
  }
  ```
  - 테스트 실패: 1000L 사이에 데이터 통지가 있었기 때문에 실패한다.
  - 테스트 성공: 1000L 뒤에 데이터를 통지하기 때문에 100L 동안에는 통지가 없기 때문에 성공한다.

#### assertValues와 assert

- assertValues
  - 통지된 데이터가 한개 이상인 경우에 사용
    - assertValue는 1개의 데이터인지 확인
  - 즉, 통지된 데이터의 값과 순서가 파라미터로 입력된 데이터의 값과 순서와 일치하면 테스트에 성공

- assertNoValues
  - 해당 시점까지 통지된 데이터가 없으면 테스트에 성공한다.
  - 완료 통지와 에러 통지는 테스트 대상에서 제외된다.
    - assertEmpty와의 차이

- 예시
  ```java
  @Test
  void assertNoValueTest() {
      Observable.range(0, 10)
            .subscribeOn(Schedulers.computation())
            .filter(num -> num == 5)
            .test()
            .awaitDone(10L, TimeUnit.MILLISECONDS)
            .assertValue(5);
  }  
  ```
  - 구독하는 시점에 새로운 스레드를 만들었고 이 때 awaitDone은 대기 시간 이후에 값을 통지받을 수 있도록 해준다.
  
- 예시
  ```java
  @Test
  void assertValuesTest() {
      Observable.just(2, 4, 6, 7, 9)
          .subscribeOn(Schedulers.computation())
          .filter(num -> num % 2 != 0)
          .test()
          .awaitDone(1L, TimeUnit.MILLISECONDS)
          .assertValues(7, 9);
  }
  ```
  - assertValues를 통해 한개 이상의 값을 확인할 수 있다.

- 예시
  ```java
  @Test
  void assertNoValuesTest() {
      Observable.interval(200L, TimeUnit.MILLISECONDS)
          .doOnNext(data -> System.out.println("DO ON NEXT: " + data))
          .filter(data -> data > 5)
          .test()
          .awaitDone(1000L, TimeUnit.MILLISECONDS)
          .assertNoValues();
  }
  ```
  - doOnNext에서 0 ~ 4 까지 로그가 찍힌다.
  - 하지만, filter로 해당 데이터를 보내지 않고, 테스트를 통해 1000L 동안 데이터 통지되지 않았기 때문에 테스트 성공

#### assertResult, assertError, assertComplete 와 assertNotComplete

- assertResult
  - 해당 시점까지 통지를 완료했고, 통지된 데이터와 파라미터로 입력된 데이터의 값과 순서가 같으면 테스트에 성공한다.
  - assertValues와의 차이점은 해당 시점까지 완료 통지를 받았느냐 받지 않았느냐이다.
  - 예시
    ```java
    // 테스트 실패 예제
    @Test
    void assertResultFailTest() {
        Observable.interval(200L, TimeUnit.MILLISECONDS)
            .doOnNext(data -> System.out.println("DO ON NEXT: " + data))
            .filter(data -> data >3)
            .test()
            .awaitDone(1100L, TimeUnit.MILLISECONDS)
            .assertResult(4L);
    }
    
    // 테스트 성공 예제
    @Test
    void assertResultSuccessTest() {
        Observable.interval(200L, TimeUnit.MILLISECONDS)
            .take(5)
            .doOnNext(data -> System.out.println("DO ON NEXT: " + data))
            .filter(data -> data > 2)
            .test()
            .awaitDone(1100L, TimeUnit.MILLISECONDS)
            .assertResult(3L, 4L);
    }
    ```
    - take를 주지 않았을 때에는 완료 통보가 없어서 실패한다.
    - 정확히 5개로 개수를 지정하면 마지만 4L을 받을 때 완료 통지도 함께 진행되어 성공한다.
    
- assertError
  - 해당 시점까지 에러 통지가 있으면 테스트에 성공한다.
  - 단순히 에러 통지가 있었는지의 여부와 구체적으로 발생한 에러가 맞는지를 테스트할 수 있다.
  - 예시
    ```java
    // 예외 클래스 발생 여부만 확인
    @Test
    void assertErrorTest01() {
        Observable.interval(100L, TimeUnit.MILLISECONDS)
            .map(data -> 10 / data)
            .test()
            .awaitDone(1000L, TimeUnit.MILLISECONDS)
            .assertError(Throwable.class);
    }
	
    // 구체적인 예외 클래스 비교
    @Test
    void assertErrorTest02() {
        Observable.interval(100L, TimeUnit.MILLISECONDS)
            .map(data -> 10 / data)
            .test()
            .awaitDone(1000L, TimeUnit.MILLISECONDS)
            .assertError(error -> error.getClass() == ArithmeticException.class);
    }	  
    ```
    - 구체적인 에러까지 비교할 수 있다.

- assertComplete
  - 해당 시점까지 완료 통지가 있으면 테스트에 성공한다.
  - 예시
    ```java
    @Test
    void assertCompleteTest() {
        Observable.just(2, 4, 6, 8)
            .zipWith(Observable.just(1, 3, 5, 7), (a, b) -> {
                Thread.sleep(100L);
                return a + b;
            })
            .test()
            .awaitDone(500L, TimeUnit.MILLISECONDS)
            .assertComplete();		
    }
    ```

- assertNotCopmlete
  - 해당 시점까지 완료 통지가 없으면 테스트에 성공한다.
  - 예시
    ```java
    @Test
    void assertNotCompleteTest() {
        Observable.interval(100L, TimeUnit.MILLISECONDS)
            .take(5)
            .test()
            .awaitDone(300L, TimeUnit.MILLISECONDS)
            .assertNotComplete();		
    }
    ```

#### await 함수

- await
  - 생산자쪽에서 완료 통지나 에러 통지가 있을 때까지 쓰레드를 대기시킨다.
  - 파라미터로 지정된 시간동안 대기하며, 대기 시간내에 완료 통지가 있었는지 여부를 검증한다.
  - 예제
    ```java
    // 생산자 쪽에서 완료 통지를 보낼때까지 대기한 후, 완료 통지된 데이터 개수를 검증하는 예제
    @Test
    void awaitTest01() throws InterruptedException {
        Observable.interval(100L, TimeUnit.MILLISECONDS)
            .doOnNext(data -> System.out.println("DO ON NEXT: " + data))
            .take(5)
            .doOnComplete(() -> System.out.println("DO ON COMPLETE"))
            .doOnError(error -> System.out.println("DO ON ERROR: " + error.getMessage()))
            .test()
            .await()
            .assertComplete()
            .assertValueCount(5);
    }
	
    // 생산자 쪽에서 완료 통지를 보낼때까지 대기한 후, 완료 및 통지된 데이터 개수를 검증하는 예제
    @Test
    void awaitTest02() throws InterruptedException {
        boolean result = Observable.interval(100L, TimeUnit.MILLISECONDS)
                .doOnNext(data -> System.out.println("DO ON NEXT: " + data))
                .take(5)
                .doOnComplete(() -> System.out.println("DO ON COMPLETE"))
                .doOnError(error -> System.out.println("DO ON ERROR: " + error.getMessage()))
                .test()
                .await(300L, TimeUnit.MILLISECONDS);
        assertFalse(result);
    }
    ```
    - 생산자 쪽에서 완료 통지를 보낼때까지 대기한 후, 완료 통지된 데이터 개수를 검증하는 예제
    - 생산자 쪽에서 완료 통지를 보낼때까지 대기한 후, 완료 및 통지된 데이터 개수를 검증하는 예제

- awaitDone
  - 파라미터로 지정된 시간 동안 대기시키거나 지정된 시간 전에 완료 통지나 에러 통지가 있다면 통지가 있을때까지만 대기시킨다.
  - 예제
    ```java
    // 지정된 시간까지 완료 통지 없이, 해당 시점까지 전달받은 데이터의 개수가 맞는지 검증
    @Test
    void awaitDoneTest01() {
        Observable.interval(200L, TimeUnit.MILLISECONDS)
            .doOnNext(data -> System.out.println("DO ON NEXT: " + data))
            .take(5)
            .doOnComplete(() -> System.out.println("DO ON COMPLETE"))
            .doOnError(error -> System.out.println("DO ON ERROR: " + error.getMessage()))
            .test()
            .awaitDone(500L, TimeUnit.MILLISECONDS)
            .assertNotComplete()
            .assertValueCount(2);
    }
	
    // 지정된 시간 전에 완료 통지가 있어, 완료 통지 시점까지만 대기하는 검증 예제
    @Test
    void awaitDoneTest02() {
        Observable.interval(100L, TimeUnit.MILLISECONDS)
            .doOnNext(data -> System.out.println("DO ON NEXT: " + data))
            .take(3)
            .doOnComplete(() -> System.out.println("DO ON COMPLETE"))
            .doOnError(error -> System.out.println("DO ON ERROR: " + error.getMessage()))
            .test()
            .awaitDone(500L, TimeUnit.MILLISECONDS)
            .assertComplete()
            .assertValueCount(3);
    }
    ```
    - 지정된 시간까지 완료 통지 없이, 해당 시점까지 전달받은 데이터의 개수가 맞는지 검증
    - 지정된 시간 전에 완료 통지가 있어, 완료 통지 시점까지만 대기하는 검증 예제

- awaitCount
  - 파라미터로 지정된 개수만큼 통지될 때까지 쓰레드를 대기시킨다.
  - 예시
    ```java
    // 지정된 개수만큼 대기하고 완료 통지 유무, 통지된 데이터 개수 및 데이터의 값과 순서를 검증
    @Test
    void awaitCountTest() {
        Observable.interval(200L, TimeUnit.MILLISECONDS)
            .doOnNext(data -> System.out.println("DO ON NEXT: " + data))
            .take(5)
            .doOnComplete(() -> System.out.println("DO ON COMPLETE"))
            .doOnError(error -> System.out.println("DO ON ERROR: " + error.getMessage()))
            .test()
            .awaitCount(3)
            .assertNotComplete()
            .assertValueCount(3)
            .assertValues(0L, 1L, 2L);
    }
    ```
    - 지정된 개수만큼 대기하고 완료 통지 유무, 통지된 데이터 개수 및 데이터의 값과 순서를 검증

#### is 함수

- 완료, 구독 에러 등 이벤트가 발생했는지 확인하는 함수로 true/false를 반환한다.

- isTerminated
  - 에러 또는 완료 이벤트가 발생했으면 true를 리턴한다.

- isCancelled
  - Flowable에서 구독 해지 이벤트가 발생했으면 true를 리턴한다.

- isDisposed
  - Observable에서 구독 해지 이벤트가 발생했으면 true를 리턴한다.

- hasSubscriptoin
  - 구독이 발생하면 true를 리턴한다.

- 예시
  ```java
  @Test
  void isTerminalEventTest() {
      boolean result = Observable.interval(200L, TimeUnit.MILLISECONDS)
              .doOnNext(data -> System.out.println("DO ON NEXT: " + data))
              .take(5)
              .doOnComplete(() -> System.out.println("DO ON COMPLETE"))
              .doOnError(error -> System.out.println("DO ON ERROR: " + error.getMessage()))
              .test()
              .awaitCount(3)
              .isTerminated();
		
      assertTrue(result);
  }
  ```

#### 참고

Kevin의 알기 쉬운 RxJava 2부
