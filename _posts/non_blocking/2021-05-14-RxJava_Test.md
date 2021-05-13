---
layout: post
title: RxJava, Test
summary: RxJava
author: devhtak
date: '2021-05-14 21:41:00 +0900'
category: RxJava
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

#### 참고

Kevin의 알기 쉬운 RxJava 2부
