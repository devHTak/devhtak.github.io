---
layout: post
title: Effective Java Ch08. 메서드
summary: Effective Java 3판 공부
author: devhtak
date: '2020-11-29 15:41:00 +0900'
category: java
---

### ITEM 49. 매개변수가 유효한지 검사하라

- 매개변수 검사
  - 매개변수를 제대로 검사하지 못하면 문제가 발생할 수 있다. (실패 원자성, failure aotmicity을 어기는 결과를 낳을 수 있다.)
    - 메서드가 수행되는 중간에 모호한 예외를 던지며 실패할 수 있다.
    - 메서드가 잘 수행되지만 잘못된 결과를 반환할 수 있다.
    - 메서드는 문제없이 수행됐지만, 어떤 객체를 이상한 상태로 만들어 놓아 알 수 없는 시점에 해당 메서드와 상관없는 오류를 낼 수 있다.
    
  - public, protected 메서드는 매개변수 값이 잘못됐을 때 던지는 예외를 문서화해야 한다.
    - IllegalArgumentException, IndexOutOfBoundsException, NullPointerException 중 하나
    
      ```java
      /*
      * (현재 값 mod m) 값을 반환한다. 이 메서드는
      * 항상 음이 아닌 BigInteger를 반환한다는 점에서 remainder 메서드와 다르다
      *
      * @param m 계수(양수여야 한다.)
      * @return 현재 값 mod m
      * @throws ArithmeticException m이 0보다 작거나 같으면 발생한다.    
      */
      public BigInteger mod(BigInteger m) {
          if( m.signum() <= 0 ) {
              throws new ArithmeticException("계수(m)은 양수여야 합니다. " + m);
          }
          //...
      }
      ```
      - m 이 null이면 m.signum()을 호출할 때 NullPointerException을 던진다. 하지만 메서드 설명에 해당 내용을 기술하지 않았다.
      - 그 이유는 BigInteger 클래스 수준에서 기술했기 때문이다.
      - java.util.Objects.requireNonNull 메서드를 사용하여 null 검사를 수동으로 하지 않아도 된다.
      ```java
      // 자바의 null 검사 기능 사용하기
      this.strategy = Objects.requireNonNull(strategy, "전략");
      ```
      - 9버전 부터는 Objects 객체를 활용하여 범위 검사 기능도 더해졌다.
        - checkFromIndexSize, checkFromToIndex, checkIndex
    
  - private 메서드 통제
    - assert를 사용해 매개변수 유효성을 검증할 수 있다.
    ```java
    private static void sort(long[] a, int offset, int length) {
        assert a != null;
        assert offset >= 0 && offset <= a.length;
        assert length >= 0 && length <= a.length - offset;
        // 계산 수행
    }
    ```
    - 여기서 이 assert 문들은 자신이 단언한 조건이 무조건 참이라고 선언한다는 것이다.
      - 실패하면 AssertionError를 던진다.
      - 런타임에 아무런 효과도, 아무런 성능 저하도 없다.

  - 매개변수 유효성 검사 예외
    - 유효성 검사 비용이 지나치케 높거나 실용적이지 않을 때
    - 계산 과정에서 암묵적으로 검사가 수행될 때
      - 하지만 암묵적 유혀성 검사에 너무 의존했다가 실패 원자성을 해칠 수 있다.
      - 때로는 계산 과정에서 필요한 유효성 검사가 이뤄지지만 실패했을 때 잘못된 예외를 던지기도 한다.

### ITEM 50. 적시에 방어적 복사본을 만들라

- 방어적으로 프로그래밍 하라
  ```java
  public final class Period {
      private final Date start;
      private final Date end;
      
      /*
      * @param start 시작 시각
      * @param end 종료 시각, 시작 시각보다 뒤여야 한다.
      * @throws IllegalArgumentException 시작 시각이 종료 시각보다 늦을 때 발생
      * @throws NullPointerException start나 end가 null인 경우 발생
      public Period(Date start, Date end) {
          if(start.compareTo(end) > 0) throw new IllegalArgumentException(start + " after " + end);
          this.start = start;
          this.end = end;
      }
      public Date start() { return this.start; }
      public Date end() { returh this.end; }
      // ... 나머지 코드 생략
      */
  }
  ```
  - 불변 클래스로 보이며, 시작 시간이 종료 시각보다 늦을 수 없다는 불변식이 무리 없이 지켜질 것 같다
    - 하지만 Date가 가변이라는 사실을 이용하면 어렵지 않게 그 불변식을 깨트릴 수 있다.
    ```java
    Date start = new Date();
    Date end = new Date();
    Period p = new Period(start, end);
    end.setYear(78); // p의 내부를 수정할 수 있다.
    ```
    - 8버전 이후부터 Date 대신 Instant를 사용하면 된다 (또는 LocalDateTime, ZonedDateTime)
    - 외부 공격으로부터 Period 인스턴스의 내부를 보호하려면 생성자에서 받은 가변 매개변수를 각각을 방어적으로 복사해야 한다.
    ```java
    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());
        if(this.start.compareTo(this.end) > 0 ) throw new IllegalArgumentException(this.start + " after " + this.end);
    }
    ```
    - 방어적 복사본을 만든 후에 유효성을 검사했다.
  - 매개변수가 제 3자에 의해 확장될 수 있는 타입이라면 방어적 복사본을 생성할 때 clone을 사용하면 안된다.
  - 가변인 내부 객체를 반환할 때는 반드시 심사숙고해야 하며 안심할 수 없다면 방어적 복사본을 반환해야 한다.
    ```java
    Date start = new Date();
    Date end = new Date();
    Period p = new Period(start, end);
    p.end().setYear(78);
    ```
    - 해당 공격을 대비하기 위해서는 방어적 복사본을 반환하라.
    ```java
    public Date end() {
        return new Date(this.end.getTime());
    }
    ```
    - 생성자와 달리 접근자 메서드에는 방어적 복사에 clone()을 사용해도 된다.
  - 방어적 복사에는 성능 저하가 따르고, 또 같은 패키지에 속하는 등의 이유로 항상 쓸 수 있는 것도 아니다. 
  
### ITEM 51. 메서드 시그니처를 신중히 설계하라

- 메서드 이름을 신중히 짓자
  - 표준 명명 규칙을 따르자.
  
- 편의 메서드를 너무 낳이 만들지 말자.
  - 메서드가 너무 많은 클래스 또는 인터페이스는 익히고, 사용, 문서화, 테스트, 유지보수등이 어렵다
  
- 매개변수는 목록은 짧게 유지하자
  - 4개 이하가 좋다.
  - 같은 타입의 매개변수 여러 개가 연달아 나오는 경우 특히 해롭다.
    - 실수로 순서를 바꿔 입력해도 그대로 컴파일되고 실행되기 때문
  - 긴 매개변수 목록을 짧게 줄여주는 기술
    - 여러 메서드로 쪼갠다. 
      - 자칫 메서드의 수가 많아질 수 있지만 직교성(orthogonality)를 높여 오히려 메서드 수를 줄여주는 효과가 있을 수 있다.
      - 직교성은 수학에서 온 용어로 직각을 이루며 교차한다는 뜻이다.
      - SW에서 직교성이 높다는 것은 공통점 없이 기능들을 잘 분리되어 있다는 의미다.
    - 매개변수 여러개를 묶어주는 도우미 클래스를 만들자
    - 객체 생성에 사용한 빌더 패턴을 메서드 호출에 응용하자.
      - 먼저 모든 매개변수를 하나로 추상화한 객체를 정의하고, 클라이언트에서 세터 메서드를 호출해 필요한 값을 설정하게 하는 것

- 매개변수의 타입은 클래스보다는 인터페이스가 더 낫다.
  - 다형성을 존중하며 사용 범위가 더 높아진다.
  - 입력 데이터가 다른 형태로 존재한다면 명시한 특정 구현체의 객체로 옮겨 담는 비싼 복사 비용을 치룬다.
  
- boolean 보다는 원소 2개짜라 열거타입을 사용하자
  - 메서드 이름상 boolean을 받아야 의미가 더 명확할 때에는 예외다.
  
### ITEM 52. 다중 정의는 신중히 사용하라

- 재정의한 메서드는 동적으로 선택되고, 다중정의한 메서드는 정적으로 선택된다.
  - 재정의한 메서드는 하위클래스에서 정의한 재정의 메서드가 실행된다.
  - 다중정의된 메서드는 객체의 런타임 타입은 전혀 중요하지 않다.
    - 선택은 컴파일 타임에, 오직 매개변수의 컴파일 타임 타입에 의해 이뤄진다.
  - 매개변수 수가 같은 다중 정의는 만들지 말자.
    - 매개변수에 타입 결정이 클라이언트에 계산과 다를 수 있다.
    
- 다중정의하는 대신 메서드 이름을 다르게 지워주는 길도 있다.
  - write 메서드 대신 writeBoolean, writeInt, writeLong 등과 같은 형식 사용
  
- 한편 생성자는 이름을 다르게 지을 수 없으므로 무조건 다중정의가 된다.
  - 헷갈릴 만한 매개변수는 형변환하여 정확한 다중정의 메서드가 선택되도록 해야 한다.
  - 불가능하면, 기존 클래스를 수정해 새로운 인터페이스를 구현해야 할 때는 같은 객체를 입력받는 다중정의 메서드들이 모두 동일하게 동작하도록 만들어야 한다.
  
- 메서드를 다중정의할 때, 서로 다른 함수형 인터페이스라도 같은 위치의 인수로 받아서는 안된다.
  - 서로 다른 함수형 인터페이스라도 서로 근본적으로 다르지 않기 때문이다.
  
### ITEM 53. 가변 인수는 신중히 사용하라.

- 가변인수 메서드
  - 명시한 타입의 인수를 0개 이상 받을 수 있다.
  ```java
  static int sum(int... args) {
      int sum = 0;
      for(int arg : args) {
          sum += arg;
      }
      return sum;
  }
  ```
  - 1개 이상 인수를 받아야 할때
  ```java
  static int min(int firstArg, int... args) {
      int min = firstArg;
      for(int arg: args) {
          if(arg < min)
            min = arg;
      }
      return min;
  }
  ```

- 성능에 민감한 상황이면 가변인수가 걸림돌이 된다.
  - 가변인수 메서드는 호출될 때마다 배열을 새로 하나 할당하고 초기화한다.
  - 가변인수의 유연성이 필요할 때는 선택할 수 있는 멋진 패턴이 있다.
    - public void foo() {} public void foo(int a1) {}... public void foo(int a1, int a2, int a3, int... rest) {}
    - 95%가 3개 이하의 foo를 사용하고 나머지 5%가 이상을 사용할 때 메서드 호출 중 단 5%만 배열을 생성한다.
    - EnumSet의 정적 팩터리도 이 기법을 사용해 열거 타입 집합 생성 비용을 최소화한다.
      - EnumSet은 비트 필드를 대체하면서 성능까지 유지해야 하므로 아주 적절하게 사용한 것
      
### ITEM 54. null이 아닌 빈 컬렉션이나 배열을 반환하라

- null이 반환될 가능성이 있는 경우
  ```java
  List<Cheese> cheeses = shop.getChesses();
  if( cheeses != null && cheeses.contains(Cheese.STILTON) ) {
    //...
  }
  ```
  - 항상 예제와 같이 null임을 체크하는 방어적 코드를 넣어주어야 한다.
  
- 빈 컬렉션이나 배열을 반환하자
  ```java
  public List<Cheese> getCheeses() {
      // return new ArrayList<>(cheesesInStock);
      return cheesesInStock.isEmpty() ? Collections.emptyList() : new ArrayList<>(cheeseInStock);
  }  
  ```
  - 성능 분석 결과 해당 할당이 성능 저하의 주범이 확인되지 않는 한 이 정도의 성능 차이는 신경 쓸 수진이 못된다.
  - 빈 컬렉션과 배열은 굳이 새로 할당하지 않고도 반환할 수 있다.
  - 만약, 할당이 성능을 눈에 띄게 떨어뜨린다면,
    - '불변' 컬렉션을 반환하자.
    - Collections.emptyList, Collections.emptySet, Collections.emptyMap 등
  
  - 배열
  ```java
  private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];
  public Cheese[] getCheeses() {
      // return cheeseInStock.toArray(new Cheese[0]);
      return cheeseInStock.toArray(EMPTY_CHEESE_ARRAY);
  }
  ```
    - 만약 이 또한 성능이 걱정되면 길이 0 짜리 배열을 미리 선언해두고 해당 배열을 반환하자.
    - 길이가 0인 배열은 모두 불변이기 때문이다.
   
### ITEM 55. Optional 반환은 시중히 하라

- Optional
  - 8 버전 이전에는 메서드가 특정 조건에서 값을 반환할 수 없을 때에는
    - 예외를 던지거나 -> 예외는 진짜 예외적인 상황에서만 사용해야 하며, stack 전체를 갭처하므로 비용이 많이 든다.
    - null을 반환했다. -> null 값을 어딘가에 저장해두면 언젠가 NullPointerException이 발생할 수 있다.
  - 8 버전 이후 Optional<T> 객체를 반환할 수 잇게 되었다.

    ```java
    public static <E extends Comparable<E>> E max(Collection<E> c) {
        if(c.isEmpty()) throw new IllegalArgumentException("빈 컬렉션");
        E result = null;
        for(E e : c) {
            if(result == null || e.compareTo(result) > 0) result = Objects.requireNonNull(e);
        }
        return result;
    }
    ```
    ```java
    public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
        if(c.isEmpty()) return Optional.empty();
        E result = null;
        for(E e: c) {
            if(result == null || e.compareTo(result) > 0) result = objects.requireNonNull(e);
        }
        return Optional.of(result);
    }
    ```
    - Optinoal을 반환하는 메서드에서는 절대 null을 반환하지 말자.
    ```java
    //stream 활용
    public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
        return c.stream().max(Comparator.naturalOrder());
    }
    ```
    - null 반환, 에외를 던지는 대신 Optional 반환을 선택해야 하는 기준
      - Optional은 검사 예외와 취지가 비슷하다.
      - 즉, 반환값이 없을 수도 있음을 API 사용자에게 명확히 알려주는 것이다.
      ```java
      String lastWordInLexicon = max(words).orElse("단어 없음..."); //기본값을 정할 수 있다.
      String lastWordInLexicon = max(words).orElseThrow(TemperTantrumException::new); //예외를 던질 수 있다.
      String lastWordInLexicon = max(words).get(); //항상 값이 채워져 있다고 가정한다.
      ```
  - 9 버전 이후부터는 Optional에 stream(0 메서드가 추가되었다.
    - Optional을 Stream으로 반환해주는 어댑터다.
    - Stream의 flatMap 메서드와 조합하면 명료하게 바꿀 수 있다.
    ```java
    streamOfOptionals.flatMap(Optional::stream);
    ```
  
  - 컬렉션, 스트림, 배열, 옵셔널과 같은 컨테이너 타입은 옵셔널로 감싸면 안된다.
    - Optional<List<T>> 보다는 빈 List<T>를 반환하는 것이 좋다.
    
  - 박싱된 기본 타입을 담은 옵셔널을 반환하는 일은 없도록 하자.
    
### ITEM 56. 공개된 API 요소에는 항상 문서화 주석을 작성하라.

- 문서화 주석
  - 5 버전. @literal, @code,  8 버전. @implSpec, 9 버전. @index
  - API를 올바로 문서화하려면 공개된 모든 클래스, 인터페이스, 매서드, 필드 선언에 문서화 주석을 달아야 한다.
  - 메서드용 문서화 주석에는 해당 메서드와 클라이언트 사이의 규약을 명료하게 기술해야 한다.
    - 무엇을 하는지를 기술하자. 
    - @throws: 비검사 예외를 선언하여 암시적으로 기술하자. 
    - @param: 조건에 영향받는 매개변수에 기술할 수 있다.
    - @return: 반환값을 설명하는 명사구를 작성하자.
    - 문서화 주석은 HTML로 변환하므로 문서화 주석 안의 태그들을 사용할 수 있다.

- 자기사용 패턴(self-use pattern)
  - @implSpec 태그 사용
  - 해당 메서드와 하위 클래스 사이의 계약을 설명하여, 하위 클래스들이 그 메서드를 상속하거나 super 키워드를 이용해 호출할 때 그 메서드가 어떻게 동작하는지를 명확히 인지하고 사용하도록 해줘야 한다.
  ```java
  /**
  * Returns tru if this collection is empty.
  * @implSpec
  * This implementation returns {@code this.size() === 0 },
  * @return true if this collectoin is empty
  */
  public boolean isEmpty() {}
  ```

- 주석의 첫번째 문장
  - 해당 요소의 요약 설명으로 간주한다.
  - 요약 설명에는 반드시 대상의 기능을 고유하게 기술해야 한다.
  - 헷갈리지 않으려면 한 클래스(혹은 인터페이스) 안에서 요약 설명이 똑같은 멤버(혹은 생성자)가 둘 이상이면 안된다.
  - 요약설명에는 마침표(.)를 주의하자. 마침표가 나오는 부분까지 설명되기 때문이다.
    - {@literal Mrs.} 사용
  - 메서드와 생성자의 요약 설명은 해당 메서드와 생성자의 동작을 설명하는 주어가 없는 동사구여야 한다.
  - 클래스, 인터페이스, 필드의 요약설명은 대상을 설명하는 명사절이어야 한다.
  
- 인덱스 기능
  - 9 버전 이후
  - ex) This method compiles with the {@index IEEE 754} standard.

- 제네릭
  - 제네릭 타입이나 제네릭 메서드를 문서화 할 때에는 모든 타입 매개변수에 주석을 달아야 한다.
  ```java
  /*
  * An object that maps keys to values. A map cannot contain
  * duplicate keys; each key can map to at most one value.
  * (Remainder ommited)
  * @param <K> the type of keys maintained by this map
  * @param <V> the type of mapped values
  */
  public interface Map<K, V> {}
  ```

- 열거타입
  - 상수들도 주석을 달아야 한다.
  
- 애너테이션 타입
  - 멤버들에도 모두 주석을 달아야 한다.
    
- 패키지
  - 패키지를 설명하는 문서화 주석은 package-info.java 파일에 작성한다.
 
- 주의점
  - 자주 누락되는 설명
    - 스레드 안전성 : 클래스 혹은 정적 메서드가 스레드 안전하든 그렇지 안든 스레드 안전 수준을 반드시 API 설명에 포함해야 한다.
    - 직렬화 가능성 : 직렬화할 수 있는 클래스라면 직렬화 형태도 API 설명에 기술해야 한다.
  - 자바독은 메서드 주석을 '상속' 시킬 수 있다.
  - 자바독은 프로그래머가 자바독 문서를 올바르게 작성했는지 확인하는 기능을 제공한다.
