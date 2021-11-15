---
layout: post
title: React Programming과 React Stream 기본
summary: Reactive Programming
author: devhtak
date: '2021-05-04 21:41:00 +0900'
category: Reactive
---

#### Reactive Programming

```
In computing, reactive programming is a declarative programming
paradigm concerned with data streams and the propagation of change.
(변화의 전파와 데이터 흐름과 관련된 선언적 프로그래밍 패러다임이다.)
```
- 변화의 전파와 데이터 흐름: 데이터가 변경될 때마다 이벤트를 발생시켜 데이터를 계속적으로 전달한다.
- 선언적 프로그래밍: 실행할 동작을 구체적으로 명시하는 명령형 프로그래밍과 달리 선언형 프로그래밍은 단순히 목표를 선언한다.
  ```java
  // 선언형 프로그래밍 방법
  List<Integer> numbers = Arrays.asList(1, 3, 21, 10, 8, 11);
  int sum1 = numbers.stream().filter(number -> number > 6 && (number % 2 != 0)).mapToInt(number -> number).sum();
  System.out.println("선언형 프로그래밍 사용: " + sum1);
  
  // 명령형 프로그래밍 
  int sum2 = 0;
  for(Integer i : numbers) {
      if(i > 6 && (i % 2 != 0)) {
          sum2 += i
      }
  }
  System.out.println("명령형 프로그래밍 사용: " + sum2);
  ```
  - 명령형은 for, if문등을 활용하여 구체적인 알고리즘을 명시

#### Reactive Programming 등장 배경

- 전통적인 아키텍처에서는 동기 블로킹방식을 사용하였다.
  - (동기 블로킹 방식) 하나의 요청에 대해 하나의 스레드를 통해 처리하며 모든 데이터를 가져와서 처리할 때까지 해당 스레드를 블로킹한다.
  - 멀티스레드의 문제점을 해결할 수 있다.
    - Thread Pool 최적화
    - Thread Context-Switching 발생
      ```
      Thread Context-Switching과 Process Context-Switching의 차이점
      - Context Switching이란?
      CPU내에 존재하는 레지스터들은 현재 실행중인 프로세스 관련 데이터들로 채워진다. 
      실행중인 프로세스가 변경이 되면, CPU내 레지스터들의 값이 변경되어야 하는데, 
      변경되기 전에 이전 프로세스가 지니고 있던 데이터들을 PCB에 저장해 주어어야 한다.
      그리고 새로 실행되는 프로세스가 아니라면 이전에 실행될 때 레지스터들이 지니고 있던
      데이터들을 불러와서 이어서 실행해야 한다. 이 과정이 컨텍스트 스위칭이다.
      - 차이점
      쓰레드는 공유하는 영역이 많기 때문에 컨텍스트 스위칭이 빠르다.
      캐쉬는 CPU와 메인메모리 사이에 위치하며 CPU에서 한번 이상 읽어들인
      메모리의 데이터를 저장하고 있다가, CPU가 다시 그 메모리에 저장된 
      데이터를 요구할 때, 메인메모리를 통하지 않고 데이터를 전달해 주는 용도이다.
      프로세스 컨텍스트 스위칭이 일어났을 경우, 공유하는 데이터가 없으므로
      캐쉬가 지금껏 쌓아놓은 데이터들이 무너지고 새로 캐쉬정보를 쌓아야 한다.
      이것이 프로세스 컨텍스트 스위칭에 부담이 되는 요소이다. 
      반면, 쓰레드라면 저장된 캐쉬 데이터는 쓰레드가 바뀌어도 공유하는 
      데이터가 있으므로 의미있다. 그러므로 컨텍스트 스위칭이 빠른 것이다. 
      ```
      
- Reactive Stream에서의 데이터 처리 방식은 1thread에 여러 요청을 처리할 수 있다.
  - Context Switching을 줄일 수 있고 여러 요청을 한번에 처리할 수 있다.

#### Blocking/Non-Blocking vs Synchronous/Asynchronous

- blocking & non-blocking
  ```
  이 그룹은 호출되는 함수에 대한 제어권을 바로 반납하느냐 마느냐가 관심사이다.
  ```
  - 행위자가 취한 행위 자체가, 또는 그 행위로 인해 다른 무엇이 막혀버린, 제한된, 대기하는 상태.
  - 대개의 경우에는 나 이외의 대상으로 하여금 내가 Block 당하겠지만(Blocked), 어찌 되었든 문자 자체로는 나라는 단일 개체 스스로의 상태를 나타낸다.
  - 호출된 함수가 자신이 할 일을 모두 마칠 때까지 제어권을 계속 가지고서 호출한 함수에게 바로 돌려주지 않으면 Block
  - 호출된 함수가 자신이 할 일을 채 마치지 않았더라도 바로 제어권을 건네주어(return) 호출한 함수가 다른 일을 진행할 수 있도록 해주면 Non-block
  
- synchronous & asynchronous
  ```
  이 그룹은 호출되는 함수의 작업 완료 여부를 누가 신경쓰느냐가 관심사이다.
  ```
  - 동시에 발생하는 것들(always plural, can never be singular).
  - 동시라는 것은 즉, 시(time)라는 단일계(system)에서 같이, 함께 무언가가 이루어지는 두 개 이상의 개체 혹은 이벤트를 의미한다고 볼 수 있겠습니다.
  - 호출된 함수의 수행 결과 및 종료를 호출한 함수가(호출된 함수뿐 아니라 호출한 함수도 함께) 신경 쓰면 Synchronous
  - 호출된 함수의 수행 결과 및 종료를 호출된 함수 혼자 직접 신경 쓰고 처리한다면(as a callback fn.) Asynchronous

- 조합
  ![image](https://user-images.githubusercontent.com/42403023/141741806-f440f561-1653-4c43-bc50-7c32abef5542.png)

  ** 이미지 출처: https://velog.io/@kjh3865/BlockingNon-Blocking-SyncAsync-%EC%A1%B0%ED%95%A9-%EC%A0%95%EB%A6%AC
  
  - blocking + Synchronous
    - 결과가 처리되어 나올때까지 기다렸다가 return 값으로 결과를 전달한다.
    
  - blocking + asynchronous
    - 호출되는 함수가 바로 return하지 않고, 호출하는 함수는 작업 완료 여부를 신경쓰지 않는다.
    - 이 조합은 사실 이점이 없어서 일부러 이 방식을 사용하진 않는다고 한다.)

  - non-blocking + Asynchronous
    - 작업 요청을 받아서 별도의 프로세서에서 진행하게 하고 바로 return(작업 끝)한다. 결과는 별도의 작업 후 간접적으로 전달(callback)한다.
  
  - non-blocking + synchronous
    - 결과가 없다면 바로 return한다. 결과가 있으면 바로 결과를 return 한다.
    - polling: 결과가 생길때까지 계속 완료 되었는지 확인

#### Reactive Programming 기반 기술

- Observer Pattern
  - 한 객체의 상태가 바뀌면 그 객체에 의존하는 다른 객체들한테 연락을 하고, 자동으로 내용이 갱신되는 방식에 디자인패턴
  - push 방식: 데이터의 변화가 발생했을 때 변경이 발생한 곳에서 데이터를 보내주는 방식
  - Observable은 하나 또는 여러개의 Observer를 가지며 Observable은 notifyObservers 메소드를 호출함으로 변화를 Observer들에게 알려준다고 한다
    ```java
    // Source(Observable) -> Event/Data -> Target(Observer)
    static class IntObservable extends Observable implements Runnable { 
      @Override
      public void run() {
        for(int i = 1; i <= 10; i++) {
          setChanged();
          notifyObservers(i);
        }
      }		
    }
    public static void main(String[] args) {
      Observer observer = new Observer() {
        public void update(Observable o, Object arg) {
          // 이벤트를 받았을 때 진행하는 메서드
          System.out.println(arg);
        }
      };
      IntObservable intObservable = new IntObservable();
      intObservable.addObserver(observer);
      intObservable.run();
      return "ok";
    }
    ```

- Iterator Pattern
  - 컬렉션 구현 방법을 노출시키지 않으면서 컬렉션 안에 들어있는 모든 엘리먼트에 접근할 수 있는 방식을 구현한 패턴
  - pull 방식: 변경된 데이터가 있는지 요청을 보내 질의하고 변경된 데이터를 가져오는 방식
  - Iterable interface 구현체
    ```java
    // for-each
    List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
    for(Integeri: list) {}
    
    // iterable
    Iterable<Integer> iter = () -> new Iterator<Integer>() {
      int i = 0;
      private final static int MAX = 10;
      public Integer next() {
        return ++i;
      }
      public boolean hasNext() {
        return i < MAX;
      }
    };
    
    for(Iterator it = iterable.iterator(); it.hasNext();) {}
    ```
    
    - 컬렉션은 Iterable 인터페이스를 구현하기 떄문에 for-each가 가능하다.
    - Iterable 인터페이스 안에 Iterator가 존재하며 Iterator의 next(), hasNext() 의 메소드 통해 순회할 수 있다.
  - Iterable은 Iterator를 통해 데이터를 꺼내오고, Iterator의 next()를 통해 데이터를 가져온다

- Reactive Programming은 Observer Pattern + Iterator Pattern
  - Iterator로 스트림의 '끝'을 나타내고, Observer pattern의 async한 이벤트 실행을 결합하여 나타낸다.
    - Iterator의 next() -> Observable onNext(), hasNext() -> onComplete(), error에 대한 처리로 onError()를 제공
    - 데이터를 전송하는 방식은 Observer 패턴을 사용한다.

#### Reactive Stream

- non-blocking backPressure(배압) 을 이용하여 비동기 서비스를 할 때 기본이 되는 스펙
  - 리액티브 프로그래밍 라이브러리의 표준 사양
    - https://github.com/reactive-streams/reactive-streams-jvm
    - 리액티브 프로그래밍에 대한 인터페이스만 제공한다.
    - RxJava는 Reactive Streams의 인터페이스들을 구현한 구현체이다.
  - Reactive Streams는 Publisher, Subscriber, Subscription, Processor 라는 4개의 인터페이스를 제공한다.
    - Publisher: 데이터를 생성하고 통지한다.
    - Subscriber: 통지된 데이터를 전달받아 처리한다.
    - Subscription: 전달 받은 데이터의 개수를 요청하고 구독을 해지한다.
    - Processor: Publisher, Subscribe의 기능이 모두 있다.
    - 구성도
    
      ![image](https://user-images.githubusercontent.com/42403023/116971008-e55c6480-acf3-11eb-87d8-ef6107f6faa5.png)
      
      (이미지 출처: https://sightstudio.tistory.com/14)
      
      ```java
      Iterable<Integer> iter = Arrays.asList(1, 2, 3, 4, 5);
      Publisher<Integer> publisher = new Publisher<Integer>() {
        @Override
        public void subscribe(Subscriber subscriber) {
          Subscription subscription = new Subscription() {
            Iterator it = iter.iterator();
            @Override
            public void request(long n) {
              if(it.hasNext()){
                subscriber.onNext(it.next());
              } else {
                subscriber.onComplete();
              }
            }					
            @Override
            public void cancel() {}
          };
          subscriber.onSubscribe(subscription);
        }
      };	
      Subscriber<Integer> subscriber = new Subscriber<Integer>() {
        @Override
        public void onSubscribe(Subscription subscription) {
          System.out.println("onSubscribe");
          subscription.request(Long.MAX_VALUE);
        }
        @Override
        public void onNext(Integer item) {
          System.out.println("onNext: " + item);
        }
        @Override
        public void onError(Throwable throwable) {
          System.out.println("onError: " + throwable.getMessage());
        }
        @Override
        public void onComplete() {
          System.out.println("onComplete");
        }
      };
      ```
    
- java 1.9 버전에 추가된 Flow 역시 reactive stream 스펙을 채택하여 사용하고 있다.

- Observer 패턴과 Iterator 패턴을 결합하여 사용
  - 처리할 수 있는 양만큼 가져와 처리한다.

- Reactive Programming은 Observer 패턴에 Backpressure 를 결합하였다.
  - BackPressure(배압)
    - 한 컴포넌트가 부하를 이겨내기 힘들 때, 시스템 전체가 합리적인 방법으로 대응해야 한다. 
    - 과부하 상태의 컴포넌트에서 치명적인 장애가 발생하거나 제어 없이 메시지를 유실해서는 안 된다. 
    - 컴포넌트가 대처할 수 없고 장애가 발생해선 안 되기 때문에 컴포넌트는 상류 컴포넌트들에 자신이 과부하 상태라는 것을 알려 부하를 줄이도록 해야 한다. 
    - 이러한 배압은 시스템이 부하로 인해 무너지지 않고 정상적으로 응답할 수 있게 하는 중요한 피드백 방법이다.
    - 배압은 사용자에게까지 전달되어 응답성이 떨어질 수 있지만, 이 메커니즘은 부하에 대한 시스템의 복원력을 보장하고 시스템 자체가 부하를 분산할 다른 자원을 제공할 수 있는지 정보를 제공할 것이다.

- blocking과 non-blocking
  - blocking: 자신의 수행 결과가 끝날 때 까지 제어권을 갖고 있는 것을 의미
  - non-blocking: 자신이 호출되었을 때 제어권을 바로 자신을 호출한 쪽으로 넘기며, 자신을 호출한 쪽에서 다른 일을 할 수 있도록 하는 것을 의미
   

#### 마블 다이어그램

- 리액티브 프로그래밍을 통해 발생하는 비동기적인 데이터의 흐름을 시간의 흐름에 따라 시각적으로 표시한 다이어그램
- 마블 다이어그램 알아야 하는 이유
  - 문장으로 적혀 있는 리액티브 연산자(Operators)의 기능을 이해하기 어려움
  - 리액티브 연산자의 기능이 시각화되어 있어서 이해하기 쉬움
  - 리액티브 프로그래밍의 핵심인 연산자를 사용하기 위한 핵심 도구

![image](https://user-images.githubusercontent.com/42403023/116775768-b948a580-aa9f-11eb-99c0-8091ea5b5ba5.png)

  - map operator: Observable의 값을 다니며 선언된 함수에 대한 연산을 진행하여 Observable을 return한다.
    ```java
    Observable observable = Observable.just(1, 25, 9, 15, 7, 30)
       .map(x -> x * 10)
       .subscribe(x - > System.out.println(x)); // 10, 250, 90, 150, 70, 300 출력
    ```
  - filter operator: Observable의 값을 다니며 선언된 조건에 부합한 데이터만 Observable을 return한다.
    ```java
    Observable observable = Observable.just(1, 25, 9, 15, 7, 30)
       .filter(x -> x > 10)
       .subscribe(x - > System.out.println(x)); // 25, 15, 30 출력
    ```
  - just 함수: 데이터 가공, 변화하는 함수가 아닌 Observable, Flowable을 생성하는 함수이다.
  
#### 출처

- Kevin의 RxJava 강의

- 참고: \[Java] Reactive Stream 이란? 
  (URL: https://sabarada.tistory.com/98)
  
- 참고: \[Reactive] Reactive Programming 과 Reactive Stream
  (URL: https://sightstudio.tistory.com/14)
  
- 참고: Reactive Stream - Observer, Iterator, Reactive Stream
  (URL: https://phantasmicmeans.tistory.com/entry/Observer-Iterator-Reactive-Stream)
  
- 출처: Blocking & Non-blocking & Synchronous & Asynchronous
  (URL: https://musma.github.io/2019/04/17/blocking-and-synchronous.html)
  
- 참고: 토비의 봄 TV 5회 스프링 리액티브 프로그래밍 (1) - Reactive Streams
  (URL: https://www.youtube.com/watch?v=8fenTR3KOJo&list=PLv-xDnFD-nnmof-yoZQN8Fs2kVljIuFyC&index=10&ab_channel=TobyLee)
