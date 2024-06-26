---
layout: post
title: Kotlin 제니릭
summary: Kotlin
author: devhtak
date: '2022-03-01 14:41:00 +0900'
category: Kotlin
---

##### 제네릭 타입 파라미터
- 제네릭 타입의 인스턴스를 만들기 위해서는 타입 파라미터를 구체적인 타입 인자(Type Argument)로 치환해야 한다.
  - List 타입이 있다고 했을 때, 명확하게 문자열을 담는 리스트 
    - List\<String>
  - Map 타입은 제네릭 타입변수로 Map\<K, V>형태로 선언이 되어 있고, 이를 인스턴스화 할때 Map\<String, Person>처럼 구체적인 타입 인자를 넘겨 인스턴스화 할수 있다.
- 코틀린 컴파일러의 경우, 보통 타입과 마찬가지로 타입 인자도 추론이 가능하다
```kotlin
val authors = listOf("John", "Sveltna")
```
- 빈 리스트의 경우, 추론할 값이 없기 때문에 직접 타입을 명시 해줘야 한다.
```kotlin
val authors: List<String> = emptyList();
val authors = emptyList<String>();
```
- 제네릭 함수를 호출할 때는 반드시 구체 타입으로 타입 인자를 넘겨줘야 한다.
  - 아래 List<T>의 slice 확장 함수를 예로 들면, 타입 파라미터 T가 수신객체와 반환 타입으로 사용된다. 
  - 타입 추론이 가능한 구체 타입으로 사용 시, 별도로 제네릭 지정이 불필요하다.
  ```kotlin
  public fun <T> List<T>.slice(indices: IntRange): List<T>
  ```
- 타입 파라미터 선언은 클래스, 인터페이스, 클래스/인터페이스의 메소드, 확장 함수, 최상위 함수에서 선언이 가능하다.
- 코틀린에서는 아래와 같이 제네릭 확장 프로퍼티 선언이 가능하다. 
  - 확장 프로퍼티에 대해서만 가능하며 일반 프로퍼티는 타입 파라미터를 가질 수 없다.
  ```kotlin
  val <T> List<T>.penultimate: T
	    get() = this[size - 2]
  ```
- 자바와 마찬가지로 제네릭 클래스를 확장/구현하기 위해서는 하위 클래스에서 기저 클래스의 구체 타입을 명시하거나 혹은 타입 파라미터로 받은 타입을 넘겨줘야 확장/구현이 가능하다.
```kotlin
class StringList: List<String> { ... }
class ArrayList<T>: List<T> { ... }
```
- 타입 파리미터는 사용할 수 있는 타입을 제한할 수 있다. 
  - 타입 파라미터에 대한 상한(upper-bound) 을 지정한다고 표현하며 제네릭 타입을 인스턴스화 하는 시점에는 상한 타입이거나 혹은 상한 타입의 하위 타입만 가능하게 된다. 
  - 상한을 지정하게 되면 해당 타입의 값은 상한 타입으로 취급이 가능하게 된다.
  ```kotlin
  fun <T : Number> List<T>.sum(): T // Kotlin
  ```
  ```java
  <T extend Number> T sum(List<T> list) // Java
  ```
- 두 인자값 중 큰 값을 찾는 제네릭 함수를 만든다고 할때, Comparable 인터페이스를 활용하여 코드를 작성한다고 해보자.
```kotlin
fun <T: Comparable<T>> max(first: T, second: T): T {
	// 코틀린 컴파일러에 의해 first.compareTo(second) > 0으로 변한다.
	return if (first > second) first else second
}

println(max("kotlin", "java")) // kotlin
println(max("kotlin", 42")) // 42의 경우 첫 번째 인자의 타입 정보와 일치하지 않기 때문에 컴파일 에러
```
- 타입 파라미터에 대해 두 가지 이상의 제약을 걸수도 있다. 
  - 아래 코드에서는 CharSequence와 Appendable 인터페이스를 구현해야 하는 제약을 건 예제이다.
  ```kotlin
  fun <T> ensureTraillingPeriod(seq: T) where T: CharSequence, T: Appendable {
    if (!seq.endsWith('.')) seq.append('.')
  }
  ```
  - 아무런 상한을 정하지 않을 경우 기본적으로 \<T: Any?>로 선언한 것과 같다. 
  - Nullable이 아닌 타입 파라미터를 받고자 한다면 \<T: Any>로 지정해야 한다.

- 만약 \<T:Any>로 선언해둔 타입파라미터에 String?등과 같은 널 허용 타입을 지정하게 되면 String?은 Any의 하위 타입이 아니기 때문에 컴파일 에러가 발생하게 된다.
```kotlin
class A<T> {
	// 타입 파라미터 T에 대해 별도의 상한을 지정하지 않았기 때문에 `Nullable`
	fun process(value: T) {
		value?.hashCode()
	}
}

class B<T: Any> {
	// 타입 파라미터 T에 대해 Any로 상한을 지정하였기 때문에 NonNull
	fun process(value: T) {
		value.hashCode()
	}
}
```

##### 실행 시 제네릭스의 동작: 소거된 타입 파리미터와 실체화된 타입 파라미터
- JVM에서 제네릭은 타입 소거(Type Erasure)를 사용해 구현되며 실행 시점에 제네릭 클래스의 인스턴스에 타입 인자 정보가 들어있지 않다는 뜻이 된다.
  - List의 인스턴스를 만들더라도 런타임엔 List라는 타입정보만 남을뿐 String 원소를 저장하는지에 대한 정보를 알수 없다. 
  - 타입 정보가 지워지는 것이 단점만 있는 것은 아니고 전반적인 메모리 사용량이 줄어들기 때문에 나름 장점도 존재한다.
  ```kotlin
  val list1: List<String> = listOf("a","b")
  val list2: List<Int> = listOf(1,2,3)
  ```
    - list1과 list2는 실제 런타임 시점에서는 List 타입으로만 취급되며 List의 제네릭 타입이 String인지 Int인지는 체크하는 것이 불가능하다. 
    - is를 통한 타입 검사등은 할 수 없다는 이야기이다.

- 코틀린에서는 inline 함수를 사용해 타입 정보가 지워지지 않게 할 수 있는데 이러한 것을 가리켜 실체화(reified)라고 한다.
- 제네릭 타입을 알수 없을 경우 스타 프로젝션을 사용하여 포현하면 된다.
- 타입을 알수 없는 경우에도 as 혹은 as? 캐스팅은 가능하다. 
  - 하지만, 타입 인자가 다른 타입으로 캐스팅해도 캐스팅 자체를 성공해버리기 때문에 이점은 조심해야 한다.

```kotlin
fun printSum(c: Collection<*>) {
    val intList = c as? List<Int> 
    ?: throw IllegalArgumentException("List is Expected")
    println(intList.sum())
}
```
  - 코드 자체는 컴파일에 성공하지만 인자로 넘어오는 값에 따라 여러 예외 상황을 맞이 할 수 있다. 
  - Set 인스턴스를 넘기게 될 경우 엘비스 이후에 있는 예외 코드가 실행이 되게 될것이고, List<String>의 인스턴스를 넘기게 될 경우 캐스팅은 성공하겠지만 sum() 함수를 호출하는 과정에서 ClassCastException을 만나게 될 것이다.

- 만약 컴파일 시점에 타입 정보를 추론할 수 있는 상황이라면 코틀린 컴파일러는 is 검사를 허용해준다. 
```kotlin
fun printSumB(c: Collection<Int>) {
    if (c is List<Int>) {
        println(c.sum())
    }
}
```
  - Collection<Int>로 넘어오는 인자 타입을 추론할 수 있는 상황이기 때문에 is 검사를 허용하게 해준다.
  - IDE에서 테스트해보면 안전하지 못한 is 체크는 금지하고 위험한 as 캐스팅은 warning을 출력한다.

- inline 함수를 사용하여 타입 파라미터 실체화를 사용하게 되면, 실행 시점에 인라인 함수의 타입인자를 알수 있게 된다.
  - 만약 인라인 함수가 람다를 인자로 사용하는 상황이라면 별도의 익명 클래스나 객체가 생성되지 않기 때문에 성능상으로 좀 더 나아질 수도 있다.
- 컴파일러는 인라인 함수의 본문을 구현한 바이트코드를 함수가 호출되는 모든 지점에 삽입하게 되고 컴파일러는 호출한 부분의 타입 인자를 활용해 해당 함수에서 사용하는 정확한 타입 정보를 알수 있게 된다.
- Iterable.filterIsInstance 표준 함수를 예시로 살펴보면, 실제 filterIsInstance를 호출하는 함수 내부에서 inline 함수를 호출하기 때문에 해당 함수 본문의 바이트코드가 호출한 부분에 채워지게 된다. 
  - 본문을 대치하게 됨으로써 실제 제네릭으로 넘어간 타입 정보 역시 런타임에서도 타입 정보가 지워지지 않고 유지된다.
```kotlin
fun reifiedFilterList(): List<String> {
    return listOf("one", 2, "three")
        .filterIsInstance<String>()
}

// 함수 선언부
public inline fun <reified R> Iterable<*>.filterIsInstance(): List<R>
```
  - 자바코드에서 코틀린 코드를 호출할 경우, inline 함수의 실체화된 타입 파라미터는 사용할 수 없다고 한다.

- 자바에서는 코틀린의 함수를 보통 함수처럼 호출하기 때문에 코틀린에서처럼 인라인 함수의 본문 대치가 불가능하다.
- 보통 inline 함수에서는 람다를 사용한 코드가 일반적
  - 위 filterIsInstance의 경우 람다 선언이 없이 구체화된 타입 정보를 런타임에 알기 위해 사용한 것이다. 
  - 일반적으로 람다를 사용하지 않는 인라인 함수는 성능적으로 유리하지 않으며 JVM내에서도 일반 함수를 호출해도 강력하게 인라이닝할 수 있도록 기술적인 장치가 존재한다고 한다.
  - baeldung_jvm-method-inlining

- java.lang.Class 타입 인자를 인자로 받아서 처리하는 코드를 만들 때, inline 함수의 실체화 타입 파라미터를 사용하는 것 또한 유용하다. 
  - 클래스 타 타입 인자를 받아서 처리하는 API 중 대표적인 유스케이스로는 JDK의 ServiceLoader가 있고, 또한 자주 사용하는 jackson과 같은 직렬화/역직렬화 라이브러리에서도 클래스 타입 인자를 넘겨서 처리하는 API가 존재한다.

- 타입 파라미터로 전달된 T에 대해 ::class.java 구문을 사용하여 클래스에 대한 참조를 얻는 방법도 가능하다.
```kotlin
inline fun <reified T: Any> getServiceLoader(): ServiceLoader<T> {
    return ServiceLoader.load(T::class.java)
}

// jackson-module-kotlin
inline fun <reified T> jacksonTypeRef(): TypeReference<T> = object: TypeReference<T>() {}
```

- 실체화 타입 파라미터의 사용 가능한 경우와 제약사항
  - 사용 가능한 경우
    - 타입 검사와 캐스팅(is, as)
    - 리플렉션 API
    - 코틀린 타입에 대응하는 자바 Class 타입 얻기(::class.java)
    - 다른 함수 호출 시, 타입 인자로 사용
  - 제약 사항
    - 타입 파라미터의 인스턴스 생성
    - 타입 파라미터 클래스의 동반 객체 메소드 호출
    - 일반 타입파라미터를 실체화 타입 파라미터를 인자로 받는 함수에 넘기기
    - 클래스, 프로퍼티, 인라인 함수가 아닌 함수의 타입 파라미터를 reified로 지정

- 실체화 타입 파라미터를 사용하는 인라인 함수에서 람다를 사용하게 될 경우, 경우에 따라 인라이닝을 할 수 없는 경우도 발생하기도 하며, 성능 문제로 인라이닝을 안하고 싶을 수도 있다. 
  - noinline 키워드를 사용하여 인라이닝을 금지 시킬수 있다.

##### 변성(variance)
- 기저 타입이 같고 타입 인자가 다른 여러 타입이 서로 어떤 관계가 있는지 설명하는 개념
  - List\<String>, List\<Any>와 같이 기저 타입이 같고 타입이 인자가 다를 경우
- 변성은 코드에서 위험할 여지가 있는 메소드를 호출할 수 없게 만듬으로써 제네릭 타입의 인스턴스 역할을 하는 클래스 인스턴스를 잘못 사용하는 일이 없게 방지하는 역할을 수행한다.
- String의 경우 Any를 확장하기 때문에 Any타입을 받는 함수에 String타입의 인자를 넘겨도 안전하겠지만, List\<String>, List\<Any>의 경우 확실한 안정성을 이야기할 순 없다. 
  - 문자열로 선언된 mutableList에 정수값 42가 추가되고 maxBy 함수를 호출하는 과정에서 Integer에 대해 캐스팅 익셉션이 발생하는 걸 확인할 수 있다.
  ```kotlin
  fun printContents(list: List<Any>) {
      println(list.joinToString())
  }
  fun addAnswer(list: MutableList<Any>) {
      list.add(42)
  }
  val strings = mutableListOf("abc", "bac")
  addAnswer(strings)
  // ClassCastException: Integer cannot be cast to String
  println(strings.maxBy { it.length }) 
  ```

- 어떤 함수가 만약 리스트의 값을 추가/변경하면 타입 불일치가 생길 수 있으니 List\<Any> 대신 List\<String>을 넘길 수 없다. 
  - 원소의 추가/변경이 없을 경우 List\<Any> 대신 List\<String>을 넘겨도 안전하다. 
  - 코틀린에서는 리스트의 변경 가능성에 따라 적절한 인터페이스를 선택하게 되면 추가/변경을 막을 수 있어 안전하지 못한 호출을 막을 수 있다.

- 타입 =! 클래스
  - 제네릭 클래스가 아닌 경우 클래스 이름을 타입으로 바로 사용할 수 있다. 
    - 단순한 예제로 val x: String가 있다.
  - 제네릭 클래스에서는 상황이 복잡하다. List는 타입이 아니고 클래스이며 실제 타입 파라미터가 추가된 List\<String>, List\<Int> 등이 제대로 된 타입이다. 
    - 제네릭 클래스는 결국 무수하게 많은 타입을 만들 수 있게 된다.
- 하위 타입
  - A 타입의 값을 받을 수 있는 인자에 B 타입 값이 들어갈 수 있다면 A타입은 B타입의 상위 타입, B타입은 A타입의 하위 타입 으로 정리할 수 있다. 
    - 예를 들어, Number타입의 인자를 받을 수 있는 함수에 Int값을 넘길 수 있기에 둘 사이의 관계는 Number타입은 Int타입의 상위 타입, Int타입은 Number타입의 하위 타입으로 상/하위 타입 관계를 표현할 수 있게 된다.

- 타입 관계의 중요성은 컴파일러가 변수 대입이나 인자 전달 시 하위 타입 검사를 매번 수행하기 때문이다. 
  - Int 타입의 인자를 받는 함수에 Number 타입 인자를 넣어 호출하게 되면 하위 타입 검사에서 Int 타입의 하위 타입이 Number가 아니기 때문에 컴파일이 되지 않게 된다.
- 간단한 경우 하위 타입은 하위 클래스와 같게 된다. 
  - 앞에서의 예처럼 Int 클래스는 Number 클래스의 하위 클래스이므로 Int는 Number의 하위 타입이다.
- 코틀린의 Nullable과 NonNull 타입의 관계
  - Int? 타입 선언에는 Int타입의 값을 넣거나 변수 선언을 해도 무관하지만, 반대로 Int타입엔 Int?를 선언하는 건 불가능하다
  - 그래서 둘 사이의 관계는 Int?타입은 Int타입의 상위 타입 혹은 Int타입은 Int?타입의 하위 타입이라고 정리할 수 있다
```kotlin
val s: String = "abc"
val t: String? = s // String이 String?의 하위 타입이므로 이 대입은 가능
```
- 제네릭 타입을 인스턴스화 할때, 타입 인자에 서로 다른 타입이 들어가고 서로 하위 타입 관계가 성립하지 않은 걸 가리켜 무공변(invariant)라고 한다.
- 타입 A가 B의 하위 타입이면 List\<A>는 List\<B>의 하위 타입이며 이런 관계를 공변적(covariant)라고 한다.

- 공변성(covariant)
  - 공변적인 관계를 표현할 때는 out 키워드를 넣어 표현한다.
  ```kotlin
  interface Producer<out T> {
      fun produce(): T
  }
  ```
  - 클래스의 타입 파라미터를 공변적으로 만들면 함수 정의에 사용한 파라미터 타입과 타입 인자의 타입이 정확히 일치하지 않더라도 그 클래스의 인스턴스를 함수 인자나 반환 값으로 사용할 수 있다.
  - 타입 파라미터를 공변적으로 지정하면 클래스 내부에서 그 파라미터를 사용하는 방법을 제한한다
    - 타입 안정성을 위해 항상 아웃(out) 위치에 있어야 한다. 
    - 클래스가 타입의 값을 생상할 수 있지만 값을 소비할 수 없다.
  - 클래스 맴버 선언 시, 타입 파라미터를 사용할 수 있는 지점은 out/in으로 나뉘며, out으로 선언할 경우 반환 타입(생산)에 쓰이며 in 으로 선언할 경우 값을 소비하는 역할로 한정이 된다.

 ```kotlin
interface Transformaer<T> {
    // t의 인자는 in 위치, 반환 타입 T는 out 위치
    fun transform(t: T): T
}
```
  - 공변성은 하위 타입 관계가 유지되며, 아웃 키워드를 통해 공변성을 표현하며 아웃으로 선언하게 되면 아웃 위치에서만 사용이 가능하다.
```kotlin
class Herd<out T: Animal> {
    val size: Int
        get() = ...
    operator fun get(i: Int): T { ... }
}
```
  - 위 클래스는 Herd 클래스에 Animal의 하위 타입을 받는 타입이며, 공변성을 표현하고 있고 타입 한정을 Animal로 지정하여 T타입은 자연스럽게 Animal 타입으로 취급된다. 
    - 또한 공변성을 표현하고 있기 때문에 반환 타입으로만 해당 타입을 지정하는 것이 가능하다.
    - 또한, Animal뿐만 아니라 이를 상속하는 타입이 있다면 해당 타입을 타입 파라미터로 지정하는 것이 가능하다. 
    - 예를 들어 Cat이 Animal의 하위 타입이라면 get을 호출하는 부분에서 Cat을 반환해도 아무런 문제가 없게 된다.
- 타입 파라미터는 함수 파라미터 타입이나 반환 타입으로만 쓰이지 않고, 다른 타입 파라미터의 인자로도 사용할 수 있다.
- 생성자 파라미터는 인이나 아웃 어느 쪽도 아니며 타입 파라미터가 out 으로 선언되어 있어도 그 타입을 생성자 파라미터 선언에 사용할 수 있다. 생성자는 나중에 호출할 수 있는 메소드가 아니기 때문에 위험할 여지가 없다.
- val, var 키워드를 생성자 파라미터에 사용할 경우 게터 혹은 세터를 정의하는 것고 같기 때문에 읽기 전용 프로퍼티는 아웃 위치에 그리고 변경 가능 프로퍼티는 인 위치에 해당한다.
```kotlin
class Herd<T: Animal>(var leadAnimal: T, vararg animals: T) { ... }
```

- 반공변성(contravariance)
  - 타입 B가 A의 하위 타입인 경우, Consumer\<A>가 Consumer\<B>의 하위 타입인 관계가 성립하면 제네릭 클래스 Consumer는 타입 인자 T에 대해 반공변을 표현한다고 할 수 있다.
  - Animal의 하위 타입인 Cat이 Consumer\<Animal>가 Consumer\<Cat>의 하위 타입 관계가 성립될 경우 이를 반공변 관계가 성립한다고 할 수 있다.
  - in이라는 키워드는 그 키워드가 붙은 타입이 이 클래스의 메소드 안으로 전달돼 메소드에 의해 소비된다는 뜻이다. 다시 말해, 오직 인 위치에서만 사용할 수 있다는 뜻이다.

- 공변 vs 반공변 vs 무공변
  - 공변성
    - Producer\<out T>
    - 타입 인자의 하위 타입 관계가 제네릭 타입에서도 유지된다.
    - Producer\<Cat>은 Producer\<Animal>의 하위 타입이다.
    - T를 아웃 위치에서만 사용
  - 반공변성
    - Consumer\<in T>
    - 타입 인자의 하위 타입 관계가 제네릭 타입에서 뒤집힌다.
    - Consumer\<Animal>은 Consumer\<Cat>의 하위 타입이다.
    - T를 인 위치에서만 사용
  - 무공변성
    - MutableList\<T>
    - 하위 타입 관계가 성립하지 않음
    - T를 아무 위치에서나 사용 가능
```kotlin
interface Function1<in P, out R> {
    operator fun invoke(p: P): R
}
```
  - Function 에서 P 타입에 대해서는 반공변성을 유지하며, R 타입에 대해서는 공변성을 띄게 된다.
  - Cat과 Animal의 예제를 위 Function1에 대입해보면 (Cat) -> Number와 (Animal) -> Int에서도 하위 타입 관계가 성립하게 된다. 
  - Function1의 경우 코틀린의 표현으로 바꿔보자면 (Cat) -> Number를 받는 인자에 (Animal) -> Int의 함수 인스턴스를 넘기더라도 하위 타입 관계가 성립하기 때문에 호출에 문제가 없게 되는 것이다.
  - 반공변의 경우 타입의 하위 관계를 반대로, 공변의 경우 타입의 하위 관계를 그대로 유지, 무공변의 경우 타입 파라미터간의 하위 타입 관계가 전혀 없는 것으로 정리할 수 있다.

- 변성의 종류
  - 선언 지점 변성(declaration site variance): 클래스 선언에 지정하는 것을 선언 지점 변성이라 부르며, 한번의 선언만으로 변성을 추가 지정할 필요가 없으므로 코드가 간결해진다.
  - 사용 지점 변성(use site variance): 자바에서처럼 타입 파라미터가 있는 타입을 사용할때 마다 해당 타입 파라미터를 하위 타입이나 상위 타입 중 어떤 타입으로 대치할 수 있는지 명시하는 방법을 사용 지점 변성이라고 한다.   - 자바의 한정 와일드카드 타입(bounded wildcard)을 사용하는 것을 생각하면 된다.(? extends , ? super) 물론 코틀린도 자바에서처럼 사용 지점 변성을 지원한다. 
    - 공변적이거나 혹은 반공변적인지 선언할 수 없다면 사용 지점 변성을 사용하여 타입 파라미터가 나오는 지점에 변성을 정의하면 된다.
  - 변성을 지정하지 않은 클래스를 사용할 때 변성에 대한 지정이 가능하다. 대표적인 예제로 MutableList가 있다. 
    - MutableList는 타입 파라미터로 넘어는 값을 생산할 수도 있고 소비할수도 있게 되는데, 변성에 대한 지정을 통하여 이를 제어할 수 있게 된다.

```kotlin
fun <T> copyData(source: MutableList<out T>, destination: MutableList<T>) {
    for(item in source) {
        destination.add(item)
    }
}
```
  - source의 타입 선언을 MutableList\<out T>로 지정해뒀는데, out 키워드를 붙임으로서 무공변인 MutableList에서 공변으로 제약을 붙인 타입으로 프로젝션을 시킬 수 있게 된다. 
    - 이럴 경우 값을 소비하는 add 함수등을 사용하게 될 경우 Out-projected type ....과 같은 에러를 만나게 될 수 있다.
- 타입 선언에서 \[타입 파라미터를 사용하는 위치]라면 어디에나 \[변성 변경자]를 붙일 수 있으며, \[파라미터 타입, 로컬 변수 타입, 함수 반환 타입] 등에 타입 파라미터가 쓰이는 경우 in이나 out 변경자를 붙일 수 있다. 
  - 이를 가리켜 타입 프로젝션(type projection)이라 부른다.
  - List\<out T>처럼 이미 변성 변경자가 붙어 있는 클래스에 out 프로젝션을 하는 것은 의미가 없으므로 컴파일러와 IDE가 친절하게(?) 경고를 하게 될 것이다.

##### 출처
- Kotlin In Action (http://www.yes24.com/Product/Goods/55148593)
