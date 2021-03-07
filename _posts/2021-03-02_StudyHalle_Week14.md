---
layout: post
title: 스터디 할래 14. 제네릭
summary: 백기선님과 스터디 할래
author: devhtak
date: '2021-03-02 21:41:00 +0900'
category: Java Study
---

#### 학습할 것
- 제네릭 사용법
- 제네릭 주요 개념 (바운디드 타입, 와일드 카드)
- 제네릭 메소드 만들기
- Erasure

#### 제네릭 사용법

- Generic 이란?
  - 데이터 타입을 일반화하는 것을 의미한다.
  - 클래스나 메서드에서 컴파일 시에 미리 지정하는 방법으로 이렇게 컴파일 시 type check를 하면 장점이 있다.
    - 클래스나 메소드 내부에서 사용되는 객체 타입의 안전성을 높일 수 있다.
    - 반환값에 대한 타입 변환 및 타입 검사에 들어가는 노력을 줄일 수 있다.
  - Java 5 이전에 Object
    - 제네릭이 나오기 전 Object 타입을 사용했다.
    - 이 경우에는 반환된 Object를 다시 type casting을 해야 하고, 이 때 오류가 발생할 가능성도 있다.
    - Generic은 컴파일 시에 미리 타입이 정해지므로, 타입 검사, 변환과 같은 번거로운 작업을 생략할 수 있다.

- Generic 사용 이유
  - Generic 타입을 사용함으로써 잘못된 타입이 사용될 수 있는 문제를 컴파일 과정에서 제거할 수 있다.
  - 자바 컴파일러는 코드에서 잘못 사용된 타입 때문에 발생하는 문제점을 제거하기 위해 제네릭 코드에 대해 강한 타입 체크를 하기 때문에 런타임시에 에러가 나는 것을 막아준다.
  - 타입 변환을할 필요가 없기 때문에 프로그램 성능 향상 효과가 있다.

- 사용법

```java
class SampleGeneric<T> {
    T element;
    void setElement(T element) { this.element = element; }
    T getElement() { return this.element; }
}
```
  - 타입 변수
    - 아무런 이름이나 지정해도 컴파일하는 데 전혀 상관이 없다.
    - 현존하는 클래스를 사용해도 되고 존재하지 않는 것을 사용해도 된다.
    - 임의의 참조형 타입을 의미한다.
    - 여러개의 탕입 변수는 쉼표(,)로 구분하여 명시할 수 있다 
      - <K, V>
    - 타입 변수는 클래스에서뿐 아니라 메소드의 매개변수나 반환값으로도 사용할 수 있다.
      - T getElement() ... void setElement(T element)

  - 타입별 기능
    
    |타입|기능|
    |---|---|
    |E|요소(Element, Collection에서 주로 사용)|
    |K|키|
    |V|값|
    |T|타입|
    |S,U,V|두, 세, 네번째에 선언된 타입|
    
  - 주의점
    - static 멤버에 타입변수 T를 사용할 수 없다.
      - T는 인스턴스 변수로 간주되기 때문이다.
      - static 멤버는 인스턴스 변수를 참조할 수 없다.
    - 제네릭 타입의 배열을 생성하는 것도 허용되지 않는다.
      - 제네릭 배열 타입의 참조변수를 선언하는 것은 가능하다.
      - new T[10]과 같은 방법으로 배열을 생성하는 것은 안된다.
        - new 연산자 떄문
      - 해결책
        - 꼭! T[]을 만들어야 한다면,
        - Arrays.newInstance(); 메소드나 Object[] 배열을 생성하여 clone()하는 방법을 사용할 수 있다.

#### 제네릭 주요 개념 (바운디드 타입, 와일드 카드)

- Bounded type parameter
  ```java
  public class SampleBoundType<T extends Number> {
      T element;      
  }  
  ```  
  - 바운드타입은 특정 타입의 서브타입으로 제한한다. 
  - 위 예제에서는 Number 클래스를 구현한 Integer와 같은 타입만 제네릭에 대입할 수 있다.

- WildCard
  - 제네릭 단점
    - 제네릭으로 구현된 메소드의 경우 선언된 타입으로만 매개 변수를 입력해야 한다.
    - 이를 상속받은 클래스 혹은 부모 클래스를 사용하고 싶어도 불가능하기 때문에 다형성을 구현하기 어렵다.
  - 해결책: 와일드카드!
  
  - Unbounded WildCard
    - Unbounded -> 무한한...
    - List<?>와 같은 방식으로 ?로 종의한다.
    - 내부적으로는 Object로 구현되어 있어 모든 타입의 인자를 받을 수 있다.
    - Object에 정의된 메소드만으로 충분한 경우에 사용
    - 타입 파라미터에 의존저기지 않는 경우 사용한다.
      - List.clear, List.size 등

  - UpperBounded WildCard
    ```java
    List<? extends Foo>
    ```
    - Foo를 상속받는 하위 클래스는 모두 올 수 있다
    - Foo에서 정의된 기능만으로 사용이 가능하다.
    
  - LowerBounded WildCard
    ```java
    List<? upper Foo>
    ```
    - Foo가 구현한 부모 클래스들이 모두 올 수 있다.

  

#### 제네릭 메소드 만들기

- 메서드 선언분에 제네릭 타입이 선언된 메서드를 제네릭 메서드라 한다.
- 대표 예제
  - Collections.sort()
  ```java
  static <T> void sort(List<T> list, Comparator<? super T> c)
  ```
- 제네릭 클래스에 정의된 타입 매개변수와 제네릭 메소드에 정의된 타입 매개변수는 전혀 별개의 것이다.
  - 같은 타입 문자 T를 사용해도 같지 않다.
- 제네릭 메소드는 매개변수의 타입이 복잡할 때 유용하게 사용할 수 있다.  
  - 기존
  ```java
  public static void printAll(List<? extends Product> list) {
      for(Product p : list) {
          System.out.println(p.getName());
      }
  }
  ```
  - 제네릭 메소드로 변경
  ```java
  public static <T extends Product>void printAll(List<T> list) {
      for(Product p : list) {
          System.out.println(p.getName());
      } 
  }
  ```
#### Erasure

- 컴파일러는 제네릭 타입을 이용해 소스파일을 체크하고, 필요한 곳에 형변환을 넣어준다. 그 후에 제네릭 타입을 제거한다.
- 클래스 파일을 보면 제네릭 타입 정보는 사라지는 데 이를 Type Erasure라 한다.
- 이유
  - 제네릭이 도입되기 이전의 소스코드와의 호환성을 유지하기 위해서이다.

- 예제
  - 각기 다른 제네릭을 컴파일해 보자
    - java 파일
    ```java
    public class Test {
        public static void main(String[] args) { 
            List<String> strings = new ArrayList<>();
            List<Integer> integers = new ArrayList<>();
            List list = new ArrayList();        
        }
    }   
    ```
    - class 파일
    ```java
    public class Test {
        public Ori() {}
        public static void main(String[] args) { 
            new ArrayList();
            new ArrayList();
            new ArrayList();        
        }
    }   
    ```
  - 제네릭 타입 제거 - unbounded
    ```java
    public class Node<T> {
        private T data;
        private Node<T> next;
        
        public Node(T data, Node<T> next) {
            this.data = data;
            this.next = next;
        }
        
        public T getData() {
            return data;
        }
    }
    ```
    - class 파일
    ```java
    public class Node {
        private Object data;
        private Node next;
        
        public Node(Object data, Node next) {
            this.data = data;
            this.next = next;
        }
        
        public Object getData() {
            return data;
        }
    }
    ```
    - 일반적인 Object로 대체 사용되는 것을 알 수 있다.
    
  - 제네릭 타입 제거 - bounded 사용
    ```java
    public class Node<T extends Number> {
        private T data;
        private Node<T> next;
        
        public Node(T data, Node<T> next) {
            this.data = data;
            this.next = next;
        }
        
        public T getData() {
            return data;
        }
    }
    ```
    
    - class 파일
    ```java
    public class Node {
        private Number data;
        private Node<Number> next;
        
        public Node(Number data, Node<Number> next) {
            this.data = data;
            this.next = next;
        }
        
        public Number getData() {
            return data;
        }
    }
    ```
    - extends 되어 있는 변수로 대체되어 사용되는 것을 알 수 있다.
     
  - 메소드 제네릭 타입 제거
    ```java
    public static <T> int count(T[] anArray, T element) {
        int cnt = 0;
        for(T e: anArray) {
            if(e.equals(element)
                cnt+=1;
        }
        return cnt;
    }
    ```
    - class 파일
    ```java
    public static int count(Object[] anArray, Object element) {
        int cnt = 0;
        for(Object e: anArray) {
            if(e.equals(element)
              cnt += 1;
        }
        return cnt;
    }
    ```
    - T가 Object로 대체된다.

#### Generic 예제

- 백기선님 스터디에서 언급된 Generic을 활용하여 Repository를 만드는 예제를 따라했습니다.

- Bananna 객체와 Apple 객체를 저장하는 Repository 생성
  - Banana.class
  ```java
  @Getter @Setter
  public class Banana {	
      private Long id;
      private String name;
  }
  ```
  
  - Apple.class
  ```java
  @Getter @Setter
  public class Apple {	
      private Long id;
      private String name;
  }
  ```

- 제네릭 사용하기 이전 Respository
  - 객체마다 Repository를 만들어야 한다.
  - 단순 CRUD에 경우 코드가 중복된다.
  - BananaRepository.java
  ```java
  public class BananaRepository {	
      private static Map<Long, Banana> store = new HashMap<>();
      public void save(Banana banana) {
          store.put(banana.getId(), banana);
      }
      public void findById(Long id) {
          store.get(id);
      }
  }
  ```
  - AppleRepository.java
  ```java
  public class AppleRepository {	
      private static Map<Long, Apple> store = new HashMap<>();
      public void save(Apple apple) {
          store.put(apple.getId(), apple);
      }
      public void findById(Long id) {
          store.get(id);
      }
  }
  ```
- 제네릭을 사용하여 Repository의 중복코드를 삭제하자.
  - Repository.java
  ```java
  public class Repository<E, K> {
      private Map<K, E> store = new HashMap<>();
      public void store(E element) {
          store.put(element.getId(), element);
      }
      public E findById(K id) {
          return store.get(id);
      }
  }
  ```
  - 하지만 value.getId()에서 오류가 발생한다.
  - 이유는, V 타입에 getId() 메소드가 모른다, 이것을 해결해도 리턴하는 K타입이 무엇인지 모를 수 있다.
  - 해결책: Apple, Banana 객체를 추상화하여 bounded generic으로 수정
  
  ```java
  public interface Entity<K> {
      public K getId();
  }
  ```
  ```java
  public class Banana extends Entity<Long> {
      private Long id;
      private String name;
      
      public Long getId() {
          return this.id;
      }
  }
  ```
  ```java
  public class Apple implements Entity<Long> {
      private Long id;
      private String name;
      
      public Long getId() {
          return this.id;
      }
  }
  ```
  
  - 오류를 수정한 Repository
  ```java
  public class Repository<E extends Entity<K>, K> {	
      private Map<K, E> store = new HashMap<>();
      public void store(E element) {
          store.put(element.getId(), element);
      }
      public E findById(K id) {
          return store.get(id);
      }
  }
  ```
  ```java
  public class AppleRepository extends Repository<Apple, Long> {}
  ```
  ```java
  public class BananaRepository extends Repository<Banana, Long> {}
  ```
  
  - Generic을 활용해서 추상화하였기 때문에 Repository에 Element, Key를 넘겨줌으로써 중복 코드를 없앨 수 있다.
  - 그렇다면 현재 코드에서 AppleRepository와 BananRepository를 만들 필요가 있을까?
    ```java
    Repository<Banana, Long> bananaRepository = new Repository<>();
    Repository<Apple, Long> appleRepository = new Repository<>();
    ```
    - 이렇게 Generic한 Repository로도 충분히 중복된 메소드를 사용하여 객체별 처리를 할 수 있다.
    - 그렇다면 왜?! AppleRepository와 BananaRepository를 만들었을까?
      - 단순 CRUD 말고도 Apple, Banana 특성에 맞는 메소드가 필요할수있기 때문이다.
