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
    |규약|내용|
    |:---:|:---|
    |반사성(reflexivity)|null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true이다.|
    |대칭성(symmetry)|null이 아닌 모든 참조 값 x, y에 대해 x.equals(y)가 true이면 y.equals(x)도 true이다.|
    |추이성(transivity)|null이 아닌 모든 참조값 x, y, z에 대해 x.equals(y)가 true이고, y.equals(z)도 true이면, x.equals(z)도 true이다.|
    |일관성(consistency)|null이 아닌 모든 참조값 x, y에 대해 x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.|
    |null-아님|null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false다.|
    
    - 반사성은 자기 자신과 같아야 한다는 뜻
    - 대칭성은 서로에 대한 동치 여부에 똑같이 답해야 한다는 뜻이다.
      
    ```java
    public class CaseInsentiveString { // 대칭성 위배 예제
      private final String s;

      public CaseInsentiveString(String s) {
        this.s = Objects.requireNonNull(s);
      }

      @Override public boolean equals(Object o) {
        // TODO Auto-generated method stub
        if( o instanceof CaseInsentiveString)
          return s.equalsIgnoreCase( ((CaseInsentiveString) o).s);

        if( o instanceof String ) // 한방향으로만 작동
          return s.equals((String) o);

        return false;
      }
    }
    ```
    ```java
    public class CaseInsentiveStringMain {
      public static void main(String[] args) {
        CaseInsentiveString cis = new CaseInsentiveString("Polish");
        String s = "polish";

        System.out.println(cis.equals(s)); // true
        System.out.println(s.equals(cis)); // false
      }
    }
    ```
    cis.equals(s) 는 true, s.equals(cis) 는 false를 반환하여 대칭성을 명백히 위반한다.
    ```java
    @Override public boolean equals(Object o) {
      // TODO Auto-generated method stub
      return o instanceof CaseInsentiveString && ((CaseInsentiveString) o).s.equals(s);
    }
    ```
    CaseInsentiveString의 equals를 String과 비교할 생각을 버리면 위와 같이 간단하게 구현할 수 있다.
    
    - 추이성은 첫 번째 객체와 두 번째 객체가 같고, 두 번째 객체와 세 번째 객체가 같다면, 첫 번째 객체와 세 번째 객체도 같아야 한다는 의미
    ```java
    public class Point {
      private final int x;
      private final int y;

      public Point(int x, int y) {
        this.x = x; this.y = y;
      }

      @Override
      public boolean equals(Object o) {
        // TODO Auto-generated method stub
        if( !(o instanceof Point))
          return false;
        Point p = (Point)o;
        return p.x == x && p.y == y;
      }	
    }
    
    public class ColorPoint extends Point{	
      private Color color;
      public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
      }
      /// 생략
    }
    ```
    ColorPoint에 equals를 구현하지 않은 상태인 경우 equals 규약에 위배되지 않는다. 다만 ColorPoint의 color 를 놓치게 된다.
    
    ```java
    @Override public boolean equals(Object o) {
        // TODO Auto-generated method stub
        if( !(o instanceof ColorPoint) )
          return false;
        return super.equals(o) && this.color == ((ColorPoint) o).color;
      }
    ```
    위와 같이 ColorPoint에서 equals를 재정의 하는 경우 Point와 ColorPoint는 동치성 위배가 된다. ColorPoint에 equals에서 instanceof ColorPoint 가 계속 false이기 때문이다.
    
    ```java
    @Override public boolean equals(Object o) {
      if( !(o instanceof Point) )
        return false;

      if( !(o instanceof ColorPoint) )
        return o.equals(this);

      return super.equals(o) && this.color == ((ColorPoint) o).color;		
    }
    ```
    위와 같이 수정하면 동치성은 성립하지만 추이성은 위배된다
    ```java
    public class EqualsMain {
      public static void main(String[] args) {
        ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
        Point p2 = new Point(1, 2);
        ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);

        System.out.println(p1.equals(p2)); // true
        System.out.println(p2.equals(p3)); // true
        System.out.println(p1.equals(p3)); // false
      }
    }
    ```
    p1과 p2, p2와 p3를 비교할 때에는 색상을 무시했지만 p1과 p3 비교에서는 색상까지 고려했기 때문이다.
    
    구체 클래스의 하위 클래스에서 값을 추가할 방법은 없지만 우회 방법이 존재한다.
    -> Point를 상속하는 대신 Point를 ColorPoint의 private 필드로 두고 ColorPoint와 같은 위치의 일반 Point를 반환하는 뷰 메서드를 public으로 추가하는 것
    
    ```java
    public class ColorPoint {
      private Point point;
      private Color color;
      public ColorPoint(int x, int y, Color color) {
        point = new Point(x, y);
        this.color = color;
      }
      public Point asPoint() {
        return point;
      }
      @Override public boolean equals(Object o) {
        if( !(o instanceof ColorPoint) )
          return false;

        return this.point.equals(((ColorPoint) o).point) 
            && this.color == ((ColorPoint) o).color;		
      }
    }
    ```
    - 일관성은 두 객체가 같다면 앞으로 영원히 같아야 한다.
    -> 가변 객체는 비교 시점에 따라 서로 다를 수도 같을 수도 있지만, 불변 객체는 한번 다르면 끝까지 달라야 한다.
    -> 하지만, 클래스가 불변이든 가변이든 equals의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안된다.
    
    - null-아님은 모든 객체가 null과 같지 않아야 한다는 뜻이다.
    
- equals 메서드 구현 방법
  - == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
  - instanceof 연산자로 입력이 올바른 타입인지 확인한다.
  - 입력을 올바른 타입으로 형변환한다.
  - 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사한다.
  - equals를 재정의할 땐 hashCode도 반드시 재정의하자
  - 너무 복잡하게 해결하려하지 말자
  - Object 외에 타입을 매개변수로 받는 equals 메서드는 선언하지 말자
  - 꼭 필요한 경우가 아니면 equals를 재정의하지 말자
  
