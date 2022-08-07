---
layout: post
title: Effective Java Ch02. 객체 생성과 파괴
summary: Effective Java 3판 공부
author: devhtak
date: '2020-11-02 09:41:00 +0900'
category: Effective Java
---

### Item 00. 목적

- 객체를 만들어야 할 때와 만들지 말아야 할 때를 구분하는 법
- 올바른 객체 생성 방법과 불필요한 생성을 피하는 법
- 제 때 파괴됨을 보장하고 파괴 전에 수행해야 할 정리 작업을 관리하는 요쳥

### Item 01. 생성자 대신 정적 팩터리 메소드를 고려하라

public 생성자를 사용하는 대신 정적 팩터리 메서드를 제공하는 것에 대해 장점과 단점이 존재한다.

- 장점
  
  - 이름을 가질 수 잇다.
    - 생성자에 넘기는 매개변수와 생성자 자체만으로 반환될 객체의 특성을 제대로 설명하지 못한다.
    - 만약, 입력 매개변수들의 순서를 다르게 한 생성자를 새로 추가하는 식??? 좋지 않은 발상이다.
      - 매개변수 별 목적에 맞는 네이밍으로 정적 팩토리 메소드를 생성한다면 사용하기 편하다
    
  - 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.
    - Immutable Class (불변 클래스) 
      - 불필요한 객체 생성을 피하게 해준다.
      - Immutable Class 란? 어떠한 변경도 허용하지 않는다는 의미로 사용
      	- ex) String 객체는 한번 만들어지면 절대 값을 변경할 수 없다. -> 대입할 때에 새로운 객체 생성하게 된다.
    - Singleton
    
  - 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.
    - 반환될 객체의 클래스를 자유롭게 선택할 수 있는 유연성 제공
    - Java 8 부터는 인터페이스가 정적 메서드를 가질 수 없다는 제한이 풀렸다.
    - Java 8 이전에는 Companion Class(동반 클래스) 를 만들어 그 안에 정의하는 것이 관례였다.
    
  - 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
    
  - 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
  
- 단점
  - 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메소드만 제공하면 하위 클래스를 만들 수 없다.
  
  - 정적 펙터리 메서드는 프로그래머가 찾기 어렵다.
    - 문서화 필요
    - 명명 방식
    
    |메서드 명|내용|
    |:---:|:---|
    |from|매개변수를 하나 받아 해당 타입의 인스턴스로 반환하는 형변환 메서드|
    |of|여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드|
    |valueOf|from과 of의 더 자세한 버전|
    |instance 혹은 getInstance|(매개변수를 받는다면) 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지 않는다.|
    |create 혹은 newInstance|instance 혹은 getInstance와 같지만, 매번 새로운 인스턴스를 생성해 반환함을 보장한다.|
    |getType|getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 사용. "Type" 부분에는 반환할 객체의 타입 입력|
    |newType|newInstance와 같으나 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의. "Type" 부분에는 반환할 객체의 타입 입력|
    |type|getType과 newType의 간결한 버전|
     
### Item 02. 생성자에 매개변수가 많다면 빌더를 고려하라.

- 점층적 생성자 패턴 (telescoping constructor pattern)
  - 필수 매개변수만 받는 생성자, 1개 매개변수만 받는 생성자 .... 모든 매개변수를 받는 생성자 등 늘려가는 방식
  - 점층적 생성자 패턴을 사용할 수 있지만, 매개변수 개수가 많아지면 클라이언트 코드를 작성하거나 가독하기 어렵다.

- 자바 빈즈 패턴 (Java Beans Pattern)
  - setter 메서드를 호출해 원하는 매개변수의 값을 설정하는 방식
  - 점층적 생성자 패턴에 단점은 사라졌다
  - 하지만, 객체 하나를 만들기 위해서는 여러 메서드를 호출해야 하고, 객체가 완전히 생성되기 전까지 일관성이 무너진 상태에 놓이게 된다.
  - 일관성이 무너지는 문제 때문에 자바빈즈 패턴에서는 클래스를 불변으로 만들 수 없다.

- 빌더 패턴 (Builder Pattern)
  - 유효성 검사 생략 
    - 입력 매개변수 검사 및 build 메서드가 호출하는 생성자에서 여러 매개변수에 걸친 불변식 검사가 필요하다. 
    - 어떤 매개변수가 잘못되었는 지 알려줄 때에는 new IllegalArgumentsException 사용  
  - 가급적 Lombok의 @Builder 사용하여 Builder pattern 구현
  - 생성자나 정적 팩터리가 처리해야 할 매개변수가 많다면 빌더 패턴을 선택하는 것이 좋다. 매개변수 중 다수가 필수가 아니거나 같은 타입이면 더욱 그렇다.
    - 빌더는 점층적 생성자보다 클라이언트 코드를 읽고 쓰기가 훨씬 간결하고, 자바 빈드보다 훨씬 안정적이다. 다만, Builder 객체를 만드는 비용 발생
  
  예제)
  ```java
  public class Account {
	
    private Long id;
    private String nickname;
    private String password;

    public static class AccountBuilder {
      private Long id;
      private String nickname;
      private String password;

      public AccountBuilder() {}

      public AccountBuilder(Long id, String nickname, String password) {
        this.id = id;
        this.nickname = nickname;
        this.password = password;
      }

      public AccountBuilder id(Long id) {
        this.id = id;
        return this;
      }

      public AccountBuilder nickname(String nickname) {
        this.nickname = nickname;
        return this;
      }

      public AccountBuilder password(String password) {
        this.password = password;
        return this;
      }

      public Account build() {
        return new Account(this);
      }
    }

    public Account(AccountBuilder builder) {
      this.id = builder.id;
      this.nickname = builder.nickname;
      this.password = builder.password;
    }
  }
	```

### Item 03. private 생성자나 열거 타입으로 싱글턴임을 보장하라

- Signleton이란? 인스턴스를 오직 하나만 생성할 수 있는 클래스
- 클래스를 signleton 으로 만들면, 이를 사용하는 클라이언트를 테스트하기가 어려워 진다.

- public static final 또는 private static final + getter 을 활용한 signleton 구현
```java
public class Elvis {
	public static final Elvis INSTNACE = new Elvis();
	private Elvis(){}
}
```
  - 해당 클래스가 싱글턴임을 API에서 명백히 드러난다.
  - 코드가 간결하다.
```java
public class Presley {
	private static final Presley INSTANCE = new Presley();
	private Presley(){}

	public static Presley getInstance() {
		return INSTANCE;
	}
}
```
  - API를 변경하지 않고도 singleton을 변경할 수 있다.
  - 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다는 점
  - 정적 팩터리의 메서드 참조를 공급자로 사용할 수 있다는 점

- enum을 활용한 singleton 구현
```java
public enum Elvis {
	INSTNACE;
}
```
  - public 방식과 비슷하지만, 더 간결하며, 추가 노력없이 직렬화할 수 있다.
  - 대부분 상황에서는 원소가 하나뿐인 열거 타입이 singleton을 구현하는 데 가장 좋은 방법이 된다.
    - 다만, 만들려는 singleton이 Enum외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다. 
    - 열거 타입이 다른 인터페이스를 구현하도록 선언할 수 있다.
	
### Item 04. 인스턴스화를 막으려거든 private 생성자를 사용하라

- 정적 메서드와 정적 필드만을 담은 클래스
- java.lang.Math, java.util.Arrays 처럼 기본 타입 값이나, 배열 관련 메서드들을 모아놓거나 java.util.Collections처럼 특정 인터페이스를 구현하는 객체를 생성해주는 정적 메서드를 모아놓는 경우 
- final class와 관련한 메서드를 모아놓는 경우
  - 기본 생성자를 명시하지 않으면 컴파일러가 자동으로 기본 생성자를 만들어 주기 때문에 private 생성자가 필요하다.
  - 추상 클래스로 만드는 것 또한 상속하여 인스턴스화 하면 가능하기 때문에 private 생성자가 필요하다.
```java
public class UtilityClass {
	// 기본 생성자가 만들어지는 것을 막는다. (인스턴스화 방지용)
	private UtilityClass() {
		throw new AssertionError();
	}
}
```
  - 해당 방식은 private 생성자밖에 없기 때문에 상속을 불가능하게 막아주는 효과도 있다.

### Item 05. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라.

- 사용하는 자원에 따라 동작이 달라지는 클래스는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다.
- 대신, 클래스가 여러 자원 인스턴스를 지원해야 하며, 클라이언트가 원하는 자원을 사용해야 한다. 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식
```java
public class SpellChecker {
	private final Lexicon dictionary;
	
	public SpellChecker(Lexicon dictionary) {
		this.dictionary = Objects.requireNonNull(dictionary);
	}
}
```
  - 자원 팩터리를 넘겨주는 방식이 있다. : Factory method pattern
  - Supplier<T> 인터페이스가 팩터리를 표현한 완벽한 예 
    - (참고: https://m.blog.naver.com/PostView.nhn?blogId=writer0713&logNo=221590159146&proxyReferer=https:%2F%2Fwww.google.com%2F ) 

### Item 06. 불필요한 객체 생성을 피하라

- 똑같은 기능의 객체를 매번 생성하기 보다는 객체 하나를 재사용하는 편이 나을 때가 많다.
- 특히, 불별 객체는 언제든 재 사용할 수 있다. -> 정적 팩터리 메서드를 제공하는 불변 객체에 경우 해당 방법을 사용하여 불필요한 객체 생성을 막을 수 있다.
- auto boxing : 기본 타입과 박싱된 기본 타입의 구분을 흐려주지만 완전히 사라지는 것은 아니다.
  - 객체를 생성하는 것에 부담감을 느끼는 것이 아닌 불필요한 생성을 지양하자는 것

```java
public class RomanNumeral {
	/*
	public static boolean isRomanNumeral(String s) {
		return s.matches("^(?=.)M*(C[MD]|D?C{0,3}(X[CL]|L?X{0,3})(I[XV]|V?I{0,3}$"); 
	}
	*/
	
	private static final Pattern ROMAN = Pattern.compile("^(?=.)M*(C[MD]|D?C{0,3}(X[CL]|L?X{0,3})(I[XV]|V?I{0,3}$");
	
	public static boolean isRomanNumeral(String s) {
		return ROMAN.matcher(s).matches();
	}
}
```
- 주석처리된 부분을 반복 사용하면 같은 Pattern 인스턴스를 계속 생성한다.
- 아래와 같이 정적변수로 하나 만들어주면 계속 재사용이 가능하다. 

### Item 07. 다 쓴 객체를 해제하라.

- 다 쓴 참조를 여전히 가지고 있는 경우
  - null 처리(참조 해제)
  - null처리는 예외적인 경우여야 한다. 다 쓴 참조를 해제하는 방법 중 가장 좋은 방법은 그 참조를 담은 변수를 유효 범위 밖으로 밀어내는 것

- 캐시 
  - 객체 참조를 캐시에 넣은 후 그 객체를 다 쓴 뒤에도 한참을 그냥 두는 일
  - 해법은 키를 참조하는 동안만 엔트리가 살아있는 캐시가 필요한 경우 WeakHashMap을 사용해 캐시 생성

- 리스너 혹은 콜백
  - 콜백을 등록만 하고 명확히 해지하지 않는다면 콜백이 계속 쌓여갈 것이다.
  - 해법은 약한 참조(Weak reference)로 저장하면 가비지 컬렉터가 즉시 수거한다.

### Item 08. finalizer와 cleaner 사용을 피하라

- 객체 소멸자: finalizer 
  - 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요하다.

- 객체 소멸자: cleaner
  - finalizer보다 덜 위험하지만, 여전히 예측할 수 없고, 느리고, 일반적으로 불필요하다.

### Item 09. try-finally 보다는 try-with-resources 를 사용하라

- 라이브러리 중 close를 호출해 직접 닫아주어야 하는 자원이 많다.
  - 자원을 닫지 않으면 성능 문제로 이어진다.

- try-finally를 활용한 자원 회수 방법
```java
// 자원 회수 방법 중 try-catch-finally는 최선의 방책이 아니다.
public static String firstLineOfFile(String path) throws IOException {
	BufferedReader br = new BufferedReader(new FileReader(path));

	try {
		return br.readLine();
	} finally {
		br.close();
	}
}

// 자원이 둘 이상이면 너무 드러워 진다.
public static void copy(String src, String dist) throws IOException {
	InputStream in = new FileInputStream(src);

	try {
		OutputStream out = new FileOutputStream(dist);
		try {
			byte[] buf = new byte[BUFFER_SIZE];
			int n;
			while( (n = in.read(buf)) >= 0) {
				out.write(buf, 0, n);
			}
		} finally {
			out.close();
		}
	} finally {
		in.close();
	}
}
```
  - try-finally 를 사용하면 닫아야 하는 자원이 많을 수록 소스가 더러워진다.
  - 예외는 try 블록 뿐만이 아닌 finally 블록에서도 발생할 수 있는데, finally 블록에서 발생하면 try에서 발생한 예외를 찾기 어려워 진다.

```java
public static String firstLineOfFile(String path) throws IOException {
	try(BufferedReader br = new BufferedReader(new FileReader(path))) {
		return br.readLine();
	}
}

public static void copy(String src, String dist) throws IOException {
	try(InputStream in = new FileInputStream(src); OutputStream out = new FileOutputStream(dist)) {
		byte[] buf = new byte[BUFFER_SIZE];
		int n;
		while( (n = in.read(buf)) >= 0) {
			out.write(buf, 0, n);
		}
	}
}
```

  - 간결하기 때문에 가독성이 높다.
  - 만약 firstLineOfFile 에서 close 도 중 오류가 발생하면 readLine오류에서 발생한 예외가 기록된다.
  - 물론 catch 절도 사용할 수 있다.
  - try-finally를 사용해야 할 예외는 없다.
