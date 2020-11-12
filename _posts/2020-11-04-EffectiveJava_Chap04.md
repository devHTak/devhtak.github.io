---
layout: post
title: Effective Java Ch04. 클래스와 인터페이스
summary: Effective Java 3판 공부
author: devhtak
date: '2020-11-10 09:41:00 +0900'
category: java
---

### Item 15. 클래스와 멤버의 접근 권한을 최소화하라

: 잘 설계된 컴포넌트는 모든 내부 구현을 완벽히 숨겨, 구현과 API를 깔끔히 분리한다.

- 정보 은닉의 장점
  - 시스템 개발 속도를 높인다. 여러 컴포넌트를 병렬로 개발할 수 있기 때문
  - 시스템 관리 비용을 낮춘다. 각 컴포넌트를 더 빨리 파악하여 디버깅할 수 있고, 다른 컴포넌트로 교체하는 부담도 적다
  - 성능 최적화에 도움을 준다. 완성된 시스템을 프로파일링해 최적화할 컴포넌트를 정한 다음, 다른 컴포넌트에 영향을 주지 않고 해당 컴포넌트만 최적화할 수 있기 때문이다.
  - 소프트웨어 재사용성을 높인다. 외부 의존도가 낮은 컴포넌트라면 유용하게 쓰일 가능성이 높다.
  - 큰 시스템을 제작하는 난이도를 낮춘다. 
  
- 정보 은닉 방법
  - 선언된 위치
  - 접근 제한자 (private, package-private, protected, public)
    - 기본 원칙: 모든 클래스와 멤버의 접근성을 가능한 한 좁혀야 한다!
    - private: 멤버를 선언한 톱레벨 클래스에서만 접근할 수 있다.
    - package-private: 멤버가 소속된 패키지 안의 모든 클래스에서 접근할 수 있다. (접근 제한자를 명시하지 않는 경우의 수준, 단, 인터페이스의 멤버는 기본적으로 public 적용)
    - protected: package-private 범위를 포함하며 해당 클래스의 하위 클래스에도 접근할 수 있다.
    - public: 모든 곳에서 접근할 수 있다.
    - 제약: 상위 클래스의 메서드를 재정의할 때는 접근 수준을 상위 클래스에보다 좁게 설정할 수 없다.
    - 코드를 테스트하려는 목적으로 접근 범위를 넓힌다 하더라도 private에서 package-private 수준까지 가능하다.
    - public 클래스의 인스턴스 필드는 되도록 public이 아니어야 한다. public 가변 필드를 갖는 클래스는 일반적으로 스레드에 안전하지 않다. 불변 객체를 참조하더라도 문제가 발생할 수 있다.
    -> 예외로는 publis static final(상수) 필드로 공개해도 좋다. 하지만 배열인 경우(변경 가능), 해당 필드를 반환하는 접근자 메서드를 제공해서는 안된다.
    ```java
    public static final Thing[] PRIVATE_VALUES = {...}; //안좋은 방법 변경 가능
    // 해결방법 1. public을 private으로 두고, public 리스트 생성
    private static final Thing[] PRIVATE_VALUES = {...};
    public static final List<Thing> VALUES = Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
    // 해결방법 2. 배열을 private으로 만들고, 그 복사본을 반환하는 public 메서드를 추가
    private static final Thing[] PRIVATE_VALUES={...};
    public static final Thing[] values() {
    	return PRIVATE_VALUES.clone();
    }
    ```

### Item 16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

- public 클래스에 경우 인스턴스를 private으로 선언하고, getter/setter 로 접근하도록 한다.
- package-private, private 중첩 클래스라면 데이터 필드를 노출한다 해도 문제가 없다.
  - 클래스 내부 표현에 묶이기는 하나, 패키지 안에서만 동작하는 코드일 뿐이며, 패키지 바깥 코드는 전혀 손대지 않고도 데이터 표현 방식을 바꿀 수 있다. 
  - private 중첩 클래스의 경우라면 수정 범위가 더 좁아져서 이 클래스를 포함하는 외부 클래스까지로 제한된다.
  
### Item 17. 변경 가능성을 최소화하라.

- 불변 클래스
  - String, 기본타입의 박싱된 클래스, BigInteger, BigDecimal
  - 규칙
    - 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다.
    - 클래스를 확장할 수 없도록 한다. 클래스에 final을 붙이는 방법과 추가적인 방법이 있다.
    - 모든 필드를 final로 선언한다.
    - 모든 필드를 private으로 선언함으로써 가변 객체에 대한 접근을 제어한다.
    - 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.
  
  ```java
  public final class Complex {	
	private final double re;
	private final double im;

	public Complex(double re, double im) {
		this.re = re; this.im = im;
	}

	public double realPart() { return re; }
	public double imaginaryPart() { return im; }

	public Complex plus(Complex c) {
		return new Complex(c.re + this.re, c.im + this.im);
	}

	public Complex minus(Complex c) {
		return new Complex(this.re - c.re, this.im - c.im);
	}

	public Complex times(Complex c) {
		return new Complex(this.re * c.re - this.im * c.im, this.re * c.im + this.im * c.re);
	}

	public Complex dividexBy(Complex c) {
		double tmp = c.re * c.re + c.im * c.im;
		return new Complex((this.re * c.re + this.im * c.im) / tmp, (this.im * c.re - this.re * c.im) / tmp);
	}
	@Override public boolean equals(Object o) {
		// TODO Auto-generated method stub
		if( o == this ) return true;
		if( !(o instanceof Complex) ) return false;
		Complex c = (Complex)o;
		return Double.compare(c.re, this.re) == 0 && Double.compare(c.im, this.im) == 0;
	}

	@Override public int hashCode() {
		// TODO Auto-generated method stub
		return 31 * Double.hashCode(re) + Double.hashCode(im);
	}

	@Override public String toString() {
		// TODO Auto-generated method stub
		return "(" + re + " + " + im + "i)";
	}
  }
  
  ```
  
  - 사칙연산 메서드들이 인스턴스 자신을 수정하는 것이 아닌, 새로운 Complex 인스턴스를 만들어 반환하였다.
  - 특징
    - 불변 객체는 단순하다. 생성 시점부터 파괴될 때까지 그대로 간직한다.
    - 불변 객체는 근본적으로 스레드 안전하여 따로 동기화할 필요가 없다.
    - 불변 객체는 안심하고 공유할 수 있고, 불변 객체끼리는 내부 데이터를 공유할 수 있다.
    - 객체를 만들 때 다른 불변 객체들을 구성요소로 사용하면 이점이 많다. Map의 키와, Set의 원소로 쓰기에 좋다. 
    - 불변 객체는 그 자체로 실패 원자성을 제공한다.
    ** 실패 원자성이란, 메서드에서 예외가 발생한 후에도 그 객체는 여전히 유효한 상태여야 한다는 성질
    - 단점은, 값이 다르면 반드시 독립된 객체로 만들어야 한다는 것이다.
  
  - 불변 객체 만드는 법
  ```java
  public class Complex {
	private final double re;
	private final double im;

	private Complex(double re, double im) {
		this.re = re; this.im = im;
	}

	public static Complex valueOf(double re, double im) {
		return new Complex(re, im);
	}
    // 이하 생략
  ```
  - 객체 설계 시 주의점
    - getter가 있다고 해서 무조건 setter를 만들지 말자. 클래스는 필요한 경우가 아니면 불변이어야 한다.
    - 불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄이자. -> 합당한 이유가 없다면 모든 필드는 private final
    - 생성자는 불변식 설정이 모두 완료된, 초기화가 완벽히 끝난 상태의 객체를 생성해야 한다.

### Item 18. 상속보다는 컴포지션을 사용하라

** 구현상속을 말하며, 클래스가 인터페이스를 구현하거나, 인터페이스를 확장하는 등에 인터페이스 상속과는 무관한다.

- 상속의 주의점
  - 메서드 호출과 달리 상속은 캡슐화를 깨트린다.
    - 상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작이 이상할 수 있다.
      ex) HashSet을 상속한 클래스를 구현할 때 add 메소드에서 사이즈를 +1씩 늘리고, addAll 메서드에서 인자로 받은 Collection에 크기만큼 늘리도록 overrindg을 한다면?
      예기치 못한 size를 얻을 수 있다. (이미 addAll에서 add를 호출하도록 되어 있기 때문에 2번 size를 늘리는 것)
    - 다음 릴리스에서 상위 클래스에 새로운 메서드를 추가했을 때 하위 클래스에서 허용되지 않는 일이 벌어날 수 있다.
      ex) Hashtable과 Vector를 컬렉션 프레임워크에 포함시키자 이와 관련한 보안 구멍들을 수정해야 하는 사태가 발생했다.
    - 메서드를 overriding을 하는 것이 아닌 새로운 메서드를 추가해도 문제가 될 수 있다.
      ex) 하위 클래스에 추가한 메서드와 시그니처가 같고 반환 타입이 다르다면 컴파일 오류가 발생할 수 있고, 상위 클래스의 메서드가 요구하는 규약을 만족하지 못할 수도 있다.
- 상속을 피하는 법
  - 기존 클래스를 확장하는 대신, 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 하자.
  ** 기존 클래스가 새로운 클래스의 구성요소로 쓰인다는 뜻에서 이러한 설계를 Composition(컴포지션)이라 한다.
  ** 새 클래스의 인스턴스 메서드들은 (private 필드로 참조하는) 기존 클래스의 대응하는 메서드를 호출해 그 결과를 반환하는 데 해당 방식은 전달(forwarding)이라 부르며, 새 클레스의 메서드들은 전달 메서드(forwarding method)라 한다.
  ```java
  // Wrapper Class - 상속대신 컴포지션을 사용
  public class InstrumentedSet<E> extends ForwardingSet<E> {	
	private int addCount = 0;
	public InstrumentedSet(Set<E> s) {
		super(s);
	}
	@Override public boolean add(E e) {
		this.addCount += 1; 
		return super.add(e);
	}
	@Override public boolean addAll(Collection<? extends E> c) {
		this.addCount += c.size();
		return super.addAll(c);
	}
	public int getAddCount() {
		return this.addCount;
	}
  }  
  ```
  ```java
  public class ForwardingSet<E> implements Set<E> {
	private final Set<E> s;
	public ForwardingSet(Set<E> s) { this.s = s; }

	// 기존 메서드 구현
	public void clear() {...}
	...
   }
  ```
          
### Item 19. 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라

- 상속용 클래스를 생성할 때 고려할 점
  - 상속용 클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는지(자기사용) 문서로 남겨야 한다.
  (재정의 가능 메서드란? public, protected 메서드 중 final이 아닌 모든 메서드)
  - "Implementation Requirements"로 시작되는 절을 볼 수 있는데, 이 절은 메서드 주석에 @implSpec 태그를 붙여주면 자바독 도구가 생성해준다.
    -> @implSpec을 활성화하기 위해서는 명령줄 매개변수로 -tag "implSpec:a:Implementation Requirements:"를 지정해주면 된다.
  - 문서를 남기는 것 + 효율적인 하위 클래스를 큰 어려움 없이 만들수 있게 해야 한다.
    -> 클래스의 내부 동작 과정 중간에 끼어들 수 있는 훅(hook)을 잘 선별하여 protected 메서드 형태로 공개해야 할 수도 있다.
  - 상속용 클래스를 테스트하기 위해서는 하위클래스를 만드는 방법 밖에 없기 때문에 배포 전에 하위 클래스를 만들어 검증해야 한다.
  - 상속용 클래스의 생성자는 직/간접적으로 재정의 가능 메서드를 호출해서는 안된다.
  ```java
  public class SuperClass {
	public SuperClass() { this.overrideMe(); }
	public void overrideMe() {///}
  }
  public final class SubClass extends SuperClass {
	private final Instant instant;
	public SuperClass() { instant = Instant.now(); }
	@Override public void overrideMe() { System.out.println(instant); }

	public static void main(String[] args) {
		SubClass sub = new SubClass();
		sub.overrideMe();
	}
  }	
  ```
    - instant가 2번 실행되지 않고, 첫번째는 null을 출력한다. 인스턴스 필드가 초기화되기 전에 SuperClass 생성자에 있는 overrideMe()가 호출되기 때문이다.
  
  - Clonable 클래스를 구현한 클래스는 확장하기 어렵다.
    - clone과 readObject는 모두 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안된다.
  - Serializable을 구현한 상속용 클래스가 readResolve나 writeReplace 메서드를 갖는다면 이 메서드들은 private아닌 protected를 선언해야 한다.
  
### Item 20. 추상 클래스보다는 인터페이스를 우선하라

** 자바가 제공하는 다중 구현 메커니즘은 인터페이스와 추상 클래스가 있다.
** 자바 8 부터 인터페이스도 default method를 제공할 수 있게 되어, 인터페이스와 추상 클래스 모두 구현형태로 제공할 수 있다.
** 추상 클래스와 인터페이스의 가장 큰 차이는 추상 클래스에서 정의한 타입을 구현하는 클래스는 반드시 추상 클래스의 하위 클래스여야 한다. 자바는 단일 상속만 가능하므로 제약 사항이 될 수 있다.

  - 추상 클래스가 아닌 인터페이스를 선택해야 하는 이유
    - 기존 클래스에 손쉽게 새로운 인터페이스를 구현해 넣을 수 있다.
    - 인터페이스는 믹스인(mixin) 정의에 안성 맞춤이다.
    	- 믹스인이란 클래스가 구현할 수 있는 타입으로 기존 주된 타입 외에도 특정 선택적 행위를 제공한다고 선언하는 효과를 준다. (혼합(mixed in)한다고 해서 믹스인)
    - 인터페이스로는 계층구조가 없는 타입 프레임워크를 만들 수 있다.

  - Template Method Pattern
    - 인터페이스와 추상 골격 구현 클래스를 함께 제공하는 식으로 인터페이스와 추상 클래스의 장점을 모두 취하는 방법
    - 인터페이스로는 타입을 정의하고, 필요하면 디폴트 메서드도 제공한다. 그리고 골격 구현 클래스는 나머지 메서드들까지 구현한다.
    - 관레상 인터페이스 이름을 Interface라고 한다면, 골격 구현 클래스의 이름은 Abstractinterface로 짓는다.
    - 골격 구현 클래스는 추상 클래스처럼 구현을 도와주는 동시에, 추상 클래스로 타입을 정의할 때 따라오는 제약으로부터 자유롭다.
    - 구조상 골격 구현 클래스를 구현하지 못하더라도 default method에 대한 이점을 여전히 누릴 수 있다.
    - 골격 구현은 기본적으로 상속해서 사용하는 걸 가정하므로 설계 및 문서화 지침을 모두 따라야 한다.

### Item 21. 인터페이스는 구현하는 쪽을 생각해 설계하라

** 자바 8 전에는 기존 구현체를 깨트리지 않고, 인터페이스에 메서드를 추가할 방법이 없었다.
** 자바 8 부터는 기존 인터페이스에 메서드를 추가할 수 있도록 디폴트 메서드가 있다.
** 디폴트 메서드를 추가된 이유는 람다를 활용하기 위해서다.

  - 디폴트 메서드의 단점
    - 구현체에 디폴트 메서드를 재정의하지 않는 경우 해당 메서드를 그대로 물려받게 되며, 규약을 지키지 못할 수 있다.
    - 기존 구현체에 런타임 오류를 일으킬 수 있다.
    
  - 디폴트 메서드라는 도구가 생겼더라도! 인터페이스를 설계할 때는 주의해야 한다!

### Item 22. 인터페이스는 타입을 정의하는 용도로만 사용하라

** 인터페이스는 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할을 한다.

- 상수 인터페이스
   ```java
   public interface PhysicalConstants {
  	// 잘못된 사용
  	static final double AVOGADROS_NUMBER = ...;
	static final double BOLTZMAN_CONSTANT = ...;
	static final double ELECTRON_MASS = ...;
   }
   ```
  - 클래스 내부에서 사용하는 상수는 외부 인터페이스가 아니라 내부 구현에 해당한다. 위와 같이 구현하면 API로 노출하는 행위다.
  - 특정 클래스나 인터페이스와 강하게 연관된 상수라면 해당 클래스나 인터페이스에 구현하거나 열거 타입으로 공개, 인스턴스화 할 수 없는 유틸리티 클래스로 공개하자.
  ```java
  public class PhysicalConstants {
	// 상수 유틸리티 클래스
	private PhysicalConstants(){} // 인스턴스화 방지
	public static final double AVOGADROS_NUMBER = ...;
	public static final double BOLTZMAN_CONSTANT = ...;
	public static final double ELECTRON_MASS = ...;
  }
  ```
    
### Item 23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라

** 태그달린 클래스란? 두가지 이상의 의미를 표현할 수 있으며, 그 중 현재 표현하는 의미를 태그 값으로 알려주는 클래스

- 태그 달린 클래스의 단점
  - 가독성, 메모리 사용, 열거 타입 선언, 태그 필드, switch 문 등 쓸데없는 코드가 많다.
  - 장황하고 오류를 내기 쉽고, 비효율적이다.
  - 클래스 계층 구조를 어설프게 흉내낼 뿐이다.
  
- 해결책? 클래스 계층구조!
```java
abstract class Figure {
	abstract double area();
}
class Circle extends Figure {
	final double radius;
	Circle(double radius) { this.radius = radius; }
	@Override double area() {
		// TODO Auto-generated method stub
		return Math.PI * ( radius * radius );
	}	
}
class Rectangle extends Figure {
	final double length;
	final double width;
	Rectangle(double length, double width) { this.length = length; this.width = width; }
	@Override double area() {
		// TODO Auto-generated method stub
		return 0;
	}
}
```

### Item 24. 멤버 클래스는 되도록 static으로 만들어라

** 중첩클래스란 다른 클래스 안에 정의된 클래스
** 중첩클래스는 자신을 감싼 클래스에서만 사용 가능하며, 그 외의 쓰임새가 있다면 톱레벨 클래스로 만들어야 한다.
** 중첩클래스의 종류는 정적 멤버 클래스, (비정적) 멤버 클래스, 익명 클래스, 지역 클래스이며, 정적 멤버 클래스를 제외한 나머지는 내부 클래스이다.

- 정적 멤버 클래스 (Static neted class)
  - 다른 클래스 안에 선언되고, 바깥 클래스의 private 멤버에도 접근할 수 있다는 점만 제외하고는 일반 클래스와 독같다.
  ex) Calculator.Operation.PLUS, Calculator.Operation.MINUS ..
  
- (비정적) 멤버 클래스
  - 정적 멤버 클래스와의 차이는 static 여부
  - 밖에 있는 클래스는 내부클래스를 멤버변수처럼 사용할 수 있다. 사용하려면 new 생성자로 생성해야 한다.
  - 내부 클래스는 바깥 클래스의 자원을 직접 사용할 수 없다.
  - 멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static을 붙여서 정적 멤버 클래스로 만들자
    - 바깥 클래스에 대한 숨은 참조(바깥클래스.this)를 가져올 수 있는데 비용이 들어가며 Garbage Collector가 제때 수거하지 못해 메모리 누수가 발생할 수 있다.
  
- 익명 클래스
  - 익명클래스는 바깥 클래스의 멤버가 아니며, 바깥 클래스를 new 생성자로 생성하는 경우 생성되며, 동시에 부모 클래스를 상속받아 내부에서 오버라이딩해서 사용한다.
  - 매개변수로 사용할 수 있다.
  - 익명 클래스 내부의 변수나 메소드는 익명클래스의 밖에서 사용 불가하다.
  - 내부에 생성자를 작성할 수 없으며, 바깥 클래스의 자원은 final이 붙은 것만 사용할 수 있다.
  
- 지역 클래스
  - 메소드 내부에 클래스를 정의하는 경우로 메소드 내의 지역변수처럼 사용 가능하다
  - 메소드 내부에서 new 생성자로 사용해야 한다. 메소드 밖에서는 사용할 수 없다.
    
### Item 25. 톱레벨 클래스는 한 파일에 하나만 담으라

- 단점
  - Utensil.java 파일 안에 Utensil class와 Dessert class를 선언하더라도 사용하는 데 아무런 문제가 없다. 하지만 이후 Dessert.java 파일을 생성하면 문제가 발생한다.
  - javac Main.java 또는 javac Main.java Utensil.java를 실행하면 Utensil.java 안에 Desert class가 실행되지만, javac Main.java Dessert.java를 사용하면 Dessert.java 안에 있는 클래스가실행된다.
- 해결책
  - 소스들을 파일로 분리하면 된다.
  - 한 파일에 담고자 한다면 정적 멤버 클래스를 사용하자.
