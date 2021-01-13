---
layout: post
title: 스터디 할래 4. 제어문
summary: 백기선님과 스터디 할래
author: devhtak
date: '2021-01-13 20:41:00 +0900'
category: Java Study
---

#### 목표
- 자바가 제공하는 제어문을 학습하세요.

#### 학습할 것 (필수)
- 선택문
- 반복문

#### 선택문

- if/else 문
  - if 문에 들어가는 조건식이 참인 경우에 if문 내의 블록 코드를 실행한다.
  - 만약 조건식이 거짓인 경위 else if 문의 조건을 실행하며, 모든 조건이 거짓인 경우 else 문의 코드를 실행한다.
  - 또한, if문 안에 if 문을 사용할 수 있다. 이를 중첩 if문이라고 한다.
    ```java
    if(a < 5) {
        System.out.println("a가 5보다 작은 경우");
    } else if(a > 5) {
        System.out.println("a가 5보다 큰 경우");
    } else {
        System.out.println("a가 5인 경우");
    }
    ```
    
- switch/case 문
  - 여러 개의 if문은 가독성 및 조건 탐색을 해야하는 단점이 있다.
  - switch문은 switch의 매개변수에 맞는 조건에 따라 case 문을 실행하여 다중 if문의 단점을 개선한 선택문이다.
  - 각각의 case문에 break 키워드를 사용하지 않으면 switch문을 탈출하지 않으므로 다음 case문도 실행하기 때문에 주의해야 한다.
    ```java
    int a = 3;
    switch(a) {
        case 1 : 
          System.out.println("a가 1입니다.");
          break;
        case 2 : 
          System.out.println("a가 2입니다.");
          break;
        case 3 : 
          System.out.println("a가 3입니다.");
          break;
        default : 
          System.out.println("a가 그 외의 값입니다.");
    }
    ```
  - Java 12부터 switch/case 문에 기능이 확장되었다.
    - 
    
#### 반복문
  
- for문(초기화;조건문;증감식)
  - 초기화한 값을 가지고 조건문을 검사해 초기화한 값을 증감식의 조건에 따라 증감해가면서 for문 내부의 코드를 반복하는 구문이다.
    ```java
    for(int i = 0; i < 10; i++) {
        System.out.println("for문 실행 - 횟수: " + (i+1));
    } // 0부터 9까지 총 10회 실행
    ```
    
- for-each 문(향상된 for문)
  - for문과 동일하게 for를 사용하지만 구조가 for문보다 직관적이고, 반복할 객체를 하나씩 차례대로 가져와 사용하는 구조이다.
    ```java
    for(int num : list) {
        System.out.println(num);
    }
    ```
    
- while문(조건)
  - while문은 조건의 값이 참인 경우에는 계속 반복하는 구문이다.
  - 조건이 항상 참인 경우 무한루프에 빠질 가능성이 있기 때문에 유한적인 조건을 주거나 내부 탈출 조건을 주어야 한다.
    ```java
    int i = 0;
    while(i < 10) {
        i++;
        System.out.println("while문 실행 - 횟수: " + i);
    }
    ```
  
- do-while문(하단 조건)
  - do-while문은 do의 구문을 먼저 실행한 후 마지막에 조건을 확인함으로써 실행 후 조건확인이라는 순서의 차이가 있다.
    ```java
    int i = 0;
    do {
        i++;
        System.out.println("do-while문 실행 - 횟수: " + i);
    } while(i < 10);
    ```
    
- Iterator
  - Iterator는 Java의 Collection에 저장되어 있는 데이터를 읽어오는 방법을 표준화한 기술 중 하나다.
  - hasNext(), next(), remove() 등의 메소드를 이용해 데이터를 뽑아와 사용할 수 있다.
    ```java
    Set<String> set = new HashSet<>();
    set.add("A"); set.add("B"); set.add("C");
    
    Iterator<String> it = set.iterator();
    while(it.hasNext()) {
        System.out.println(it.next()); // 요소 출력 후 다음 요소로 
        it.remove(); // 요소 삭제
    }
    ```
