---
layout: post
title: Effective Java Ch06. 열거타입과 애너테이션
summary: Effective Java 3판 공부
author: devhtak
date: '2020-11-17 19:41:00 +0900'
category: java
---

### ITEM 34. int 상수 대신 열거 타입을 사용하라

** 열거 타입은 일정 개수의 상수 값을 정의한 다음, 그 외의 값은 허용하지 않는 타입니다.

- 정수 열거 패턴(int enum pattern)에 단점
  - 타입 안전을 보장하지 않으며, 표현력도 떨어진다.
  - 프로그램이 깨지기 쉽다.
    - 클라이언트 파일에 그대로 새겨지기 때문에, 상수의 값이 바뀌면 클라이언트도 반드시 다시 컴파일 해야 한다.
  - 정수 상수는 문자열로 출력하기 까다롭다.
  
- 문자열 열거 패턴 (string enum pattern)에 단점
  - 오타 등으로 런타임 버그가 발생한다.
  
- 열거 타입(enum type)의 장점
```java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```
  - 열거 타입 자체는 클래스이며, 상수 하나당 자신의 인스턴스를 만들어 public static final 필드로 공개한다.
  - 싱글턴은 원소가 하나뿐인 열거 타입이라고 할 수 있고, 열거 타입은 싱글턴을 일반화한 형태라고 할 수 있다.
  - 컴파일 타임 타입 안전성을 제공한다.
  - toString 메서드는 출력하기에 적합한 문자열을 제공한다.
  - 임의의 메서드나 필드를 추가할 수 있고 임의의 인터페이스를 구현하게 할 수 도 있다.
  ```java
  public enum Planet {
      MERCURY(3.302e+23, 2.439e6),
      VENUS(4.869e+24, 6.052e6),
      EARTH(5.975e+24, 6.378e6),
      MARS(6.419e+23, 3.393e6),
      JUPITER(1.899e+27, 7.149e7),
      SATURN(4.685e+26, 2.556e7),
      URANUS(8.683e+25, 2.556e7),
      NEPTUNE(1.024e+26, 2.477e7);

      private final double mass;		//질량(단위:킬로그램)
      private final double radius;		// 반지름(단위: 미터ㅏ)
      private final double surfaceGravity; //표면중력(단위: m/s^2)

      //중력상수(단위 : m^3 / kg s^2)
      private static final double G = 6.67300E-11;

      Planet(double mass, double radius) {
          this.mass = mass; this.radius = radius; 
          surfaceGravity = G * mass / ( radius * radius);
      }

      public double mass() { return mass;}
      public double radius() {return radius;}
      public double surfaceGravity() { return surfaceGravity; }

      public double surfaceWeight(double mass) {
          return mass * surfaceGravity; // f= ma
      }
  }
  ```
- 상수별 메서드 구현
  - 상수에서 자신에 맞게 재정의하는 것을 말한다.  
```java
public enum Operation {
    PLUS("+") {
        public double apply(double x, double y){ return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y){ return x - y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y){ return x / y; }
    },
    TIMES("*") {
        public double apply(double x, double y){ return x * y; }
    };   
    public abstract double apply(double x, double y);
    
    private final String symbol;
    Operation(String symbol) { this.symbol = symbol; }
    @Override 
    public String toString() { return symbol; }
}
```
  - 열거 타입에는 상수 이름을 입력받아 그 이름에 해당하는 상수를 반환해주는 valueOf(String) 메서드가 자동 생성된다.
  - toString을 재정의할 때에는 toString이 반환하는 문자열을 해당 열거 타입 상수로 변환해주는 fronString 메서드도 함께 제공하자.
  ```java
  private static final Map<String, Operation> stringtoEnum = Stream.of(values()).collect(toMap(Object::toString, e->e));
  public static Optional<Operation> fronString(String symbol) {
      return Optional.ofNullable(stringToEnum.get(symbol));
  }
  ```

- 전략 열거 타입 패턴
```java
public enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;

    private static final int MINS_PER_SHIFT = 8 * 60; // 하루 8시간
    int pay(int minutesWorked, int payRate) {
        int basePay = minutesWorked * payRate;

        int overtimePay;
        switch(this) {
            case SATURDAY: case SUNDAY: // 주말
                overtimePay = basePay / 2;
                break;
            default: // 주중
                if (minutesWorked <= MINS_PER_SHIFT) {
                    overtimePay = 0	;
                } else {
                  overtimePay = (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
                }
        }
        return basePay + overtimePay;
    }
}
```
  - 상수별 메서드에 열거 타입 상수끼리 코드를 공유하기가 어렵다라는 단점이 있다.
  - 새로운 상수를 추가하려면 그 값을 처리하는 case 문을 추가하여야 한다. 또한 잔업 수당 등에 대한 새로운 형태를 제공하기에는 장황해진다.
  - 이 때 잔업수당 계산을 private 중첩 열거 타입으로 옯기고 PayrollDay 열거 타입의 생성자에서 이를 선택하게 하면 좀 더 안전하고 유연하게 접근할 수 있다. 
```java


```java
public enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY(PayType.WEEKEND), SUNDAY(PayType.WEEKEND);
  	private final PayType payType;
	  PayrollDay(PayType payType) {
		    this.payType = payType;
	  }
    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }
	  enum PayType {
		    WEEKDAY {
			      int overtimePay(int minsWorked, int payRate) {
				        int overtimePay;
				        if (minsWorked <= MINS_PER_SHIFT) {
					          overtimePay = 0;
				        } else {
					        overtimePay = (minsWorked - MINS_PER_SHIFT) * payRate / 2;
				        }
				        return overtimePay;
			      }
		    },
		    WEEKEND {
			      int overtimePay(int minsWorked, int payRate) {
				        return minsWorked * payRate / 2;
			      }
		    };
    		abstract int overtimePay(int mins, int payRate);
    		private static final int MINS_PER_SHIFT = 8 * 60; // 하루 8시간
        int pay(int minutesWorked, int payRate) {
            int basePay = minutesWorked * payRate;
            return basePay + overtimePay(minutesWorked, payRate);
        }
	  }
}
```
- 열거 타입은 필요한 원소를 컴파일타임에 다 알수있는 상수 집합이라면 사용하자.
- 열거 타입에 정의된 사우 개수가 영원히 고정 불변일 필요는 없다.

### ITEM 35. ordinal 메서드 대신 인스턴스 필드를 사용하라

- 열거 타입 상수는 하나의 정숫값에 대응하여 ordinal이란 메서드를 통해 몇 번째 위치인지를 반환한다.
```java
public enum Ensemble {
    SOLO, DUET, TRIO, QUARTET, QUINTET, SEXTET, SEPTET, OCTET, NONET, DECTET;
    public int numOfMusicians() { return ordinal() + 1; }
}
```
- ordinal 메서드는 유지보수가 어렵다.
- index로 사용하다 보니 의미를 맞추기 위해 빈 값들이 추가되거나 해당 의미에 값을 2개 이상 추가할 수 없다.
- 그렇기 때문에 ordinal 메서드를 사용하지 말고 인스턴스 필드에 저장하자.
```java
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5), SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8), NONET(9), DECTET(10);
    private final int numberOfMusicians;
    Ensemble(int numberOfMusicians) { this.numberOfMusicians = numberOfMusicians; }
    public int numberOfMusicians() { return this.numberOfMusicians; }
}
```

### ITEM 36. 비트 필드 대신 EnumSet을 사용하라

- 비트 필드
  - 이전에는 열거한 값들이 집합으로 사용되는 경우 2의 거듭제곱 값(비트 필드)으로 할당한 정수 열거 패턴을 사용해왔다. 
  - 비트 빌드를 사용하면 비트별 연산을 사용해 합집합, 교집합과 같은 집합 연산에 용이하다.
  - 다만, 열거 상수의 단점을 그대로 지니며, 가독성이 떨어지는 추가 단점이 있고, 비트를 사용하므로 기본타입 변경이 필요할 수 있다.
- EnumSet
  - Set 인터페이스를 완벽히 구현하며 내부는 비트 연산으로 되어있다.
  ```java
  public class Text {
      public enum Style { BOLD, ITALIC, UNDERLINE; }
      // EnumSet을 사용하는 것이 좋으며 EnumSet 인스턴스를 건네는 클라이언트 코드다.
      public void applyStyles(Set<Style> styles) { ... }  
  }
  ```
  
### ITEM 37. ordinal 인덱싱 대신 EnumMap을 사용하라

- ordinal 메서드를 배열 인덱스로 사용하면 위험하다.
  - ordinal 메서드를 사용하면 정확한 정수값을 사용한다는 것을 개발자가 보증해야 한다.
- EnumMap
  - 열거 타입을 키로 사용하도록 설계한 Map 구현체
  ```java
  Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);
  for(Plant.LifeCycle lc : Plant.LifeCycle.values()) {
    plantsByLifeCycle.put(lc, new HashSet<>());
  }
  for(Plant p: garden) {
    plantsByLifeCycle.get(p.LifeCycle).add(p);
  }
  ```
  - 안전하지 않은 형변환을 사용하지 않으며, index 접근이 아닌 열거 타입을 key로 사용하기 때문에 안전하다.
- Stream을 활용한 방법
  ```java
  Arrays.stream(garden).collect(groupingBy(p -> p.lifeCycle)); // 첫번째 코드
  Arrays.stream(garden).collect(groupingBy(p-> p.lifeCycle, () -> new EnumMap<>(LifeCycle.class), toSet())); // 두번째 코드  
  ```
  
### ITEM 38. 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

- 열거 타입을 확장하기 위해서는 인터페이스를 사용하면 된다.
```java
public interface Operation {
    double apply(double x, double y);
}

public enum BasicOperation implements Operation {
    PLUS("+") {
        public double apply(double x, double y){ return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y){ return x - y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y){ return x / y; }
    },
    TIMES("*") {
        public double apply(double x, double y){ return x * y; }
    };   
    private final String symbol;
    Operation(String symbol) { this.symbol = symbol; }
    @Override 
    public String toString() { return symbol; }
}

public enum ExtendedOperation implements Operation {
    EXP("^") {
        public double apply(double x, double y) { return Math.pow(x, y); }
    }
    REMAINDER("%") {
        public double apply(double x, double y) { return x % y; }
    }
    
    private final String symbol;
    Extended(String symbol) { this.symbol = symbol; }
     @Override 
    public String toString() { return symbol; }
}
```
  - BasicOperation은 열거 타입이기 때문에 확장이 불가능하지만 Operation을 통해 확장이 가능하다.
  ```java
  public static void main(String[] args) {
      double x = args[0]; double y = args[1];
      test1(ExtendedOperation.class, x, y);
      test2(Arrays.asList(ExtendedOperation.values()), x, y));
  }
  
  private static <T extends Enum<T> & Operation> void test1(Class<T> opEnumType, double x, double y) {
      for(Operation op : opEnumType.getEnumConstants()) { 
          System.out.println("%f %s %f = %f", x, op, y, op.apply(x,y));
      }
  }
  
  private static void test2(Collection<? extends Operation> opSet, double x, double y) {
    for(Operation op : opSet) { 
        System.out.println("%f %s %f = %f", x, op, y, op.apply(x, y));
    }
  }
  ```
    - test1은 <T extends Enum<T> & Operation>을 통해 열거 타입이며 Operation 구현체를 입력받는 것을 명시한다.
    - test2는 와일드 카드 타입 Collection<? extends Operation>을 넘기는 방법이다.

### ITEM 39. 명명 패턴보다 애너테이션을 사용하라

- 명명 패턴 단점
  - jUnit3는 메소드 앞에 test를 붙여 테스트 메서드임을 알려주는 명명패턴을 사용했다.
  - 만약 오타가 발생하면 jUnit은 메서드를 무시한체 지나갔기 때문에 테스트가 실행됐는 지 확인할 수 없다.
  - 매개변수를 전달할 마땅한 방법이 없다.

- Annotation을 사용하자
  - Meta Annotation : Annotation 선언에 다는 Annotation
    - @Retention
    - @Target
  
