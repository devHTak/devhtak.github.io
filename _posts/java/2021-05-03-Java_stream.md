---
layout: post
title: Stream
summary: Stream
author: devhtak
date: '2021-05-03 21:41:00 +0900'
category: Java Study
---

#### Stream이란

- Java 8 이전에는 배열, 컬렉션을 다루기 위하여 반복문(for, for-each)를 사용하였다.
- Java 8 이후에는 stream (데이터의 흐름)을 통하여 배열 또는 컬렉션 인스턴스에 함수 여러 개를 조합하여 원하는 결과를 얻을 수 있다.
  - 람다를 사용하여 코드의 양을 줄이고 간결하게 표현할 수 있다.
  - 병렬처리가 가능하다. 하나의 작업을 둘 이상의 작업으로 나누어 동시에 진행하는 것을 병렬처리라 한다.

- 구성
  - 생성 -> 가공 -> 결과 순으로 처리할 수 있다.

#### Stream 종류

- 기본 타입형 스트림
  - Stream은 제네릭을 사용하지만, 기본 타입 스트림을 생성하여 제공하고 있다.
  - int, long, double이 있다.
  - 사용예시
    - range, rangeClosed의 차이는 범위
      ```java
      IntStream intStream = IntStream.range(1, 5); // 1, 2, 3, 4
      LongStream longStream = LongStream.rangeClosed(1, 5); // 1, 2, 3, 4, 5
      ```
    - 박싱 하기 - 제네릭을 사용하지 않기 떄문에 오토박싱이 이뤄지지 않는다. 필요한 경우 boxed() 메서드 사용
      ```java
      Stream<Integer> stream = IntStream.range(1, 5).boxed();      
      ```

- 문자열 스트림
  - 문자열을 character로 하여 ASCII 코드 값으로 저장할 수 있다.
    ```java
    IntStream charStream = "STREAM".chars();
    ```

- 파일 스트림
  - 자바 NIO 의 Files 클래스의 lines 메소드는 해당 파일의 각 라인을 스트링 타입의 스트림으로 만들어준다.
    ```java
    Stream<String> lineStream = Files.lines(Paths.get("file.txt"), StandardCharsets.UTF_8);
    ```

#### 병렬 스트림(Parallel Stream)

- parallel stream을 활용하면 병렬 스트림을 쉽게 생성할 수 있다.
- 내부적으로 Java 7 부터 도입된 Fork / Join Framework를 사용할 수 있다.

#### 생성

- 배열 스트림
  - 이미 생성된 배열을 stream으로 변경하여 사용
    ```java
    String[] arr = new String[]{ "a", "b", "c"};
    Stream<String> stream = Arrays.stream(arr);
    Stream<String> streamOfArrayPart = Arrays.stream(arr, 1, 3); // {"b", "c"}
    ```
    
- 컬렉션 스트림
  - 컬렉션 타입(Collection, List, Set) 의 경우 인터페이스에 추가된 디폴트 메소드 stream을 활용
    ```java
    List<String> list = Arrays.asList("a", "b", "c");
    Stream<String> stream = list.stream();
    Stream<String> parallelStream = list.parallelStream(); // 병렬처리 스트림
    ```
    
- Stream.builder()
  - 빌더패턴을 활용하여 stream을 생성할 수 있다.
    ```java
    Stream<String> builderStream = Stream.<String>builder()
				.add("a")
				.add("b")
				.add("c").build();
    ```
  
- Stream.generate()
  - generate 메소드를 이용하여 Supplier<T> 에 해당하는 람다로 값을 넣을 수 있다.
  - Supplier<T> 는 인자는 없고 T를 리턴하는 @FunctionalInterface
  - limit을 통해 특정 사이즈를 제한해야 한다.
    ```java
    Stream<String> generateStream = Stream.generate(()-> "a").limit(5);
    ```
  
- Stream.iterator()
  - 초기 값과 해당 값을 다루는 람다를 이용하여 스트림에 들어갈 요소를 만든다.
  - limit을 통해 특정 사이즈를 제한해야 한다.
    ```java
    Stream<Integer> iteratedStream = Stream.iterate(10, (data) -> data + 5).limit(3);
    ```
    
- 스트림 연결하기
  ```java
  Stream<Integer> stream1 = Stream.<Integer>builder().add(1).add(2).build();
  Stream<Integer> stream2 = Stream.iterate(3, (data) -> data + 1).limit(3);

  Stream<Integer> concatedStream = Stream.concat(stream1, stream2);
  ```
  
#### 가공

|메소드명|특징|
|---|---|
|filter|함수형 인터페이스인 Predicate를 인자로 받으며 조건에 맞는 데이터만 stream한다.|
|map|함수형 인터페이스인 Function을 인자로 받으며 데이터를 가공할 수 있다.|
|flatMap|함수형 인터페이스인 Function을 인자로 받으며 데이터를 가공할 수 있다. map과의 차이는 원소를 최소한으로 나누어 가공한다는 점이다.|
|sorted|stream을 정렬한다. Comparator를 인자로 받아 정렬 조건을 줄 수 있다.|
|peek|함수형 인터페이스인 Consumer를 인자로 받으며 stream 중간에 확인할 때 사용된다.|

- 전체 요소 중에서 중간단계 API를 이용하여 원하는 것만 뽑아낼 수 있다.
- stream을 리턴하기 떄문에 method chaining이 가능하다

- Filtering
  ```java
  Stream<T> filter(Predicate<? super T> predicate);
  ```
  - 스트림 내 요소들을 하나씩 평가하여 걸러내는 작업
  - 인자로 받는 Predicate는 boolean을 리턴하는 함수형 인터페이스로 평가식이 들어가게 된다.
  - 예제
    ```java
    List<String> before = Arrays.asList("Mary", "Kevin", "Test");
		long aNameCount = before.stream().filter((data) -> data.contains("a")).count(); // 1
    ```

- Map
  ```java
  <R> Stream<R> map(Function<? super T, ? extends R> mapper);
  ```
  - 스트림 내 요소들을 하나씩 특정 값으로 매핑시킨 후 매핑시킨 값을 다시 스트림으로 변환하는 중간 연산을 담당한다
  - 이 때 변환하기 위한 람다를 인자로 받는다.
  - 예제
    ```java
    List<String> before = Arrays.asList("Mary", "Kevin", "Test");
    List<String> after = before.stream().map((data) -> data.toUpperCase()).collect(Collectors.toList());
    ```

- flatMap
  ```java
  <R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper);
  ```
  - Array나 Object로 감싸져 있는 모든 원소를 단일 원소 스트림으로 반환한다.
  - 예제
    ```java
    String[][] namesArray = new String[][]{
	        {"kim", "taeng"}, {"mad", "play"},
	        {"kim", "mad"}, {"taeng", "play"}};
	        
	  Arrays.stream(namesArray)
	    	.flatMap(innerArray -> Arrays.stream(innerArray))
	    	.filter(data -> data.length() > 3)
	    	.collect(Collectors.toSet()).forEach(System.out::println); // play, taeng
    ``
    
- map과 flatMap 차이
  - map은 입력한 원소를 그대로 스트림으로 반환
  - flatMap은 입력한 원소를 가장 작은 단위의 단일 스트림으로 반환
  - 예제
    ```java
    String[][] namesArray = new String[][]{
	        {"kim", "taeng"}, {"mad", "play"},
	        {"kim", "mad"}, {"taeng", "play"}};
          
    Arrays.stream(namesArray)
	    	.flatMap(innerArray -> Arrays.stream(innerArray))
	    	.filter(data -> data.length() > 3)
	    	.collect(Collectors.toSet()).forEach(System.out::println);; // play, taeng
	    
	  Arrays.stream(namesArray)
	    	.map(innerArray -> Arrays.stream(innerArray).filter(data -> data.length() > 3).collect(Collectors.toSet()))	
	    	.collect(HashSet::new , Set::addAll, Set::addAll).forEach(System.out::println); // play, taeng
     ```
     - 2차원 배열을 stream으로 생성, flatMap을 사용하면 String 단위로 변경이 가능하여 바로 Set을 만들 수 있다.
     - map에 경우 stream으로 리턴되기 때문에 안에서 filter를 걸고 set으로 만들었다.
      - collect() 메서드
        ```java
        <R> R collect(Supplier<R> supplier, BiConsumer<R, ? super T> accumulator, BiConsumer<R, R> combiner);
        ```
        - supplier는 새로운 결과 컨테이너를 만든다.
        - accumulator는 결과에 추가 요소를 통합하기 위한 역할을 한다.
        - combiner는 계산 결과를 결합하는 역할을 담당한다.

- Sorting
  
  ```java
  List<Integer> list = Arrays.asList(30, 10, 20, 50, 40, 60);
		
  list.stream().filter(data -> data > 20)
      .sorted()
      .collect(Collectors.toList()).forEach(System.out::print); // 30 40 50 60

  list.stream().filter(data -> data > 20)
      .sorted(Comparator.reverseOrder())
      .collect(Collectors.toList()).forEach(System.out::print); // 60 50 40 30
  ```
  
  - 정렬의 방법으로 Comparator를 이용한다.
    - 인자 없이 그냥 호출할 경우 그냥 오름차순으로 정렬한다.
    
- peek
  - 스트림 내 요소들 각각을 대상으로 특정 연산을 수행하는 메소드
  - Consumer를 인자로 받기 때문에 return 객체가 없다.

#### 결과 

- 가공한 stream을 가지고 결과값을 만들어내는 단계
- 스트림을 끝내는 최종 작업

- Calculating
  - 최소, 최대, 합, 평균 등 기본형 타입으로 결과를 만들어 낼 수 있다.
  - 비어있는 stream에 경우 count와 sum은 0을 리턴, 최소 최대 평균의 경우 표현할 수 없기 때문에 Optional을 이용해 리턴

- Reduction
  ```java
  // 1개 (accumulator)
  Optional<T> reduce(BinaryOperator<T> accumulator);

  // 2개 (identity)
  T reduce(T identity, BinaryOperator<T> accumulator);

  // 3개 (combiner)
  <U> U reduce(U identity, BiFunction<U, ? super T, U> accumulator, BinaryOperator<U> combiner);
  ```
  - accumulator: 각 요소를 처리하는 계산 로직, 각 요소가 올 때마다 중간 결과를 생성하는 로직
  - identity: 계산을 위한 초기값으로 스트림이 비워서 계산할 내용이 없더라도 이 값은 리턴
  - combiner: 병렬(parallel) 스트림을 나눠 계산한 결과를 하나로 합치는 동작하는 로직 / 병렬스트림이 아닌경우 실행되지 않는다
  - 예제
    ```java
    int reducedParams = Stream.of(1, 2, 3)
				.reduce(10, Integer::sum, (a, b) -> {
					System.out.println("combinar was called");
					return a + b;
				});
		
		System.out.println(reducedParams); // 16
    ```

- Collecting
  - Collectors가 제공하는 객체를 사용
  
  |Collectors 메소드|특징|
  |---|---|
  |toList()|List로 반환|
  |toSet()|Set으로 반환|
  |joining()|스트림에서 작업한 결과를 하나의 스트링으로 이어 붙일 수 있다. delimeter, prefix, suffix를 인자로 받는다|
  |averageInt()|숫자 값(Integer value)의 평균(arithmetic mean)을 구할 수 있다.
  |summingIng()|숫자 값의 합을 나타낸다. mapToInt 등을 활용하면 더 쉽게 사용할 수 있다|
  |summarizingInt()|만약 합계와 평균 모두 필요하다면 스트림을 두 번 생성해야 할까요? 이런 정보를 한번에 얻을 수 있는 방법이다.|
  |groupingBy()|특정 조건으로 요소들을 그룹지을 수 있습니다. 여기서 받는 인자는 함수형 인터페이스 Function 입니다.|
  |partitioningBy()|partitioningBy 은 함수형 인터페이스 Predicate 를 받습니다. Predicate 는 인자를 받아서 boolean 값을 리턴합니다. true, false로 그루핑된다.|
  |collectingAndThen()|특정 타입으로 결과를 collect 한 이후에 추가 작업이 필요한 경우에 사용할 수 있다. |
  |of()|직접 collector 를 만들어 사용. accumulator 와 combiner 는 reduce 에서 살펴본 내용과 동일합니다.|

- Matching
  - 조건식 람다 Predicate를 받아 해당 조건을 만족하는 요소가 있는지 체크한 결과 리턴
  - 세가지 메소드
    - anyMatch: 하나라도 조건을 만족하는 요소가 있는지
    - allMatch: 모두 조건을 만족하는지
    - noneMatch: 모두 조건을 만족하지 않는지

- Iterating
  - forEach는 요소를 돌면서 실행하는 최종 작업
  - peek와는 중간 작업과 최종 작업의 차이가 있다.

#### 출처

- https://futurecreator.github.io/2018/08/26/java-8-streams/
- https://madplay.github.io/post/difference-between-map-and-flatmap-methods-in-java
- https://kchanguk.tistory.com/56
