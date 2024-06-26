---
layout: post
title: Kotlin 애노테이션과 리플렉션
summary: Kotlin
author: devhtak
date: '2022-03-01 14:41:00 +0900'
category: Kotlin
---

#### 애노테이션과 리플렉션
- 어떤 함수를 호출하기 위해서는 그 함수가 정의된 클래스의 이름과 함수 이름, 파라미터 이름 등을 알아야한다.
- Annotation과 Reflection을 사용하면 그런 제약을 벗어나서 미리 알지 못하는 임의의 클래스를 다룰 수 있다.
  - Annotation을 사용하면 라이브러리가 요구하는 의미를 클래스에게 부여할 수 있다
  - Reflection을 사용하면 실행 시점에 컴파일러 내부 구조를 분석할 수 있다.
- 코틀린에서 Annotation을 사용하는 방법은 자바와 똑같지만 Annotation을 선언할 때 사용하는 문법은 자바와 약간 다르다.
- Reflection 역시 일반 구조는 자바와 같지만 세부 사항에는 약간의 차이가 있다.

##### 애노테이션 적용
- 코틀린에서 자바와 같은 방법으로 애노테이션을 사용할 수 있다.
  - 그러나 애노테이션에 인자를 지정할 때의 문법이 자바와 약간 다르다.
- 코틀린과 자바 차이점
  - 클래스를 애노테이션 인자로 지정할 때는 @MyAnnotation(MyClass::class)처럼 ::class를 클래스 이름 뒤에 넣어야 한다.
  - 다른 애노테이션을 인자로 지정할 때는 인자로 들어가는 애노테이션 이름 앞에 @를 넣지 말아야한다.
  - 배열을 인자로 지정하려면 arrayOf 함수를 사용해야한다. 
    - 만약 자바에서 선언한 애노테이션을 사용하는 경우라면 파라미터가 가변길이 인자로 자동으로 변환된다.

- 애노테이션은 인자를 컴파일 타임에 알 수 있어야 한다.
  - 따라서 임의의 프로퍼티를 인자로 지정할 수는 없다.
- 프로퍼티를 애노테이션 인자로 사용하기 위해서는 const 변경자를 붙여야 한다.
  - 컴파일러는 const가 붙은 프로퍼티를 컴파일 시점 상수로 취급한다.
- 이런 const 타입의 프로퍼티는 파일의 맨 위나 object안에 선언해야 하며, 원시 타입이나 String으로 초기화해야만 한다.

- 애노테이션 대상
  - 자바에 선언된 애노테이션을 사용해 프로퍼티에 애노테이션을 붙이는 경우 사용 시점 대상(use-site target) 선언으로 애노테이션을 붙일 요소를 정할 수 있다.
  - 가령 프로퍼티의 getter에 해당 애노테이션을 붙이고 싶은 경우라면,
  ```kotlin
  @get:MyAnnotation
  val temp = Temp()
  ```
    - 위와 같이 프로퍼티의 요소에 애노테이션을 적용할 수 있다.
  - 코틀린으로 애노테이션을 선언하면 프로퍼티에 직접 적용할 수 있는 애노테이션을 만들 수 있다.
  - 사용 시점 대상을 지정
    - property : 프로퍼티 전체, 자바에서 선언된 애노테이션에는 이 사용 지점 대상을 사용할 수 없다.
    - field : 프로퍼티에 의해 생성되는 필드
    - get : 프로퍼티 게터
    - set : 프로퍼티 세터
    - receiver : 확장 함수나 프로퍼티의 수신 객체 파라미터
    - param : 생성자 파라미터
    - setparam : 세터 파라미터
    - delegate : 위임 프로퍼티의 위임 인스턴스를 담아둔 필드
    - file : 파일 안에 선언된 최상위 함수와 프로퍼티를 담아두는 클래스

- 자바와는 달리 코틀린에서는 애노테이션의 인자로 클래스나 함수선언이나 타입 외에 임의의 식을 허용한다.
  - 예시: @Suppress
  ```kotlin
  @Suppress("UNCHECKED_CAST")
  val string = list as List<String>
  ```
  - @Suppress는 컴파일러 경고를 무시하기 위한 애노테이션이다.

- 자바 API를 애노테이션으로 제어하기
  - 코틀린은 코틀린으로 선언한 내용을 자바 바이트코드로 컴파일하는 방법과 코틀린 선언을 자바에 노출하는 방법을 제어하기 위한 애노테이션을 많이 제공한다.
  - 이런 애노테이션 중 일부는 자바 언어의 일부 키워드를 대신한다.
    - @Volatile과 @Strictfp 애노테이션은 자바의 volatile과 strictfp 키워드를 대신한다.
  - 다음 애노테이션을 사용하면 코틀린 선언을 자바에 노출시키는 방법을 변경할 수 있다.
    - @JvmName은 코틀린 선언이 만들어내는 자바 필드나 메소드 이름을 변경한다.
    - @JvmStatic을 메소드, 객체 선언, 동반 객체에 적용하면 그 요소가 자바 정적 메소드로 노출된다.
    - @JvmOverloads를 사용하면 디폴트 파라미터 값이 있는 함수에 대해 컴파일러가 자동으로 오버로딩한 함수를 생성해준다.
    - @JvmField를 프로퍼티에 사용하면 게터나 세터가 없는 공개된 자바 필드로 프로퍼티를 노출시킨다.

- 애노테이션 선언
  - 코틀린의 애노테이션은 다음과 같이 선언할 수 있다.
    ```kotlin
    annotation class MyAnnoation
    ```
  - 자바에서 @interface라는 다소 모호한 이름으로 선언하던 것과 달리 확실히 annotation 클래스라는 것을 명시해주고 있다.
    - 애노테이션 클래스는 선언이나 식과 관련 있는 메타데이터의 구조를 정의하기 때문에 내부에 어떤 코드도 들어갈 수 없다.
  - 파라미터가 있는 애노테이션을 적용하고자 한다면 애노테이션 클래스의 주 생성자에 파라미터를 선언해야한다.
    ```kotlin
    annotation class MyAnnotation(val name: String)
    ```
  - 주의해야할 점은 애노테이션 클래스에서 모든 파라미터는 val를 반드시 붙여야한다는 점이다.

- 메타애노테이션
  - 애노테이션 클래스에 적용할 수 있는 애노테이션을 메타애노테이션이라고 부른다.
  - 표준 라이브러리에서 가장 일반적으로 쓰이는 메타애노테이션을 꼽으라면 당연 @Target 애노테이션일 것이다.
    - @Target 메타애노테이션은 애노테이션을 적용할 수 있는 요소의 유형을 지정한다.
  - 가령 클래스인지, 메소드, 프로퍼티인지 같은 것들을 명시해줄 수 있다.

##### 리플렉션
- 리플렉션은 실행 시점에 객체의 프로퍼티와 메소드에 접근할 수 있게 해주는 방법이다.
- 일반적으로 객체의 메소드나 프로퍼티에 접근할 때는 프로그램 소스코드 안에 구체적인 선언이 있는 메소드나 프로퍼티 이름을 사용하며, 컴파일러는 그런 이름이 실제로 가리키는 선언을 컴파일 시점에 찾아내서 선언이 실제 존재함을 보장해준다.
  - 타입과 관계 없이 객체를 다뤄야 하거나 객체가 제공하는 메소드나 프로퍼티 이름을 오직 실행 시점에만 알 수 있는 경우가 존재한다.
- JSON 직렬화 라이브러리가 바로 그 예이다.
  - 직렬화 라이브러리는 어떤 객체든 JSON으로 변환할 수 있어야 하고, 실행 시점이 되기 전까지는 라이브러리가 직렬화할 프로퍼티나 클래스에 대한 정보를 알 수 없다.
  - 이런 경우 리플렉션을 사용해야 한다.

- 코틀린에서 리플렉션을 사용하려면 서로 다른 두 API를 사용해야한다.
  - 첫 번째로는 자바에서 제공하는 java.lang.reflect 패키지이다.
    - 자바 리플렉션 API가 필요한 이유는 코틀린 클래스는 일반 자바 바이트코드로 컴파일되기 때문이다.
    - 자바 리플렉션 API는 코틀린 클래스를 컴파일한 바이트코드 또한 완벽히 지원한다.
  - 두 번째는 코틀린이 kotlin.reflect에서 제공하는 API이다.
    - 이 API는 자바에서는 없는 프로퍼티나 널이 될 수 있는 타입과 같은 코틀린 고유 개념에 대한 리플렉션을 제공한다.
    
- 코틀린 리플렉션 API : KClass, KCallable, KFunction, KProperty
  - KClass는 자바의 java.lang.Class에 해당하는 클래스이다.
    - MyClass::class라는 식으로 KClass의 인스턴스를 얻을 수 있다.
  - 실행 시점에 객체의 클래스를 얻기 위해서는 객체의 javaClass 프로퍼티를 사용해 객체의 자바 클래스를 얻어야 한다.
  - javaClass는 자바의 java.lang.Object.getClass()와 동일하다.
  - 자바 클래스를 얻었다면 .kotlin 확장 프로퍼티를 통해 자바에서 코틀린 리플렉션 API로 옮겨올 수 있다.
  ```kotlin
  val person = Person("Harry",27)
  val kClass = person.javaClass.kotlin
  println(kClass.simpleName)
  ```
  - KClass의 내부 선언은 다음과 같이 선언되어있다.
  ```kotlin
  public interface KClass<T : Any> : KDeclarationContainer, KAnnotatedElement, KClassifier {
      /**
       * The simple name of the class as it was declared in the source code,
       * or `null` if the class has no name (if, for example, it is an anonymous object literal).
       */
      public val simpleName: String?

      /**
       * The fully qualified dot-separated name of the class,
       * or `null` if the class is local or it is an anonymous object literal.
       */
      public val qualifiedName: String?

      /**
       * All functions and properties accessible in this class, including those declared in this class
       * and all of its superclasses. Does not include constructors.
       */
      override val members: Collection<KCallable<*>>

      /**
       * All constructors declared in this class.
       */
      public val constructors: Collection<KFunction<T>>

      ...
  }
  ```
  - 잘 보면 클래스의 모든 멤버 목록이 KCallable 인스턴스의 컬렉션이라는 사실을 눈치챘을 것이다.
  - KCallable은 함수와 프로퍼티를 아우르는 공통 상위 인터페이스이다.

- 내부에는 call 메소드가 존재하며 이 call을 사용하면 함수나 프로퍼티의 게터를 호출할 수 있다.
  ```kotlin
  fun foo(x:Int) = println(x)
  val kFunction = ::foo
  kFunction.call(42) // 42
  ```
  - ::foo의 값 타입이 리플렉션 API에 존재하는 KFunction 클래스의 인스턴스임을 알 수 있다.
  - ::foo 의 타입 KFunction1<Int,Unit>에는 파라미터와 반환 값 타입 정보가 들어있다.
    - 1은 이 함수의 파라미터가 1개라는 것을 의미한다.
    - 이 인터페이스를 통해 함수를 호출하기 위해서는 invoke를 사용한다.
  - 그런데 여기서는 invoke를 사용하지 않고 call을 사용해서 호출다.
    - 어쨌든 이 invoke 메소드는 호출할 때 인자 개수나 타입이 맞아 떨어져야만 컴파일이 된다.
  - 파라미터의 개수나 타입에 따라 KFunctionN이 생성되는 데, 이러한 함수 타입들은 컴파일러가 생성한 합성 타입이다.
    - 코틀린 컴파일러가 생성한 합성 타입을 사용하기 때문에 원하는 수만큼 많은 파라미터를 갖는 함수에 대한 인터페이스를 사용할 수 있으며 kotlin.runtile.jar의 크기를 줄일 수 있게 되었다.
  - KProperty는 call 메소드를 호출할 수 있고, get 메소드 또한 지원한다.
    ```kotlin
    var counter = 0
    val kProperty = ::counter
    kProperty.setter.call(21)
    println(kProperty.get()) // 21
    ```
  - 최상위 수준이나 클래스 안에 정의된 프로퍼티만 리플렉션으로 가져올 수 있고 함수의 로컬 변수에는 접근할 수 없다.

##### 출처

- Kotlin In Action (http://www.yes24.com/Product/Goods/55148593)
