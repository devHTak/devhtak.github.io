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
    - 이를 상속받은 클래스 혹은 부모 클래스를 사용하고 싶어도 불가능하기 때문에 다형성을 구현하기 어렵다는 점이 있다.
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


#### Erasure
