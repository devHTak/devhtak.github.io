---
layout: post
title: 스터디 할래 10. 멀티쓰레드 프로그래밍
summary: 백기선님과 스터디 할래
author: devhtak
date: '2021-02-03 21:41:00 +0900'
category: Java Study
---

#### 학습할 것
- Thread 클래스와 Runnable 인터페이스
- 쓰레드의 상태
- 쓰레드의 우선순위
- Main 쓰레드
- 동기화
- 데드락

#### Thread 클래스와 Runnable 인터페이스

- 프로세스와 스레드  
  - 프로세스(Process)
    - 실행 중인 프로그램
    - 운영체제에 의해 메모리 공간을 할당 받아 실행 중인 것을 말한다. 이러한 프로세스는 프로그램에 사용되는 데이터와 메모리 등의 자원 그리고 쓰레드로 구성된다.
    
  - 쓰레드(Thread)
    - 프로세스 내에 실제 작업 수행 주체
    - 모든 프로세스는 1개 이상의 쓰레드가 존재하여 작업을 수행
    - 두 개 이상의 쓰레드를 가지는 프로세스를 멀티 쓰레드 포로세스라고 한다.
    - 경량 프로세스라고 불리며 가장 작은 실행 단위이다.
    
- Thread 클래스와 Runnable 인터페이스
  - 쓰레드를 생성하는 방법으로 크게 2가지 있다.
    - Thread Class 사용
    - Runnable Class 사용
      - Runnable 인터페이스는 run 메소드만 구현되어 있는 함수형 인터페이스
      - Thread 클래스는 Runnable 인터페이스를 구현한 클래스이므로 어떤 것을 적용하느냐의 차이
      - run만 오버라이딩할 계획이면 Runnable 인터페이스를 구현하면 된다. 다만, 다른 기능들을 사용하고자 한다면 Thread 클래스를 상속하여 

- 예제
  - Thread 구현체
    ```java
    public class ThreadExample extends Thread {
        @Override
        public void run() {
            // TODO Auto-generated method stub
            for(int i = 0; i < 10; i++) {
                System.out.print("Thread");
            }
        }
    }
    ```
    
  - Runnable 구현체
    ```java
    public class RunnableExample implements Runnable{
        @Override
        public void run() {
            // TODO Auto-generated method stub
            for(int i = 0; i < 10; i++) {
                System.out.print("Runnable");
            }
        }
    }
    ```
  
  - 실행
    ```java
    public static void main(String[] args) {
        ThreadExample thread1 = new ThreadExample(); // thread 실행		
        Thread thread = new Thread(new RunnableExample()); // runnable 구현체 실행

        thread.start();
        thread1.start();
    }
    ```
    - start()와 run() 메소드
      - start 메서드는 쓰레드가 작업을 실행할 호출 스택을 만들고 그 안에 run 메서드를 올려주는 역할을 한다.
      
    
#### 쓰레드의 상태

![image](https://user-images.githubusercontent.com/42403023/116192711-2f2fd280-a769-11eb-8692-7472dd7cacc9.png)

** 이미지 출처: https://cornswrold.tistory.com/185

- Thread 상태

  |상태|의미|
  |---|---|
  |NEW|쓰레드 객체는 생성되었지만, 아직 시작되지 않은 상태|
  |RUNNABLE|쓰레드가 실행중인 상태|
  |BLOCKED|쓰레드가 실행 중지 상태이며, 모니터 락(Monitor Lock)이 풀리기를 기다리는 상태|
  |WAITING|쓰레드가 대기중인 상태|
  |TIMED_WAITING|특정 시간만큼 쓰레드가 대기중인 상태|
  |TERMINATED|쓰레드가 종료된 상태|

- 상태 관계
  - 쓰레드를 생성하고 start()를 호출하면 바로 실행되는 것이 아니라 실행 대기열에 저장되어 자신의 차례가 될 때까지 기다려야 합니다. 
    실행 대기열은 큐(queue)와 같은 구조로 먼저 실행대기열에 들어온 쓰레드가 먼저 실행됩니다.
  - 자기 차례가 되면 실행상태가 됩니다.
  - 할당된 실행시간이 다되거나 yield() 메소드를 만나면 다시 실행 대기상태가 되고 다음 쓰레드가 실행상태가 됩니다.
  - 실행 중에 suspend(), sleep(), wait(), join(), I/O block에 위해 일시정지상태가 될 수 있습니다.
    (I/O block은 입출력 작업에서 발생하는 지연상태를 말합니다. 사용자의 입력을 받는 경우를 예로 들수 있습니다.)
  - 지정된 일시정지시간이 다되거나, notify(), resume(), interrupt()가 호출되면 일시정지상태를 벗어나 다시 실행 대기열에 저장되어 자신의 차례를 기다리게 됩니다.
  - 실행을 모두 마치거나 stop()이 호출되면 쓰레드는 소멸됩니다.

- 상태 제어 메소드

  |메서드|설명|
  |---|---|
  |static void sleep(long millis)|지정된 시간동안 쓰레드를 일시정지시킨다. 지정된 시간이 지나고 나면, 자동적으로 다시 실행대기 상태가 된다.|
  |void join()|지정된 시간동안 쓰레드가 실행되도록 한다. join()을 호출한 쓰레드는 그동안 일시정지 상태가 된다. 지정된 시간이 지나거나 작업이 종료되면 join()을 호출한 쓰레드로 다시 돌아와 실행을 계속한다.|
  |void interrupt()|sleep()이나 join()에 의해 일시정지 상태인 쓰레드를 깨워서 실행대기 상태로 만든다. 해당 쓰레드에서는 InterruptedException이 발생함으로써 일시정지 상태를 벗어나게 된다.|
  |void stop()|쓰레드를 즉시 종료시킨다. deprecated 되었다.|
  |void suspend()|일시정지 된 쓰레드를 실행대기 상태로 만든다. deprecated 되었다.|
  |void resume()|suspend()에 의해 일시정지 상태에 있는 쓰레드를 실행대기 상태로 만든다. deprecated 되었다.|
  |static void yield()|실행 중에 자신이 주어진 실행시간을 다른 쓰레드에게 양보하고 자신은 실행대기 상태가 된다.|
  
  - stop(), suspend(), resume()은 쓰레드를 교착상태(dead-lock)로 만들기 쉽기 때문에 deprecated 되었다.

#### 쓰레드의 우선순위

- 쓰레드는 우선순위(priority)라는 속성(멤버변수)을 가지고 있는데, 이 우선순위의 값에 따라 쓰레드가 얻는 시간이 달라진다.
- 작업의 중요도에 다라 쓰레드의 우선순위를 다르게 지정하여 특정 쓰레드가 더 많은 작업시간을 갖도록 할 수 있다.

- 쓰레드의 우선순위를 지정하는 필드와 메소드
  - 1부터 10까지의 숫자를 가지며 높을수록 우선순위가 높다.
  - default로 5를 갖고 있다.
  
  |필드 또는 메소드|설명|
  |---|---|
  |public final static int MIN_PRIORITY|우선 순위의 최소값|
  |public final static int NORM_PRIORITY|우선 순위의 값|
  |public final static int MAX_PRIORITY|우선 순위의 최대값|
  |public final void setPriority(int newPriority)|우선순위 지정|
  |public final int getPriority()|우선순위 반환|
  
#### Main 쓰레드

- Main Thread
  - Java의 실행환경인 JVM은 하나의 프로세스로 실행된다.
  - 즉, 자바의 애플리케이션은 기본적으로 하나의 메인 쓰레드를 갖는다.
  - 메인 스레드는 메인 메소드를 실행한다.

- Daemon Thread
  - 다른 일반 쓰레드의 작업을 돕는 보조적인 역할을 수행하는 쓰레드
  - 사용자 쓰레드가 모두 종료되면 데몬 쓰레드는 강제적으로 자동 종료된다.
  - 데몬 쓰레드는 보조역할을 수행하므로 사용자 쓰레드가 모두 종료되고 나면 데몬 쓰레드의 존재는 의미가 없기 때문이다.
  - 데몬 쓰레드의 예로는 Garbage Collector 등이 있다.
  - 관련 메소드
    - isDaemon(): 데몬 쓰레드인지 확인
    - setDaemon(): 쓰레드를 데몬 쓰레드 또는 사용자 쓰레드로 변경, true이면 데몬 쓰레드가 됩니다.
  
#### 동기화

- 멀티쓰레드 프로세스의 경우 여러 쓰레드가 같은 프로세스 내의 자원을 공유하기 때문에 서로의 작업에 영향을 줄  수 있다.
- 이러한 일을 방지하기 위해 한 쓰레드가 특정 작업을 끝마치기 전까지 다른 쓰레드에 의해 방해받지 않도록 하는 것이 필요하다. 
- 그래서 도입된계 임계영역(critical section)과 잠금(locak)이다.
- 공유 데이트를 사용하는 코드 영역을 임계 영역으로 지정하고, 공유 데이터(객체)가 가지고 있는 lock을 획득한 단 하나의 쓰레드만 이 영역 내의 코드를 수행할 수 있게 한다.
- 해당 쓰레드가 임게 영역 내의 모든 코드를 수행하고 벗어나서 lock을 반납해야만 다른 쓰레드가 반나보딘 lock을 획득하여 임계영역의 코드를 수행할 수 있게 된다.
- 한 쓰레드가 진행중인 다른 쓰레드가 간섭하지 못하도록 막는 것을 "쓰레드의 동기화"라고 한다.
- synchronized 블록, 1.5부터 java.util.concurrent.locks와 java.util.concurrent.aotmic 패키지를 통해 다양한 방식으로 동기화를 구현할 수 있도록 지원하고 있다.

- synchronized 동기화
  - synchronized 동기화 방법
    - 메서드 전체를 임계영역으로 지정: ex)public synchronized void methodName() {...}
      - 쓰레드는 synchronized 메서드가 호출된 시점부터 해당 메서드가 포함된 객체의 lock을 얻어 작업을 수행하다가 메서드가 종료되면 lock을 반환한다.
    - 특정한 영역을 임계 영역으로 지정: ex) synchronized(객체의 참조변수) {...}
      - 이 때 참조변수는 락을 걸고자 하는 객체를 참조하는 것이어야 한다.
      - synchronized 블럭의 영역 안으로 들어가면서부터 쓰레드는 지정된 객체의 lock을 얻게 되고, 이 블럭을 벗어나면 lock을 반납한다.
    - 두 방법 모두 lock의 획득과 반납이 자동적으로 이뤄지므로 임게영역만 설정해주면 된다.
     
  ```java
  public class Account {
      private int balance = 1000;
      public int getBalance() {
          return this.balance;
      }
      public synchronized void withdraw(int money) {
          if(this.balance >= money) {
              try {
                  Thread.sleep(1000);
              } catch(InterruptedException e) {}

              balance -= money;
          }
      }
  }
  ```
  ```java
  public class RunnableExample implements Runnable{
      private String name;
      private Account account = new Account();

      public RunnableExample(String name) {
          this.name = name;
      }

      @Override
      public void run() {
          // TODO Auto-generated method stub
          while(account.getBalance() > 0) {
              int money = (int)(Math.random() * 3) * 100;
              account.withdraw(money);
              System.out.println(this.name + " balance: " + account.getBalance() );
          }
      }
  }
  ```
  ```java
  public static void main(String[] args) {
      Thread thread1 = new Thread(new RunnableExample("First"));
      Thread thread2 = new Thread(new RunnableExample("Second"));
      Thread thread3 = new Thread(new RunnableExample("Third"));

      thread1.start();
      thread2.start();
      thread3.start();
	}
  ```

- wait() 과 notify()
  - 동기화된 임계 영역의 코드를 수행하다가 작업을 더 이상 진행할 상황이 아니면 일단 wait()를 호출하여 쓰레드가 락을 반납하고 기다리게 한다.
  - wait()은 notify() 또는 notifyAll()이 호출될 때까지 기다리지만, 매개변수가 있는 wait()은 지정된 시간동안 기다린다.
  - Object에 정의되어 있으며 synchronized 블록 내에서만 사용할 수 있으며 보다 효율적인 동기화를 가능하게 한다.
  - 기아 현상과 경쟁상태
    - wait()중인 쓰레드가 notify()를 못받고 오랫동안 기다리는 현상을 기아 현상이라고 한다.
    - 이를 막기 위해 notify() 대신 notifyAll()을 사용해야 한다.
    
- Lock과 Condition을 이용한 동기화
  - jdk 1.5부터 추과된 java.util.concurrent.locks 패키지를 통해 동기화할 수 있다.
  - Lock 클래스
    - ReentrantReadWriteLock: 재진입이 가능한 lock. 가장 일반적인 배타 lock
    - ReendtrantReadWriteLock: 읽기에는 공유적이고, 쓰기에는 배타적인 lock
    - StampedLock: ReendtrantReadWriteLock에 낙관적인 lock을 추가
    
- ReentrantLock
  - ReentrantLock은 가장 일반적인 lock이다.
  - ‘reentrant(재진입할 수 있는)’이라는 단어가 앞에 붙은 이유는 wait() & notify()처럼, 특정 조건에서 lock을 풀고 나중에 다시 lock을 얻어 이후의 작업을 수행할 수 있기 때문이다.
  - 생성자
    ```java
    public ReentrantLock() {
        sync = new NonfairSync();
    }
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
    ```
    - fair는 공정하게 처리한다는 의미로, 가장 오래된 쓰레드를 찾는 과정이 추가된다.
    - 성능 이슈가 떨어지므로 공정함보다는 성능을 선택한다.
  - lock 관련 메서드
    ```java
    void lock() // lock을 잠근다.
    void unlock() // lock을 해제한다.
    boolean isLocked() // lock이 잠겼는지 확인한다.
    ```
    - synchronized와 달리 해당 메서드로 수동으로 lock을 잠그고 
    ```java
    if(!lock.isLocked())
    	lock.lock();
    try {
    	// 임계영역
    } finally {
    	lock.unlock();
    }
    ```
    - 위 소스와 같이 임계영역 내에서 예외가 발생하거나 return문으로 빠져나올 경우 unlock되도록 finally구문에 작성한다.
    ```java
    boolean tryLock()
    boolean tryLock(long timeout, TimeUnit unit) throws InterruptedException
    ```
    - tryLock은 다른 쓰레드에 의해 lock이 걸려 있으면 lock을 얻으려고 기다리지 않거나 지정된 시간만큼 기다린다.
    - lock을 얻으면 true, 얻지못하면 false를 반환한다.
    - lock()은 lock을 얻을 때까지 쓰레드를 블락시키므로 쓰레드의 응답성이 나빠질 수 있다. 응답성이 중요할 경우 tryLock을 이용하여 lock에 대한 결과를 사용자가 결정할 수 있게 한다.
    - InterruptedException이 발생할 수 있는 데, 지정된 시간동안 lock을 얻으려고 기다리는 중에 interrupt()에 의해 작업이 취소될 수 있도록 코드를 작성할 수 있다는 뜻이다.
  - ReentrantLock과 Condition
    - Condition은 wait() & notify()에서 쓰레드를 구분해서 연락하지 못한다는 단점을 해결하기 위한 것이다.
    - wait() & notify()로 쓰레드의 종류를 구분하지 않고, 공유 객체의 waiting pool에 같이 몰아넣은 대신, 각각의 쓰레드의 Condition을 만들어서 각각의 waiting pool에서 기다리도록 하면 문제는 해결된다.
    - Condition은 이미 생성된 lock으로부터 new Condition()을 호출해서 생성한다.

  
- ReentrantReadWriteLock
  - ReentrantReadWriteLock은 읽기를 위한 lock과 쓰기를 위한 lock을 제공한다. 
  - ReentrantLock은 배타적인 lock이라서 무조건 lock이 있어야만 임계 영역의 코드를 수행할 수 있지만, ReentrantReadWriteLock은 읽기 lock이 걸려있으면, 다른 쓰레드가 읽기 lock을 중복해서 걸고 읽기를 수행할 수 있다. 
  - 읽기는 내용을 변경하지 않으므로 동시에 여러 쓰레드가 읽어도 문제가 되지 않는다. 그러나 읽기 lock이 걸린 상태에서 쓰기 lock을 거는 것은 허용되지 않는다. 
  - 반대의 경우도 마찬가지다. 읽기를 할 때는 읽기 lock을 걸고, 쓰기 할 때는 쓰기 lock을 거는 것일 뿐 lock을 거는 방법은 같다.

- StampedLock
  - StampedLock은 lock을 걸거나 해지할 때 ‘스탬프(long타입의 정수값)’를 사용하며, 읽기와 쓰기를 위한 lock외에 ‘낙관적 읽기 lock(optimistic reading lock)’이 추가된 것이다. 
  - 읽기 lock이 걸려있으면, 쓰기 lock을 얻기 위해서는 읽기 lock이 풀릴 때까지 기다려야하는데 비해 ‘낙관적 읽기 lock’은 쓰기 lock에 의해 바로 풀린다. 
  - 낙관적 읽기에 실패하면, 읽기 lock을 얻어서 다시 읽어 와야 한다. 무조건 읽기 lock을 걸지 않고, 쓰기와 읽기가 충돌할 때만 쓰기가 끝난 후에 읽기 lock을 거는 것이다.

#### 데드락

- 교착상태(deadlock)은 두 개 이상의 작업이 서로 상대방의 작업이 끝나기를 기다리고 있어서 아무것도 완료되지 못하는 상태를 말한다.
- 교착상태 조건
  - 상호배제(Mutual exclusion): 프로세스들이 필요로 하는 자원에 대해 배타적인 통제권을 요구한다.
  - 점유대기(Hold and wait): 프로세스가 할당된 자원을 가진 상태에서 다른 자원을 기다린다.
  - 비선점(No preemption): 프로세스가 어떤 자원의 사용을 끝낼 때까지 그 자원을 뺏을 수 없다.
  - 순환대기(Circular wait): 각 프로세스는 순환적으로 다음 프로세스가 요구하는 자원을 가지고 있다.

- 예방
  - 상호배제 조건의 제거
    - 교착 상태는 두 개 이상의 프로세스가 공유가능한 자원을 사용할 때 발생하는 것이므로 공유 불가능한, 즉 상호 배제 조건을 제거하면 교착 상태를 해결할 수 있다.
  - 점유와 대기 조건의 제거
    - 한 프로세스에 수행되기 전에 모든 자원을 할당시키고 나서 점유하지 않을 때에는 다른 프로세스가 자원을 요구하도록 하는 방법이다. 자원 과다 사용으로 인한 효율성, 프로세스가 요구하는 자원을 파악하는 데에 대한 비용, 자원에 대한 내용을 저장 및 복원하기 위한 비용, 기아 상태, 무한대기 등의 문제점이 있다.
  - 비선점 조건의 제거
    - 비선점 프로세스에 대해 선점 가능한 프로토콜을 만들어 준다.
  - 환형 대기 조건의 제거
    - 자원 유형에 따라 순서를 매긴다.
  - 이 해결 방법들은 자원 사용의 효율성이 떨어지고 비용이 많이 드는 문제점이 있다.

- 회피
  - 자원이 어떻게 요청될지에 대한 추가정보를 제공하도록 요구하는 것으로 시스템에 circular wait가 발생하지 않도록 자원 할당 상태를 검사한다.
  - 회피 알고리즘
    - 자원 할당 그래프 알고리즘 (Resource Allocation Graph Algorithm)
    - 은행원 알고리즘 (Banker’s algorithm)
      - 은행에서 100원을 갖고 있고, A는 50원, B는 30원, C는 60원을 빌려달라고 한다. 은행은 A, B, C에게 일괄적으로 25원을 빌려주어 25원을 갖고있고, 이로 인해 A는 25, B 5, C 45원을 추가로 요구한다.
      - 이에 은행은 나머지 25원으로 B에게 5원을 빌려주고 갚을때까지 기다린다. 갚아서 자원이 있을 때 A에게 빌려주고, 또 돈을 갚도록 기다린 후 C에게 빌려준다.
      - 즉, 은행이 모두에 요구에 맞게 갚을 수 있도록 하는 것이 안전상태이다.
      - 불안전상태는 만약 C가 25이 아닌 45를 요구하여 빌려주면 5원밖에 남지 않아 빌려줄 방법이 없다... 이것이 불안전 상태
      - 안전상태: 교착상태 가능 X, 안전 순서열 존재, 프로세스의 요구에 맞게 자원 할당 가능
      - 불안전상태: 교착상태 가능 O, 안전 순서열 존재 X, 프로세스의 요구에 맞게 자원 할당 가능 X
      - 은행원 알고리즘은 이렇게 불안전상태에 빠지지 않도록 자원 할당을 회피하도록 하는 것

- 무시
  - 예방과 회피방법을 활용하면 성능 상 이슈가 발생하는데, 데드락 발생에 대한 상황을 고려하는 것에 대한 비용이 낮다면 별다른 조치를 하지 않을 수도 있다고 한다.

** 출처: 백기선님과 스터디할래#10 - Multi Thread Programming
