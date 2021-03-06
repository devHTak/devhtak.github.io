---
layout: post
title: 스터디 할래 3. 연산자
summary: 백기선님과 스터디 할래
author: devhtak
date: '2021-01-11 20:41:00 +0900'
category: Java Study
---

#### 목표
- 자바가 제공하는 다양한 연산자를 학습하세요.

#### 학습할 것
- 산술 연산자
- 비트 연산자
- 관계 연산자
- 논리 연산자
- instanceof
- assignment(=) operator
- 화살표(->) 연산자
- 3항 연산자
- 연산자 우선 순위
- (optional) Java 13. switch 연산자

#### 산술 연산자

- boolean 타입을 제외한 모든 Primitive Type에서 사용 가능하다.
- 부동소수점이 있다면 부동 소수점 산술이 되고 그렇지 않다면 정수 산술이 된다.

- 더하기 연산(+)
  - \+ 연산은 수를 더하는 연산이기도 하지만 문자열이 있다면 문자열을 이어 붙이는(concat) 문자열 연산으로 변환된다.
  
- 빼기 연산(-)
  - \- 연산은 수를 빼는 연산으로 문자열에서 적용하지 않는다.
  
- 곱하기 연산(*)
  - 두 숫자를 곱한다. 문자열에는 적용되지 않는다.
  
- 나누기 연산(/)
  - 두 숫자를 나눈다.
  - 피연산자가 정수이면 정수로 출력되며, 부동소수점이 있다면 결과도 부동소수점이다.
  - 정수를 0으로 나누면 AritemeticException이 발생한다.
  - 부동소수점을 0으로 나누면 결과는 Infinity 또는 NaN이 된다.
  
- 나머지 연산(%)
  - 첫번째 피연산자를 두번째 피연산자로 나누고 남은 나머지를 정수로 리턴한다.
  - 리턴된 결과는 첫번째 피연산자의 부호와 동일하다.
  - 정수를 0으로 나머지 연산하는 경우 AritemeticException이 발생한다.
  - 부동소수점을 0으로 나누면 결과는 NaN이 된다.
  
#### 비트연산자

- 비트 연산자 및 시프트 연산자는 개별 비트를 조작하는 연산자

- 비트 보수(Bitwise complement: ~)
  - NOT 연산자라고 불린다.
    ```java
    int i = ~1;  // 0b11111111_11111111_1111111_11111110
    byte j = ~1; // 0b11111110
    System.out.println(i + " " + j); // 2 2
    ```
- AND 연산자(&)
  - AND 연산 (비트곱)
  - 모두가 참이어야 참이다.(1 & 1 = 1)
    ```java
    int num1 = 10; // 0b00000000_00000000_00000000_00001010
    int num2 = 20; // 0b00000000_00000000_00000000_00010100
    int result = num1 & num2; // 0
    
    int num3 = 11; // 0b00000000_00000000_00000000_00001011
    int num4 = 15; // 0b00000000_00000000_00000000_00001111
    int result = num3 & num4; // 11
    ```
    
- OR 연산자(\|)
  - OR 연산자 (비트합)
  - 둘 중 하나가 참이면 참이다.(1 | 1 = 1, 1 | 0 = 1, 0 | 1 = 1)
    ```java
    int num1 = 10; // 0b00000000_00000000_00000000_00001010
    int num2 = 20; // 0b00000000_00000000_00000000_00010100
    int result = num1 | num2; // 30
    
    int num3 = 11; // 0b00000000_00000000_00000000_00001011
    int num4 = 15; // 0b00000000_00000000_00000000_00001111
    int result = num3 | num4; // 15
    ```

- XOR 연산(^)
  - Exclusive OR 연산
  - 두 피연산자가 모두 참이거나 거짓이면 거짓, 둘 중 하나가 참이면 참이된다.
    ```java
    int num1 = 10; // 0b00000000_00000000_00000000_00001010
    int num2 = 20; // 0b00000000_00000000_00000000_00010100
    int result = num1 ^ num2; // 30
    
    int num3 = 11; // 0b00000000_00000000_00000000_00001011
    int num4 = 15; // 0b00000000_00000000_00000000_00001111
    int result = num3 | num4; // 4
    ```
  - 백기선님 라이브 스터디 강의 발췌(XOR 활용 예제)
    - XOR의 사용법: 같은 수를 XOR 연산하면 0이되고, 0을 XOR 하면 그 수가 된다.
    - EX) 5 ^ 5 = 0000_0101 ^ 0000_0101 = 0000_0000 = 0, 5 ^ 0 = 0000_0101 ^ 0000_0000 = 0000_0101 = 5
    - 이런 논리를 이용하여 배열 안에 2개 있는 숫자가 아닌 1개만 있는 숫자를 구할 수 있다.
      ```java
      public int solution(int[] arr) {
          // arr = {1, 2, 3, 2, 1};
          int result = 0;
          for(int i = 0; i < arr.length; i++) {
              result ^= arr[i];
          }
          System.out.println(result); // 3
      }
      ```
    
- 왼쪽 시프트 연산(<<)
  - 두번째 비트 수 만큼 왼쪽으로 이동시킨다.
  - 시프트 될 때 왼쪽 비트는 삭제되고 오른쪽 비트는 0으로 채워진다.
  - x << y 는 x * 2 ^ y의 효과를 갖는다.
    ```java
    int num1 = 2;
    int num2 = 4;
    int result = num1 << num2; // 2 * 2 ^ 4 = 32 
    ```
    
- 오른쪽 시프트 연산(>>)
  - 두번째 비트 수 만큼 오른쪽으로 이동시킨다.
  - 시프트 될 때 오른쪽 비트는 삭제되고 왼쪽 비트는 양수의 경우 0, 음수의 경우 1이 채워진다.
    - 나머지는 버린다.
  - x >> y 는 x / 2 ^ y의 효과를 갖는다.
    ```java
    int num = 10;
    int result1 = num >> 1; // 5
    int result2 = num >> 2; // 2
    ```

- 부호없는(unsigned) 오른쪽 시프트 연산(>>>)
  - 두번째 비트 수 만큼 오른쪽으로 이동시킨다.
  - 시프트 될 때 오른쪽 비트는 삭제되고 왼쪽 비트는 무조건 0이 채워진다.
    ```java
    int num = -10; // 0b11111111_11111111_11111111_11110110
    int result1 = num >>> 1; // 0b01111111_11111111_11111111_11111011
    int result2 = num >>> 2; // 0b00111111_11111111_11111111_11111011
    ```
  - 백기선님 라이브 스터디 강의 발췌 (>>>을 활용할 예제)
    - 중간값 구하기 예제
      - 방법1. int mid = (min + max) / 2; // start와 end가 MAX 값에 인접하면 더할 때 overflow가 발생한다.
      - 방법2. int mid = min + (max - min) / 2; // (max-min)/2 한 값을 더하므로 overflow가 발생하지 않는다.
      - 방법3. int mid = (min + max) >>> 2; // 보통 오버플로우가 발생하면 부호 비트가 1이 되는 데, shift할 때 0을 채우기 때문에 양수가 되므로 overflow가 일어나도 숫자 변형이 발생하지 않는다.
    
#### 관계 연산자

- 같거나 같지 않음을 평가하는 비교 연산자
- 크고 작은 관게를 평가하는 관계 연산자
- true/false 결과를 도출하므로 조건문이나 반복문 등에서 사용

- 비교 연산자
  - Equals(==)
    - 정수, 부동소수점에 경우 숫자가 같은지를 비교
    - 부동소수점의 NaN의 경우 Float.isNan() or Double.isNan()을 사용한다.
    - 객체에 경우 같은 객체인지 비교(메모리 주소)
      - String의 경우 값이 같은지는 equals() 메소드를 사용한다.
  - Not Equals(!=)
    - 정수, 부동소수점에 경우 숫자가 다른지를 비교
    - 객체에 경우 같은 객체인지 비교(메모리 주소)
    
- 관계 연산자
  - 큼, 크거나 같음(>, >=)
  - 작음, 작거나 같음(<, <=)

#### 논리 연산자

- 여러 비교 연산을 사용하여 복잡도를 높인 조건식을 만들 때 주로 사용

- AND(&&)
- OR(\|\|)
- 부정 연산자(!)

- 단락 회로 평가(Short Circuit Evaluation)
  - &&, \|\| 연산을 사용할 때 첫번째 결과값이 정해졌을 때 두번째 연산자의 평가를 하지 않는 것을 말한다.
  - && 연산: 첫번째 결과가 false일 경우 두번째 연산을 평가하지 않고 false 리턴
  - \|\| 연산: 첫번째 결과가 true일 경우 두번째 연산을 평가하지 않고 true 리턴
  
#### instanceof

- 객체 또는 배열 값을 어떠한 참조 유형에 맞는 값인지를 평가하는 연산자
- null을 평가하면 false가 리턴
- 평가 결과가 true이면 비교된 참조 유형으로 안전하게 캐스팅하고 할당할 수 있다.

  ```java
  // safety casting
  if (obj instanceof MyClass) {
      MyClass c = (MyClass) obj;
  }

  i instanceof int // compile error : Primitive Type은 사용할 수 없습니다.
  System.out.println(null instanceof String); // null은 항상 false
  ```

#### assignment(=) operator

- 어떠한 변수에 값을 할당할 때 해당 연산자를 사용하며 메모리에 값을 저장하거나 할당한다는 의미
- 산술연산자와 함께 사용할 수 있다.
  - +=, -=, *=, /=, %=
- 비트 연산자, shift 연산자와 함께 사용할 수 있다.
  - &=, \|=, ^= 
  - <<=, >>=

#### 화살표(->) 연산자(Lambda Expression ->)

- Java 8 버전부터 도입된 람다 표현식에 사용되며 메소드 본문에 해당 실행 가능한 코드의 익명 컬렉션이다.
- 메소드 파라미터 목록, 연산자, 코드 블럭 순으로 구성된다.
- 코드블럭이 한문장인 경우 블럭을 생략할 수 있다.
  ```java
  Student studen = studentRepository.findById(id).orElseThrow(()->{
      throw new IllegalArgumentException();
  });
  ```

#### 3항 연산자( ? : )

- 조건 ? true일 때 결과 : false일 때 결과
- if else 조건문을 함축하여 사용할 수 있다.

#### 연산자 우선 순위

|우선 순위|연산자|연산방향|동작|
|---|---|---|---|
|1|.|->|객체 멤버 접근|
|1|[, ]|->|배열 요소 접근|
|1|(args)|->|메소드 호출|
|1|data++, data--|->|후위 증감|
|2|++data, --data|<-|전위 증감|
|2|+, -|<-|단항 증감|
|2|~, !|<-|비트 보수, 부정 연산|
|3|new|<-|객체 생성|
|3|(type)|<-|캐스팅|
|4|*, /, %|->|곱하기, 나누기, 나머지|
|5|+, -|->|더하기, 빼기|
|5|+|->|문자열 결합|
|6|<<, >>, >>>|->|왼쪽 시프트, 오른쪽 시프트, 부호없는 오른쪽 시프트|
|7|<, <=, >, >=|->|작음, 작거나 같음, 큼, 크거나 같음|
|7|instanceof|->|타입 비교|
|8|==, !=|->|같음, 같지 않음|
|9|&|->|AND|
|10|^|->|XOR|
|11|\||->|OR|
|12|&&|->|AND|
|13|\|\||->|OR|
|14|? :|<-|3항 연산자|
|15|=, *=, /=, %=, +=, -=, <<=, >>=, >>>=, &=, ^=, \|=|<-|대입 연산자|
|16|->|->|람다 표현식|

#### (optional) Java 13. switch 연산자

- Java 13버전부터 switch문을 좀 더 간편하게 사용할 수 있게 되었다.
- 기존 switch 문
  ```java
  int result;

  switch (mode) {
      case "a":
          result = 1;
          break;
      case "b":
          result = 2;
          break;
      case "c", "d", "e":
          result = 3;
      case "f", "g":
          System.out.println("this is f or g");
          result = 4;
          break;
      default:
          result = -1;
  }
  ```
  
- yield 키워드를 사용하여 switch 문을 리턴할 수 있다.
  ```java
  int result = switch (mode) {
      case "a":
          yield 1;
      case "b":
          yield 2;
      case "c", "d", "e":
          yield 3;
      case "f", "g":
          System.out.println("this is f or g");
          yield 4;
      default:
          yield -1;
  };
  ```
  
- (label rules) ->을 사용하여 case 구문 처리가 가능하다.
  ```java
  int result = switch (mode) {
      case "a" -> 1;
      case "b" -> 2;
      case "c", "d", "e" -> 3;
      case "f", "g" -> {
          System.out.println("this is f or g");
          yield 4;
      }
      default -> yield -1;
  };
  ```
  
- 출처: https://blog.baesangwoo.dev/posts/java-livestudy-3week/
