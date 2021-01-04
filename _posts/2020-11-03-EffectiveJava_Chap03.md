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
      obj.equals(obj); // true
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
    -> 자기 자신이면 true를 반환 // 성능 최적화 용
  - instanceof 연산자로 입력이 올바른 타입인지 확인한다.
  - 입력을 올바른 타입으로 형변환한다.
  - 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사한다.
    
    -> Equals를 구현했으면 세가지를 확인 및 테스트를 하자. 대칭성, 추이성, 일관성!!
    
    ```java
    public class PhoneNumber {	
      private final short areaCode;
      private final short prefix;
      private final short lineNum;
      
      public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = rangeCheck(areaCode, 999, "지역코드");
        this.prefix = rangeCheck(prefix, 999, "프리픽스");
        this.lineNum = rangeCheck(lineNum, 9999, "가입자 번호");
      }

      private short rangeCheck(int val, int max, String arg) {
        if(val < 0 || val > max) 
          throw new IllegalArgumentException(arg + ": " + val);
        return (short) val;
      }

      @Override
      public boolean equals(Object o) {
        // TODO Auto-generated method stub
        if(o == this)
          return true;

        if( !(o instanceof PhoneNumber) )
          return false;

        PhoneNumber pn = (PhoneNumber)o;
        return pn.areaCode == this.areaCode && pn.prefix == this.prefix && pn.lineNum == this.lineNum;
      }
    }
    ```

  - equals를 재정의할 땐 hashCode도 반드시 재정의하자
  - 너무 복잡하게 해결하려하지 말자
  - Object 외에 타입을 매개변수로 받는 equals 메서드는 선언하지 말자
  - 꼭 필요한 경우가 아니면 equals를 재정의하지 말자
  
### Item 11. equals를 재정의하려거든 hashCode도 재정의하자

- Object 명세
  - equals 비교에서 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 메서드는 몇번을 호출해도 일관되게 항상 같은 값을 반환해야 한다.
  단, 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관 없다.
  - equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.
  - equals(Object)가 두 객체를 다르다고 판단했더라도, 두 hashCode가 다른 값을 반환할 필요는 없다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋다.
  
  ```java
  public static void main(String[] args) {
		Map<PhoneNumber, String> map = new HashMap<>();
		map.put(new PhoneNumber(707, 867, 5309), "Jenny");
		
		System.out.println(map.get(new PhoneNumber(707, 867, 5309))); // null
	}
  ```
  -> 2개의 인스턴스를 사용했고, hashCode를 재정의하지 않았기 때문에 2개의 인스턴스는 다른 hashCode를 반환하기 때문에 Jenny가 아닌 null이 리턴되는 것
  
- hashCode 재정의
  -> 좋은 hash 함수라면, 서로 다른 인스턴스에 다른 해시코드를 반환한다. 
  -> 서로 다른 인스턴스를 32bit 정수 범위에 균일하게 분배해야 한다.
  
  - hashCode 함수 작성하는 간단 요쳥
    1-1 int 변수 result를 선언한 후 값 c로 초기화 한다. c는 해당 객체의 첫번째 핵심필드를 단계 2.a 방식으로 계산한 해시코드
    
    2-1 c의 값을 계산한다.
    
      i. 기본 타입 필드라면, Type.hashCode(f)를 수행. Type은 기본타입에 박싱 클래스다.
      
      ii. 참조 타입 필드면서 이 클래스의 equals 메서드가 이 플드의 equals를 재귀적으로 호출해 비교한다면, 이 필드의 hashCode를 재귀적으로 호출한다. 계산이 복잡해질 것 같으면, 이 필드의 표준형을 만들어 그 표준형의 hashCode를 호출한다. 필드의 값이 null이면 0을 사용
     
      iii. 필드가 배열이면, 핵심 원소 각각을 별도 필드처럼 다룬다. 이상의 규칙을 재귀적으로 적용하여 2.b의 방법으로 갱신한다. 배열에 핵심 원소가 하나도 없다면 단순히 0을 사용하고, 모든 원소가 핵심 원소면, Arrays.hashCode를 사용
      
    2-2 단계 2.a에서 계산한 해시코드 c로 result을 갱신하여 리턴한다.
      result = 31 * result + c;
      return result;
    
 ```java
 @Override
public int hashCode() {
  // TODO Auto-generated method stub
  int result = Short.hashCode(this.areaCode);
  result = 31 * result + Short.hashCode(this.prefix);
  result = 31 * result + Short.hashCode(lineNum);
  return result;
}
 ```
  - 주의점
  -> 성능을 문제로 해시코드를 계산할 때 핵심 필드를 생략해서는 안된다. 품질 문제
  -> hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말자. 그래야 클라이언트가 해당 값에 의지하지 않게 되고, 추후에 계산 방식을 바꿀 수도 있다.

### Item 12. toString을 항상 재정의하라.

- Object의 toString 메서드는 클래스이름@16진수의_hashCode로 반환한다. -> 모든 구체 클래스에서 Object의 toString을 재정의하자.
- toString 을 잘 구현한 클래스는 훨씬 사용하기 수월하고, 디버깅하기 쉬워진다.

-> 단점도 발생한다. 포맷을 한번 명시하면 그 포맷에 얽매이게 된다. 포맷을 명시하든 아니든 의도를 명확히 밝혀야 한다.
-> 포맷 명시 여부와 상관없이 toString이 반환된 값에 포함된 정보를 얻어올 수 있는 API를 제공하자.

```java
/**
 * 해당 전화번호의 문자열 표현을 반환한다.
 * XXX-YYY-ZZZZ로 X는 지역코드, Y는 프리픽스, Z는 가입자 번호다.
 * 전화번호의 마지막 부분을 재우지 못한다면 맨 앞자리에 0으로 채워 진다.
 * ex) 02, 123, 123 -> 002-123-0123 
 */
@Override
public String toString() {
  // TODO Auto-generated method stub
  return String.format("%03d-%03d-%04d", this.areaCode, this.prefix, this.lineNum);
}
```

### Item 13. clone 재정의는 주의해서 진행하자.

- Clonable 인터페이스
  -> Clonable 클래스는 복제해도 되는 클래스임을 명시하는 용도의 mixin interface 이지만 목적을 제대로 이루지 못하고 있다.
    - clone 메서드가 선언된 곳은 Clonable이 아닌 Object이고 접근 제한자가 protected이다. 하지만 해당 인터페이스는 Object의 protected clone 메서드의 동작 방식을 결정한다.

- clone 메서드의 일반 규약은 허술하다.
  -> 생성자 연쇄(constructor chaining)와 살짝 비슷한 매커니즘이다.
  clone이 super.clone을 호출하지는 것이 아닌 생성자를 호출해 얻은 인스턴스를 반환해도 컴파일러는 오류를 내지 않는다.
  하지만, 이 클래스의 하위 클래스에서 super.clone을 호출한다면 잘못된 객체가 만들어져, 결국 하위 클래스의 clone은 제대로 동작하지 않게 된다.
  
  ```java
  @Override
	public PhoneNumber clone() {
		// TODO Auto-generated method stub
		try {
			return (PhoneNumber)super.clone();
		} catch(CloneNotSupportedException e) {
			throw new AssertionError(); // 일어날 수 없는 일이다.
		}
	}
  ```
  - 먼저, super.clone을 호출한다. 그렇게 얻은 객체는 원본의 완벽한 복제본일 것이다. 또한 객체의 필드가 기본타입이거나 불변 객체를 참조한다면 이 객체는 우리가 원하는 완벽한 복제 상태다.
  - 공변 반환 타이핑(covariant return typing)으로 Object를 리턴하는 것이 아닌 PhoneNumber 객체를 리턴하도록 했다.
  - try-catch로 감싼 이유는 Object의 clone 메서드에서 CloneNotSupportedException 예외를 던질 가능성이 있기 때문이다. 하지만 비검새 예외였어야 했다.

  - 가변 객체를참조하는 경우  
  ``` java
  public class Stack {
	
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
      this.elements = new Object[this.DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
      ensureCapacity();
      elements[size++] = e;
    }

    public Object pop() {
      if(size == 0)
        throw new EmptyStackException();
      Object result = elements[--size];
      elements[size] = null; // 다쓴 객체 참조 해제
      return result;
    }

    private void ensureCapacity() {
      if(elements.length == size)
        elements = Arrays.copyOf(elements, 2 * size + 1);
    }
  }
  ```
    -> clone을 재정의하지 않은 체 clone을 사용하면, 복제할 당시에는 올바른 값을 갖겠지만, elements 필드는 원본 Stack 인스턴스와 똑같은 배열을 참조하기 때문에 수정하면 다른 하나도 수정되어 불변식을 해치게 된다.
    -> clone 메서드는 사실상 생성자와 같은 효과를 낸다. 즉, clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다.
    
    ```java
    @Override
    public Stack clone() {
      // TODO Auto-generated method stub
      try {
        Stack cloneStack = (Stack)super.clone();
        cloneStack.elements = this.elements.clone();
        return cloneStack;
      } catch(CloneNotSupportedException e) {
        throw new AssertionError();
      }
    }
    ```
    -> 만약 elements가 final로 선언되었다면 앞서의 방식은 작동하지 않는다. final 필드는 새로운 값을 할당할 수 없기 대문이며, 이는 근본적인 문제로, 직렬화와 마찬가지로 Cloneable 아키텍처는 '가변 객체를 참조하는 필드는 final로 선언하라'는 일반 용법과 충돌한다.
    
  - 재귀적 호출로 clone을 구현하는 방법이 충분하지 않을 때가 있다.
    -> 깊은 복사(deep copy)를 해야할 때가 있지만 재귀호출을 사용하게 되면 성능에 문제가 있고, stack overflow가 발생할 수 있다.
    -> deep copy를 재귀 호출 대신 반복자를 써서 순회하는 방향으로 구현하자.
    -> 아니면 리스트를 새로 생성하고, 기존 객체에 원소드를 put / add하여 return 하는 방식으로 구현하자.
    
- clone 재정의할 때에 주의할 점
  - clone 메소드는 사실상 생성자와 같은 효과를 낸다. 즉 clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다.
  - Cloneable을 구현하는 클래스는 clone을 재정의해준다. 이 때 접근제어자를 public으로 바꾸고, 반환형도 쓰기 편하게 자기 타입으로 바꾸고, throws 절도 없애는 것을 추천한다.
  - 상속용 클래스는 implements Cloneable하면 안된다. 이것은 하위클래스에서 정하게 해야한다.
  
- clone을 재정의하는 것보다 복사 생성자와 복사 팩터리로 제공할 수 있다.
  ```java
  public Yum(Yum yum) {///};
  
  public static Yum newInstance(Yum yum) {///};
  ```
  
  - clone 방식의 단점이었던 언어모순적, 위험, 허술한 스펙, final용법과 충돌, 불필요한 checked exception, 형변환 등이 모두 없다.
  
### Item 14. Comparable을 구현할지 고려하라

-> compareTo 는 Object equals와 비슷한 성격을 가지고 잇으며 단순 동치성 비교에 더해 순서까지 비교할 수 있으며 Generic하다.
  ** Collections.sort(list); Arrays.sort(arr); Comparable을 구현한 객체들은 손쉽게 정렬할 수 있다.
  
- compareTo 재구현에 규약
  - 이 객체와 주어진 객체의 순서를 비교한다. 이 객체가 주어진 객체보다 작으면 음의 정수를, 같으면 0을, 크면 양의 정수를 반환하며 비교할 수 없는 객체인 경우 ClassCastException을 던진다.
  - (대칭성) x.compareTo(y) 와 y.compareTo(x) 가 같아야 한다.
  - (추이성) x.compareTo(y) == y.compareTo(z) 이면 x.compareTo(z)도 같아야 한다.
  - 마지막으로, 동치성 테스트가 equals의 결과와 같아야 한다. 
    -> 정렬된 컬렉션들은 동치성을 비교할 때 equals를 사용하는 것이 아닌 compareTo를 사용하기 때문이다.
    
- compareTo 작성요령
  - equals와 비슷하다. 
  - 다른 점.
    - Comparable은 타입을 인수로 받는 제네릭 인터페이스이므로 인자타입을 확인하거나 형변환할 필요가 없다.
    - compareTo 메서드는 각 필드가 동치인지를 비교하는 게 아니라 순서를 비교한다.
    - 객체 참조 필드를 비교하려면 compareTo 메서드를 재귀적으로 호출한다.
    - Comparable을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 한다면 비교자(Comparator)를 대신 사용한다.(비교 연산자 등은 가급적 사용하지 않는다. jdk 1.7부터)
    -> 핵심적인 필드부터 비교해라. 비교결과가 바로나온다면 곧장 return 하자.
    ```java
    public int compareTo(PhoneNumber pn) {
      int result = Short.compare(this.areaCode, pn.areaCode); // 핵심적인 필드 부터 비교후 결과 바로 return;
      if(result != 0) 
        return result;

      result = Short.compare(this.prefix, pn.prefix);
      if(result != 0)
        return result;

      return Short.compare(this.lineNum, pn.lineNum);
    }
    ```
  - Comparator
  ```java
  private static final Comparator<PhoneNumber> COMPARATOR =
      comparingInt((PhoneNumber pn) -> pn.areaCode) 
      .thenComparingInt(pn -> pn.prefix) 
      .thenComparingInt(pn -> pn.lineNum); 
  
  public int compareTo(PhoneNumber pn) { 
    return COMPARATOR.compare(this, pn); 
  }
  ```
