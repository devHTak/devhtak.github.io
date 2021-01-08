---
layout: post
title: 스터디 할래 2. 자바 데이터 타입, 변수
summary: 백기선님과 스터디 할래
author: devhtak
date: '2021-01-08 23:41:00 +0900'
category: Java Study
---

### 자바 데이터 타입, 변수

#### 목표

- 자바의 프리미티브 타입, 변수 그리고 배열을 사용하는 방법을 익힙니다.

#### 학습할 내용

- 프리미티브 타입 종류와 값의 범위 그리고 기본 값
- 프리미티브 타입과 레퍼런스 타입
- 리터럴
- 변수 선언 및 초기화하는 방법
- 변수의 스코프와 라이프타임
- 타입 변환, 캐스팅 그리고 타입 프로모션
- 타입 추론, var

#### 프리미티브 타입 종류와 값의 범위 그리고 기본 값

- 기본형 타입(Primitive Type)
  - 총 8가지의 기본형 타입을 미리 정의하여 제공
  - 기본값이 있기 때문에 null이 존재하지 않는다.
  - 실제 값을 저장하는 공간으로 스택 메모리에 저장
  - 종류
  
    |타입|할당되는 메모리 크기|기본값|데이터의 표현 범위|
    |---|---|---|---|
    |boolean|1 byte|false|true, false|
    |byte|1 byte|0|-128 ~ 127|
    |short|2 byte|0|-32,768 ~ 32,767|
    |int|4 byte|0|-2,147,483,648 ~ 2,147,483,647|
    |long|8 byte|0|-9,223,372,036,854,775,808 ~ 9,223,372,036,854,775,807|
    |float|4 byte|0.0f|(3.4 X 10-38) ~ (3.4 X 1038) 의 근사값|
    |double|8 byte|0.0|(1.7 X 10-308) ~ (1.7 X 10308) 의 근사값|
    |char|2 byte|\u0000|0 ~ 65,535|
    
#### 프리미티브 타입과 레퍼런스 타입

- 참조형 타입 (Reference Type)
  - 기본형 타입을 제외한 타입들은 모두 참조형 타입
  - null이 존재한다. 
  - 값이 저장되어 있는 곳의 주소값이 저장하는 공간으로 Heap 메모리에 저장된다.
  - 문법상으로는 에러가 없지만 실행시켰을 때 런타임 에러가 발생할 수 있다. 
  - 종류
  
    |타입|기본값|할당되는 메모리 크기|
    |---|---|---|
    |배열(Array)|Null|4 byte(객체의 주소값)|
    |열거(Enumeration)|Null|4 byte(객체의 주소값)|
    |클래스(Class)|Null|4 byte(객체의 주소값)|
    |인터페이스(Interface)||Null|4 byte(객체의 주소값)|

- Primitive Type 과 Reference Type 출처: https://gbsb.tistory.com/6

#### 리터럴(literal)

- literal은 데이터 그 자체로 변수에 넣는 변하지 않는 데이터를 의미한다.
- ex)
  ```java
  int num = 1; // num은 변수고, 1이 리터럴이다.
  ```
  
- 정수 리터럴
  - 모든 임의의 정수 값은 정수 리터럴이다. 
  - 모든 정수형 데이터가 기본적으로 int이기 때문에 long리터럴이면 숫자 뒤에 L 또는 l을 붙인다.
  - byte, short 변수에 숫자를 저장할 때에는 숫자 범위가 해당 테이터의 자료형에 포함되면 에러가 발생하지 않는다.
  - 진수를 사용할 수 있다.

- 부동 소수점
  - 소수점 이하를 가진 10진 값
  - double형은 부동 소수점의 기본형, 숫자 뒤에 d 또는 D를 추가할 수 있다.
  
- boolean
  - 논리적인 값: 참(true) / 거짓(false)를 갖는다.
  
- 문자(character)
  - 자바에서는 모든 몬자들은 Unicode를 사용한다.

- 문자열(String)

#### 변수 선언 및 초기화하는 방법

- 변수 선언
  ```java
  int a;
  Student student;
  ```
  - 저장공간을 확보하겠다는 의미
  - primitive type에 경우 초기값이 저장되고 reference type은 null값이 저장된다.
  
- 초기화
  ```java
  a = 10;
  student = new Stuent();
  ```
  - 저장공간에 원하는 값을 의미한다.

#### 변수의 스코프와 라이프타임

- 스코프
  - 해당 변수를 사용할 수 있는 영역 범위
  - 스코프에 따라 instance, class, local 변수로 나눈다.
  - instance variables
    - 클래스 안에서 선언되고, 어떠한 method나 block안에 선언되지 않은 변수
    - scope: static method를 제외한 클래스 전체
    - lifetime: 클래스를 인스턴스화한 객체가 메모리에 사라질 때 까지
  - class variables
    - 클래스 안에서 선언되고, 어떠한 메서드나 블럭안에서 선언되지 않았으며, static 키워드가 포함되어 선언된 변수
    - scope: 클래스 전체
    - lifetime: 프로그램 종료시 까지
  - local variables
     - 인스턴스 변수, 클래스 변수가 아닌 모든 변수
     - scope: 변수가 선언된 block 내부
     - lifetime: control이 변수가 선언된 block 내부에 있는 동안
    
    ```java
    public class VariableEx {
        int num1, num2;     // instance variables
        static int result;  // class variables
        public int add(int a, int b) { // local variables
            return a + b; 
        }
    }
    ``` 
  
- 라이프 타입
  - 해당 변수가 메모리에 언제까지 살아있는지를 의미

#### 타입 변환, 캐스팅 그리고 타입 프로모션

- type casting
  - 더 큰 자료형을 크기가 더 작은 자료형에 대입
  - 강제 형 변환을 해야하며(type), 데이터 크기가 작아지기 때문에 데이터 손실이나 변형이 올 수 있다.
  - ex) long -> int
    ```java
    long a = 10L;
    int b = (int)a;
    ```
    
- type promotion
  - 타입 캐스팅과는 밴대로 작은 자료형에서 큰 자료형으로 대입
  - 자동으로 변환 가능하며 데이터 크기가 커지기 때문에 손실이나 변형이 발생하지 않는다.
  - ex) int -> long
    ```java
    int a = 10;
    long b = a;
    ```
- reference type에서의 type casting과 type promotion
  - reference에서는 상속관계 가능하다.
  - type casting: 부모 인스턴스를 자식 인스턴스의 대입
    ```java
    Person p = new Person();
    Student s = (Student)p;
    ```
  - type promotion: 자식 인스턴스를 부모 인스턴스의 대입
    ```java
    Student s = new Student();
    Person p = s;
    ```

#### 타입 추론, var    

- 타입추론이란 데이터 타입을 소스코드에 명시하지 않고, 컴파일 단계에서 컴파일러가 타입을 유추하는 것
- 5 버전 이후 Generic, 8 q버전 이후 lamda 에서 타입추론이 되며, 10 버전부터는 var라는 local variable type-inference가 추가되었다.
  ```java
  var str = "hello"; // String str = "hello";
  var b = 10;       // int b = 10;
  ```
