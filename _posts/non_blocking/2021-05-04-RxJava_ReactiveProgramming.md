---
layout: post
title: RxJava, 다양한 Reactive Stream 객체
summary: RxJava
author: devhtak
date: '2021-05-04 21:41:00 +0900'
category: RxJava
---

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

#### Publisher와 Subscriber 간의 프로세스 흐름

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

#### Flowable과 Observable의 결정적 차이. 배압 이란?

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
    - doOnNext: 데이터를 전송할 때 해당 전달한 함수가 실행된다.
    - observerOn(Schedulers.computation()): 게시자를 수정하여 지정된 파라미터(scheduler)에서 버퍼 사이즈만큼 슬롯의 제한된 버퍼를 사용하여 비동기식으로 방출 및 알림을 수행합니다.
    - subscribe: 게시자를 구독하고 발행하는 항목과 발행하는 오류 또는 완료 알림을 처리하기위한 콜백을 제공합니다. (onNext, onError, onComplete)
    - 1부터 계속 publisher가 데이터를 요청하는데 처리하지를 못하고 대기한다. 128번째에 MissingBackpressureException(Can't deliver value 128 due to lack of requests)이 발생한다.
  
  - Flowable에서 데이터를 통지하는 속도가 Subscriber에서 통지된 데이터를 전달받아 처리하는 속도보다 빠를 때 밸런스를 맞추기 위해 데이터 통지량을 제어하는 기능을 말한다.
  - RxJava에서 BackpressureStrategy를 통해 Flowable이 통지 대기 중인 데이터를 어떻게 다룰지에 대한 배압 전략을 제공

#### 배압 전략 
- Missing 전략
  - 배압을 적용하지 않는다.
  - 나중에 onBackpressureXXX()로 배압 적용을 할 수 있다.

- Error 전략
  - 통지된 데이터가 버퍼의 크기를 초과하면 MissinbBackpressureException 에러를 통지한다.
  - 즉, 소비자가 생산자의 통지 속도를 따라 잡지 못할 때 발생

- Buffer 전략: DROP_LATEST
  - 버퍼가 가득 찬 시점에 버퍼내에서 가장 최근에 버퍼로 들어온 데이터를 DROP한다.
  - DROP 된 빈자리에 버퍼 밖에서 대기하던 데이터를 채운다.

    ![image](https://user-images.githubusercontent.com/42403023/118387966-7f110380-b65c-11eb-81c5-508df46e4143.png)	
      - 11이 drop되고 13이 버퍼에 들어가게 된다.

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

    ![image](https://user-images.githubusercontent.com/42403023/118387998-a5cf3a00-b65c-11eb-8f7b-b6a65d86864c.png)

      - 2가 drop되고 13이 buffer에 들어가게 된다.
      
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
  
    ![image](https://user-images.githubusercontent.com/42403023/118388017-cf886100-b65c-11eb-8884-b6a536a5c9e1.png)

      - 버퍼가 빌 때까지 13 이후 데이터는 drop된다.
      
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
  
    ![image](https://user-images.githubusercontent.com/42403023/118388039-0199c300-b65d-11eb-969f-5f6a2a11b683.png)

      - 버퍼가 빌때까지 쌓인 데이터는 버리고 그 다음 데이터 부터 쌓인다.
      - 여기서는 20~29가 다 처리된 후에 들어오는 39부터 다시 버퍼에 쌓인다.
      
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

#### 다양한 데이터 스트림

- Single (onSucess, onError)
  - 데이터를 1건만 통지하거나 에러를 통지한다.
  - 데이터 통지 자체가 완료를 의미하기 때문에 완료 통지는 하지 않는다.
  - 데이터를 1건만 통지하므로 데이터 개수를 요청할 필요가 없다.
  - onNext(), onComplete()가 없으며 이 둘을 합한 onSuccess()를 제공한다.
  - Single의 대표적인 소비자는 SingleObserver이다.
  - 클라이언트의 요청에 대응하는 서버의 응답이 Single을 사용하기 좋은 대표적인 예이다.
  - 예제
    ```java
    Single<String> single = Single.create(new SingleOnSubscribe<String>() {
        @Override
        public void subscribe(SingleEmitter<String> emitter) throws Exception {
            // TODO Auto-generated method stub
            emitter.onSuccess(LocalDateTime.now().format(DateTimeFormatter.ISO_DATE_TIME));
        }			
    });
    	
    single.subscribe(new SingleObserver<String>() {
        @Override
        public void onSubscribe(Disposable d) {} // 아무것도 하지 않음 
        @Override
        public void onSuccess(String t) {System.out.println("SUCCESS: " + t);}
        @Override
        public void onError(Throwable e) { System.out.println("ERROR: " + e.getMessage()); }			
    });
    ```
  - 람다도 가능하다.    
    ```java
    Single<String> single = Single.create(emitter ->
        emitter.onSuccess(LocalDateTime.now().format(DateTimeFormatter.ISO_DATE_TIME)));
		
    single.subscribe(
        data -> System.out.println("SUCCESS: " + t),
        error -> System.out.println("ERROR: " + e.getMessage())
    );
    ```
  - just 사용
    ```java
    Single.just(LocalDateTime.now().format(DateTimeFormatter.ISO_DATE_TIME))
        .subscribe(
            data -> System.out.println("SUCCESS: " + data),
            error -> System.out.println("ERROR: " + error.getMessage())
        );
    ```
    
- Maybe (onSucess, onCompletable, onError)
  - 데이터를 1건만 통지하거나 1건도 통지하지 않고 완료 또는 에러를 통지한다.
  - 데이터 통지 자체가 완료를 의미하기 때문에 완료 통지는 하지 않는다.
  - 단, 데이터를 1건도 통지하지 않고 처리가 종료될 경우에는 완료 통지를 한다.
  - Maybe의 대표적인 소비자는 MaybeObserver이다.
  - 예제
    ```java
    Maybe<String> maybe = Maybe.create(new MaybeOnSubscribe<String>() {
        @Override
        public void subscribe(MaybeEmitter<String> emitter) throws Exception {
            // TODO Auto-generated method stub
            emitter.onComplete();
        }			
    });
		
    maybe.subscribe(new MaybeObserver<String>() {
        @Override
        public void onSubscribe(Disposable d) {} // 아무것도 하지 않음 
        @Override
        public void onSuccess(String t) {System.out.println("SUCCESS: " + t);}
        @Override
        public void onError(Throwable e) { System.out.println("ERROR: " + e.getMessage()); }
        @Override
        public void onComplete() { System.out.println("COMPLETE!"); }
    });
    ```
  - 람다를 사용할 수 있다.
    ```java
    Maybe<String> maybe = Maybe.create(emitter -> emitter.onComplete());
		
    maybe.subscribe(
        data -> System.out.println("SUCCESS: " + t),
        error -> System.out.println("ERROR: " + error.getMessage()),
        () -> System.out.println("COMPLETE")
    );
    ```
  - just 사용
    - 1건을 보내어 onSuccess를 호출하였다.
      ```java
      Maybe.just(LocalDateTime.now().format(DateTimeFormatter.ISO_DATE_TIME))
          .subscribe(
              data -> System.out.println("SUCCESS: " + data),
              error -> System.out.println("ERROR: " + error.getMessage()),
              ()-> System.out.println("COMPLETE")
          );
      ```
  - empty 사용
    - 1건도 통제하지 않고 바로 onComplete 메서드가 호출된다.
      ```java
      Maybe.empty()
            .subscribe(
                data -> System.out.println("SUCCESS: " + data),
                error -> System.out.println("ERROR: " + error.getMessage()),
                ()-> System.out.println("COMPLETE")
            );
      ```
  - Single 객체로부터 Maybe 객체로 변환하여 사용할 수 있다.
    ```java
    Single<String> single = Single.just(LocalDateTime.now().format(DateTimeFormatter.ISO_DATE_TIME));
    Maybe.fromSingle(single)
          .subscribe(
              data -> System.out.println("SUCCESS: " + data),
              error -> System.out.println("ERROR: " + error.getMessage()),
              ()-> System.out.println("COMPLETE")
          );
    ```
    
- Completable (onCompletable, onError)
  - 데이터 생산자이지만 데이터를 1건도 통지하지 않고 완료 또는 에러를 통지한다.
  - 데이터 통지의 역할 대신에 Completable 내에서 특정 작업을 수행한 후, 해당 처리가 끝났음을 통지하는 역할을 한다.
  - Completable의 대표적인 소비자는 CompletableObserver이다.
  - 예제
    ```java
    Completable completable = Completable.create(new CompletableOnSubscribe() {
        @Override
        public void subscribe(CompletableEmitter emitter) throws Exception {
            // TODO Auto-generated method stub
            int sum = 0;
            for(int i = 0; i < 100; i++) { sum += i; }
            System.out.println("합계: " + sum);
            emitter.onComplete();
        }			
    });
		
    completable.subscribeOn(Schedulers.computation())
        .subscribe(new CompletableObserver() {
            @Override
            public void onSubscribe(Disposable d) {} // 아무것도 하지 않음 
            @Override
            public void onError(Throwable e) { System.out.println("ERROR: " + e.getMessage()); }
            @Override
            public void onComplete() { System.out.println("COMPLETE!"); }
    });
    ```
  - 람다를 사용할 수 있다.
    ```java
    Completable completable = Completable.create(emitter -> {
        int sum = 0;
        for(int i = 0; i < 100; i++) { sum += i; }
        System.out.println("합계: " + sum);
        emitter.onComplete();
    });
		
    completable.subscribeOn(Schedulers.computation())
        .subscribe(
            () -> System.out.println("COMPLETE"), 
            error -> System.out.println("ERROR: " + error.getMessage())
        );
    ```
  
#### 출처

- Kevin의 RxJava 강의
