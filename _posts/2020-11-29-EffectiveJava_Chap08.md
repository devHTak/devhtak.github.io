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
    
  - private 메서드 통제
    - assert를 사용해 매개변수 유효성을 검증할 수 있다.
    ```java
    private static void sort(long[] a, int offset, int length) {
        assert a != null;
        assert offset >= 0 && offset <= a.length;
        assert length >= 0 && length <= a.length - offset;
        // 계산 수행
    }
    ```
    - 여기서 이 assert 문들은 자신이 단언한 조건이 무조건 참이라고 선언한다는 것이다.
      - 실패하면 AssertionError를 던진다.
      - 런타임에 아무런 효과도, 아무런 성능 저하도 없다.

  - 매개변수 유효성 검사 예외
    - 유효성 검사 비용이 지나치케 높거나 실용적이지 않을 때
    - 계산 과정에서 암묵적으로 검사가 수행될 때
      - 하지만 암묵적 유혀성 검사에 너무 의존했다가 실패 원자성을 해칠 수 있다.
      - 때로는 계산 과정에서 필요한 유효성 검사가 이뤄지지만 실패했을 때 잘못된 예외를 던지기도 한다.

### ITEM 50. 적시에 방어적 복사본을 만들라

- 방어적으로 프로그래밍 하라
  ```java
  public final class Period {
      private final Date start;
      private final Date end;
      
      /*
      * @param start 시작 시각
      * @param end 종료 시각, 시작 시각보다 뒤여야 한다.
      * @throws IllegalArgumentException 시작 시각이 종료 시각보다 늦을 때 발생
      * @throws NullPointerException start나 end가 null인 경우 발생
      public Period(Date start, Date end) {
          if(start.compareTo(end) > 0) throw new IllegalArgumentException(start + " after " + end);
          this.start = start;
          this.end = end;
      }
      public Date start() { return this.start; }
      public Date end() { returh this.end; }
      // ... 나머지 코드 생략
      */
  }
  ```
  - 불변 클래스로 보이며, 시작 시간이 종료 시각보다 늦을 수 없다는 불변식이 무리 없이 지켜질 것 같다
    - 하지만 Date가 가변이라는 사실을 이용하면 어렵지 않게 그 불변식을 깨트릴 수 있다.
    ```java
    Date start = new Date();
    Date end = new Date();
    Period p = new Period(start, end);
    end.setYear(78); // p의 내부를 수정할 수 있다.
    ```
    - 8버전 이후부터 Date 대신 Instant를 사용하면 된다 (또는 LocalDateTime, ZonedDateTime)
    - 외부 공격으로부터 Period 인스턴스의 내부를 보호하려면 생성자에서 받은 가변 매개변수를 각각을 방어적으로 복사해야 한다.
    ```java
    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());
        if(this.start.compareTo(this.end) > 0 ) throw new IllegalArgumentException(this.start + " after " + this.end);
    }
    ```
    - 방어적 복사본을 만든 후에 유효성을 검사했다.
  - 매개변수가 제 3자에 의해 확장될 수 있는 타입이라면 방어적 복사본을 생성할 때 clone을 사용하면 안된다.
  - 가변인 내부 객체를 반환할 때는 반드시 심사숙고해야 하며 안심할 수 없다면 방어적 복사본을 반환해야 한다.
    ```java
    Date start = new Date();
    Date end = new Date();
    Period p = new Period(start, end);
    p.end().setYear(78);
    ```
    - 해당 공격을 대비하기 위해서는 방어적 복사본을 반환하라.
    ```java
    public Date end() {
        return new Date(this.end.getTime());
    }
    ```
    - 생성자와 달리 접근자 메서드에는 방어적 복사에 clone()을 사용해도 된다.
  - 방어적 복사에는 성능 저하가 따르고, 또 같은 패키지에 속하는 등의 이유로 항상 쓸 수 있는 것도 아니다. 
  
### ITEM 51. 메서드 시그니처를 신중히 설계하라


    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
