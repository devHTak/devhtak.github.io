---
layout: post
title: Effective Java Ch02. 객체 생성과 파괴
summary: Effective Java 3판 공부
author: devhtak
date: '2020-11-02 09:41:00 +0900'
category: java
---

### Item 00. 목적

- 객체를 만들어야 할 때와 만들지 말아야 할 때를 구분하는 법
- 올바른 객체 생성 방법과 불필요한 생성을 피하는 법
- 제 때 파괴됨을 보장하고 파괴 전에 수행해야 할 정리 작업을 관리하는 요쳥

### Item 01. 생성자 대신 정적 팩터리 메소드를 고려하라

public 생성자를 사용하는 대신 정적 팩터리 메서드를 제공하는 것에 대해 장점과 단점이 존재한다.

- 장점
  -> 이름을 가질 수 잇다.
  
  ```
  public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
  }
  ```
