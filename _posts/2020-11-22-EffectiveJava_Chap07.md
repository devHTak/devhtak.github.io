---
layout: post
title: Effective Java Ch07. 람다와 스트림
summary: Effective Java 3판 공부
author: devhtak
date: '2020-11-22 11:41:00 +0900'
category: Effective Java
---

### ITEM 42. 익명 클래스보다는 람다를 사용하라

- 람다식
  - Java 8 이전에는 함수 타입을 표현할 때 추상 메서드 하나만을 담은 인터페이스 또는 추상 클래스를 사용했다.
  - Java 8 이후부터는 함수형 인터페이스라 부르는 람다식을 사용해 만들 수 있게 되었다.
  ```java
  // Java 8 이전
  Collections.sort(words, new Comparator<String>() {
      public int compare(String s1, String s2) { return Integer.compare(s1.length(), s2.length()); }
  });
  // Java 8 이후 - 람다 사용
  Collections.sort(words, (s1, s2) -> Integer.compare(s1.length(), s2.length()));
  ```
  - 람다를 사용하면 매개 변수에 대한 타입, return type 등에 언급이 없다. 컴파일러가 문맥을 살펴 타입을 추론한다.
  - 람다는 이름이 없고 문서화도 못한다. 따라서 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수가 많아지면 람다를 쓰지 말아야 한다.
  
- 익명 클래스를 사용해야 할 때
  - 람다는 함수형 인터페이스에서만 사용된다. 추상 클래스의 인스턴스를 만들 때 람다를 쓸수 없으니 익명 클래스를 사용해야 한다.
  - 여러 추상 메서드를 갖는 인터페이스의 인스턴스를 만들 때에도 익명 클래스를 사용해야 한다.
  - 람다는 자신을 참조할 수 없다. 람다에서의 this는 바깥 인스턴스를 가리키는 반면 익명 클래스의 this는 익명 클래스의 인스턴스 자신을 가리킨다.
  - 가상머신별 / 구현별로 다를 수 있기 때문에 람다를 직렬화 해서는 안된다.
  
### ITEM 43. 람다 보다는 메서드 참조를 사용하라
- 메서드 참조
  - 람다보다 더 간결하게 표현할 수 있다.
  - 예시) 임의의 키와 Integer값으 매핑을 관리하는 프로그램 일부로 키가 map 안에 없으면 1을 매핑하고, 없다면 기존 매핑 값을 증가시킨다.  
  ```java
  // 람다 사용
  map.merge(key, 1, (count, increase) -> count + increase);
  // 메서드 참조 사용
  map.merge(key, 1, Integer::sum);
  ```
  - 메서드 참조를 사용하면 가독성이 좋다. 다만 클래스, 메서드 이름이 길거나 할 때에는 메서드 참조를 사용할 때 가독성이 더 떨어질 수도 있다.
  
- 메서드 참조 유형 
  |메서드 참조 유형|예|같은 기능을 하는 람다|
  |:---|:---|:---|
  |정적|Integer::parseInt|str->Integer.parseInt(str)|
  |한정적(인스턴스)|Instant.now()::isAfter|Instant then = Instant.now(); t -> then.isAfter(t)|
  |비한정적(인스턴스)|String:toLowerCase|str -> str.toLowerCase()|
  |클래스생성자|TreeMap<K,V>::new|()-> new TreeMap<K,V>()|
  |배열생성자|int[]::new|len->new int[len]|
  
- 람다로는 불가능하나 메서드 참조로는 가능한 유일한 예는 바로 제네릭 함수타입 구현이다.

### ITEM 44. 표준 함수형 인터페이스를 사용하라
- 람자 지원 이후 변경 사항
  - 상위 클래스의 기본 메서드를 재정의해 원하는 동작을 구현하는 템플릿 메서드 패턴의 사용이 줄었다.
    - LinkedHashMap: removeEldestEntry를 재정의하여 캐시로 사용할 수 있다. 
    ```java
    protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
        return size() > 100;
    }
    ```
    - 원소의 개수가 100개 이상이 되면 가장 오래된 원소를 삭제하여 100개를 유지한다.
    ```java
    @FunctionalInterface
    interface EldestEntryRemovalFunction<K,V>{
        boolean remove(Map<K,V> map, Map.Entry<K,V> eldest);
    }
    ```
    - 함수형 인터페이스를 제공하여 필요한 용도에 맞게 빠르게 구현할 수 있다.

- 기본 함수형 인터페이스
  |인터페이스|함수시그니처|예|
  |:---|:---|:---|
  |UnaryOperator<T>|T apply(T t)|String::toLowerCase|
  |BinaryOperator<T>|T apply(T t1, T t2)|BigInteger::add|
  |Predicate<T>|boolean test(T t)|Collection::isEmpty|
  |Functoin<T,R>|R apply(T t)|Arrays::asList|
  |Supplier<T>|T get()|Instant::now|
  |Comsumer<T>|void accept(T t)|System.out::println|
  
  - 주의점
    - 기본 함수형 인터페이스에 박싱된 기본 타입을 넣어 사용하지는 말자. 계산량이 많을 때는 성능이 처참히 느려진다.
  - Comparator<T> 인터페이스
    - 구조적으로 ToIntBiFunction<T,U>와 동일하다.
    - 특성 1. 자주 쓰이며, 이름 자체가 용도를 명확히 설명해준다.
    - 특성 2. 반드시 따라야 하는 규약이 있다.
    - 특성 3. 유용한 디폴트 메서드를 제공할 수 있다.

- @FunctionalInterface
  - 직접 만든 함수형 인터페이스에는 항상 사용하자.
  - 이유 1. 해당 클래스의 코드나 설명 문서를 읽을 이에게 그 인터페이스가 람다용으로 설계된 것임을 알려준다.
  - 이유 2. 해당 인터페이스가 추상 메서드를 오직 하나만 가지고 있어야 컴파일되게 해준다.
  - 이유 3. 결과 유지보수 과정에서 누군가 실수로 메서드를 추가하지 못하게 막아준다.

- 마지막으로 서로 다른 함수형 인터페이스를 같은 위치의 인수로 받는 메서드들을 다중정의해서는 안된다.

### ITEM 45. 스트림은 주의해서 사용하라.
- Stream API의 추상 개념 중 핵심
  - 핵심 1. Stream은 데이터 원소의 유한 혹은 무한 시퀀스를 뜻한다.
  - 핵심 2. Stream Pipeline은 이 원소들로 수행하는 연산 단계를 표현하는 개념이다.
  - Stream의 원소들은 대표적으로 컬렉션, 배열, 파일, 정규표현식 패턴 매처, 난수 생성기, 다른 스트림이 있다.
  
- Stream Pipeline
  - Source Stream으로 시작해 종단 연산(terminal operation)으로 끝나며, 그 사이에 하나 이상의 중간 연산(intermediate operation)이 있다.
  - 지연 평가(lazy evaluation)
    - 종단 연산이 호출될 때 평가가 이뤄지며 종단 연산에 쓰이지 않는 데이터 원소는 계산ㅇ ㅔ쓰이지 않는다.

- Stream 주의점
  - Stream을 과용하면 프로그램이 읽거나 유지보수하기 어려워진다.
  - char 값들을 처리할 때에는 스트림을 삼가는게 좋다
  ```java
  "Hello World".chars().forEach(System.out::print); // 720....이 출력된다.
  "Hello World".chars().forEach(ch->System.out.println((char)ch));
  ```
    - chars()는 int값이기 때문이다.
  - 기존 코드는 스트림을 사용하도록 리펙터링하되, 새 코드가 더 나아 보일 때만 반영하자

- Stream 사용
  - 원소들의 시퀀스를 일관되게 변환한다.
  - 원소들의 시퀀스를 필터링한다.
  - 원소들의 시퀀스를 하나의 연산을 사용해 결합한다.(더하기, 연결하기, 최솟값 구하기 등)
  - 원소들의 시퀀스를 컬렉션에 모든다(아마도 공통된 속성을 기준으로 묶어가며)
  - 원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾는다.

- Stream 사용이 어려운 경우
  - 한 데이터가 파이프라인의 여러 단계를 통과할 때 이 데이터의 각 단계에서의 값들에 동시에 접근하기 어려운 경우
  - Stream Pipeline은 일단 한 값을 다른 값에 매핑하고 나면 원래의 값은 잃는 구조이기 때문이다.
  
  ```java
  // 데카르트 곱 계산을 반복 방식으로 구현
  private static List<Card> newDeck() {
      List<Card> result = new ArrayList<>();
      for(Suit suit: Suit.values()) {
          for(Rank rank: Rank.values()) 
              result.add(new Card(suit,rank));
      }
      return result;
  }
  // 데카르트 곱 계산을 스트림 방식으로 구현
  private static List<Card> newDeck() {
      return Stream.of(Suit.values())
          .flatMap(suit -> Stream.of(Rank.values()).map(rank->new Card(suit, rank)))
          .collect(toList());
  }
  ```

### ITEM 46. 스트림에서는 부작용 없는 함수를 사용하라.
```java
Map<String, Long> freq = new HashMap<>();
try( Stream<String> words = new Scanner(file).tokens() ) {
    words.forEach(word -> {
        freq.merge(word.toLowerCase(), 1L, Long::sum);        
    });
}
```
- Stream API의 장점을 살리지 못하며 반복문을 사용하는 것보다 복잡하게 느껴진다.
```java
Map<String, Long> freq = new HashMap<>();
try( Stream<String> words = new Scanner(file).tokens() ) {
    freq = words.collect(groupingBy(String::toLowerCase, counting()));
}
```
- 스트림을 제대로 활용해 빈도표를 초기화했다.

- Collector
  - 축소 전략을 캡슐화한 블랙박스 객체 (축소: 스트림의 원소들을 객체 하나에 취합한다는 의미)
  - toList(), toSet(), toCollection(collectionFactory)
  - 빈도표에서 가장 흔한 단어 10개를 뽑아내는 파이프라인
  ```java
  List<String> topTen = freq.keySet().stream().sorted(comparing(freq::get).reversed()).limit(10).collect(toList());
  ```
  - toMap(keyMapper, valueMapper)
    - 스트림 원소를 키에 매핑하는 함수와 값에 매핑하는 함수를 인수로 받는다.
    ```java
    // Enum 에서 fromString 구현
    private static final Map<String, Operation> stringToEnum = Stream.of(value().collect(toMap(Object::String, e->e));
    ```
    - 3번째 인수는 어떤 키와 그 키에 연관된 원소들 중 하나를 골라 연관 짓는 맵을 만들 때 유용하다.
    - 4번째 인수는 Map Factory를 받는다.
  - groupingBy()
    - 사용 1. 메서드의 입력으로 분류 함수(classifier)를 받고 출력으로는 원소들을 카테고리별로 모아 놓은 맵을 담은 수집기를 반환한다.
    - 분류 함수는 입력받은 원소가 속하는 카테고리를 반환한다.
    ```java
    // 알파벳화한 단어를 알파벳화 결과가 같은 단어들의 리스트로 매핑하는 맵을 생성
    words.collect(groupingBy(word -> alphabetize(word)));
    ```
    - 사용 2. toSet()을 넘겨 groupingBy는 원소들의 리스트가 아닌 집합을 값으로 갖는 맵을 만들 수 있다. 또는 toCollection(collectionFactory)를 건네어 컬렉션을 값으로 갖는 맵을 생성할 수 있다.
    - 사용 3. 다운스트림 수집기에 더해 맵 팩터리도 지정할 수 있게 해준다.
  - minBy(), maxBy()
    - 인수로 받은 비교자를 이용해 스트림에서 값이 가장 작거나 큰 원소를 찾아 반환한다.
  - joining()
    - 원소들을 연결(concatenate)하는 수집기를 반환한다.
    
### ITEM 47. 반환 타입으로는 스트림보다 컬렉션이 낫다
- 자바 8 이전 원소 시퀀스
  - Collection, Set, List와 같은 Collection Interface
  - Iterable (일부 Collection 메서드를 구현할 수 없을 때)
  - 배열
  
- Stream 특징
  - Stream은 반복(Iteration)을 지원하지 않는다.
    - Iterable 인터페이스가 정의한 추상 메서드를 전부 포함하고, 정의한 방식대로 동작하지만 for-each를 사용할 수 없다. Iterable을 확장하지 않았기 때문이다.
    - Stream 을 Iterator로 중개해주는 어댑터
    ```java
    public static <E> Iterable<E> iterableOf(Stream<E> stream) {
        return stream::iterator;
    }
    ```
    ```java
    for(ProcessHandle p: iterableOf(ProcessHandle.allProcesses())) { ...}
    ```
    - 하지만, Iterable을 반환하면 스트림 파이프라인에서 처리하기 어려워 진다.
  - 원소 시퀀스를 반환하는 공개 API 반환 타입에는 Collection이나 그 하위 타입을 쓰는게 일반적으로 최선이다.
    - Collection 인터페이스는 Iterable의 하위 타입이고, stream 메서드도 제공하기 때문에 반복과 스트림을 동시에 지원한다.

- Collection 내의 시퀀스가 크면 전용 Collection을 구현하라
  - 반환하는 시퀀스의 크기가 메모리에 올려도 안전할 만큼 작다면 ArrayList나 HashSet같은 표준 컬렉션 구현체를 반환하는게 최선일 수 있다. 하지만 단지 컬렉션을 반환한다는 이유로 덩치 큰 시퀀스를 메모리에 올려서는 안된다.
  - Stream이 나을 때도 있다.
 
### ITEM 48. 스트림 병렬화는 주의해서 적용하라
- Stream과 병렬화
  - 중간 연산으로 limit 등을 사용하면 병렬화로는 성능 개선이 불가능하다.
  - Stream의 소스가 ArrayList, HashMap, HashSet, ConcurrentHashMap의 인스턴스거나 배열, int, long 범위일 때 병렬화의 효과가 가장 좋다.
  - 종단 연산의 동작 방식 역시 병렬 수행 효율에 영향을 준다.
    - 병렬화의 적합한 것은 축소(reduction) 종단 연산이다.
    - 축소는 파이프라인에서 만들어진 모든 원소를 하나로 합치는 작업으로 reduce 메서드 중 하나, min, max, count, sum 또는 anyMatch, allMatch, noneMatch처럼 조건에 맞으면 바로 반환하는 메서드가 병렬화에 적합하다.
  - Stream을 잘못 병렬화하면 성능이 나빠질 뿐만 아니라 결과 자체가 잘못되거나 에상 못한 동작이 발생할 수 있다.

- Stream 병렬화는 오직 성능 최적화 수단이다.
  - 소수계산 스트림 파이프라인(병렬화를 통한 성능 최적화한 예시)
  ```java
  static long pi(long n) {
      return LongStream.rangeClosed(2, n)
          .parallel()
          .mapToObj(BigInteger::valueOf)
          .filter(i -> i.isProbablePrime(50))
          .count();
  }
  ```
  

  
  
  
