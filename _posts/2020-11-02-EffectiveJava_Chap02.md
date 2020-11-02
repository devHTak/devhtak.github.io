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
  
  - 이름을 가질 수 잇다.
    - 생성자에 넘기는 매개변수와 생성자 자체만으로 반환될 객체의 특성을 제대로 설명하지 못한다.
    - 만약, 입력 매개변수들의 순서를 다르게 한 생성자를 새로 추가하는 식??? 좋지 않은 발상이다.
    -> 매개변수 별 목적에 맞는 네이밍으로 정적 팩토리 메소드를 생성한다면 사용하기 편하다
    
  - 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.
    - Immutable Class (불변 클래스) -> 불필요한 객체 생성을 피하게 해준다.
      -- Immutable Class 란? 어떠한 변경도 허용하지 않는다는 의미로 사용, ex) String 객체는 한번 만들어지면 절대 값을 변경할 수 없다. -> 대입할 때에 새로운 객체 생성하게 된다.
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
  - 유효성 검사 생략 ->입력 매개변수 검사 및 build 메서드가 호출하는 생성자에서 여러 매개변수에 걸친 불변식 검사가 필요하다. 어떤 매개변수가 잘못되었는 지 알려줄 때에는 new IllegalArgumentsException 사용  
  - 가급적 Lombok의 @Builder 사용하여 Builder pattern 구현
  - 생성자나 정적 팩터리가 처리해야 할 매개변수가 많다면 빌더 패턴을 선택하는 것이 좋다. 매개변수 중 다수가 필수가 아니거나 같은 타입이면 더욱 그렇다.
  => 빌더는 점층적 생성자보다 클라이언트 코드를 읽고 쓰기가 훨씬 간결하고, 자바 빈드보다 훨씬 안정적이다. 다만, Builder 객체를 만드는 비용 발생
  
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
```
	public class Elvis {
		public static final Elvis INSTNACE = new Elvis();
		private Elvis(){}
	}
```
  - 해당 클래스가 싱글턴임을 API에서 명백히 드러난다.
  - 코드가 간결하다.
```
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
```
	public enum Elvis {
		INSTNACE;
	}
```
  - public 방식과 비슷하지만, 더 간결하며, 추가 노력없이 직렬화할 수 있다.
  - 대부분 상황에서는 원소가 하나뿐인 열거 타입이 singleton을 구현하는 데 가장 좋은 방법이 된다.
  	다만, 만들려는 singleton이 Enum외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다. -> 열거 타입이 다른 인터페이스를 구현하도록 선언할 수 있다.
	
### Item 04. 인스턴스화를 막으려거든 private 생성자를 사용하라

- 정적 메서드와 정적 필드만을 담은 클래스
- java.lang.Math, java.util.Arrays 처럼 기본 타입 값이나, 배열 관련 메서드들을 모아놓거나 java.util.Collections처럼 특정 인터페이스를 구현하는 객체를 생성해주는 정적 메서드를 모아놓는 경우 
- final class와 관련한 메서드를 모아놓는 경우
-> 기본 생성자를 명시하지 않으면 컴파일러가 자동으로 기본 생성자를 만들어 주기 때문에 private 생성자가 필요하다.
-> 추상 클래스로 만드는 것 또한 상속하여 인스턴스화 하면 가능하기 때문에 private 생성자가 필요하다.
```
	public class UtilityClass {
		// 기본 생성자가 만들어지는 것을 막는다. (인스턴스화 방지용)
		private UtilityClass() {
			throw new AssertionError();
		}
	}
```
-> 해당 방식은 private 생성자밖에 없기 때문에 상속을 불가능하게 막아주는 효과도 있다.

### Item 05. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라.

- 

  
    
