---
layout: post
title: Effective Java Ch08. 메서드
summary: Effective Java 3판 공부
author: devhtak
date: '2020-11-29 15:41:00 +0900'
category: java
---

### ITEM 49. 매개변수가 유효한지 검사하라

- 매개변수 검사
  - 매개변수를 제대로 검사하지 못하면 문제가 발생할 수 있다. (실패 원자성, failure aotmicity을 어기는 결과를 낳을 수 있다.)
    - 메서드가 수행되는 중간에 모호한 예외를 던지며 실패할 수 있다.
    - 메서드가 잘 수행되지만 잘못된 결과를 반환할 수 있다.
    - 메서드는 문제없이 수행됐지만, 어떤 객체를 이상한 상태로 만들어 놓아 알 수 없는 시점에 해당 메서드와 상관없는 오류를 낼 수 있다.
    
  - public, protected 메서드는 매개변수 값이 잘못됐을 때 던지는 예외를 문서화해야 한다.
    - IllegalArgumentException, IndexOutOfBoundsException, NullPointerException 중 하나
    
      ```java
      /*
      * (현재 값 mod m) 값을 반환한다. 이 메서드는
      * 항상 음이 아닌 BigInteger를 반환한다는 점에서 remainder 메서드와 다르다
      *
      * @param m 계수(양수여야 한다.)
      * @return 현재 값 mod m
      * @throws ArithmeticException m이 0보다 작거나 같으면 발생한다.    
      */
      public BigInteger mod(BigInteger m) {
          if( m.signum() <= 0 ) {
              throws new ArithmeticException("계수(m)은 양수여야 합니다. " + m);
          }
          //...
      }
      ```
      - m 이 null이면 m.signum()을 호출할 때 NullPointerException을 던진다. 하지만 메서드 설명에 해당 내용을 기술하지 않았다.
      - 그 이유는 BigInteger 클래스 수준에서 기술했기 때문이다.
      - java.util.Objects.requireNonNull 메서드를 사용하여 null 검사를 수동으로 하지 않아도 된다.
      ```java
      // 자바의 null 검사 기능 사용하기
      this.strategy = Objects.requireNonNull(strategy, "전략");
      ```
      - 9버전 부터는 Objects 객체를 활용하여 범위 검사 기능도 더해졌다.
        - checkFromIndexSize, checkFromToIndex, checkIndex
    
    
