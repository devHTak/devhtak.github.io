---
layout: post
title: Effective Java Ch11. 동시성
summary: Effective Java 3판 공부
author: devhtak
date: '2022-08-06 22:41:00 +0900'
category: Effective Java
---

#### Chap11. 동시성

##### Item 78. 공유중인 가변 데이터는 동기화해 사용하라

- synchronized 키워드는 해당 메서드나 블록을 한번에 한 스레드에 수행하도록 보장
  - 한 객체가 일관된 상태를 가지고 생성되고, 이 객체에 접근하는 메서드는 그 객체에 락을 건다.
    - 락을 건 메서드는 객체의 상태를 확인하고 필요하면 수정
  - 동기화 없이는 한 스레드가 만든 변화를 다른 스레드에서 확인하지 못할 수 있다.
  - long, double 외의 변수를 읽고 쓰는 동작은 원자적이다.
    - 여러 스레드가 변수를 동기화없이 수정하더라도 저장한 값을 온전히 읽어옴을 보장

- 동기화는 배타적 실행 뿐 아니라 스레드 사이의 안정적인 통신에 꼭 필요하다

```java
private static boolean stopRequested;

public static void main(String[] args) throws InterruptedException {
    Thread backgroundThread = new Thread(() -> {
        int i = 0;
        while(!stopRequested)
            i++;
    });
    backgroundThread.start();
    
    TimeUnit.SECONDS.sleep(1);
    stopRequested = true;
}
```
  - 1초 후에 끝날 것 같지만 영원히 끝나지 않는다.
  - 원인은 동기화에 있다. 메인 스레드가 수정한 값을 백그라운드 스레드가 언제 볼지 보증할 수 없다.

```java
private static boolean stopRequested;
private static synchronized void requestStop() { stopRequested = true; }
private static synchronized boolean isStopRequested() { return stopRequested; }

public static void main(String[] args) throws InterruptedException {
    Thread backgroundThread = new Thread(() -> {
        int i = 0;
        while(!isStopRequested())
            i++;
    });
    backgroundThread.start();
    
    TimeUnit.SECONDS.sleep(1);
    requestStop();
}
```
- stopRequested 동기화를 통하여 1초 후 종료 보장
  - 쓰기, 읽기 메서드를 동기화 하였다.(쓰기와 읽기 모두 동기화되지 않으면 동작을 보장하지 않는다)
- 또는 getter/setter 대신 volatile 을 선언하면 동기화를 생략해도 된다.
  ```java
  private static volatile boolean stopRequested;
  ```
  - volatile 를 선언하면 변수를 cpu 캐시가 아닌 메인 메모리로부터 읽어온다
  ```java
  private static volatile int serialNumber = 0;
  public static int generateNextSerailNumber() {
      this.serialNumber++;
  }
  ```
  - 하지만 해당에 경우엔 동기화를 보장하지 않는다.
  - serialNumber += 1; 형태로 더하고, 저장하는 과정에서 일관되지 않은 값을 가져올 수 있다.

- java.util.concurrent.atomic 패키지 사용
  - AtomicLong ..
  - 해당 패키지는 락 없이도 스레드 안전한 프로그래밍을 지원하는 클래스들이 담겨 있다.
  
##### Item 79. 과도한 동기화는 피하라

- 과도한 동기화는 성능을 떨어트리고, 교착상태에 빠뜨리고, 예측할 수 없는 동작을 낳기도 한다.
  - 동기화된 영역 안에서는 재정의할 수 있는 메서드는 호출하면 안된다
  - 클라이언트가 넘겨준 함수 객체를 호출해서도 안된다

- 잘못된 코드 예시
  ```java
  public class ObservableSet<E> extends ForwardingSet<E> {
      public ObservableSet(Set<E> set) {
          super(set);
      }

      private final List<SetObserver<E>> observers = new ArrayList<>(); // 관찰자리스트 보관

      public void addObserver(SetObserver<E> observer) { // 관찰자 추가
          synchronized (observers) {
              observers.add(observer);
          }
      }

      public boolean removeObserver(SetObserver<E> observer) { // 관찰자제거
          synchronized (observers) {
              return observers.remove(observer);
          }
      }

      private void notifyElementAdded(E element) { // Set에 add하면 관찰자의 added 메서드를 호출한다.
          synchronized (observers) {
              for (SetObserver<E> observer : observers)
                  observer.added(this, element);
          }
      }

      @Override
      public boolean add(E element) {
          boolean added = super.add(element);
          if (added)
              notifyElementAdded(element);
          return added;
      }

      @Override
      public boolean addAll(Collection<? extends E> c) {
          boolean result = false;
          for (E element : c)
              result |= add(element);  // notifyElementAdded를 호출한다.
          return result;
      }
  }
  ```
  - addObserver, removeObserver를 동기화하였다

- ConcurrentModificationException
  ```java
  public static void main(String[] args) {
      ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());
      
      set.addObserver(new SetObserver<>() {
          public void added(ObservableSet<Integer> s, Integer e) {
              System.out.println(e);
              if(e == 23) 
                  s.removeObserver(this);
          }
      });
      
      for(int i = 0; i < 100; i++)
          s.add(i);
  }
  ```
  - 0 ~ 22까지 정상동작 하다 23 출력 후, ConcurrentModificationException 을 던진다.
  - set.add()를 호출 -> notifyElementAdded가 호출
    - removeObserver를 통해 리스트에서 원소를 삭제하려고 하는데, notifyElementAdded를 통해 리스트를 순회하는 동작으로 허용되지 않는 동작

- 교착상태
  ```java
  set.addObserver(new SetObserver<>() {
      public void added(ObservableSet<Integer> s, Integer e) {
          System.out.println(e);
          if(e == 23) {
              ExcutorService service = Excutors.newSignleThreadExecutor();
              try {
                  exec.submit(() -> s.removeObserver(this)).get();
              } catch(ExcutionException | InterruptedException ex) {
                  throw new AssertionError(ex);
              } finally {
                  exec.shutdown();
              }
              
          }
      }
  });
  ```
  - 백그라운드 스레드가 s.removeobserver를 호출하여 옵저버를 잠그려고 하지만 락을 얻을 수 없다.
  - 메인 스레드에서는 백그라운드 스레드가 observer를 삭제하기를 기다리며 데드락 상태에 빠진다.

- 자바언어의 락은 재진입(reentrant)를 허용하므로 교착상태에 빠지지 않는다.

- 해결 방법 1. 외계인 메서드를 동기화 외부로 옮겼다.
  ```java
  private void notifyElementAdded(E element) {
      List<SetObserver<E>> snapshot = null;
      synchronized (observers) {
          snapshot = new ArrayList<>(observers); 
      }
      for (SetObserver<E> observer : snapshot) // 락 필요없이 순회할 수 있다.
          observer.added(this, element);
  }
  ```
  - 락없이도 리스트를 순회할 수 있다.
- 해결 방법 2. CopyOnWriteArrayList 와 같이 동시성 컬렉션 라이브러리 사용

- 동기화 기본 원칙은 동기화 영역에서는 가능한 할일을 적게 하는 것
  - 가변클래스를 작성하려면, 동기화를 전혀 하지 말고 그 클래스를 동시에 사용해야 하는 클래스가 외부에서 동기화하도록 하자
  - 또한, 동기화를 내부에서 수행해 스레드 안전한 클래스로 만들자
    - 내부에서 동기화 하기로 했다면, 락 분할, 락 스트라이핑, 비차단 동시성 제어 등 다양한 기법을 동원하여 동시성을 높여줄 수 있다.
    - 락 분할(lock splitting) - 하나의 클래스에서 기능적으로 락을 분리해서 사용하는 것(두개 이상의 락 - ex: 읽기전용 락, 쓰기전용 락)
    - 락 스트라이핑(lock striping) - 자료구조 관점에서 한 자료구조 전체가 아닌 일부분(buckets or stripes 단위)에 락을 적용하는 것

##### Item 80. 스레드보다는 실행자, 태스크, 스트림을 애용하라

- java.util.concurrent 패키지는 실행자 프레임워크라고 하는 인터페이스 기반의 유연한 태스크 실행 기능을 담고 있다
  - ExcutorService
    ```java
    ExcutorService exec  = Executors.newSingleThreadExcutor(); // 작업 큐를 생성하다
    
    exec.execute(runnable); // 다음은 이 Excutor에 실행할 태스크(task; 작업)를 넘기는 방법이다

    exec.shutdown(); // 다음은 Excutor를 우아하게 종료시키는 방법이다(이 작업이 실패하면 VM 자체가 종료되지 않을 것이다)
    ```
    - 주요 특징
      - 특정 태스크가 완료되기를 기다린다
      - 태스크 모음 중 아무것 하나(invokeAny 메서드) 혹은 모든 태스크(invokeALL 메서드)가 완료되기를 기다린다
      - 실행자 서비스가 종료하기를 기다린다(awaitTermination 메서드)
      - 완료된 태스크들의 결과를 차례로 받는다(ExecutorCompletionService 이용)
      - 태스크를 특정 시간에 혹은 주기적으로 실행하게 한다(ScheculedThread PoolExecutor 이용)
    - 큐를 둘 이상의 스레드가 처리하게 하고 싶다면 간단히 다른 정적 팩터리를 이용하여 다른 종류의 실행자 서비스(스레드 풀)를 생성하면 된다
    - 평범하지 않은 실행자를 원한다면 ThreadPoolExecutor 클래스를 직접 사용해도 된다. 이 클래스로는 스레드 풀 동작을 결정하는 거의 모든 속성을 설정할 수 있다
    - 작은 프로그램이나 가벼운 서버라면 Executors.newCachedThreadPool을 사용하라. 특별히 설정할 게 없고 일반적인 용도에 적합하게 동작한다
    - 무거운 프로덕션 서버에서는 스레드 개수를 고전한 Executors.newFixedThreadPool을 선택하거나 완전히 통제할 수 있는 ThreadPoolExecutor를 직접 사용하는 편이 훨씬 낫다
    - 주의 점
      - 작업 큐를 손수 만드는 일은 삼가야 하고, 스레드를 직접 다루는 것도 일반적으로 삼가야 한다
      - 스레드를 직접 다루면 Thread가 작업 단위와 수행 메커니즘 역할을 모두 수행하게 된다
      - 반면 실행자 프레임워크에서는 작업 단위와 실행 매커니즘이 분리되어 있어서 의미가 명확하다

- ForkJoinTask
  - 자바 7이 되면서 실행자 프레임워크는 포크-조인 태스크를 지원하도록 확장
  - ForkJoinTask의 인스턴스는 작은 하위 태스크로 나뉠 수 이고, ForkJoinPool을 구성하는 스레드들이 이 태스크들을 처리
    - 일을 먼저 끝낸 스레드는 다른 스레드의 남은 태스크를 가져와 대신 처리 가능하다
  - 포크-조인에 적합한 형태의 작업에서는 포크-조인 태스크를 직접 작성하고 튜닝하기란 어려운 일이지만, 포크-조인 풀을 이용해 만든 병렬 스트림을 이용하면 적은 노력으로 그 이점을 얻을 수 있다

##### Item 81. wait, notify 보다는 동시성 유틸리티를 애용하라

- wait, notify는 올바르게 사용하기 까다로운 고수준 동시성 유틸리티를 사용해야 한다
- java.util.concurrent 패키지
  - Executor 실행자 프레임워크
  - 동시성 컬렉션
    - 동시성 컬렉션에서는 동시성을 무력화하는 것은 불가능하며 외부에서 락을 추가로 사용하면 오히려 속도가 느려진다
    - ConcurrentMap: 동시성이 뛰어나며 속도도 빠르다
    ```java
    public static String intern(String s){
        String result = map.get(s);
            if(result == null){
                result = map.putIfAbsent(s, s);
                if(result == null)
                    result = s;
            }
        return result;
    }
    ```
    - BlockingQueue
  - 동기화 장치(synchronized)
    - CountDownLatch
      - 일회성 장벽으로 하나 이상의 스레드가 또 다른 하나 이상의 스레드 작업이 끝날 때까지 기다리게 한다.
    ```java
    public static long time(Executor executor, int concurrency, Runnable action) throws InturruptedException {
        CountDownLatch ready = new CountDownLatch(concurrenycy);
        CountDownLatch start = new CountDownLatch(1);
        CountDownLatch done = new CountDownLatch(concurrency);

        for (int i = 0; i < concurrency; i++) {
            executor.execute(() -> {
                // 타이머에게 준비를 마쳤음을 알린다.
                ready.countDown();
                try {
                    // 모든 작업자 스레드가 준비될 때까지 기다린다.
                    start.await();
                    action.run();
                } catch (InturruptedException e){
                    Thread.currentThread().interrupt();
                } finally {
                    // 타이머에게 작업을 마쳤음을 알린다.
                    done.countDown();
                }
            });
        }

        ready.await(); // 모든 작업자가 준비될 때까지 기다린다
        long startNanos = System.nanoTime();
        start.countDown(); // 작업자들을 깨운다
        done.await(); // 모든 작업자가 일을 끝마치기를 기다린다
        return System.nanoTime() - startNanos;
    }
    ```
      - ready 래치는 작업자 스레드들이 준비가 완료됐음을 타이머 스레드에게 통지할 때 사용
        - 통지를 끝낸 작업자 스레드들은 두 번째 래치인 start가 열리기를 기다린다.
        - 마지막 작업자 스레드가 ready.countDown을 호출하면 타이머 스레드가 시작 시각을 기록
      - start.countDown()을 호출해 기다리던 작업자 스레드들을 깨운다
      - 그 직후 타이머 스레드는 세 번째 래치인 done이 열리기를 기다린다.
        - done 래치는 마지막 남은 작업자 스레드가 동작을 마치고 done.countDown을 호출하면 열린다
        - 타이머 스레드를 done 래치가 열리자마자 깨어나 종료 시각을 기록한다
    - Semaphore

- wait, notify
  - 새로운 코드라면 wait과 notify 대신 동시성 유틸리티를 사용해야 하지만 레거시를 다뤄야 할 때도 있다
  - wait 메서드를 사용하는 표준 방식
    ```java
    synchronized (obj) {
        while(<조건이 충족되지 않았다>)
            obj.wait(); // 락을 놓고, 깨어나면 다시 잡는다.

        // ... // 조건이 충족되면 동작을 수행한다.
    }
    ```
    - wait 메서드를 사용할 때는 반드시 대기 반복문(wait loop) 관용구 사용
    - 반복문 바깥에서는 절대 호출하지 말아야 하며 이 구문은 wait호출 전 후로 조건이 만족되었는지 확인해야 한다
    - 대기 전에 조건을 검사해 조건이 이미 충족되었다면 wait를 건너뛰게 하는 것은 응답 불가 상태를 예방하는 조치
    - 만약 조건이 충족되었는데 스레드가 notify 메서드를 호출하고 대기 상태로 빠지면, 그 스레드를 다시 깨울 수 있다고 보장할 수 없다.
    - 대기 후에 조건을 검사하여 조건이 충족되지 않았다면 다시 대기하게 하는 것은 안전 실패를 막는 조치
      - 만약 조건이 충족되지 않았는데 스레드가 동작을 이어가면 락이 보호하는 불변식을 깨뜨릴 위험이 있다

  - 조건이 만족되지 않아도 스레드가 깨어날 상황이 몇가지가 있다.
    - 스레드가 notify를 호출하고 대기중이던 스레드가 깨어나는 사이 다른 스레드가 락을 얻어 그 락이 보호하는 상태 변경
    - 조건이 만족되지 않았음에도 다른 스레드가 실수로 혹은 악의로 notify를 호출
    - 깨우는 스레드가 지나치게 관대해 일부 조건만 만족했는데도 notifyAll을 호출하는 경우
    - 대기 중인 스레드가 드물게 notify 없이도 깨어나는 경우 (허위 각성 현상, spurious wakeup)
  
  - notify 대신 notifyAll
    - 일반적으로는 notify 보다는 notifyAll을 호출하는 것이 합리적이고 안전한 조언이다
    - notifyAll은 관련 없는 스레드가 실수로 혹은 악의로 wait를 호출하는 공격으로 보호할 수 있다
    - 깨어나야 하지만 깨어나지 못한 스레드들을 깨울 수 있다. 다른 스레드가 깨워질수 있지만, 조건이 만족되지 않았다면 그 스레드들은 다시 대기할 것이다
    - 모든 스레드가 같은 조건을 기다리고, 조건이 한 번 충족될때마다 하나의 스레드만 깨우는 경우라면 최적화 하기 위한 용도로 notify를 사용해도 된다

##### Item 82. 스레드 안전성 수준을 문서화하라

- API 문서 synchronized 한정자
  - 메서드 선언에 synchronized 한정자를 선언할지는 구현 이슈일 뿐 API에 속하지 않는다
  - API 문서에 synchronized 한정자가 보인다고해서 이 메서드가 스레드 안전하다고 믿기 어렵다
  - 멀티 스레드 환경에서 안전하게 사용하려면 지원하는 스레드 안전성 수준을 정확히 명시해야한다

- 스레드 안전성 수준
  - 불변(immutable)
    - 이 클래스의 인스턴스는 마치 상수와 같아서 외부 동기화가 필요없다
    - String , Long, BigInteger
    - @Immutable
 
  - 무조건적 스레드 안전(unconditionally thread-safe)
    - 이 클래스의 인스턴스는 수정될 수 있으나, 내부에서 충실히 동기화하여 별도의 외부 동기화 없이 동시에 사용해도 안전하다
    - AtomicLong ,ConcurrentHashMap
    - @ThreadSafe
 
  - 조건부 스레드 안전(conditionally thread-safe)
    - 무조건적 스레드 안전과 같으나, 일부 메서드는 동시에 사용하려면 외부 동기화가 필요하다
    - Collections.synchronized 래퍼 메서드가 반환한 컬렉션들
    - @ThreadSafe
 
  - 스레드 안전하지 않음(not thread-safe)
    - 이 클래스의 인스턴스는 수정될 수 있다. 동시에 사용하려면 각각의 메서드 호출을 클라이언트가 선택한 외부 동기화 메커니즘으로 감싸야한다
    - ArrayList , HashMap
    - @NotThreadSafe
 
  - 스레드 적대적(thread-hostil)
    - 이 클래스는 모든 메서드 호출을 외부 동기화로 감싸더라도 멀티스레드 환경에서 안전하지 않다
    - 이 수준의 클래스는 일반적으로 정적 데이터를 아무 동기화 없이 수정한다
    - 문제를 고쳐 재배포 하거나 사용자제 (deprecated) API로 지정한다
 
- Conditionally Thread-safe의 문서화
  - 어떤 순서로 호출할 때 외부 동기화가 필요한지
  - 그 순서로 호출하려면 어떤 락 혹은 락들을 얻어야하는지
  
- Lock 제공
  - 외부에서 사용할 수 있는 락을 제공하면 클라이언트에서 메서드 호출을 원자적으로 수행할 수 있다
  - 내부에서 처리하는 고성능 동시성 제어 메커니즘과 혼용이 안된다
    - CurrentHashMap과는 사용 하면 안된다
  - 클라이언트가 공개된 락을 오래 쥐고 놓지 않는 서비스 거부 공격 (denail-of-service attack)을 수행할 수도 있다
  - synchronized 메서드도 공개된 락에 속한다
  - 비공개 락 객체를 사용해야한다
    - 비공개 락 객체, final 선언으로 락 객체 교체 상황 방지해야 한다
  ```java
  private final Object lock = new Object();

  public void someMethod() {
      synchronized(lock) {
          // do something
      }
  }
  ```

- 락필드는 항상 final로 선언하라
  - 비공개 락 객체 관용구는 무조건적 스레드 안전 클래스에서만 사용가능
  - 조건부 스레드 안전클래스에서는 특정 호출 순서에 필요한 락을 알려줘야하므로 이 관용구를 사용할 수 없다

##### Item 83. 지연 초기화는 신중히 사용하라

- 지연 초기화(lazy initialization)는 필드의 초기화 시점을 그 값이 처음 필요할 때까지 늦추는 기법
  - 최적화 용도로 사용하며, 클래스와 인스턴스 초기화 때 발생할 수 있는 순환 문제를 해결하는 효과도 있다

- 지연 초기화는 장단점을 가지고 있다.
  - 생성시의 초기화 비용은 줄어들지만, 지연초기화는 필드에 접근하는 비용이 커진다.
  - 해당 클래스의 인스턴스 중 그 필드를 사용하는 인스턴스의 비율이 낮은 반면, 그 필드를 초기화하는 비용이 크다면 지연 초기화는 올바른 역할을 한다.
  - 하지만 정말 그런지, 지연 초기화 적용 전, 후의 성능을 측정해보아야 한다

- 멀티스레드 환경에서 지연초기화는 까다롭다
  - 대부분의 상황에서는 일반적인 초기화가 지연 초기화보다 낫다
    - 인스턴스 필드를 선언할 때 수행하는 일반적인 초기화 모습
      ```java
      private final FieldType field = computeFieldValue();
      ```
  - 멀티쓰레드에서 안전하게 지연 초기화 하는 방법
    - 지연초기화가 초기화 순환성을 깨트릴 것 같으면 synchronized 를 단 접근자를 사용하자
    ```java
    private FieldType field;
    private synchronized FieldType getField() {
        if(field == null)
            this.field = computeFieldValue();
        return field;
    }
    ```
  - 성능 때문에 정적 필드를 지연 초기화해야 한다면 지연 초기화 홀더 클래스(lazy initialization holder class) 관용구를 사용하자
    ```java
    private static class FieldHolder {
        static final FieldType field = computeFieldValue();
    }
    private static FieldType getField() {
        return FileHolder.field;
    }
    ```
    - getFeild() 가 처음 호출되는 순간 FieldType.field가 처음 읽히면서, FieldHolder 클래스 초기화를 촉발
    - getField 메서드가 필드에 접근하면서 동기화를 전혀 하지 않으니 성능에 영향이 없다.
  - 성능 때문에 인스턴스 필드를 지연 초기화해야 한다면 이중검사 관용구(double-check)를 사용하자(volatile)
    - 초기화된 필드에 접근할 때의 동기화 비용을 없애준다
    ```java
    private volatile FieldType field;
    private FieldType getField() {
        FieldType result = field;
        if(result != null) // 첫번째 검사 (락 사용 안함) 
            return result;
         synchronized(this) {
            if(field == null)
                field = computeFieldValue();
            return field;
         }
    }
    ```
    - result 지역변수를 사용한 이유는 필드가 이미 초기화된 상황에서 그 필드를 한번만 읽도록 보장하는 역할을 한다
    - 단일 검사 관용구에 경우 초기화가 중복되어 발생할 수 있다
  - 3가지 초기화 기법은 기본타입 필드, 객체 참조 필드 모두에 적용될 수 있다.

##### Item 84. 프로그램의 동작을 스레드 스케줄러에 기대지 마라

- 여러 쓰레드가 실행중이면 쓰레드 스케줄러가 어떤 스레드를 얼마나 오래 실행할 지 정한다
  - 운영체제마다 스케줄링 정책은 다를 수 있다
  - 정확성이나 성능 스레드 스케줄러에 따라 달라지는 프로그램이면 다른 플랫폼에 이식하기 어렵다

- 실행 가능한 스레드의 평균적인 수를 프로세서 수보다 많아지지 않도록 하는 것
  - 실행 가능한 스레드 수를 적게 유지하는 주요 기법은 각 스레드가 유용한 작업을 완료한 후에 추가 일거리가 발생하기 전까지 대기하는 것
  - 스레드는 당장 처리할 작업이 없다면 실행돼서는 안된다

- 바쁜 대기(bysh waiting)
  - 공유 객체의 상태가 바뀔때까지 쉬지않고 검사해서는 안된다.
  - 스레드 스케줄러의 취약하며 프로세서에 큰 부담을 준다
  ```java
  public class SlowCountDownLatch {
      private int count;
      
      public SlowCountDownLatch(int count) {
          this.count = count;
      }
      
      public void await() {
          while(true) {
              synchronized(this) {
                  if(count == ) return;
              }
          }
      }
      
      public synchronized void countDown() {
          if(count != 0) count--;
      }
  }
  ```
  - 1000개의 스레드를 만드는 기존 CountDownLatch를 사용하는 것보다 10배가 느려진다

- Thread.yield
  - 특정 스레드가 다른 스레드들과 비교해 CPU 시간을 충분히 얻지 못해서 간신히 돌아가는 프로그램을 보더라도 Thread.yield를 써서 문제를 고치려 하지 말자
  - 성능에 효과가 있는 yield 도, 다른 OS에 이식됨에 따라 효과가 없을 수도, 오히려 느려지게 할 수 있다.
  - 테스트할 수 있는 수단도 
