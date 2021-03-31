---
layout: post
title: 스터디 할래 6. 상속
summary: 백기선님과 스터디 할래
author: devhtak
date: '2021-01-19 21:41:00 +0900'
category: Java Study
---

#### 학습할 것 (필수)

- 자바 상속의 특징
- super 키워드
- 메소드 오버라이딩
- 다이나믹 메소드 디스패치 (Dynamic Method Dispatch)
- 추상 클래스
- final 키워드
- Object 클래스

#### 자바 상속의 특징

- 상속이란?
  - 부모클래스의 변수와 메소드를 물려받는 것을 말한다.
  - extends 키워드를 통해 상속을 받는다.
    ```java
    class Child extends Parent {
        // ...
    }
    ```
    
- 상속 대상
  - 접근 지정자에 따라 상속하여도 사용할 수 없는 변수 및 메소드들이 있다.
  - 
    |접근 지정자|같은 클래스|같은 패키지의 멤버|자식 클래스의 멤버|그 외의 영역|
    |---|---|---|---|---|
    |public|O|O|O|O|
    |protected|O|O|O|X|
    |default|O|O|X|X|
    |private|O|X|X|X|
    
- 상속의 특징
  - 상속은 단일 상속만 가능하다
  - 자바의 계층 구조의 최상위 객체는 java.lang.Object 클래스다
  - 자바에서는 상속의 횟수 제한이 없다.
  - 부모의 메소드와 변수만 상속되며, 생성자는 상속되지 않는다.
    - 부모의 메소드는 재정의하여 사용가능(오버라이딩)
    
#### super 키워드

- super 키워드는 자식클래스가 부모클래스로부터 상속받은 멤버를 사용하고자 할 때 사용한다
  - cf) this 키워드: 본인 클래스의 멤버, 메소드 접근 시 사용
- 부모 클래스의 생성자를 호출할 수 있다. (super(); super(a, b); ...)
  - cf) this(): 본인 클래스의 생성자 호출
  
#### 메소드 오버라이딩

- 메소드 오버라이딩은 부모 클래스의 메소드를 재정의하는 것
  - 함수명, 리턴 값, 파라미터가 모두 동일해야 한다.
    ```java
    public class Parent {
        public void print() {
            System.out.println("PARENT");
        }
    }    
    ```
    ```java
    public class Child extends Parent {
        @Override
        public void print() {
            // TODO Auto-generated method stub
            System.out.println("CHILD");
        }
    }
    ```
  - cf) 메소드 오버로딩
    - 메소드 오버로딩은 클래스 내에서 리턴값과 메소드명은 같지만 파라미터 개수, 타입 등이 다른 것을 의미한다.

#### 다이나믹 메소드 디스패치 (Dynamic Method Dispatch)

- 어떤 메소드를 호출할 지 경청하여 실제로 실행시키는 과정을 말한다.
- 정적 메소드 디스패치, 동적 메소드 디스패치, 더블 디스페치가 존재한다.
- 정적 메소드 디스패치 (Static Method Dispatch)
  ```java
  public class Parent {
      public void print() {
          System.out.println("PARENT");
      }
  }    
  ```
  ```java
  public class Child extends Parent {
      @Override
      public void print() {
          // TODO Auto-generated method stub
          System.out.println("CHILD");
      }
  }
  ```
  ```java
  public class Main {
      public static void main(String[] args) {
          Child child = new Child();
          child.print(); // CHILD 출력
      }
  }
  ```
    - 오버라이딩 소스와 동일 (오버로딩 또한 정적 메소드 디스패치)
    - child.print()를 호출 할 때 Child 클래스에 오버라이딩한 메소드가 실행될 것을 알고 있다.
    - 컴파일러 또한 Child클래스에 메소드를 실행해야 하는 것을 정확히 알고 있다.

- 동적 메소드 디스패치 (Dynamic Method Dispatch)
  - 동적 메소드 디스패치는 정적 메소드 디스패치와는 다르게 컴파일러가 어떤 메소드를 호출해야 하는 지 모르는 것을 의미한다.
    ```java
    interface InterfaceExample() {
        public void print();
    }
    ```
    ```java
    public class ClassExample {
        private InterfaceExample interfaceExample;
        public ClassExample(InterfaceExample interfaceExample) {
            this.interfaceExample = interfaceExample;
        }
        public void print() {
            this.interfaceExample.print();
        }
    }
    ```
    ```java
    public class InterfaceImplementOne implements InterfaceExample {
        public void print() {
            System.out.print("ONE");
        }
    }
    ```
    ```java
    public class InterfaceImplementTwo implements InterfaceExample {
        public void print() {
            System.out.print("TWO");
        }
    }
    ```
      - InterfaceExample을 구현한 구현 클래스는 2가지가 있다.
      - ClassExample 인스턴스를 생성할 때 사용하는 InterfaceImplementOne, InterfaceImplementTwo 종류에 따라 print되는 함수가 달라진다.
      - 즉, 컴파일러가 실행되기 전까지 어떤 메소드가 실행될 지 전혀 모르는 것이다. 런타임 시점에야 알 수 있다.

- 더블 디스패치(Double Dispatch)
  - Dynamic Dispatch를 두번하는 것
    ```java
    interface Post {
        void postOn(SNS sns);
    }
    class Text implements Post {
        public void postOn(SNS sns) {
            sns.post(this);
        }
    }
    class Picture implements Post {
        public void postOn(SNS sns) {
            sns.post(this);
        }
    }
    ```
    ```java
    interface SNS {
        void post(Text text);
        void post(Picture picture);
    }
    class Facebook implements SNS {
        public void post(Text text) {
            // text -> facebook
        }
        public void post(Picture picture) {
            // picture -> facebook
        }
    }
    class Twitter implements SNS {
        public void post(Text text) {
            // text -> twitter
        }
        public void post(Picture picture) {
            // picture -> twitter
        }
    }
    ```
    - 두번의 동적 메소드 디스패치가 일어난다.
      - Post의 postOn 메소드에서 SNS 구현체(Facebook, Twitter)의 post() 메서드
      - postOn 메소드 내의 SNS 구현체(Facebook, Twitter) 의 post 메서드에서, Post 구현체(Text, Picture)에 대한 오버로딩
  
  - 방문자 패턴(Vistor Pattern)
    - 더블 디스패치를 사용하는 대표적인 패턴
    - 객체 구조에서 데이터와 메소드를 구분하기 위해 사용한다
    
#### 추상 클래스

- 추상 메소드
  - 메서드의 선언부만 작성하고 구현부는 작성하지 않은 체 남겨두는 메서드를 말한다.
  - 보통 어떤 기능을 수행할 지 주석 처리하고, 구현부는 자식 클래스에서 오버라이딩한다
  - 만약 자식 클래스가 추상메서드를 구현하지 않는 경우 자식클래스 또한 추상클래스가 되어야 한다.
  - 추상 메서드의 접근 지정자는 private일 수 없다.
    - private 은 상속관계에서 접근이 불가능하기 때문
    ```java
    abstract 리턴타입 메서드명(파라미터); // 추상 메소드
    ```

- 추상 클래스
  - 하나 이상의 추상 메서드를 포함하고 있어야 한다.
  - 자체 인스턴스 생성 불가능 -> 추상메서드를 구현한 자식클래스를 인스턴스로 생성할 수 있다
  - 생성자와 멤버변수, 일반 메서드 모두 가질 수 있다.
    ```java
    abstract class 클래스명 { //  추상클래스
        // ..추상 메서드 하나 이상 포함
    }
    ```
  
- 인터페이스
  - 인터페이스는 오직 추상 메소드와 상수만을 멤버로 가질 수 있다.
    - 상수: public static final -> 생략 가능하다
      - 구현 객체의 같은 동작을 보장하기 위한 목적
      - 인터페이스의 변수는 스스로 초기화 될 권한이 없다
      - 인스턴스를 생성하지 못하기 때문에 인스턴스가 존재하지 않는 시점이기 때문
    - 메서드: public abstract -> 생략 가능하다
    - Java 8부터 인터페이스에서 default method, static method 구현이 가능하다
      - 인터페이스의 모든 추상메소드를 구현하는 것을 막기 위해 default method가 가능해졌다. 오버라이딩이 가능하다.
      - static method가 가능해지므로써 인터페이스를 이용한 간단한 기능을 가지는 유틸리티성 인터페이스를 만들 수 있게 되었다.
        ```java
        public interface Calculator {
            public int plus(int a, int b);
            public int multiple(int a, intb);
            default int exec(int i, int j) { // default method
                return i + j; 
            }
            public static exec2(int i, int j) { // static method
                return i * j;
            }
        }
        ```
  - 인터페이스의 구현(implements)은 상속이 아니기 때문에 다중 구현이 가능하다

#### final 키워드

- final 키워드는 엔티티를 한 번만 할당하겠다는 의미
- final 변수
  - 상수를 의미한다. 생성자, 대입연산자를 통해 한번 초기화 가능
- final 메소드
  - 메소드 오버라이딩하거나 숨길 수 없음을 의미한다(캡슐화)
- final 클래스
  - 상속할 수 없음을 의미한다.
  - 상속 계층에서 가장 마지막(자식) 클래스임을 의미한다.
  
#### Object 클래스

- 모든 클래스의 최상위 클래스이다.
- Object 클래스를 모두 상속하기 때문에 오버라이딩이 가능하다.
  - 주의점이 있다 (Effective Java 참고)
  
  |메소드|설명|
  |---|---|
  |boolean equals(Object obj)|두 객체가 같은 지 비교|
  |String toString()|객체의 문자열을 반환|
  |protected Object clone()|객체 복사|
  |protected void finalize()|GC 직전에 객체의 리소스를 정리할 때 호출|
  |Class getClass()|객체의 클래스형 반환|
  |int hashCode()|객체의 코드값 반환|
  |void notify()|wait된 스레드 실행을 재개할 때 호출|
  |void notifyAll()|wait된 모든 스레드 실행을 재개할 때 호출|
  |void wait()|스레드를 일시적으로 중지할 때 호출|
  |void wait(long timeout)|timeout 시간만큼 스레드를 일시적으로 중지할 때 호출|
  |void wait(long timeout, int nanos)|timeout 시간만큼 스레드를 일시적으로 중지할 때 호출|

\[출처] 온라인 자바 스터디#6 - 자바 상속(메소드 오버라이딩, 추상클래스, 다이나믹 메소드 디스패치)
