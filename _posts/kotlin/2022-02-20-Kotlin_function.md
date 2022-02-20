---
layout: post
title: Kotlin 함수
summary: The Java
author: devhtak
date: '2022-02-20 14:41:00 +0900'
category: Kotlin
---

##### 컬렉션
- 컬렉션 만들기
  - 코틀린은 자체 컬렉션을 제공하지 않고 표준 자바 컬렉션을 활용하여 자바 코드와의 상호작용 쉽게 한다.
    ```kotlin
    val set = hashSetOf(1, 7, 53) // java.util.HashSet 의 객체
    val list = arrayListOf(1, 7, 53) // java.util.ArrayList 의 객체
    val map = hashMapOf(1 to "one", 7 to "seven", 53 to "fifty-three") // java.util.HashMap 의 객체    
    ```
  - 이름 붙인 인자
    - 코틀린으로 작성한 함수를 호출할 때는 함수에 전달하는 인자 중 일부 또는 전부의 이름을 명시할 수 있다
      ```kotlin
      println(joinToString(list, separator = "; ", prefix = "(", postfix = ")"))
      ```

- 디폴트 파라미터 값
  - 자바에서 너무 많은 오버로딩 메소드가 있다는 것이 문제. 코틀린에서는 함수 선언에서 파라미터의 디폴트값을 지정할 수 있다.
  - 함수의 디폴트 파라미터 값은 함수를 호출하는 쪽이 아니라 함수 선언 쪽에서 지정된다. 
  - 다음은 컬렉션을 받아서 구분자, 접두사, 접미사를 인자로 받아 원하는 형식으로 컬렉션을 출력하는 제네릭 함수 예제다
    ```kotlin
    fun <T> joinToString(
        // 디폴트 파라미터값 지정도 가능하다.
        collection: Collection<T>,
        separator: String = ", ",
        prefix: String = " ",
        postfix: String = ""
    ): String {

        val result = StringBuilder(prefix)

        for ((index, element) in collection.withIndex()) {
            if (index > 0) result.append(separator) // 첫 원소 앞에는 구분자를 붙이면 안된다.
            result.append(element) // 컬렉션의 원소를 StringBuilder의 뒤에 덧붙인다.
        }

        result.append(postfix)
        return result.toString()
    }
    ``` 

##### 정적인 유틸리티 클래스 없애기 : 최상위 함수

- 자바에서는 모든 코드를 클래스의 메소드로 작성해야하는데, 어느 한 클래스에 포함시키기 어려운 코드가 있는 경우가 있다.
- 정적 메소드를 모아두는 역할만 담당하며, 특별한 상태나 인스턴스 메소드는 없는 클래스가 왕왕 생겨난다.
- 코틀린에서는 함수를 클래스 안에 선언할 필요가 전혀 없다. 함수를 직접 소스파일의 최상위 수준, 모든 다른 클래스의 밖에 위치시키면 된다.
  ```kotlin
  package strings
  fun joinToString(...): String { ... }
  ```
  - 코틀린 컴파일러는 최상위 함수가 들어있던 코틀린 소스 파일의 이름과 대응하는 클래스를 생성한다.
  ```koglin
  package strings;

  public class JoinKt {
    public static String joinToString{...} {...}
  }
  ```
  - 코틀린 파일의 모든 최상위 함수는 이 클래스의 정적인 메소드가 된다.
  - 자바에서 joinToString을 호출하는 코드는 다음과 같다.
  ```kotlin
  import strings.JointKt;

  JointKt.joinToString(list, ", ", "", "");
  ```
  - 만약, 코틀린 최상위 함수가 포함되는 클래스의 이름을 바꾸고 싶다면 파일 맨 앞, 패키지 선언 이전에 @JvmName 어노테이션을 추가하면 된다. 

##### 정적인 유틸리티 클래스 없애기 : 최상위 프로퍼티

```kotlin
var opCount = 0
fun performOperation() {
    opCount++
    //...
}

fun reportOperationCount() {
    println("Operation performed $opCount times")
}
```

- 메소드를 다른 클래스에 추가: 확장 함수와 확장 프로퍼티
- 기존 코드와 코틀린 코드를 자연스럽게 통합하는것은 중요하다. 
- 완전한 코틀린으로만 이루어진 프로젝트라 하더라도, 서드파티 프레임워크들은 자바 라이브러리 기반으로 만들어진 경우가 많기 때문이다.
- 코틀린을 기존 자바 프로젝트에 통합하는 경우 코틀린으로 직접 변환할 수 없거나 미처 변환하지 않은 기존 자바 코드를 처리할 수 있어야 한다. 
- 기존 자바 API를 재작성하지 않고도 코틀린이 제공하는 기능들을 사용할 수 있도록 해주는 것이 확장 함수이다. 

##### 확장 함수
- 어떤 클래스의 멤버 메소드인 것처럼 호출할 수 있지만 그 클래스의 밖에 선언된 함수다.
```kotlin
fun String.lastChar(): Char = this.get(this.length - 1) // String : 수신 객체 타입, this : 수신 객체
```
  - 확장할 클래스 이름 = 수신 객체 타입
  - 확장 함수가 호출되는 대상이 되는 값(객체) = 수신 객체
  - String 클래스의 소스 코드를 소유하지 않았지만, 자바 클래스로 컴파일한 클래스 파일이 있는 한 그 클래스에 원하는 대로 확장을 추가할 수 있다.

- 확장 함수 본문에서 this 생략 가능
```kotlin
fun String.lastChar(): Char = get(length - 1)
```
  - 확장 함수 내부에서는 일반적인 인스턴스 메소드의 내부와 같이 수신 객체의 메소드나 프로퍼티를 바로 사용할 수 있지만, 확장 함수가 캡슐화를 깨지는 않는다. 
  - 클래스 안에서 정의한 메소드와 달리 확장 함수 안에서는 클래스 내부에서만 사용할 수 있는 private 멤버나 protected 멤버를 사용할 수 없다. 

##### 임포트와 확장 함수
- as 키워드를 사용하여 임포트한 클래스나 함수를 다른 이름으로 부를 수 있다.
```kotlin
import strings.lastChar as last
val c = "Kotlin".last()
```
  - 이렇게 이름을 바꿔서 임포트 하면 다른 여러 패키지에 속해있는 이름이 같은 함수를 가져와 사용해야 할 경우 충돌을 막을 수 있다. 

- 확장 함수로 유틸리티 함수 정의
```kotlin
fun <T> Collection<T>.joinToString(
        separator: String = ", ",
        prefix: String = "",
        postfix: String = ""
): String {
    val result = StringBuilder(prefix)

    for ((index, element) in this.withIndex()) { // this는 수신 객체(= T타입 컬렉션)
        if (index > 0) result.append(separator)
        result.append(element)
    }

    result.append(postfix)
    return result.toString()
}
```

- 확장 함수는 오버라이드 할 수 없다.
  - 확장 함수는 정적 메소드와 같은 특징을 가지므로, 확장 함수를 하위 클래스에서 오버라이드할 수 없다.
  - 확장 함수는 수신 객체로 지정한 변수의 정적 타입에 의해 어떤 함수가 호출될지 정적으로 결정된다. ( 확장 함수는 첫 번째 인자가 수신 객체인 정적 자바 메소드로 컴파일하기 때문 )
  - 코틀린의 일반 메소드는, 그 변수가 저장된 객체의 동적인 타입에 의해 확장함수가 결정된다.

- 확장 프로퍼티
```kotlin
// 확장 프로퍼티 선언
val String.lastChar: Char
    get() = get(length - 1)

var StringBuilder.lastChar: Char
    get() = get(length - 1) // 프로퍼티 게터
    set(value: Char) {
        this.setCharAt(length - 1, value) // 프로퍼티 세터
    }
```

##### 컬렉션 처리: 가변 길이 인자, 중위 함수 호출, 라이브러리 지원
- vararg 키워드를 사용하면 호출 시 인자 개수가 달라질 수 있는 함수를 정의할 수 있다
- 중위 함수 호출 구문을 사용하면 인자가 하나뿐인 메소드를 간편하게 호출할 수 있다
- 구조 분해 선언을 사용하면 복합적인 값을 분해해서 여러 변수에 나눠 담을 수 있다

- 자바 컬렉션 API 확장
  - 위에서 코틀린 컬렉션은 자바와 같은 클래스를 사용하지만 더 확장된 API를 제공한다
  ```kotlin
  fun main() {
    val view: View = Button()
    view.showOff()
    
    var strings: List<String> = listOf("first", "second", "fourteenth")
    println(strings.last())
    
    val numbers: Collection<Int> = setOf(1, 14, 2)
    println(numbers.maxOrNull())
  }
  ```
  - 자바 라이브러리 클래스의 인스턴스인 컬렉션에 대해 코틀린이 새로운 기능을 추가 할 수 있었던 이유는 last(), max()는 모두 확장 함수.

- 가변 인자 함수: 인자의 개수가 달라질 수 있는 함수 정의
  ```kotlin
  fun main(args: Array<String>) {
    val list = listOf("args: ", *args)
    println(list)
  }
  ```
  - 코틀린에서도 자바와 비슷하게 가변 길이 인자를 사용할 수 있습니다.

- 값의 쌍 다루기: 중위 호출과 구조 분해 선언
  ```kotlin
  fun main() {
    val map = mapOf(1 to "one", 7 to "seven", 53 to "fifty-three")
    println(1.to("one"))  // 일반적인 방식으로 호출
    println(1 to "one")   // 중위 호출 방식으로 호출
  }
  ```
  - 여기서 to는 코틀린 키워드가 아니다. 중위 호출이라는 방식으로 to라는 일반 메소드를 호출한 것.
  - 인자가 하나뿐인 일반 메소드나 인자가 하나뿐인 확장 함수에 중위 호출을 사용할 수 있으며 함수를 중위 호출에 사용하게 허용하고 싶다면 infix 변경자를 함수 선언 앞에 추가해야 한다
  ```kotlin
  public infix fun <A, B> A.to(that: B): Pair<A, B> = Pair(this, that)
  fun main() {
      val (number, name) = 1 to "one"
      println("number: $number, name : $name")
  }
  ```
  - 구조 분해 선언: 위와 같이 두 변수를 즉시 초기화하여 선언하는 방식

- 문자열 나누기
  ```kotlin
  public class Test {
      public static void main(String[] args) {
          String[] split = "12.345-6.A".split(".");
          System.out.println(Arrays.toString(split)); // []
      }
  }
  ```
  - 자바에서는 split 함수로 위와 같이 마침표(.)을 분리시키면 [12, 345-6, A]와 같은 배열로 반환되는 것이 아니라 빈 배열로 반환.
  - split의 구분 문자열은 실제로는 정규식으로 마침표(.)는 모든 문자를 나타내는 정규식으로 해석된다.
  - 코틀린에서는 자바의 이러한 혼란스러운 split() 메소드의 매개변수를 String이 아니라 Regex 타입의 값을 받습니다.
    ```kotlin
    println("12.345-6.A".split(".")) // [12, 345-6, A]
    ```

##### 코드 다듬기: 로컬 함수와 확장 
- 코틀린에서는 함수에서 추출한 함수를 원 함수 내부에 중첩시킬 수 있다. 따라서, 코드 중첩을 로컬 함수를 통해서 해소가능하다. 
```kotlin
class User(val id: Int, val name: String, val address: String)

fun saveUser(user: User) {
    if (user.name.isEmpty()) {
        throw IllegalArgumentException (
            "Can't save user ${user.id} : empty Name"
                )
    }

    if (user.address.isEmpty()) {
        throw IllegalArgumentException (
            "Can't save user ${user.id} : empty Address"
        )
    }

    // user를 데이터베이스에 저장한다.
}
```
- 로컬 함수는 자신이 속한 바깥 함수의 모든 파라미터와 변수를 사용할 수 있다.
- 확장함수 추출하기
```kotlin
class User(val id: Int, val name: String, val address: String)

fun User.validateBeforeSave() {
    fun validate(value: String, fieldName: String) {
        if (value.isEmpty()) {
            throw IllegalArgumentException (
                "Can't save user $id : empty $fieldName" // User의 프로퍼티 직접 사용
            )
        }
    }

    validate(name, "Name")
    validate(address, "Address")
}

fun saveUser(user: User) {
    user.validateBeforeSave() // 확장 함수를 호출

    // user를 데이터베이스에 저장한다.
}
```
- 한 객체만을 다루면서 객체의 비공개 데이터를 다룰 필요는 없는 함수는 확장 함수로 만들면 객체.멤버 처럼 수신 객체를 지정하지 않고도 공개된 멤버 프로퍼티나 메소드에 접근할 수 있다.
