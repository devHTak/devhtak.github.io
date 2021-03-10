---
layout: post
title: 스터디 할래 15. 람다식
summary: 백기선님과 스터디 할래
author: devhtak
date: '2021-03-10 21:41:00 +0900'
category: Java Study
---

#### 학습할 것

- 람다식 사용법
- 함수형 인터페이스
- Variable Capture
- 메소드, 생성자 레퍼런스

#### 람다식

- 람다 표현식(Lambda Expressions)
  - 식별자 없이 실행 가능한 함수
  - 메소드를 하나의 식으로 표현하는 것이라고 볼 수 있다.
  - 람다식으로 표현하면 return이 없어지므로 람다식을 anonymous function(익명 함수) 이라고도 한다.

- 람다 표현식의 장단점
  - 장점
    - 코드를 간결하게 만들 수 있다.
    - 가독성이 향상된다.
    - 함수를 만드는 과정 없이 한번에 처리하기에 생산성이 높아진다.
  - 단점
    - 람다로 인한 무명함수는 재사용이 불가능하다.
    - 디버깅이 많이 까다롭다.
    - 람다를 무분별하게 사용하면 코드가 클린하지 못하다
    - 재귀로 만들경우 부적합하다.

#### 람다식 사용법

- 사용법

  ```java
  // 표현 방법 1. function body가 한줄인 경우
  (arg1, arg2) -> function body
  
  // 표현 방법 2. 다양한 arguments, 여러 라인의 body 사용 가능
  (arg1) -> {
      // function body
  }
  
  // 표현 방법 3. 매개변수가 없는 경우, body가 한줄인 경우
  () -> function body
  
  // 표현 방법 4. 매개변수가 없고, body가 여러줄인 경우
  () -> {
      // function body
  }
  ```

- 람다 사용 예제
  - Optional을 활용한 람다 예제
    - Optional에 orElseThrow 활용
    
    ```java
    Member member = memberService.findById(1L).orElseThrow(()-> new IllegalArgumentException());
    ```
    - orElseThrow의 구현체
    
    ```java
    /**
     * If a value is present, returns the value, otherwise throws an exception
     * produced by the exception supplying function.
     *
     * @apiNote
     * A method reference to the exception constructor with an empty argument
     * list can be used as the supplier. For example,
     * {@code IllegalStateException::new}
     *
     * @param <X> Type of the exception to be thrown
     * @param exceptionSupplier the supplying function that produces an
     *        exception to be thrown
     * @return the value, if present
     * @throws X if no value is present
     * @throws NullPointerException if no value is present and the exception
     *          supplying function is {@code null}
     */
    public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X {
        if (value != null) {
            return value;
        } else {
            throw exceptionSupplier.get();
        }
    }
    ```
      - (Supplier) 람다를 파라미터로 받고 해당 return 값을 throw하도록 구성되어 있다.

- @FunctionalInterface

  - Java 8부터 람다식이 추가되고 하나의 변수에 하나의 함수를 매핑함으로써 함수형 프로그래밍이 가능해졌다.
  - @FunctionalInterface 는 오직 하나의 메소드 선언을 갖는 인터페이스를 말한다.
    - 인터페이스는 구현체를 생성하여 사용하거나, 직접 메소드 오버라이딩을 정의해서 사용하였다.
    
      ```java
      String[] arr = new String[] {"A", "B", "C"};
      Arrays.sort(arr, new Comparator<String>() {
          @Override
          public int compare(String o1, String o2) {
              // TODO Auto-generated method stub
              return o1.compareToIgnoreCase(o2);
          }			
      });
      ```
    - Lambda를 활용할 수 있다.
    
      ```java
      // 변수로 저장하여 사용
      Comparator<String> comp = (o1, o2) -> o1.compareToIgnoreCase(o2);		
      Arrays.sort(arr, comp);

      // 직접 구현
      Arrays.sort(arr, (o1, o2)-> o1.compareToIgnoreCase(o2));
      ```
      
    - Java 8 이전 원래 abstract 메소드의 선언만 갖을 수 있었다. 하지만 Java 8 이후 부터 default 접근제한자와 함께 메소드 구현도 갖을 수 있다.
    - Comparator는 @FunctionalInterface로 선언되어 있고, 여러 메소드 구현을 갖고 있다.
    
- (바이트코드) INVOKEDYNAMIC CALL

  - 

#### 함수형 인터페이스

- 기본 함수형 인터페이스
  
  |Interface|Function Descriptor|Abstract Method|
  |---|---|---|
  |Predicate<T>|(T) -> boolean|boolean test(T t);|
  |Consumer<T>|(T) -> void|void accept(T t);|
  |Function<T, R>|(T) -> R|R apply(T t);|
  |Supplier<T>|() -> T|T get();|
  |UnaryOperator<T>|(T) -> T|T apply(T t);|
  
  - 등등: https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html

- Runnable
  - 인자를 받지 않고, 리턴값도 없는 인터페이스
  - 쓰레드에서 Runnable 인터페이스로 실행한 것이라고 보면 된다.
  
    ```java
    Runnable runnable = () -> System.out.println("Runnable run"); // run() 정의
    runnable.run(); // Runnable run 출력
    ```
   
  - Runnable은 run()을 호출해야 한다. 
  - 함수형 인터페이스마다 run()과 같은 실행 메소드 이름이 다르다. 인터페이스 종류마다 만들어진 목적이 다르고, 인터페이스 별 목적에 맞는 실행 메소드 이름을 정하기 때문이다.

- Supplier<T>
  - 인자를 받지 않고 T 타입의 객체를 리턴한다.
  
    ```java
    public interface Supplier<T> {
        T get();
    }
    ```
  - 예제
  
    ```java
    Supplier<String> supplier = () -> "Hello Lambda";
    System.out.println(supplier.get()); // Hello Lambda 출력
    ```
  
- Consumer<T>
  - T타입의 객체를 인자로 받고 리턴값이 없다.
  
    ```java
    public interface Consumer<T> {
        void accept(T t);
        default Consumer<T> andThen(Consumer<? super T> after) {
            Objects.requireNotNull(after);
            return (T t) -> { 
                accept(t);
                after.accept(t);
            };
        }
    }
    ```
  - 예제
    - Consumer에 구현된 andThen을 사용하면 두개 이상의 Consumer를 사용할 수 있다.
    
    ```java
    Consumer<String> hello = (c) -> System.out.println("A : " + c); 
		Consumer<String> consumer = (c) -> System.out.println("B : " + c);
		
		hello.andThen(consumer).accept("Hello Consumer"); // A: Hello Consumer, B: Hello Consumer 출력
    ```

- Function<T, R>
  - T 타입의 인자를 받아, R 타입의 객체로 리턴한다.
    
    ```java
    public interface Function<T, R> {
        R apply(T t);
        
        default <V> Function<V, R> compose(Function<? super V, ? textends T> before) {
            Objects.requiredNonNull(before);
            return (V v) -> apply(before.apply(v));
        }
        
        default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
            Objects.requiredNonNull(after);
            return (T t) -> apply(after.apply(t));
        }
        
        static <T> Function<T, T> identity() {
            return t -> t;
        }
    }
    ```
  - 예제
    ```java
    Function<Integer, Integer> add = (val) -> val + 2;
    Function<Integer, Integer> sub = (val) -> val - 2;
    
    Function<Integer, Integer> addAndSub = add.compose(sub);
    System.out.println(addAndSub.apply(10)); // 10
    ```  
    
- Predicate
  - T 타입 인자를 받고 결과로 boolean을 리턴한다.
    ```java
    public interface Predicate<T> {
        boolean test(T t);
        
        default Predicate<T> and(Predicate<? super T> other) {
            Objects.requireNonNull(other);
            return (t) -> test(t) && other.test(t);
        }
        
        default Predicate<T> negate() {
            return (t) -> !test(t);
        }
        
        default Predicate<T> or(Predicate<? super T> other) {
            Objects.requireNonNull(other);
            return (t) -> test(t) || other.test(t);
        }

        static <T> Predicate<T> isEqual(Object targetRef) {
            return (null == targetRef)
                    ? Objects::isNull
                    : object -> targetRef.equals(object);
        }
        
        @SuppressWarnings("unchecked")
        static <T> Predicate<T> not(Predicate<? super T> target) {
            Objects.requireNonNull(target);
            return (Predicate<T>)target.negate();
        }
    }
    ```
    - and(), or()로 다른 Predicate와 함께 사용이 가능하다.
    - isEquals()은 static 메소드로 인자로 전달되는 객체와 같은 지 체크하여 객체를 만들어 준다.

#### Variable Capture


#### 메소드, 생성자 레퍼런스
