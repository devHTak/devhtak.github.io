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
  
  int sum2 = 0;
  for(Integer i : numbers) {
      if(i > 6 && (i % 2 != 0)) {
          sum2 += i
      }
  }
  System.out.println("명령형 프로그래밍 사용: " + sum2);
  ```
  - 명령형은 for, if문등을 활용하여 구체적인 알고리즘을 명시

- 리액티브의 개념이 적용된 예
  - PUSH 방식: 데이터의 변화가 발생했을 때 변경이 발생한 곳에서 데이터를 보내주는 방식
    - 리액티브 방식
  - PULL 방식: 변경된 데이터가 있는지 요청을 보내 질의하고 변경된 데이터를 가져오는 방식

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

#### Reactive stream
   
- non-blocking backPressure(배압) 을 이용하여 비동기 서비스를 할 때 기본이 되는 스펙
- java의 RxJava, Spring의 WebFlux의 Core에 있는 Project Reactor 프로젝트 모두 해당 스펙을 따르고 있다.
- java 1.9 버전에 추가된 Flow 역시 reactive stream 스펙을 채택하여 사용하고 있다.
   
- blocking과 non-blocking
  - blocking: 자신의 수행 결과가 끝날 때 까지 제어권을 갖고 있는 것을 의미
  - non-blocking: 자신이 호출되었을 때 제어권을 바로 자신을 호출한 쪽으로 넘기며, 자신을 호출한 쪽에서 다른 일을 할 수 있도록 하는 것을 의미
   
- BackPressure(배압)
  - 한 컴포넌트가 부하를 이겨내기 힘들 때, 시스템 전체가 합리적인 방법으로 대응해야 한다. 
  - 과부하 상태의 컴포넌트에서 치명적인 장애가 발생하거나 제어 없이 메시지를 유실해서는 안 된다. 
  - 컴포넌트가 대처할 수 없고 장애가 발생해선 안 되기 때문에 컴포넌트는 상류 컴포넌트들에 자신이 과부하 상태라는 것을 알려 부하를 줄이도록 해야 한다. 
  - 이러한 배압은 시스템이 부하로 인해 무너지지 않고 정상적으로 응답할 수 있게 하는 중요한 피드백 방법이다.
  - 배압은 사용자에게까지 전달되어 응답성이 떨어질 수 있지만, 이 메커니즘은 부하에 대한 시스템의 복원력을 보장하고 시스템 자체가 부하를 분산할 다른 자원을 제공할 수 있는지 정보를 제공할 것이다.

- Reactive Streams
  - 리액티브 프로그래밍 라이브러리의 표준 사양
    - https://github.com/reactive-streams/reactive-streams-jvm
  - 리액티브 프로그래밍에 대한 인터페이스만 제공한다.
  - RxJava는 Reactive Streams의 인터페이스들을 구현한 구현체이다.
  - Reactive Streams는 Publisher, Subscriber, Subscription, Processor 라는 4개의 인터페이스를 제공한다.
    - Publisher: 데이터를 생성하고 통지한다.
    - Subscriber: 통지된 데이터를 전달받아 처리한다.
    - Subscription: 전달 받은 데이터의 개수를 요청하고 구독을 해지한다.
    - Processor: Publisher, Subscribe의 기능이 모두 있다.

- Publisher와 Subscriber 간의 프로세스 흐름

  ![image](https://user-images.githubusercontent.com/42403023/116811901-aeb50b80-ab86-11eb-8984-19a466d30fd8.png)

  - publisher와 subscriber 인터페이스에 해당 추상 메서드가 있다.

- Cold Publisher & Hot Publisher
  - Cold Publisher

    ![image](https://user-images.githubusercontent.com/42403023/116811989-461a5e80-ab87-11eb-8e85-ccf8cafb2d20.png) 

    - 생산자는 소비자가 구독할때마다 데이터를 처음부터 새로 통지한다.
    - 데이터를 통지하는 새로운 타임라인이 생성된다.
    - 소비자는 구독 시점과 상관없이 통지된 데이터를 처음부터 전달 받을 수 있다.
    - 예제
      ```java
      public static void main(String[] args) {
          Flowable<Integer> flowable = Flowable.just(1, 3, 5, 7);

          flowable.subscribe(data -> System.out.println("구독자 1: " + data));
          flowable.subscribe(data -> System.out.println("구독자 2: " + data));
      }
      ```
      - 새로운 타임라인을 만들어 구독자 1, 2 모두 1, 3, 5, 7 전달 받을 수 있다.

  - Hot Publisher
    
    ![image](https://user-images.githubusercontent.com/42403023/116812032-782bc080-ab87-11eb-9d00-66d6925dc2e1.png)
    
    - 생산자는 소비자 수와 상관없이 데이터를 한번만 통지한다.
    - 즉, 데이터를 통지하는 타임 라인은 하나이다.
    - 소비자는 발행된 데이터를 처음부터 전달받는게 아니라 구독한 시점에 통지된 데이터들만 전달받을 수 있다.
    - 예제
      ```java
      public static void main(String[] args) {
          PublishProcessor<Integer> processor = PublishProcessor.create();

          processor.subscribe(data -> System.out.println("구독자 1: " + data));
          processor.onNext(1);
          processor.onNext(3);

          processor.subscribe(data -> System.out.println("구독자 2: " + data));
          processor.onNext(5);
          processor.onNext(7);

          processor.onComplete();
      }
      ```
      - 구독자 1은 1, 3, 5, 7을 전달받고 구독자 2는 5, 7을 전달받는다.

#### Flowable과 Observable 비교

|Flowable|Observable|
|---|---|
|Reactive Streams 인터페이스를 구현|Reactive Streams 인터페이스를 구현하지 않았다.|
|Subscriber에서 데이터를 처리한다.|Obsrever에서 데이터를 처리한다.|
|데이터 개수를 제어하는 배압기능이 있다.|데이터 개수를 제어하는 배압 기능이 없다.|
|Subscription으로 전달 받는 데이터 개수를 제어할 수 있다.|배압 기능이 없기 때문에 데이터 개수를 제어할 수 없다.|
|Subscriptoin으로 구독을 해지한다.|Disposable으로 구독을 해지한다.|

- Flowable과 Observable의 결정적 차이. 배압 이란?

  ![image](https://user-images.githubusercontent.com/42403023/116812383-7a8f1a00-ab89-11eb-805c-ef65643ef325.png)
  
  - 예제
    ```java
    public static void main(String[] args) throws InterruptedException {
        Flowable.interval(1L, TimeUnit.MILLISECONDS)
            .doOnNext(data -> System.out.println("doOnNext: " + data))
            .observeOn(Schedulers.computation())
            .subscribe(data -> {
                System.out.println("# 소비자 처리 대기중");
                TimeUnit.SECONDS.sleep(1);
                System.out.println("subscribe: " + data);
            }, 
            error -> System.out.println(error), 
            () -> System.out.println("Logger.oc()"));

        Thread.sleep(2000L);
    }
    ```
    - doOnNext: lambda 파라미터가 호출될 때 조치할 수 있도록 publisher를 수정한다.
    - observerOn: 게시자를 수정하여 지정된 파라미터(scheduler)에서 버퍼 사이즈만큼 슬롯의 제한된 버퍼를 사용하여 비동기식으로 방출 및 알림을 수행합니다.
    - subscribe: 게시자를 구독하고 발행하는 항목과 발행하는 오류 또는 완료 알림을 처리하기위한 콜백을 제공합니다. 
    - 1부터 계속 publisher가 데이터를 요청하는데 처리하지를 못하고 대기한다. 128번째에 MissingBackpressureException(Can't deliver value 128 due to lack of requests)이 발생한다.
  
  - Flowable에서 데이터를 통지하는 속도가 Subscriber에서 통지된 데이터를 전달받아 처리하는 속도보다 빠를 때 밸런스를 맞추기 위해 데이터 통지량을 제어하는 기능을 말한다.
  - RxJava에서 BackpressureStrategy를 통해 Flowable이 통지 대기 중인 데이터를 어떻게 다룰지에 대한 배압 전략을 제공
  - Missing 전략
    - 배압을 적용하지 않는다.
    - 나중에 onBackpressureXXX()로 배압 적용을 할 수 있다.
  - Error 전략
    - 통지된 데이터가 버퍼의 크기를 초과하면 MissinbBackpressureException 에러를 통지한다.
    - 즉, 소비자가 생산자의 통지 속도를 따라 잡지 못할 때 발생
  - Buffer 전략: DROP_LATEST
    - 버퍼가 가득 찬 시점에 버퍼내에서 가장 최근에 버퍼로 들어온 데이터를 DROP한다.
    - DROP 된 빈자리에 버퍼 밖에서 대기하던 데이터를 채운다.
      ```java
      public static void main(String[] args) throws InterruptedException {
          System.out.println("# start: " + LocalDateTime.now());

          Flowable.interval(300L, TimeUnit.MILLISECONDS)
              .doOnNext(data -> System.out.println("#doOnNext: " + data))
              .onBackpressureBuffer(
                    2,
                    ()-> System.out.println("overflow!"),
                    BackpressureOverflowStrategy.DROP_LATEST)
              .doOnNext(data -> System.out.println("onBackpressureBuffer doOnNext(): " + data))
              .observeOn(Schedulers.computation(), false, 1)
              .subscribe(data -> {
                  TimeUnit.SECONDS.sleep(1);
                  System.out.println("SUBSCRIBE: " + data);
              }, error -> {
                  System.out.println("ERROR: " + error);
              });

          TimeUnit.SECONDS.sleep(2);
          System.out.println("# end: " + LocalDateTime.now());
      }
      ```
  - BUFFER 전략: DROP_OLDEST
    - 버퍼가 가득찬 시점에 버퍼내에서 가장 오래전에(먼저) 버퍼로 들어온 데이터를 DROP 한다.
    - DROP 된 빈자리에는 버퍼 밖에서 대기하던 데이터를 채운다.
      ```java
      public static void main(String[] args) throws InterruptedException {
          System.out.println(Thread.currentThread().getName() +  " # start: " + LocalDateTime.now());

          Flowable.interval(300L, TimeUnit.MILLISECONDS)
              .doOnNext(data -> System.out.println(Thread.currentThread().getName() + " #doOnNext: " + data))
              .onBackpressureBuffer(
                    2,
                    ()-> System.out.println("overflow!"),
                    BackpressureOverflowStrategy.DROP_OLDEST)
              .doOnNext(data -> System.out.println(Thread.currentThread().getName() +  " onBackpressureBuffer doOnNext(): " + data))
              .observeOn(Schedulers.computation(), false, 1)
              .subscribe(data -> {
                  TimeUnit.SECONDS.sleep(1);
                  System.out.println(Thread.currentThread().getName() +  " SUBSCRIBE: " + data);
              }, error -> {
                  System.out.println(Thread.currentThread().getName() +  " ERROR: " + error);
              });

          TimeUnit.SECONDS.sleep(2);
          System.out.println(Thread.currentThread().getName() +  " # end: " + LocalDateTime.now());
      }
      ```
  - DROP 전략
    - 버퍼에 데이터가 모두 채워진 상태가 되면 이후에 생성되는 데이터를 버리고(DROP), 버퍼가 비워지는 시점에 DROP 되지 않은 데이터부터 다시 버퍼에 담는다.
      ```java
      public static void main(String[] args) throws InterruptedException {
          System.out.println(Thread.currentThread().getName() +  " # start: " + LocalDateTime.now());

          Flowable.interval(300L, TimeUnit.MILLISECONDS)
              .doOnNext(data -> System.out.println(Thread.currentThread().getName() + " #doOnNext: " + data))
              .onBackpressureDrop(dropData -> System.out.println("drop: " + dropData))
              .observeOn(Schedulers.computation(), false, 1)
              .subscribe(data -> {
                  TimeUnit.SECONDS.sleep(1);
                  System.out.println(Thread.currentThread().getName() +  " SUBSCRIBE: " + data);
              }, error -> {
                  System.out.println(Thread.currentThread().getName() +  " ERROR: " + error);
              });

          TimeUnit.SECONDS.sleep(3);
          System.out.println(Thread.currentThread().getName() +  " # end: " + LocalDateTime.now());
      }
      ```
  - LATEST 전략
    - 버퍼에 데이터가 모두 채워진 상태가 되면 버퍼가 비워질 때까지 통지된 데이터는 버퍼 밖에서 대기하여 버퍼가 비워지는 시점에 가장 나중(최근)에 통지된 데이터부터 버퍼에 담는다.
      ```java
      public static void main(String[] args) throws InterruptedException {
          System.out.println(Thread.currentThread().getName() +  " # start: " + LocalDateTime.now());

          Flowable.interval(300L, TimeUnit.MILLISECONDS)
              .doOnNext(data -> System.out.println(Thread.currentThread().getName() + " #doOnNext: " + data))
              .onBackpressureLatest()
              .observeOn(Schedulers.computation(), false, 1)
              .subscribe(data -> {
                  TimeUnit.SECONDS.sleep(1);
                  System.out.println(Thread.currentThread().getName() +  " SUBSCRIBE: " + data);
              }, error -> {
                  System.out.println(Thread.currentThread().getName() +  " ERROR: " + error);
              });

          TimeUnit.SECONDS.sleep(3);
          System.out.println(Thread.currentThread().getName() +  " # end: " + LocalDateTime.now());
      }
      ```
  - 
- Flowable
  ```java 
  public abstract class Flowable<T> implements Publisher<T> {
      // ...
  }
  ```
  - Flowable을 사용해야하는 경우
    - 10,000개 이상의 데이터 흐름이 발생하는 경우
    - 디스크에서 파일을 읽는 경우 (기본적으로 Blocking/Pull-based 방식)
    - JDBC에서 데이터베이스를 읽는 경우 (기본적으로 Blocking/Pull-based 방식)
    - 네트워크 IO 실행 시
    - Blocking/Pull-based 방식을 사용하고 있는데 나중에 Non-Blocking 방식의 Reactive API/드라이버에서 데이터를 가져올 일이 있는 경우

  - 예제
    ```java
    public static void main(String[] args) throws InterruptedException {
        Flowable<String> flowable = Flowable.create(new FlowableOnSubscribe<String>() {
            @Override
            public void subscribe(@NonNull FlowableEmitter<String> emitter) throws Exception {
                // TODO Auto-generated method stub
                String[] datas = {"Hello", "RxJava"};
                for(String data: datas) {
                    if(emitter.isCancelled()) {
                        return;
                    }

                    emitter.onNext(data); // 데이터 통지
                }
                emitter.onComplete(); // 데이터 통지 완료를 알린다.
            }			
        }, BackpressureStrategy.BUFFER); // 구독자의 처리가 늦을 경우 데이터를 버퍼에 담아두는 설정

        flowable.observeOn(Schedulers.computation())
            .subscribe(new Subscriber<String>() {
                private Subscription subscription; // 데이터 개수 요청 및 구독을 취소하기 위한 subscription 객체
                @Override
                public void onSubscribe(Subscription subscription) {
                    this.subscription = subscription;
                    this.subscription.request(Long.MAX_VALUE);
                }
                @Override
                public void onNext(String data) { System.out.println("ON_NEXT: " + data); }

                @Override
                public void onError(Throwable error) { System.out.println("ERROR: " + error);}

                @Override
                public void onComplete() { System.out.println("ON COMPLETE"); }				
            });
        Thread.sleep(5000);
    }
    ```
    - FlowableOnSubscribe, Subscriber는 lambda 표현식으로 변경할 수 있다.
    
- Observable
  ```java
  public abstract class Observable<T> implements ObservableSource<T> {
      // ...
  }
  ```
  - Observable을 사용해야하는 경우
    - 1,000개 미만의 데이터 흐름이 발생하는 경우
    - 적은 데이터 소스만을 활용하여 OutOfMemoryException이 발생할 확률이 적은 경우
    - 마우스 이벤트나 터치 이벤트와 같은 GUI 프로그래밍을 하는 경우 (초당 1,000회 이하의 이벤트는 Observable의 sample()이나 debounce()로 핸들링 가능)
    - 동기적인 프로그래밍이 필요하지만 플랫폼에서 Java Streams을 지원하지 않는 경우
  - 예제
    ```java
    public static void main(String[] args) throws InterruptedException {
        Observable<String> observable = Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(ObservableEmitter<String> emitter) throws Exception {
                // TODO Auto-generated method stub
                String[] datas = {"Hello", "RxJava"};
                for(String data: datas) {
                    if(emitter.isDisposed()) {
                        return;
                    }
                    emitter.onNext(data);
                }
                emitter.onComplete();
            }
        });

        observable.observeOn(Schedulers.computation())
            .subscribe(new Observer<String>() {
                @Override
                public void onSubscribe(Disposable d) { 
                    // 아무 처리도 하지 않는다. 					
                }
                @Override
                public void onNext(String data) { System.out.println("ON_NEXT: " + data); }

                @Override
                public void onError(Throwable error) { System.out.println("ERROR: " + error);}

                @Override
                public void onComplete() { System.out.println("ON COMPLETE"); }					
            });
        Thread.sleep(5000);
    }
    ```

#### 출처

- Kevin의 RxJava 강의

- 참고: [Java] Reactive Stream 이란? 
  (URL: https://sabarada.tistory.com/98)
