---
layout: post
title: Effective Java Ch05. 제네릭
summary: Effective Java 3판 공부
author: devhtak
date: '2020-11-12 19:41:00 +0900'
category: java
---

### 00. 들어가기

** 제네릭은 자바 5부터 사용할 수 있다.
** 제네릭을 사용하기 전에는 컬렉션에서 객체를 꺼낼 때마다 형변환을 해야 했고, 엉뚱한 타입의 객체를 넣어두면 런타임에 형변환 오류가 난다.
** 제네릭을 사용함으로써 컴파일러에게 컬렉션이 담을 수 있는 타입을 알려주며 알아서 형변환 코드를 추가할 수 있게 된다.

### Item 26. Row type은 사용하지 말라

- 단점
  - Row Type을 쓰면 형변환 등에 문제가 발생할 수 있다.
  - 하지만 지원하는 이유는 호환성 때문이다.
  
- List vs List<Object>
  - List<Object>같은 매개변수화 타입을 사용할 때와 달리 List 같은 로 타입을 사용하면 타입 안전성을 잃게 된다.
  ```java
  public static void main(String[] args) {
  	List<String> strings = new ArrayList<>();
  	unsafeAdd(strings, Integer.valueOf(42));
  	String s = strings.get(0); // 컴파일러가 자동으로 형변환 코드를 넣어준다. 하지만 Integer가 들어있기 때문에 경고 발생
  }
  public static void unsafeAdd(List list, Object o) {
  	list.add(o);
  }
  ```
  - strings.get(0) 에서 ClassCastException 발생
  
- Row Type을 사용 예외
  - class literal에는 row type을 써야 한다. ex) List.class, String[].class, int.class는 허용하고 List<String>.class, List<?>.class는 허용하지 않는다.
  - instanceof 연산자
  ```java
  if( o instanceof Set){ // Row Type 사용 가능
  	Set<?> s = (Set<?>)o; 
  }
  ```

### Item 27. 비검사 경고를 제거하라

** 제네릭을 사용하면 비검사 형변환 경고, 비검사 메서드 호출 경고, 비검사 매개화변수화 가변인수 타입 경고, 비검사 변환 경고 등
** 제네릭 사용 도 중 발생하는 비검사 경고는 중요하니 무시하지 말자. ClassCastException을 일으킬 수 있는 잠재적 가능성을 뜻한다.
** Compiler가 경고에 대해 친절히 설명해줄 수 있다. 다만, 어려운 경고도 존재한다.
** 하지만, 할 수 있는 한 모든 비검사 경고를 제거하자. 타입 안전성이 보장되기 때문이다.

- 경고 제거 / 숨기는 방법
  - 타입이 안전하다고 확신한다면, @SuppressWarnings("unchecked") 애너테이션을 달아 경고를 숨기자.
  - 해당 애너테이션은 개별 지역 변수부터 클래스 전체까지 어떤 선언에도 달 수 있지만 가능한 좁은 범위에 적용하자.
  ex) 변수 선언, 아주 짧은 메서드, 생성자
  - 경고를 무시해도 되는 이유를 항상 주석으로 남기자
  ```java
  public <T> T[] toArray(T[] a) {
	  if( a.length < size ) {
		// 생성한 배열과 매개변수로 받은 배열의 타입이 모두 T[]로 같으므로 올바른 형변환이다.
		@SuppressWarnings("unchecked")T[] result = (T[])Arrays.copyOf(elements, size, a.getClass());
		return result;
	  }
	  System.arraycopy(elements, 0, a, 0, size);
	  if(a.length > size) 
		a[size] = null;
	  return a;
  }  
  ```

### Item 28. 배열보다는 리스트를 사용하라
  
- 배열과 제네릭 타입에 중요 차이
  - 1. 배열은 공변이고, 제네릭은 불공변이다.
    - Sub 클래스가 Super 클래스의 하위 클래스인 경우 배열 Sub[]는 Super[] 배열의 하위타입이다. 즉, 함께 변한다.
    - Type1, Type2가 있을 때 List<Type1>, List<Type2>는 하위타입도 상위 타입도 아니다.
  ```java
  Object[] objectArray = new Long[1];
  objectArray[0] = "타입이 달라 넣을 수 없다."; // ArrayStoreException을 일으킨다.

  List<Object> olist = new ArrayList<Long>(); // 호환되지 않는 타입이다.
  olist.add("타입이 달라 넣을 수 없다.">;
  ```
    - 배열은 런타임 과정에서 원인이 파악되고, 제네릭 타입은 컴파일할 때 알 수 있다.

  - 2. 배열은 실체화 된다. 배열은 런타임에도 잣니이 담기로 한 원소의 타입을 인지하고 확인한다. 
  
- 제네릭과 배열의 호환 문제
  - 제네릭 배열은 타입이 안전하지 않기 때문에 생성하지 못하게 한다.
  - 배열을 제네릭으로 만들 수 없다.
  - 만약 둘을 함께 사용하다 컴파일 에러가 발생하면 Class Cast에 좋은 리스트로 대체하자.
  
### Item 29. 이왕이면 제네릭 타입으로 만들어라.

** 클라이언트에서 직접 형변환하는 타입보다 제네릭 타입이 더 안전하고 쓰기 편하다.
- 제네릭 타입 객체 만들기
  - 먼저 object로 단순한 stack 생성
  ```java
  public class Stack {

  	private Object[] elements;
    	private int size = 0;
    	private static final int DEFAULT_INITIAL_CAPACITY = 16;

    	public Stack() {
      		elements = new Object[DEFAULT_INITIAL_CAPACITY];
    	}

    	public void push(Object e) {
      		ensureCapacity();
      		this.elements[size++] = e;
    	}

    	public Object pop() {
      		if(this.size == 0)
        		throw new EmptyStackException();
      		Object result = elements[--size];
      		elements[size] = null; // 다쓴 참조 해제
      		return result;
    	}

    	public boolean isEmpty() {
      		return this.size == 0;
    	}

    	public void ensureCapacity() {
      		if(this.size == elements.length) {
        		elements = Arrays.copyOf(elements, this.size * 2 + 1);
		}
      	}
  }
  ```
  ** 해당 클래스는 제네릭 타입으로 작성해야 한다.
  
  - 첫단계. Object를 E로 변환
  ```java
  public class Stack<E> {
	
  	private E[] elements;
    	private int size = 0;
    	private static final int DEFAULT_INITIAL_CAPACITY = 16;

    	public Stack() {
      		elements = new E[DEFAULT_INITIAL_CAPACITY];
    	}

    	public void push(E e) {
      		ensureCapacity();
      		this.elements[size++] = e;
    	}

    	public E pop() {
      		if(this.size == 0)
        		throw new EmptyStackException();
      		E result = elements[--size];
      		elements[size] = null; // 다쓴 참조 해제
      		return result;
    	}	

    	public boolean isEmpty() {
      		return this.size == 0;
    	}

    	public void ensureCapacity() {
      		if(this.size == elements.length) {
        		elements = Arrays.copyOf(elements, this.size * 2 + 1);
      		}
    	}
  }
  ```
  // elements = new E[DEFAULT_INITIAL_CAPACITY]에서 오류가 발생한다. (E와 같은 실체화 불가 타입으로 배열을 만들 수 없다.)
  // 해결 방법 첫번째. Object 배열을 생성한 뒤 제네릭 배열로 형변환
  // Object 배열을 만든 후 형변환하는 방법은 컴파일러가 형변환 경고를 발생한다. @SuppressWarnings("unchecked")를 하자
  // 해당 방법은 가독성에는 좋]배열의 런타임 타입이 컴파일 타임 타입과 다르기 때문에 힙 오염을 일으킨다.
  ```java
  // 배열 elements는 push(E)로 넘어온 E 인스턴스만 담기 때문에 타입 완전성을 보장한다.
  // 하지만 이 배열의 런타임 타입은 E[]가 아닌 Object[]이다.
  @SuppressWarnings("unchecked")
  public Stack() {
  	elements = (E[])new Object[DEFAULT_INITIAL_CAPACITY];
  }
  ```
  
  // 해결 방법 두번째. elements의 타입을 E[]가 아닌 Object[]로 변환
  // E result = elements[--size]; 에서 컴파일 오류가 발생하는 데 이를 E타입으로 형변환하자
  // 형변환 경고가 발생하면 판단 후 @SuppressWarnings("unchecked")를 하자
  ```java
  public E pop() {
	if(this.size == 0)
		throw new EmptyStackException();
	// push에서 E 타입만 허용하므로 이 형변환은 안전하다.
	@SuppressWarnings("unchecked") E result = (E)elements[--size];
	elements[size] = null; // 다쓴 참조 해제
	return result;
  }
  ```
  - 해당 제네릭 타입을 사용하면 문제는 기본 타입을 사용하지 못한다. 박싱된 타입을 사용하여 우회하는 방법이 있다.

### Item30. 이왕이면 제네릭 메서드로 만들어라

** 클래스와 마찬가지로, 메서드도 제네릭으로 만들 수 있다.

- 제네릭 메서드 만드는 방법
```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
	Set<E> result = new HashSet<E>(s1);
	result.addAll(s2);
	return result;
}
public static void main(String[] args) {
	Set<String> guys = Set.of("톰", "딕", "해리");
	Set<String> stooges = Set.of("래리", "모에", "컬리");
	Set<String> aflCio = union(guys, stooges);
	System.out.println(aflCio);
}
```
  - 타입 매개변수들을 선언하는 타입 매개변수 목록은 메서드의 제한자와 반환 타입 사이에 온다.
  - 해당 코드에서 타입 매개변수 목록은 <E>이고 반환 타입은 Set<E>이다.
  - 현재 union 메서드에 입력2개, 반환1개에 타입이 모두 같아야 한다. 이는 와일드카드 타입을 사용하면 더 유연하게 개선할 수 있다.

- 제네릭 싱글턴 팩터리
  - 때때로 불변 객체를 여러 타입으로 활용할 수 있게 만들어야 할 때가 있다. 이 때 요청한 타입 매개변수에 맞게 매번 그 객체의 타입을 바꿔주는 정적 팩터리를 만들어야 한다. 이를 제네릭 싱글턴 팩터리라고 한다.
  - 만약 제네릭을 사용하지 않으면 타입마다 형변환하는 정적 팩터리를 만들어야 한다.
  - 예로는 Collections.reverseOrder, Collections.emptySet

- 재귀적 타입 한정
  - 자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정할 수 있다.
  - 주로 타입의 자역적 순서를정하는 Copmarable 인터페이스와 함께 사용된다.
  ```java
  public static <E extends Comparable<E>> E max(Collection<E> c) {
  	if( c.isEmpty() ) throw new IllegalArgumentException("컬렉션이 비어있습니다.");
	E result = null;
	for(E e : c) {
		if(result == null || e.compareTo(result) > 0) {
			result = Objects.requireNonNull(e);
		}
	}
	return e;
  }
  ```
  - 재귀적 타입 한정을 이용해 상호 비교할 수 있음을 표현했다.

### Item 31. 한정적 와일드카드를 사용해 API 유연성을 높여라

** 매개변수화 타입은 불공변(invarient)이다. 즉, 서로 다른 타입 Type1, Typ2가 있을 때 List<Type1>은 List<Type2>의 하위 타입도 상위 타입도 아니다.
** 하지만 때로는 유연한 방식이 필요할 때도 필요하다.
	
- 한정적 와일드카드 타입
  - 유연성을 극대화하기 위해 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용하라.
  ```java
  public void pushAll(Iterable<? extends E> src) {
  	for(E e : src) { push(e); } 
  }
  ```
  - pushAll의 입력 매개변수 타입은 'E의 Iterable'이 아니라 'E의 하위 타입의 Iterable'이어야 한다.
  ```java
  public void popAll(Collection<? super E> dst) {
  	while(!isEmpty()) { dst.add(pop()); }
  }
  ```
  - PECS: producer-extends, consumer-super
  - 매개변수화 타입 T가 생산자라면 <? extends T>를 사용하고, 소비자라면, <? super T>를 사용하라.
  ```java
  public static <E extends Comparable<? super e>> max(Collection<? extends E> c);
  ```

- 타입 매개변수와 와일드카드
  ```java
  public static <E> void swap1(List<E> list, int i, int j); // 비한정적 타입 매개변수 사용
  public static void swap2(List<?> list, int i, int j); // 비한정적 와일드카드 사용
  ```
  - public API라면 간단한 두번째 swap2가 낫다. 메서드 선언에 타입 매개변수가 한번만 나오면 와일드 카드로 대체하라.
  - 하지만 swap2의 문제점이 있다. List<?>에는 null외에는 어떤 값도 넣을 수 없다. 그래서 컴파일되지 않는 경우가 발생하며, 내부에서 다시 swap1을 호출해야 한다.
  
### Item 32. 제네릭과 가변인수를 함께 쓸때는 신중하라.

** 가변인수는 메서드에 넘기는 인수에 개수를 클라이언트가 조절할 수 있게 해주지만 구현 방식에 허점이 있다.

- 제네릭과 가변인수를 함께 사용하면 안전성이 깨진다.
  ```java
  static void dangerous(List<String>... stringLists) {
  	List<Integer> intList = List.of(42);
	Object[] objects = stringLists;
	object[0] = intList; // 힙 오염 발생
	String s = stringLists[0].get(0); //ClassCastException
  }
  ```
  - 제네릭 varargs 배열 매개변수에 값을 저장하는 것은 안전하지 않다.
  - 가변인자를 받는 메서드를 호출하면 호출하는 시점에서 varargs 제네릭 배열이 만들어진다.

- @SafeVarargs
  - 제네릭 가변인수 메서드 작성자가 클라이언트 측에서 발생하는 경고를 숨길 수 있게되었다.
  - @SafeVarargs 메서드 작성자가 그 메서드가 타입 안전함을 보장하는 장치다.
  - 타입 안전한 경우
    - @SafeVarargs로 제대로 애노테이트된 또 다른 varargs 메서드에 넘기는 것은 안전하다.
    - 그저 이 배열 내용의 일부 함수를 호출만 하는 일반 메서드에 넘기는 것도 안전하다.
  - 사용해야 할 때를 정하는 규칙
    - 제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 모든 메서드에 @SafeVarargs를 달라.
    
### Item33. 타입 안전 이종 컨테이너를 고려하라.

** 제네릭은 Set<E>, Map<E> 등의 컬렉션과 ThreadLocal<T>. AtomicReference<T> 등의 단일원소 컨테이너에 사용된다.
** 매개변수화되는 대상은 원소가 아닌 컨테이너 자신이다. 따라서 하나의 컨테이너에서 매개변수화 할 수있는 타입은 제한된다.

- 타입 안전 이종 컨테이너(hetereogeneous) 패턴
  - 컨테이너 대신 키를 매개변수화한 다음, 컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공하는 방법
  ```java
  public class Favorites {
  	private Map<Calss<?, Object> favorites = new HashMap<>();
  	public <T> void putFavorite(Class<T> type, T instance) {
		favorites.put(Object.requireNorNull(type), instance);
	}
	public <T> void T getFavorite(Class<T> type) {
		return type.cast(favorites.get(type));
	}
  }
  
  public class MainTest {
  	public static void main(String[] args) {
		Favorites f = new Favorites();
		f.putFavorite(String.class, "Java");
		f.putFavorite(Integer.class, 0xcafebabe);
		
		System.out.println(f.getFavorite(String.class) + " " + f.getFavorite(Integer.class) );
	}
  }
  ```
  - Favorites 인스턴스는 타입 안전하다.
  - Map<Class<?>, Object>: 와일드카드 타입이 중첩(nested)되었다는 점을 알아야 한다. 맵이 아니라 키가 와일드카드 타입이다.
  - 안전성이 깨지는 경우
    - row 타입을 넘기면 안전성이 쉽게 깨진다.
    - 실체화 불가 타입에는 사용할 수 없다.
      - List.class는 가능하나 List<String>.class, List<Integer>.class는 불가능하다. (즉 List.class 하나만 담긴다.)
      - 이를 해결하기 위해 슈퍼타입토큰을 사용할 수 있다.
