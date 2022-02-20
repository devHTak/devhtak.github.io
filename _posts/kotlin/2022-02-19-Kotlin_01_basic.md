---
layout: post
title: Kotlin 기초
summary: The Java
author: devhtak
date: '2022-02-19 14:41:00 +0900'
category: Kotlin
---

#### 코틀린이란 무엇이며 왜 필요한가

##### 코틀린의 주요 특성

- 정적 타입 지정 언어
  - 모든 프로그램 구성 요소의 타입을 컴파일 시점에 알 수 있다
  - 객체의 필드나 메소드를 사용할 때 마다 컴파일러가 타입을 검증
  - Java와 달리 '타입 추론(type inference)' 를 통해 컴파일러가 문맥을 고려해 변수 타입을 결정
    - 모든 변수의 타입을 프로그래머가 직접 명시할 필요가 없음
  - 널이 될 수 있는 타입(nullable type)을 지원
  - 장점
    - 성능: 실행 시점에 어떤 메소드를 호출할지 알아내는 과정이 필요 없어서, 메소드 호출이 빠르다
    - 신뢰성: 컴파일러가 프로그램의 정확성(correctness)를 검증해서 실행시 프로그램이 오류로 중단될 가능성이 적다
    - 유지 보수성: 코드에서 다루는 객체가 어떤 타입에 속하는지 알 수 있기 때문에 처음 보는 코드를 다루기 쉽다
    - 도구 지원: 안전하게 리팩토링 가능, 도구는 더 정확한 코드 완성 기능을 제공 가능
  - 동적 타입 언어란? (ex: 그루비 / JRuby)
    - 타입과 관계 없이 모든 값을 변수에 넣을 수 있다
    - 메소드나 필드 접근에 대한 검증이 실행 시점에 일어난다
    - 장점 : 코드가 짧고, 데이터 구조가 유연
    - 단점 : 컴파일 시 간단한 타입 오류도 체크하지 못하고, 실행시점에 체크

- 함수형 프로그래밍
  - 함수형 프로그래밍의 특징
    - 일급 시민인(first-class) 함수
    - 함수를 일반 값(value)처럼 사용 가능
      - 변수에 저장 / 파라미터로 전달 등
  - 불변성(immutability)
    - 일단 만들어지고 나면 내부 상태가 절대로 바뀌지 않는 불변 객체를 사용해 프로그램을 작성
    - 부수 효과(side effect) 없음
    - 입력이 같으면 항상 같은 출력을 낸다
    - 다른 객체의 상태를 변경하지 않고, 함수 외부와 상호작용 하지 않는 순수 함수(pure function)을 사용
  - 함수형 프로그래밍 장점
    - 코드의 간결성
    - 순수함수를 통해 더 강력한 추상화 가능
    - 코드 중복을 줄일 수 있음
    - 다중 스레드 안정성
    - 불변 데이터 구조와 순수 함수를 통해 같은 데이터를 여러 스레드가 변경하지 못하게 한다
    - 테스트의 용이성
    - 부수효과가 없어서 환경을 구성하기 위해 필요한 준비 코드(setup-code)가 필요하지 않다(독립적 실행 가능)

##### 코틀린의 철학

- 실용성
  - 실제 문제를 해결하기 위해 만들어진 실용적인 언어
  - 항상 도구의 활용을 염두에 두고 설계되어 왔다
  - 더 간결한 구조로 바꾸는 대부분의 코드 패턴을 도구가 자동으로 감지해서 수정하라고 제안

- 간결성
  - 코드를 읽을 때 의도를 쉽게 파악할 수 있는 구문 구조를 제공
  - 의도를 달성하는 방법을 이해할 때 방해가 될 수 있는 부가적 코드가 적다
  - getter, setter 생성자 파라미터 등 부가적 코드를 묵시적으로 제공
  - 기능이 다양한 표준 라이브러리를 제공 => 반복되는 코드를 라이브러리로 대체 가능

- 안전성
  - 프로그램에서 발생할 수 있는 오류중에서 일부 유형의 오류를 프로그램 설계가 원천적으로 방지
  - 안전성과 생산성은 트레이드 오프(trade-off) 관계
    - 더 큰 안전성을 얻기 위해서는 더 많은 정보를 덧붙여야 하기 때문
  - JVM 기반으로 메모리 안전성 / 버퍼 오버플로 방지 / 동적 메모리 관리 측면의 안전성 제공
  - 타입 추론(type inference)을 통해서 적은 비용으로 타입 안전성 사용 가능
  - 컴파일 시점 검사를 통해 오류를 방지
  - ? 연산자를 통해 null이 될 수 있는지 여부를 표시 가능
  - 타입 검사(type check)와 캐스트(cast)가 한 연산자에 의해 이뤄진다

- 상호 운용성
  - 자바(Java) 코드에서 코틀린(Kotlin) 코드를 호출할 때에서 아무런 노력이 필요 없다
  - 자체 컬렉션 라이브러리 제공 X => 기존 자바 라이브러리를 가능하면 최대한 활용
  - Java와 Kotlin 코드가 섞여 있어도 컴파일 문제 X
  
  
#### 코틀린 기초

##### 함수
- 식(expression)
  - 문(statement)와 다르게, 값을 만들어 내며 다른 식의 하위 요소로 계산에 참여
  - Java에서는 모든 제어 구조가 문(statement)였지만, Kotlin에서는 대부분의 제어 구조가 식(expression)

- 식이 본문인 함수
  - 함수의 본문이 등호와 식으로 이루어진 함수
  - Kotlin에서 식이 본문인 함수가 매우 많이 사용된다
  - 식이 본문인 함수인 경우, 함수의 반환 타입이 생략 가능(블록이 본문인 함수는 불가능)
  ```kotlin
  /* 반환타입 O */
  fun max(a: Int, b: Int) : Int = if(a > b) a else b
  
  /* 반환타입 X */
  /* 컴파일러가 함수 본문을 분석해서 타입 추론(type inference)를 해줘서 생략 가능 */
  fun max(a: Int, b: Int) = if(a > b) a else b
  ```
  
##### 변수
- val과 var (기본적으로 val을 권장)
  ```kotlin
  /* val 변수는 참조 자체는 불변, 참조가 가리키는 객체 내부 값은 변경 가능! */
  val langs = arrayListOf("Java")
  langs.add("Kotlin")
  ```
  - val(value)
    - 변경 불가능한 참조를 저장하는 변수
    - 초기화하고 나면 재대입이 불가능
    - Java의 final에 해당
  - var(variable)
    - 변경 가능한 참조
    - Java의 일반변수에 해당
    
- 초기화 식이 있는 경우 변수 타입을 생략할 수 있다
  ```kotlin
  /* 변수 타입 생략 전 */
  val answer : Int = 3
  /* 변수 타입 생략 후 */
  val answer = 3;
  ```
  - 초기화 식이 없으면 컴파일러가 추론할 수 없음;
  - 정수는 Int / 부동소수점 상수는 Double로 기본 추론
  - 컴파일러는 초기화식으로 부터 변수의 타입을 추론

##### 문자열 템플릿 (string template)
```kotlin
val name : String = "hue"
println("${name}님 반갑습니다.")
```
- 문자열 리터럴 안에서 변수를 사용하는방법
- $ 연산자를 통해서 변수를 사용 가능
- ${} 처럼 중괄호를 쓰면 식을 대입할 수 있다
- 문자 $를 넣고 싶다면 \$ 처럼 역슬래시를 사용
- 한글 처리를 할 때에도 $만 쓰면 오류
  - 이러한 오류들이 있어서 ${} 로 문자열 템플릿을 쓰는 습관을 권장

##### 클래스(class)와 프로퍼티(property)

- 값 객체
  ```kotlin
  /* Kotlin은 기본적으로 public 접근제한자를 사용해서 생략 가능 */
  class Person (val name: String)
  ```
  - 코드가 없이 데이터만 저장하는 클래스

- 프로퍼티(property)
  ```kotlin
  class Person{
    val name: String // 읽기 전용, getter만 존재
    var isMarried: Boolean // 읽기, 쓰기 가능, getter setter 모두 존재
  }
  /* 객체 생성, Kotlin에서는 new 연산자 사용 X */
  val person = Person("Hue", false) 

  /* Java와 다르게 필드의 이름으로 직접 접근 */
  println(person.name)
  println(person.isMarried)

  /* setter역시 이름으로 접근 */
  person.isMarried = true
  ```
  - 필드와 접근자를 묶어 말하는 것
  - Kotlin에서는 val과 var을 사용 + getter, setter도 함께 제공
    - val: 읽기 전용, getter만 제공
    - var: 읽기 / 쓰기 가능, getter / setter 모두 제공

##### enum과 when

- enum 클래스
  ```kotlin
  enum class Color {
    RED, ORANGE, YELLOW, GREEN 
  }
  ```
  - enum은 유일하게 Java보다 Kotlin 선언에서 더 많은 키워드를 쓴다;
  - enum class 키워드 사용
  - 프로퍼티와 메소드를 갖는 enum
    ```kotlin
    enum class Color {
      val r: Int, val g: Int, val b: Int
    }{
      /* 상수 생성시 프로퍼티 값을 지정할 때 반드시 마지막에 세미콜론(;) 필요 */
      RED(255,0,0), ORANGE(255,165,0), YELLOW(255,255,0), GREEN(0,255,0);
      fun rgb() = (r * 256 + g) * 256 + b
    }
    ```
    
- when
  ```kotlin
  /* 값을 Set객체로 만드는 setOf() 메소드 사용 */
  fun mix(c1: Color, c2: Color) = 
    when(setOf(c1, c2)){
      setOf(RED, YELLOW) -> ORANGE
      setOf(RED, GREEN) -> YELLOW
      else -> BLACK
    }
  ```
  - 다른 언어의 Switch와 키워드와 같은 역할
  - 분기 조건에 상수만을 사용하는 Java의 Switch와 달리, Kotlin은 임의의 객체를 허용
  - when에 아무 인자가 없으면 -> 각 분기의 조건이 Boolean 결과를 계산하는 식이어야 한다
  
- 스마트 캐스트
  - 타입 검사(type check)와 타입 캐스트(type cast)를 한번에 해결
  - Kotlin에서는 is 키워드를 사용해서 변수의 타입을 검사
  - Java는 타입검사 이후 사용하려면 명시적으로 타입 캐스트를 해줬어야 했다
    - Kotlin은 타입을 검사할 때 파악해서 컴파일러가 대신 해준다!
  - 주의
    - 반드시 값이 바뀔 수 없는 val 이어야 한다 (커스텀 접근자도 X)
      - 그렇지 않으면 항상 같은 타입을 반환한다고 보장할 수 없기 때문
    - var이라서 명시적으로 타입 캐스팅(type casting)을 하려면 as 키워드 사용
    ```kotlin
    /* is 키워드로 e의 타입을 검사 */
    if(e is Sum) {
      /* 타입을 검사하며 컴파일러가 e의 캐스팅을 해주기 떄문에 명시적 캐스팅 필요 X -> 바로 사용 가능! */
      return eval(e.right) + eval(e.left)
    }
    ```
  - 블록(Block) 사용
    - if나 when에서 Block을 사용할 경우 마지막 문장이 블록 전체의 결과가 된다
    - 블록이 본문인 함수인 경우에 해당
      - 내부에 반드시 return을 가지고 있어야 하기 때문
    ```kotlin
    fun eval(e: Expr): Int = 
      when(e) {
        is Num -> {
         ...
         e.value // 결과(return)
        }
        is Sum -> {
          ...
          ...
          e.value + 5 // 결과(return)
        }
      }
    ```

##### While / for 루프
- while
  - Java의 while과 동일
  - while / do~while 존재

- for
  - Java의 for 루프처럼 초깃값, 증가 값, 최종 값을 사용한 루프가 없다
  - 이를 대신하기 위해 범위(range)를 사용
  - 기본적으로 for(v in array) 구문 사용
  - Kotlin의 구간은 폐구간(close)이다 => 양 끝을 포함하는 구간
  - .. 연산자를 통해서 범위 지정
    - 1..10 처럼 범위에 속한 정수를 일정한 순서로 이터레이션 하는 경우를 '수열' 이라고 함
  - 관련 키워드
    - untill : 끝 값을 제외
    - downTo : 역방향을 만드는 키워드
    - step : 증가값을 설정/* 기본 */
  ```kotlin
  for( value in array ) => value이 array의 요소를 순회
  /* .. 연산자 */
  for( i in 1..100 ) // i가 1부터 100까지 순회
  for( c in 'a'..'z' ) // c가 'a'부터 'z'까지 순회
  for( c in 'A'..'Z' ) // c가 'A'부터 'Z'까지 순회
  /* step 키워드 */
  for( i in 1..100 step 2 ) // i가 1부터 100까지 순회하는데 2개의 스텝으로 이동. 즉, 1,3,5,7,9… 99 해당
  /* untill 키워드 */
  for( i in 1 until 100 ) // 1부터 100 아래인 99까지 순회
  /* 비구조화 할당으로도 접근 가능 */
  for( (name, age) in person ) // 인덱스와 값을 동시에 참조 가능

- in
  - 어떤 값이 범위에 속하는지 검사하는 연산자
  - !in 은 속하지 않는지를 검사할 수 있음
  - 내부적인 로직은 라이브러리의 범위 클래스 구현 안에 숨겨져 있음
  ```kotlin
  /* when에서 사용 */
  fun recognize(c: Char) = when(c) {
    in '0'..'9' -> "digit!"
    in 'a'..'z', 'A'..'Z' -> "letter!"
    else -> "I don't know"
  }
  /* 특정 문자열 비교 */
  println("Kotlin in setOf("Java", "Scala")") // false
  ```
  
##### Kotlin 예외 처리
- Java처럼 try ~ catch ~ finally 사용
- Kotlin에서는 함수가 던질 수 있는 예외를 선언하지 안아도 된다 (throws)
- try의 본문은 반드시 중괄호 {} 로 둘러 싸야 한다
- Kotlin은 체크 예외(checked exception)와 언체크 예외(unchecked exception)를 구분하지 않는다
  - 체크 예외(checked exception) : Exception 및 하위
  - 언체크 예외(unchecked exception) : RuntimeException 및 하위
- 예외처리를 강제하는 것이 오히려 불필요한 코드를 만드는경우가 많아서 최신 언어에서는 구분하지 않는다
  ```kotlin
  /* ? 연산자가 있으면 nullable type을 의미 */
  fun readNumber(reader: BufferReader): Int? {
    try{
      val line = reader.readLine()
      return Integer.parseInt(lint)
    }
    catch(e: NumberFormatException){
      return null
    }
    finally{
      reader.cloase()
    }
  }
  ```

#### 출처
- Kotlin In Action (http://www.yes24.com/Product/Goods/55148593)
