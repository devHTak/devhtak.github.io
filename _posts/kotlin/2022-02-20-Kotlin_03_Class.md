---
layout: post
title: Kotlin 클래스
summary: The Java
author: devhtak
date: '2022-02-20 14:41:00 +0900'
category: Kotlin
---

##### 클래스 계층 정의
- 인터페이스
  - 코틀린 인터페이스 안에는 추상 메서드뿐 아니라 구현이 있는 메서드도 정의할 수 있다.
  - 상태(필드)는 들어갈 수 없다.
  ```kotlin
  //인터페이스 정의
  interface Clickable {
      fun click()
  }
  //인터페이스 구현
  class Button : Clickable { //코틀린은 클래스 확장과 인터페이스 구현 모두 콜론(:)을 붙인다.
      //오버라이드 표시
      override fun click() = println("I was clicked")
  }
  ```
  - 자바와 마찬가지로 인터페이스는 개수 제한없이 마음대로 구현할 수 있지만, 클래스는 오직 하나만 확장할 수 있다.
  - 상위 클래스에 있는 메서드와 시그니처가 같은 메서드를 우연히 하위 클래스에 선언하는 경우, 컴파일이 안 되기 때문에 override를 붙이거나 메서드 이름을 바꿔야 한다.
  - 예시
    ```kotlin
    interface Clickable {
        fun click()
        fun showOff() = println("I'm clickable!") //디폴트 구현 정의
    }
    interface Focusable {
        fun setFocus(b: Boolean) =
            println("I ${if (b) "got" else "lost"} focus.")
        fun showOff() = println("I'm focusable!")
    }
    ```
    - 클래스가 구현하는 두 상위 인터페이스에 정의된 showOff 구현을 대체할 오버라이딩 메서드를 직접 제공하지 않으면 컴파일 오류가 발생한다.
    - 이름과 시그니처가 같은 멤버 메서드에 대해 둘 이상의 디폴트 구현이 있는 경우, 인터페이스를 구현하는 하위 클래스에서 명시적으로 새로운 구현을 제공해야 한다.
    ```kotlin
    class Button : Clickable, Focusable {
        override fun click() = println("I was clicked")

        override fun showOff() {
            super<Clickable>.showOff()
            super<Focusable>.showOff()
        }
    }
    ```
    - super 처럼 꺽쇠 안에 타입을 지정한다. (자바에서는 Clickable.super)
  - 코틀린은 자바 6과 호환되므로, 자바에서는 코틀린의 디폴트 메서드 구현에 의존할 수 없다.
  - 디폴트 메서드가 있는 인터페이스를 일반 인터페이스와 디폴트 메서드 구현이 정적 메서드로 들어있는 클래스를 조합해 구현된다.

- open, final, abstract
  - 코틀린의 클래스와 메서드는 기본적으로 final이다.
  - 상속을 허용하려면 open 변경자를 붙여야 한다.
  - 오버라이드를 허용하고 싶은 메서드나 프로퍼티 앞에도 open 변경자를 붙여야 한다.
  ```kotlin
  open class RichButton : Clickable { //이 클래스는 열려 있다.
      fun disable() {} //이 함수는 오버라이드할 수 없다.
      open fun animate() {} //오버라이드 가능하다.
      override fun click() {} //상위 클래스의 메서드를 오버라이드한다. (오버라이드한 메서드는 기본으로 열려있다.)
  }
  open class RichButton : Clickable {
      final override fun click() {} //오버라이드한 메서드의 구현을 하위 클래스에서 오버라이드하지 못하게 하려면 final을 붙인다.
  }
  ```
  - abstract: 추상 클래스 선언
    - 추상 멤버는 항상 열려 있다. -> open을 붙일 필요가 없다.
  - 인터페이스 멤버는 final, open, abstract를 사용하지 않는다.
    - 인터페이스 멤버는 항상 열려 있으며 final로 변경할 수 없다.
  - 변경자	오버라이드	설명
    - fianl: 불가능, 클래스 멤버의 기본 변경자
    - open: 가능, open을 명시해야 오버라이드 가능
    - abstract: 필수, 추상 클래스의 멤버에만 붙일 수 있다.
  - 추상 멤버는 구현이 있으면 안 된다.

- 가시성 변경자
  - 선언에 대한 클래스 외부의 접근을 제어한다.
  - 아무 변경자도 없는 경우 모두 public 이다.
  - 코틀린은 package-private을 사용하지 않는다. (패키지를 네임스페이스 관리 용도로만 사용하기 때문)
    - 대신 internal을 도입: 모듈 내부에서만 볼 수 있음
  - 코틀린은 최상위 선언(클래스, 함수, 프로퍼티)에 대해 private 가시성을 제공한다.
    - 그 선언이 들어있는 파일 내부에서만 사용할 수 있다.
  - 변경자
    - public(default)
      - 클래스 멤버: 모든 곳에서
      - 최상위 선언: 모든 곳에서
    - internal		
      - 클래스 멤버: 같은 모듈 안에서만
      - 최상위 선언: 같은 모듈 안에서만
    - protected		
      - 클래스 멤버: 하위 클래스 안에서만
      - 최상위 선언: (최상위 선언에 적용 불가능)
    - private	
      - 클래스 멤버: 같은 클래스 안에서만	
      - 최상위 선언: 같은 파일 안에서만
    - 어떤 클래스의 기반 타입 목록에 들어있는 타입이나 제네릭 클래스의 타입 파라미터에 들어있는 타입의 가시성은 클래스 자신의 가시성과 같거나 높아야 하고, 메서드 시그니처에 사용된 모든 타입의 가시성은 그 메서드의 가시성과 같거나 높아야 한다.
  - 코틀린에서는 외부 클래스가 내부 클래스나 중첩된 클래스의 private 멤버에 접근할 수 없다.

- 내부 클래스와 중첩된 클래스
  - 코틀린의 중첩 클래스는 명시적으로 요청하지 않는 한 바깥 클래스 인스턴스에 대한 접근 권한이 없다.
  - 바깥 클래스에 대한 참조를 포함하고 싶다면 inner 변경자를 붙여야 한다.

  ![image](https://user-images.githubusercontent.com/42403023/154828405-9604cbb9-51f1-4498-a894-160eccfff27d.png)

  - 클래스 계층을 만들되 그 계층에 속한 클래스의 수를 제한하고 싶은 경우 중첩 클래스를 쓰면 편리하다.

- sealed 클래스
  - 상위 클래스를 상속한 하위 클래스 정의를 제한할 수 있다.
  - sealed 클래스의 하위 클래스를 정의할 때는 반드시 상위 클래스 안에 중첩시켜야 한다.(1.0 버전)
  - 1.1 버전부터는 같은 파일 안에 하위 클래스를 만들 수 있다. 1.5 버전부터는 sealed 인터페이스도 정의 가능하다.
  ```kotlin
  sealed class Expr {
      class Num(val value: Int) : Expr()
      class Sum(val left: Expr, val right: Expr) : Expr()
  }
  ```
  - when 식에서 sealed 클래스의 모든 하위 클래스를 처리한다면 else 분기는 필요 없다.

##### 뻔하지 않은 생성자와 프로퍼티를 갖는 클래스 선언
- 코틀린은 주 생성자(primary constructor), 부 생성자(secondary constructor), 초기화 블럭을 제공한다.
- constructor 키워드로 주 생성자나 부 생성자를 정의한다.

- 클래스 초기화: 주 생성자와 초기화 블럭
  - 주 생성자
    - 생성자 파라미터 지정
    - 초기화되는 프로퍼티를 정의
  ```kotlin
  class User constructor(_nickname: String) { //주 생성자 정의
      val nickname: String
      init { //초기화 블럭
          nickname = _nickname
      }
  }
  ```
  - 초기화 블럭은 인스턴스화될 때 실행될 초기화 코드가 들어가며, 주 생성자와 함께 사용된다.
    - (주 생성자는 별도의 코드를 포함할 수 없으므로 초기화 블럭이 필요하다.)
  - 초기화 블럭은 여러 개 생성 가능하다.
  - 프로퍼티를 초기화하는 식이나 초기화 블럭 안에서만 주 생성자의 파라미터를 참조할 수 있다.
  - 주 생성자의 파라미터로 프로퍼티를 초기화한다면, 그 파라미터에 val을 추가하는 방식으로 프로퍼티 정의와 초기화를 간략히 쓸 수 있다.
  ```kotlin
  class User(val nickname: String,
             val isSubscribed: Boolean = true) //생성자 파라미터에도 디폴트 값을 정의할 수 있다.
  ```
    - 객체 생성 시에는 new 키워드없이 직접 호출한다.
  ```kotlin
  val alice = User("Alice")
  ```
    - 모든 생성자 파라미터에 디폴트 값을 지정하면 컴파일러가 자동으로 파라미터가 없는 생성자를 만들어준다. 
    - 별도로 생성자를 정의하지 않았을 때도 자동으로 파라미터가 없는 디폴트 생성자를 만들어준다. 
    - 기반 클래스가 있는 경우, 반드시 기반 클래스의 생성자를 호출해야 한다.
  ```kotlin
  open class User(val nickname: String) { ... }
  class TwitterUser(nickname: String) : User(nickname) { ... } //기반 클래스에 생성자 파라미터를 넘김
  class RadioButton: Button() //Button의 생성자 호출
  ```
    - 클래스 외부에서 인스턴스화하지 못하게 막고 싶다면 모든 생성자를 private으로 만든다.
  ```kotlin
  class Secretive private constructor() {} //주 생성자를 private으로 만드는 예시
  ```

- 부 생성자
  - 클래스에 주 생성자가 없다면 모든 부 생성자는 반드시 상위 클래스를 초기화하거나 다른 생성자에게 생성을 위임해야 한다.
  - 부 생성자의 주 목적은 자바 상호운용성이지만, 클래스 인스턴스를 생성할 때 파라미터 목록이 다른 생성 방법이 여럿 존재하는 경우에는 부 생성자를 여럿 둘 수 밖에 없다.
  ```kotlin
  class MyButton : View {
      constructor(ctx: Context): this(ctx, MY_STYLE) { //this: 자신의 다른 생성자에 위임한다.
          // ...
      }
      constructor(ctx: Context, attr: AttributeSet): super(ctx, attr) { //super: 상위 클래스의 생성자를 호출한다.
          // ...
      }
  }
  ```
  ![image](https://user-images.githubusercontent.com/42403023/154828508-6727eea4-fb04-4324-983f-43112ba25c93.png)

- 인터페이스에 선언된 프로퍼티 구현
  - 코틀린에서는 인터페이스에 추상 프로퍼티 선언을 넣을 수 있다.
    ```kotlin
    interface User {
        val nickname: String
    }
    ```
    - nickname의 값을 얻을 수 있는 방법을 제공해야 한다는 뜻이다.
  - 구현 방법 1) 주 생성자 안에 직접 선언하기
    ```kotlin
    class PrivateUser(override val nickname: String) : User
    ```
    - User의 프로퍼티를 구현하는 것이므로, override를 붙여줘야 한다.
  - 구현 방법 2) Custom Getter
    ```kotlin
    class SubscribingUser(val email: String) : User {
        override val nickname: String
            get() = email.substringBefore('@') )
    }
    ```
    - 뒷받침하는 필드(backing field)에 저장하지 않고, 매번 이메일 주소에서 nickname을 계산해 반환한다.
  - 구현 방법 3) 프로퍼티 초기화 식
    ```kotlin
    class FacebookUser(val accountId: Int) : User {
        override val nickname = getFacebookName(accountId)
    }
    ```
    - 초기화 단계에서 getFacebookName이 한 번만 호출된다.
    - 객체 초기화 시 계산한 데이터를 뒷받침하는 필드에 저장했다가 불러오는 방식을 활용한다.
  - 인터페이스에는 추상 프로퍼티 뿐만 아니라 Getter와 Setter가 있는 프로퍼티를 선언할 수 있다.
    - Getter, Setter는 뒷받침하는 필드를 참조할 수 없다.
    ```kotlin
    interface User {
        val email: String //반드시 오버라이드해야 한다.
        val nickname: String
            get() = email.substringBefore('@')
    }
    ```
- Getter와 Setter에서 뒷받침하는 필드(Backing field) 접근
  ```kotlin
  class User(val name: String) {
      var address: String = "unspecified"
          set(value: String) {
              println("""
                  Address was changed for $name:
                  "$field" -> "$value".""".trimIndent()) //field로 값 읽기
              field = value //backing field 값 변경
          }
  }
  ```
  - field라는 식별자를 통해 뒷받침하는 필드에 접근할 수 있다.
  - Getter에서는 field 값을 읽을 수만 있고, Setter에서는 field 값을 읽거나 쓸 수 있다.
  - field를 사용하지 않는 커스텀 접근자를 구현하면 뒷받침하는 필드는 존재하지 않는다.
    - (val인 경우 Getter에 field가 없으면 되지만, var인 경우 Getter/Setter 모두 없어야 한다.)

- 접근자의 가시성 변경
  - 접근자의 가시성은 기본적으로 프로퍼티의 가시성과 같다.
  - 변경을 원한다면 get이나 set 앞에 가시성 변경자를 추가한다.
    ```kotlin
    class LengthCounter {
        var counter: Int = 0
            private set //Setter를 private으로 만든다.
    }
    ```

##### 데이터 클래스와 클래스 위임
- 모든 클래스가 정의해야 하는 메서드
  - toString()
    - 기본 제공되는 객체의 문자열 표현은 Client@5e9f23b4와 같은 방식인데, 기본 구현을 바꾸려면 toString()을 오버라이드 해야 한다. (자바와 동일)
  - equals()
    - 객체의 동등성 검사
    - 코틀린에서는 == 연산자로 equals 검사를 한다.
    - 참조 동일성 검사는 ===을 이용한다.
    
- data class
  - 데이터를 저장하는 역할만 하는 클래스
  - data class는 필요한 메서드를 컴파일러가 자동으로 만들어 준다.
    ```kotlin
    equals()
    hashCode()
    toString()
    ```
  - equals와 hashCode는 주 생성자에 나열된 모든 프로퍼티를 고려해 만들어진다.
    - 주 생성자 밖에 정의된 프로퍼티는 고려 대상이 아니다.

- copy()
  - 데이터 클래스의 모든 프로퍼티를 읽기 전용으로 만들어서 불변 클래스로 만드는 것을 권장한다.
    - (HashMap 등의 컨테이너에 객체를 담는 경우에는 불변성이 필수적이다.)
  - 다중 스레드 프로그램인 경우 불변성이 더욱 중요하다.
  - 데이터 클래스를 불변 객체로 더 쉽게 활용할 수 있도록 객체를 복사하면서 일부 프로퍼티를 바꿀 수 있는 copy 메서드를 제공한다.

- 클래스 위임: by
  - by 키워드를 통해 그 인터페이스에 대한 구현을 다른 객체에 위임중이라는 것을 명시할 수 있다.
    ```kotlin
    class CountingSet<T>(
            val innerSet: MutableCollection<T> = HashSet<T>()) : MutableCollection<T> by innerSet { //MutableCollection의 구현을 innerSet에 위임한다.
        var objectsAdded = 0
        override fun add(element: T): Boolean { //이 메서드는 위임하지 않고 새로운 구현을 제공한다.
            objectsAdded++
            return innerSet.add(element)
        }
    }
    ```

##### object: 클래스 선언과 인스턴스 생성
- 클래스를 정의하면서 동시에 인스턴스를 생성한다.
  - object는 싱글턴 정의 방법 중 하나이다.
  - companion object(동반객체)는 인스턴스 메서드는 아니지만 어떤 클래스와 관련있는 메서드와 팩토리 메서드를 담을 때 쓰인다.
  - object 식은 자바의 무명 내부 클래스 대신 쓰인다.

- 객체 선언: 싱글턴 쉽게 만들기
  - 클래스 선언과 클래스에 속한 단일 인스턴스의 선언을 합친 말이다.
  - 객체 선언에 생성자는 쓸 수 없다.
  - 싱글턴 객체가 객체 선언문이 있는 위치에서 즉시 만들어진다.
  - 구현 내부에 다른 상태가 필요하지 않은 경우에 유용하다. 
  - 코틀린 객체를 자바에서 사용할 때
    - 코틀린 객체 선언은 유일한 인스턴스에 대한 정적 필드가 있는 자바 클래스로 컴파일된다.
    - 인스턴스 필드 이름은 항상 INSTANCE 이다.

- 동반 객체: 팩토리 메서드와 정적 멤버가 들어갈 장소
  - 코틀린은 static 키워드를 지원하지 않는다.
  - 대신 최상위 함수와 객체 선언을 제공한다. (대부분의 경우 최상위 함수를 더 권장한다.)
  - 최상위 함수는 클래스의 private 멤버에 접근할 수 없다.
    - 클래스 내부 정보에 접근해야 하는 함수가 필요할 때는 클래스에 중첩된 객체 선언의 멤버 함수로 정의해야 한다.
  - 동반 객체는 companion object 키워드로 만들 수 있다.
    - 동반 객체는 자신을 둘러싼 클래스의 모든 private 멤버, private 생성자에 접근할 수 있다.
    - 팩토리 패턴을 구현하기 적합하다.
  ```kotlin
  class User private constructor(val nickname: String) { //주 생성자를 비공개로 만든다.
      companion object {
          fun newSubscribingUser(email: String) = //동반 객체 안에 팩토리 메서드를 정의한다.
              User(email.substringBefore('@'))
          fun newFacebookUser(accountId: Int) =
              User(getFacebookName(accountId))
      }
  }
  ```

- 동반 객체를 일반 객체처럼 사용
  - 동반 객체는 클래스 안에 정의된 일반 객체이다.
    - 객체에 이름을 붙이거나, 동반 객체가 인터페이스를 상속하거나, 동반 객체 안에 확장 함수와 프로퍼티를 정의할 수 있다.
    ```kotlin
    class Person(val name: String) {
        companion object Loader { //동반 객체에 이름을 붙인다.
            fun fromJSON(jsonText: String): Person = ...
        }
    }
    //접근할 때
    Person.Loader.fromJSON(...)
    ```
  - 동반 객체에서 인터페이스 구현
    ```kotlin
    interface JSONFactory<T> {
        fun fromJSON(jsonText: String): T
    }
    class Person(val name: String) {
        companion object : JSONFactory<Person> {
            //...
        }
    }
    ```
    - 동반 객체의 인스턴스를 전달할 수도 있다.
  ```kotlin
  fun loadFromJSON<T>(factory: JSONFactory<T>): T {
      //...
  }
  ```
    - loadFromJSON(Person) //인스턴스를 넘길 때 Person 클래스(동반 객체를 감싸는 클래스)의 이름으로 넘긴다.

  - 동반 객체 확장
    - 클래스에 동반 객체가 있으면 그 객체 안에 함수를 정의함으로써 클래스에 대해 호출할 수 있는 확장 함수를 만들 수 있다.

- 객체 식: 무명 내부 클래스를 다른 방식으로 작성
  - 무명 객체(anonymous object)를 정의할 때도 object 키워드를 쓴다.
  ```kotlin
  window.addMouseListener(
      object : MouseAdapter() {
          override fun mouseClicked(e: MouseEvent) {
              // ...
          }
          override fun mouseEntered(e: MouseEvent) {
              // ...
          }
      }
  }
  ```
  - 객체에 이름을 붙여야 한다면 변수에 무명 객체를 대입하면 된다.
  - 코틀린의 무명 클래스는 여러 인터페이스를 구현하거나 클래스를 확장하면서 인터페이스를 구현할 수 있다.
  - 무명 객체는 싱글턴이 아니다. 자바와 달리 final이 아닌 변수도 객체 식 안에서 사용할 수 있다.

#### 출처
- Kotlin In Action (http://www.yes24.com/Product/Goods/55148593)
