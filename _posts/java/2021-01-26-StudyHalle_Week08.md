---
layout: post
title: 스터디 할래 8. 인터페이스
summary: 백기선님과 스터디 할래
author: devhtak
date: '2021-01-26 21:41:00 +0900'
category: Java Study
---

#### 학습할 것

- 인터페이스 정의하는 방법
- 인터페이스 구현하는 방법
- 인터페이스 레퍼런스를 통해 구현체를 사용하는 방법
- 인터페이스 상속
- 인터페이스의 기본 메소드 (Default Method), 자바 8
- 인터페이스의 static 메소드, 자바 8
- 인터페이스의 private 메소드, 자바 9

#### 인터페이스란

- Interface
  - 객체의 사용 방법을 정의한 타입
  - 객체의 교환성을 높여주어 다형성을 구현하는 데에 중요한 역할을 한다.
  - 개발 코드를 수정하지 않고, 사용하는 객체를 변경할 수 있도록 하기 위해서 인터페이스를 사용한다.
  - 인터페이스는 하나의 객체가 아니라, 여러 객체들과 사용이 가능하다 (다양화)

#### 인터페이스 정의하는 방법

- interface 키워드
  
  ```
  [public] interface 인터페이스명 { 
      // 상수
      변수_타입 상수명 = 값;
      
      // 추상 메소드
      타입 메소드(매개변수, ...);
      
      // 디폴트 메소드
      default 리턴타입 메소드명(매개변수, ...) {
          // 구현부
      }
      
      // 정적 메소드
      static 리턴타입 메소드명(매개변수, ...) {
          // 구현부
      }
  }
  ```
  - 클래스 이름 작성 방법과 동일
  
- 인터페이스 특징
  - 인터페이스는 상수와 메소드만을 구성 멤버로 가진다.
    - 클래스는 필드, 생성자, 메소드를 구성 멤버로 가진다.
    - 인터페이스는 객체 생성이 불가능하기 때문에 생성자를 갖지 않는다.
  - 자바 7 이전 까지는 인터페이스의 메소드는 실행 블록이 없는 추상 메소드만 선언 가능
    - 자바 8 이후부터는 디폴트 메소드와 정적 메소드 선언이 가능한다.
    
- 상수 필드(Constant Field)
  - 인터페이스는 객체 사용 설명서이므로 런타임 시 데이터를 저장할 수 있는 필드를 선언할 수 없다.
  - 그러나, 상수 필드는 선언이 가능하다
    - 상수는 인터페이스에 고정된 값으로 런타임 시에는 데이터를 변경할 수 없기 때문이다.
    - 인터페이스는 static 블록을 사용하지 못하므로 선언과 동시에 초기값 지정이 필요하다.
  - 인터페이스는 public static final 생략 가능  
    
- 추상 메소드
  - 구현(상속)하는 객체가 포함되어야 하는 기능을 명시
  - 호출 시, 어떤 매개값과 리턴타입이 무엇인지만 알려준다.
  - 실제 실행부는 객체(구현 객체)가 가지고 있다.
  - public abstract 생략 가능
  
- 디폴트 메소드
  - 자바 8부터 도입
  - 인터페이스에 선언되지만 사실은 구현 객체가 가지고 있는 인스턴스 메소드

- 정적 메소드
  - 자바 8부터 도입
  - 디폴트 메소드와는 달리 객체가 없더라도 인터페이스만으로 호출 가능
  
- private method, private static method
  - 자바 9 부터 작성 가능

#### 인터페이스 구현하는 방법

```java
public class 구현 클래스 명 implements 인터페이스 명 {
    // 인터페이스에 선언된 추상 메소드의 실체 메소드 선언
}
```
- 개발 코드가 인터페이스 메소드를 호출하면 인터페이스는 객체의 메소드를 호출
- 객체는 인터페이스에서 정의된 추상 메소드와 동일한 메소드 이름, 매개 타입, 리턴타입을 가진 실제 메소드를 가지고 있어야 한다.
  - 이런 객체를 인터페이스의 구현(implement) 객체라고 하고, 구현 객체를 생성하는 클래스를 구현 클래스라고 한다.

- 구현 클래스
  - 보통의 클래스와 동일하다
  - 인터페이스 타입으로 사용할 수 있음을 알려주기 위해 클래스 선언에 implements 키워드를 추가하고 인터페이스명을 명시
  - 인터페이스에 선언된 추상 메소드의 실체 메소드를 구현
  
- 익명 구현 객체
  ```
  인터페이스 변수 = new 인터페이스() {
      // 추상메소드 구현
  }
  ```
  - 구현 클래스를 만들어 사용하는 것이 일반적이고, 클래스를 재사용할 수 있기 때문에 편리하지만, 일회성의 구현 객체를 만들기 위해 소스 파일을 만들고, 클래스를 선언하는 것은 비효율적이다.
  - 인터페이스에 선언된 추상 메소드를 실체 메소드로 선언해야 하며, 그렇지 않으면 오류 발생
  - Java 8. @FunctionalInterface -> lambda
  
- 다중 인터페이스 구현
  - 객체는 다수의 인터페이스를 사용할 수 있다.

#### 인터페이스 상속과 인터페이스 레퍼런스를 통해 구현체를 사용하는 방법

- 인터페이스의 상속 구조
  - 서브 인터페이스는 슈퍼 인터페이스의 메서드까지 모두 구현해야 한다.
  - 인터페이스 레퍼런스는 인터페이스를 구현한 클래스의 인스턴스를 가리킬 수 있고, 해당 인터페이스에 선언된 메서드(슈퍼 인스턴스 메소드 포함)만 호출할 수 있다.
  
- 인터페이스 다중 상속
  - 인터페이스는 클래스와 달리 다중 상속 가능
  - 인터페이스의 메서드는 추상 메서드로 구현하기 전의 메서드이기 때문에 어떤 인터페이스의 메서드를 상속받아도 같기 때문
    - 그러나, 인터페이스 구현 클래스의 부모 인터페이스가 동일한 메소드명, 동일한 파라미터를 갖지만 다른 리턴타입의 메소드를 갖고 있으면 컴파일 오류 발생

#### 인터페이스의 기본 메소드 (Default Method), 자바 8

- 인터페이스에 메소드 선언이 아니라 '구현체'를 제공
- 해당 인터페이스를 구현한 클래스의 어떠한 영향없이 새로운 기능을 추가하는 방법
- default method는 해당 인터페이스를 구현한 구현체가 모르게 추가된 기능임으로 그만큼 리스크가 따른다.
  - 컴파일 에러는 발생하지 않지만, 특정한 구현체의 로직에 따라 런타임 에러가 발생할 수 있다
  - 사용하게 된다면, 구현체가 잘못사용하지 않도록 반드시 문서화해야 한다.
- Object가 제공하는 기능(equals, hashCode)와 같은 기본 메소드는 제공할 수 없다.
  - 구현체가 재정의 하여 사용하는 것은 상관없다.
- 본인이 수정할 수 있는 인터페이스에만 기본 메소드를 제공할 수 있다.
- 인터페이스를 상속받은 인터페이스에서 다시 추상 메소드로 변경할 수 있다.
- 인터페이스 구현체가 default method를 재정의할 수 있다.
- 형태
  ```
  default 리턴타입 메소드명(파라미터, ...) {
      // 구현
  }
  ```

#### 인터페이스의 static 메소드, 자바 8

- 해당 인터페이스를 구현한 모든 인스턴스, 해당 타입에 고나련되어 있는 유틸리티, 헬퍼 메소드를 제공하고 싶다면 ?
  - static method로 제공할 수 있다.
- 인스턴스 없이 수행할 수 있는 작업을 정의할 수 있는 것이라 볼 수 있다.
- 형태
  ```
  static 리턴타입 메소드명(파라미터, ...) {
      // 구현
  }
  ```

#### 인터페이스의 private 메소드, 자바 9

- default method와 static method 의 단점은 내부적으로 처리하는 메소드임에도 public 으로 선언해야 하는 단점이 있다.
- private 메소드의 4가지 규칙
  - private 메소드는 구현부를 가져야만 한다.
  - 오직 인터페이스 내부에서만 사용할 수 있다.
  - private static 메소드는 다른 static 또는 static이 아닌 메소드에서 사용할 수 있다.
  - static이 아닌 private 메소드는 다른 private static 메소드에서 사용할 수 없다.

- 형태
  ```
  private static 리턴타입 메소드명(파라미터, ...) {
      // 구현
  }
  private 리턴타입 메소드명(파라미터, ...) {
      // 구현
  }
  ```
  
  ** 출처: 백기선님과 스터디할래 #8 인터페이스
