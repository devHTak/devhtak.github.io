---
layout: post
title: Effective Java Ch06. 열거타입과 애너테이션
summary: Effective Java 3판 공부
author: devhtak
date: '2020-11-17 19:41:00 +0900'
category: java
---

### int 상수 대신 열거 타입을 사용하라

** 열거 타입은 일정 개수의 상수 값을 정의한 다음, 그 외의 값은 허용하지 않는 타입니다.

- 정수 열거 패턴(int enum pattern)에 단점
  - 타입 안전을 보장하지 않으며, 표현력도 떨어진다.
  - 프로그램이 깨지기 쉽다.
    - 클라이언트 파일에 그대로 새겨지기 때문에, 상수의 값이 바뀌면 클라이언트도 반드시 다시 컴파일 해야 한다.
  - 정수 상수는 문자열로 출력하기 까다롭다.
  
- 문자열 열거 패턴 (string enum pattern)에 단점
  - 오타 등으로 런타임 버그가 발생한다.
  
- 열거 타입(enum type)의 장점
```java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```
  
