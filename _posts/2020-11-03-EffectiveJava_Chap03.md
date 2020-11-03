---
layout: post
title: Effective Java Ch03. 모든 객체의 공통 메서드
summary: Effective Java 3판 공부
author: devhtak
date: '2020-11-03 09:41:00 +0900'
category: java
---

### 들어가기

- Object 에서 equals, hashCode, toString, clone, finalize 는 모두 재정의(overriding)을 염두에 두고 설계된 것이라 재정의 시 지켜야 하는 일반 규약이 명확히 정의되어 있다.
- 해당 메서드를 재정의할 때에는 일반 규약이 명확히 정의되어 있는데, 만약 잘못 구현하면 대상 클래스가 이 규약을 준수한다고 가정하는 클래스(HashMap, HashSet 등)를 오작동하게 만들 수 있다.

### Item 10. equals는 일반 규약을 지켜 재정의하라

- equals를 재정의하지 말아야 할 때
  - 각 인스턴스가 본질적으로 고유하다.
    ex) Thread와 같이 값을 표현하는 것이 아닌 동작하는 개체를 표현하는 클래스
    
  - 인스턴스의 '논리적 동치성(logical equality)'을 검사할 일이 없다.
    ex) java.util.regex.Pattern
    equals를 재정의해서 두 Pattern의 인스턴스가 같은 정규표현식을 나타내는지를 검사하는, 즉 논리적 동치성을 검사하는 방법도 있다. -> equals 재정의
    클라이언트가 해당 방식을 원하지 않거나 애초에 필요하지 않다고 판단할 수도 있다. -> Object의 equals 사용
    
  - 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어 맞는다.
    ex) Set의 구현체는 AbstractSet이 구현한 equals를 상속받아 쓰고, List 구현체는 AbstractList로부터, Map 구현체는 AbstractMap으로부터 상속받아 그대로 사용한다.

  - 클래스가 private거나 package-private이면 equals 메서드를 호출할 일이 없다.
    ex) 호출되는 것을 막기 위해 Exception 처리
    ```java
    @Override public boolean equals(Object o) {
      throw new AssertionError(); // 호출 금지
    }
    ```

- equals를 재정의해야 할 때
  - 객체 식별성(Object identity: 두 객체가 물리적으로 같은가)이 아니라 논리적 동치성을 확인해야 하는데, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때.
    주로 값 클래스(Integer, String과 같이 값을 표현하는 클래스), 객체가 같은지가 아닌 값이 같은지 확인하고자 사용
    
  - 규약
    - Object 명세에 나오는 규약
    |!--|!--|
