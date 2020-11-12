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
  
  
  
  
  
  
  
  
  
  
  
