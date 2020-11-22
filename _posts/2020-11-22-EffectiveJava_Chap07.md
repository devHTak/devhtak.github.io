---
layout: post
title: Effective Java Ch07. 람다와 스트림
summary: Effective Java 3판 공부
author: devhtak
date: '2020-11-22 11:41:00 +0900'
category: java
---

### ITEM 42. 익명 클래스보다는 람다를 사용하라
- 람다식
  - Java 8 이전에는 함수 타입을 표현할 때 추상 메서드 하나만을 담은 인터페이스 또는 추상 클래스를 사용했다.
  - Java 8 이후부터는 함수형 인터페이스라 부르는 람다식을 사용해 만들 수 있게 되었다.
  ```java
  // Java 8 이전
  Collections.sort(words, new Comparator<String>() {
      public int compare(String s1, String s2) { return Integer.compare(s1.length(), s2.length()); }
  });
  // Java 8 이후 - 람다 사용
  Collections.sort(words, (s1, s2) -> Integer.compare(s1.length(), s2.length()));
  ```
  - 람다를 사용하면 매개 변수에 대한 타입, return type 등에 언급이 없다. 컴파일러가 문맥을 살펴 타입을 추론한다.
  - 람다는 이름이 없고 문서화도 못한다. 따라서 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수가 많아지면 람다를 쓰지 말아야 한다.
  
- 익명 클래스를 사용해야 할 때
  - 람다는 함수형 인터페이스에서만 사용된다. 추상 클래스의 인스턴스를 만들 때 람다를 쓸수 없으니 익명 클래스를 사용해야 한다.
  - 여러 추상 메서드를 갖는 인터페이스의 인스턴스를 만들 때에도 익명 클래스를 사용해야 한다.
  - 람다는 자신을 참조할 수 없다. 람다에서의 this는 바깥 인스턴스를 가리키는 반면 익명 클래스의 this는 익명 클래스의 인스턴스 자신을 가리킨다.
  - 가상머신별 / 구현별로 다를 수 있기 때문에 람다를 직렬화 해서는 안된다.
  
### ITEM 43. 람다 보다는 메서드 참조를 사용하라
- 메서드 참조
  - 람다보다 더 간결하게 표현할 수 있다.
  - 예시) 임의의 키와 Integer값으 매핑을 관리하는 프로그램 일부로 키가 map 안에 없으면 1을 매핑하고, 없다면 기존 매핑 값을 증가시킨다.  
  ```java
  // 람다 사용
  map.merge(key, 1, (count, increase) -> count + increase);
  // 메서드 참조 사용
  map.merge(key, 1, Integer::sum);
  ```
  - 메서드 참조를 사용하면 가독성이 좋다. 다만 클래스, 메서드 이름이 길거나 할 때에는 메서드 참조를 사용할 때 가독성이 더 떨어질 수도 있다.
  
- 메서드 참조 유형 
  - 
  
  
