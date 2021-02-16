---
layout: post
title: 스터디 할래 11. Enum
summary: 백기선님과 스터디 할래
author: devhtak
date: '2021-02-03 21:41:00 +0900'
category: Java Study
---

#### 학습할 것

- enum 정의하는 방법
- enum이 제공하는 메소드 (values()와 valueOf())
- java.lang.Enum
- EnumSet

#### enum 정의하는 방법

- enum이란?
  - 서로 관련된 상수를 편리하게 선언하기 위한 것으로 상수를 여러개 정의할 때 사용한다.
  - enum은 여러 상수를 정의한 후, 정의된 것 이외의 값을 호용하지 않는다.
  
- 정의 방법
  ```java
  enum EnumName {
    상수명1, 상수명2, ....;
  }
  ```
  
- enum 특징
  - enum에 정의된 상수들은 해당 enum type의 객체이다.
  - 생성자와 메서드를 추가할 수 있다.
  - 상수간의 비교가 가능하다.
    - == 연산자
    - 단, >, <를 사용할 수 없고, compareTo() 를 사용할 수 있다.
    
- enum은 언제 사용하는가
  ```
  필요한 원소를 컴파일 타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자.  
  태양계 행성, 한 주의 요일, 체스 말처럼 본질적으로 열거 타입인 타입은 당연히 포함된다.
  그리고 메뉴 아이템, 연산 코드, 명령줄 플래그 등 허용하는 값 모두를 컴파일타임에 이미 알고 있을 때도 쓸 수 있다. 
  열거 타입에 정의된 상수 개수가 영원히 고정 불변일 필요는 없다.
  열거 타입은 나중에 상수가 추가돼도 바이너리 수준에서 호환되도록 설계되었다
  - 이펙티브 자바 3/E
  ```

    - Type Safety를 위해 사용한다.
      - 오타, 상수 값에 대한 비교 하는 경우를 예방할 수 있다.

#### enum이 제공하는 메소드 (values()와 valueOf())

|메서드|설명|
|---|---|
|T[] values()|해당 enum 타입에 정의된 상수 배열을 반환한다.|
|Class<E> getDeclaringClass()|열거형의 객체를 반환한다.|
|String name()|열거형 상수의 이름을 문자열로 반환한다.|
|int ordinal()|열거형 상수가 정의된 순서를 반환한다.(0부터 시작)|
|T valueOf(Class<T> enumType, String name)|지정된 열거형에서 name과 일치하는 열거형 상수를 반환한다.|

```java
public enum Fruit {
    APPLE, ORANGE, PINEAPPLE;
}
```
```java
public class EnumMain {
    public static void main(String[] args) {
        for(var e: Fruit.values()) {
            System.out.println(e.name()); // APPLE, ORANGE, PINEAPPLE 출력
            System.out.println(e.ordinal()); // 0, 1, 2 출력
        }
        
        Fruit apple = Enum.valueOf(Fruit.class, "APPLE");
        Fruit orange = Fruit.valueOf("ORANGE");
    }
}
```
- values()
  - 모든 enum이 가지고 있는 것으로 컴파일러가 자동으로 추가해준다.
  
- ordinal()
  - 상수가 정의한 순서를 반환한다.
  - enum 내부에서 사용하도록 만든 것이므로 해당 메서드를 의존하여 코드를 작성하는 것은 안티패턴이다.
  
- T valueOf(Class<T> enumType, String name)
  - 지정된 열거형에서 name과 일치하는 열거형을 반환한다.
  
#### java.lang.Enum

- java.lang.Enum은 모든 Enum 클래스의 조상 클래스이다.
- 모든 열거형은 Enum 클래스를 상속받기 때문에 enum type은 별도의 상속을 받을 수 없다.

#### EnumSet

- EnumSet은 enum을 위해 고안된 특별한 Set 인터페이스 구현체이다.
- HashSet과 비교했을 때 성능 상의 이점이 많기 때문에 열거형 데이터를 위한 SEt이 필요한 경우 EnumSet을 사용하는 것이 좋다.
- 특징
  - EnumSet은 AbstractSet 클래스를 상속하고 Set 인터페이스를 구현한다.
  - 오직 열거형 상수만을 값으로 가질 수 있다. 또한 모든 값은 같은 enum type이어야 한다.
  - null value를 추가하는 것을 허용하지 않는다. NullPointerException을 던지는 것도 허용하지 않는다.
  - ordinal 값의 순서대로 요소가 저장된다.
  - tread-safe하지 않다. 동기식으로 사용하려면 Collections.synchronizedMap을 사용하거나, 외부에서 동기화를 구현해야한다.
  - 모든 메서드는 arithmetic bitwise operation을 사용하기 때문에 모든 기본 연산의 시간 복잡도가 O(1)이다.
  
- 예제
  ```java
  public enum Fruit {
      APPLE, ORANGE, PINEAPPLE, WATERMELLON, MELLON;
  }
  ```
  ```java
  public class EnumMain {
      public static void main(String[] args) {
          EnumSet<Fruit> set1 = EnumSet.allOf(Fruit.class);
          EnumSet<Fruit> set2 = EnumSet.of(Fruit.APPLE, Fruit.ORANGE);
          EnumSet<Fruit> set3 = EnumSet.complementOf(set2);
          EnumSet<Fruit> set4 = EnumSet.range(Fruit.ORANGE, Fruit.WATERMELLON);
          
          EnumSet<Fruit> set5 = EnumSet.noneOf(Fruit.class);
          set5.add(Fruit.WATERMELLON);
          set5.add(Fruit.PINEAPPLE);
          set5.remove(Fruit.WATERMELLON);
          
          System.out.println(set1); // [APPLE, ORANGE, PINEAPPLE, WATERMELLON, MELLON]
          System.out.println(set2); // [APPLE, ORANGE]
          System.out.println(set3); // [PINEAPPLE, WATERMELLON, MELLON]
          System.out.println(set4); // [ORANGE, PINEAPPLE, WATERMELLON]
          System.out.println(set5); // [PINEAPPLE]
      }
  }
  ```
