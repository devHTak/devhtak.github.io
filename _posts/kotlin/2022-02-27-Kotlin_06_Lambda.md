---
layout: post
title: Kotlin 람다와 고차함수
summary: Kotlin
author: devhtak
date: '2022-02-27 14:41:00 +0900'
category: Kotlin
---

#### 람다

##### 람다 식과 멤버 참조
- 람다는 자바 8에 도입되어 자바에서도 비로소 람다를 사용할 수 있게 되었다.
  - 자바에서 익명 클래스를 통해 버튼 클릭 리스너 구현
  
  ```java
  /* 자바 */
  button.setOnClickListener(new OnClickListener() {
      @Override
      public void onClick(View view) {
          /* 클릭시 동작 */
      }
  });
  ```
- 람다로 리스너 구현
  ```kotlin
  button.setOnClickListener { /* 클릭시 동작 */}
  ```
  - 두 구현은 동일한 동작이지만 람다를 이용한 경우 코드가 훨씬 간결하고 읽기 쉬워진다.

- 람다와 컬렉션
  - 컬렉션에서 가장 큰 값 찾기: 직접 구현
  ```kotlin
  fun findTheOldest(people: List<Person>) {
      var maxAge = 0
      var theOldest: Person? = null

      for (person in people) {
          if (person.age > maxAge) {
              maxAge = person.age
              theOldest = person
          }
      }
      println(theOldest)
  }

  val people = listOf(Person("Alice", 29), Person("Bob", 31))
  findTheOldest(people)
  ```
  - 컬렉션에서 가장 큰 값 찾기: 람다 이용
  ```kotlin
  val people = listOf(Person("Alice", 29), Person("Bob", 31))
  println(people.maxBy { it.age })

  println(people.maxBy(Person::age))
  ```
    - 자바 컬렉션에 대해 (자바 8 이전, 람다 지원 전) 수행하던 대부분의 불편했던 작업들은 람다나 멤버 참조를 인자로 취하는 라이브러리 함수를 통해 개선이 가능하다. 
    - 이렇게 개선된 후에는 코드가 한결 짧아지고 더 이해하기 쉬워졌다.

- 람다 식의 문법
  ```kotlin
  { x: Int, y: Int -> x + y } // 파라미터 -> 본문
  ```
  - 람다는 값처럼 여기저기 전달할 수 있는 동작의 모음이다.
  - 추가로 람다를 따로 선언해서 변수에 저장도 가능하다.
    - 그렇다곤 해도 보통은 함수에 인자로 넘기면서 바로 람다를 정의하는 경우가 대부분이다.
  ```kotlin
  val sum = { x: Int, y: Int -> x + y}
  println(sum(1, 2))
  ```

  - 람다식을 직접 호출도 가능하다.
  ```kotlin
  { println(42) }()
  ```
    - 굳이 이렇게 쓰는 것보다는 가독성 때문에라도 람다 본문을 직접 실행하는 편이 낫다.
      ```kotlin
      // 인자로 받은 람다를 실행해주는 라이브러리 함수인 run을 사용하여 람다 본문 실행
      run { println(42) }
      ```
      - 코틀린 람다 식은 항상 중괄호로 둘러싸여있다.
      - 인자 목록 주변에 괄호가 없다.
      - 화살표(->)를 통해 인자 목록과 람다 본문을 구분한다.

- 람다식 사용 예제
  ```kotlin
  val people = listOf(Person("Alice", 29), Person("Bob", 31))
  println(people.maxBy { it. age })

  people.maxBy({ p: Person -> p.age})
  people.maxBy() { p: Person -> p.age}

  people.maxBy { p: Person -> p.age}
  ```
    - 간단한 경우라면 괄호 없이 람다식만으로 명시해도 나쁘지 않을 것으로 생각되지만 웬만하면 괄호를 사용하여 해당 람다식이 메소드에 포함된 람다식이라는 것을 확실히 하는 것이 가독성에 더 좋을 것 같다.

- 이름 붙인 인자를 사용해 람다 넘기기
  ```kotlin
  val people = listOf(Person("이몽룡", 29), Person("성춘향", 31))
  val names = people.joinToString(seprator = " ", transform = { p: Person -> p.name })
  println(names)

  val names = people.joinToString(" ") { p: Person -> p.name}
  ```

  - 람다 파라미터 타입 제거하기
  ```kotlin
  people.maxBy { p: Person -> p.age }
  people.maxBy { p -> p.age } // 파라미터 타입 생략 (컴파일러가 추론)

  // 변수에 람다식 담는 경우 컴파일러가 타입 추론 불가
  val getAge = { p: Person -> p.age }
  people.maxBy(getAge)
  ```
    - 컴파일러는 람다 파라미터의 타입도 추론 할 수 있다.
    - people은 Person을 담은 컬렉션이므로 컴파일러는 Person 객체가 파라미터로 들어올 것을 추론할 수 있다.
    - 람다식을 변수에 담는 경우 파라미터의 타입을 추론할 문맥이 존재하지 않으므로 파라미터를 명시해야만 한다.

  - 디폴트 파라미터 이름 it 사용하기
  ```kotlin
  people.maxBy { it.age }
  ```
    - it는 자동 생성된 파라미터 이름이다.
    - 람다의 파라미터가 하나뿐이고 그 타입을 컴파일러가 추론할 수 있는 경우 it를 사용할 수 있다.
    - 람다 내에 람다가 중첩되는 경우나 문맥에서 람다 파라미터의 의미나 파라미터의 타입을 쉽게 알 수 없는 경우에는 파라미터를 명시적으로 선언하는 것이 가독성에 더 좋다.

  - 본문이 여러줄로 이뤄진 람다식
  ```kotlin
  val sum = { x: Int, y: Int -> 
      println("Computing the sum of $x and $y...")
      x + y
  }
  ```
    - 본문이 여러줄로 이루어진 경우 맨 마지막에 있는 식이 람다식의 결과값이 된다.
  - 현재 영역에 있는 변수에 접근
  ```kotlin
  fun printMessageWithPrefix(messages: Collection<String>, prefix: String) {
      messages.forEach {
          println("$prefix $it") // 람다 내부에서 함수의 "prefix" 변수 사용
      }
  }
  ```
    - 람다를 함수 안에서 정의하면 함수의 파라미터에 접근이 가능하다. 뿐만 아니라 정의의 앞에 선언된 로컬 변수까지 람다에서 모두 사용할 수 있다.
  ```kotlin
  fun printProblemCounts(response: Collection<String>) {
      var clientErrors = 0 // 람다 외부에 로컬 변수 선언
      var serverErrors = 0 // 람다 외부에 로컬 변수 선언

      response.forEach {
          if (it.startsWith("4")) {
              clientErrors++ // 람다 내부에서 외부의 로컬 변수 값 변경
          } else if (it.startsWith("5")) {
              serverErrors++ // 람다 내부에서 외부의 로컬 변수 값 변경
          }
      }
      println("$ClientErrors client errors, $serverErrors server errors")
  }
  ```
    - 람다 내부에서는 final 변수가 아닌 변수에 접근이 가능하다.
    - 람다 내부에서 람다 외부의 변수 변경도 가능하다.
    - 람다 내부에서 사용하는 람다 외부 변수를 람다가 포획한 변수라고 부른다. (위 예제들의 prefix, clientErrors, serverErrors)
      - 람다를 실행 시점에 표현하는 데이터 구조는 람다에서 시작하는 모든 참조가 포함된 닫힌(closed) 객체 그래프를 람다 코드와 함께 저장해야 한다. 
      - 그런 데이터 구조를 클로저(closure) 라고 부른다. 함수를 쓸모 있는 1급 시민으로 만들려면 포획한 변수를 제대로 처리해야 하고, 포획한 변수를 제대로 처리하려면 클로저가 꼭 필요하다. 그래서 람다를 클로저라고 부르기도 한다.

- 로컬 변수의 생명주기와 함수의 생명주기가 다른 경우

  - 포획한 변수가 있는 람다를 저장해서 함수가 끝난 뒤에 실행해도 람다의 본문 코드는 여전히 포획한 변수를 읽거나 쓸 수 있다.
    - final 변수인 경우: 람다 코드를 변수 값과 함께 저장하여 함수가 끝난 뒤에도 포획한 변수에 접근이 가능하다.
    - final 변수가 아닌 경우: 변수를 특별한 래퍼로 감싸서 나중에 변경하거나 읽을 수 있게 한 다음, 래퍼에 대한 참조를 람다 코드와 함께 저장한다.
    - 코틀린에서도 자바와 같이 약간의 꼼수(?)로 변경 가능한 변수를 포획하게 된다.
  ```kotlin
  // 실제 코드
  var counter = 0
  val inc = { counter++ }

  // 내부 동작을 보여주는 코드
  class Ref<T>(var value: T)
  val counter = Ref(0)
  val inc = { counter.value++ }
  ```
    - Ref 라는 클래스로 래핑하여 해당 클래스를 final 하게 선언하고 그 내부에 멤버 변수에 counter 값을 저장한다.
    - 그 이후 람다식에서는 클래스의 변수값에 접근하여 변경 가능한 변수를 포획한다.

- 람다를 이벤트 핸들러 등 비동기 실행 코드로 활용하는 경우

  ```kotlin
  fun tryToCountButtonClicks(button: Button): Int {
      var clicks = 0
      button.onClick { clicks++ }
      return clicks
  }
  ```
    - 람다를 비동기적으로 실행되는 코드로 활용하는 경우 함수 호출이 끝난 다음에 로컬 변수가 변경될 수도 있다는 점을 인지해 유의하여 사용해야 한다.
    - 위 예시 코드에서 해당 함수는 항상 0을 반환한다.
      - onCiick 핸들러는 버튼이 클릭될 때마다 clicks 변수를 증가시키지만 그 때에는 함수 호출이 종료된 이후이기 때문이다.
      - 즉, 해당 clicks 변수를 확인할 수 있도록 클래스의 프로퍼티나 전역 프로퍼티 등의 위치로 빼서 나중에 해당 변수를 확인할 수 있도록 해야 한다.

- 멤버 참조
  - 이미 선언된 함수를 값으로 사용해야 할 때 멤버 참조 :: 를 사용하면 된다.
  ```kotlin
  people.maxBy(Person::age) // 멤버 참조
  people.maxBy { p -> p.age }
  people.maxBy { it.age }

  fun Person.isAdult() = age >= 21
  val predicate = Person::isAdult // 확잠 함수도 동일하게 멤버 참조를 사용할 수 있음

  fun salute() = println("Salute!")
  run(::salute) // 최상위 함수 참조

  // sendEmail 함수에게 작업 위임
  val action = { person: Person, message: String -> sendEmail(person, message) }
  // 람다 대신 멤버 참조 사용
  val nextAction = ::sendEmail

  // 생성자 참조
  data class Person(val name: String, val age: Int)
  val createPerson = ::Person // 생성자 참조 저장
  val p = createPerson("Alice", 29) // 생성자 참조를 이용해 인스턴스 생성
  ```
    - 멤버 참조는 프로퍼티나 메소드를 단 하나만 호출하는 함수 값을 만들어준다.
    - :: 는 클래스 이름과 참조하려는 멤버(프로퍼티나 메소드) 이름 사이에 위치한다.
    - 멤버 참조 뒤에는 괄호를 넣으면 안된다.
    - 멤버 참조는 그 멤버를 호출하는 람다와 같은 타입이다.
    - 최상위 함수, 최상위 프로퍼티 참조도 가능하다.
      - 클래스 이름을 생략하고 :: 로 참조를 바로 시작하면 된다.
    - 생성자 참조를 사용하면 클래스 생성 작업을 연기하거나 저장해둘 수 있다.
      - :: 뒤에 클래스 이름을 넣으면 생성자 참조를 만들 수 있다.

- 바운드 멤버 참조 (1.1부터 사용 가능)
  ```kotlin
  // 1.0 멤버 참조
  val p = Person("Dmistry", 34)
  val personAgeFunction = Person::age
  println(personAgeFunction(p))

  // 1.1 바운드 멤버 참조
  val p = Person("Dmistry", 34)
  val ageFunction = p::age // p에 엮인 멤버 참조
  println(ageFunction())
  ```

##### 컬렉션 함수형 API

- filter 함수
  ```kotlin
  val list = listOf(1, 2, 3, 4)
  println(list.filter { it % 2 == 0 }) // 짝수만 filtering

  val personList = listOf(Person("Bob", 31), Person("Alice", 29))
  val filterList = personList.filter { it.age > 30 }
  println(filterList)
  ```
  - 컬렉션을 이터레이션하면서 주어진 람다에 각 원소를 넘겨 람다가 true인 원소를 모은다.
  - 만족하는 원소들을 모아 새로운 컬렉션으로 반환한다.

- map 함수
  ```kotlin
  val list = listOf(1, 2, 3, 4)
  println(list.map { it * it }) // 자기자신을 곱함

  val personList = listOf(Person("Bob", 31), Person("Alice", 29))
  val mapList = personList.map { it.age } // 나이만으로 컬렉션을 만듦
  println(mapList)

  // 멤버 참조 사용
  val memberRefMapList = personList.map(Person::name)
  println(memberRefMapList)
  ```
  - 주어진 람다를 컬렉션의 각 원소에 적용한 결과를 모아서 새 컬렉션을 만든다.

- maxBy + filter 조합
  ```kotlin
  val list = listOf(Person("Bob", 31), Person("Alice", 29))
  val filterAndMaxBy = list.filter { it.age == list.maxBy(Person::age)!!.age}
  ```
  - 위 코드의 단점은 filter가 이터레이션하기 때문에 maxby 함수가 컬렉션 수 만큼 호출되며 처리된다는 것이다.
  ```kotlin
  val list = listOf(Person("Bob", 31), Person("Alice", 29))
  val maxAge = list.maxBy(Person::age)!!.age
  val filterAndMaxBy = list.filter { it.age == maxAge }
  ```
  - 이터레이션 된다는 것을 항상 기억하고 불필요한 작업을 반복하지 않도록 유의해야 한다.

- 컬렉션 맵에서의 filter, map
  ```kotlin
  val numbers = mapOf(0 to "zero", 1 to "one", 2 to "two", 3 to "three", 4 to "four")
  val filterValuesMap = numbers.filterValues { it == "zero"}
  val mapValuesMap = numbers.mapValues { it.value.toUpperCase() }
  println(filterValuesMap)
  println(mapValuesMap)

  val filterKeysMap = numbers.filterKeys { it == 1 }
  val mapKeysMap = numbers.mapKeys { it.key % 2 }
  println(filterKeysMap)
  println(mapKeysMap)
  ```
  - 맵에서의 filter와 map은 별도의 API가 존재한다.
  - 맵의 filterValues, filterKeys 의 it 는 각각 value와 key를 가르킨다.

- 컬렉션에 술어 사용: all, any, count, find
  ```kotlin
  val list = listOf(Person("Alice", 27), Person("Bob", 31), Person("hzoou", 25), Person("txxbro", 28), Person("iyj", 28), Person("WooVictory", 27))

  // 술어 선언
  val canBeInClub27 = { p: Person -> p.age <= 27 }
  println("all: ${list.all(canBeInClub27)}")
  println("any: ${list.any(canBeInClub27)}")
  println("count: ${list.count(canBeInClub27)}")
  println("find: ${list.find(canBeInClub27)}")
  ```
  - all: 컬렉션의 모든 원소가 조건을 만족하는지 판단
  - any: 컬렉션의 모든 원소 중 하나라도 조건을 만족하는지 판단
  - count: 조건을 만족하는 원소의 갯수를 반환
  - find: 조건을 만족하는 첫 번째 원소를 반환, 만족하는 원소가 없을 경우 null을 반환

  - 함수형 API 사용시 고려할 점
    - 함수형 API count 와 컬렉션에 포함된 함수 size() 의 차이?
    - count의 경우 조건을 만족하는 원소의 개수만 추적할 뿐 원소를 따로 저장하지 않는다.
    - size()의 경우 만족하는 원소를 가진 객체를 생성 시키게 된다.
    - 위 예제 코드의 결과에서 보듯이 all과 any는 서로 부정으로 대응한다. 하지만 가독성을 이유로 any 대신 !all 이나 all 대신 !any는 사용하지 않는 것이 좋다.

- groupBy
  ```kotlin
  val list = listOf(Person("Alice", 27), Person("hzoou", 25), Person("txxbro", 28), Person("iyj", 28), Person("WooVictory", 27))

  println("groupBy: ${list.groupBy { it.age }}")

  val strs = listOf("12", "345", "11", "456")
  println(strs.groupBy { it.length })
  ```
  - groupBy: 리스트를 특정 기준에 맞춰 맵으로 변경하여 반환
  - key, value(list)

- flatMap과 flatten: 중첩된 컬렉션 안의 원소 처리
  ```kotlin
  val strings = listOf("abc", "def")
  println(strings.flatMap { it.toList() })

  data class Book(val title: String, val authors: List<String>)

  val books = listOf(Book("책1", listOf("작가1")),
                   Book("책2", listOf("작가2", "작가3")), 
                   Book("책3", listOf("작가4", "작가1")))

  println("toSet(): ${books.flatMap { it.authors }.toSet()}")
  println("기본: ${books.flatMap { it.authors }}")

  println("flatten(): ${books.map { it.authors }.flatten()}")
  ```
  - flatMap: 인자로 주어진 람다를 컬렉션의 모든 객체에 적용(매핑)하고 람다를 적용한 결과 얻어지는 여러 리스트를 한 리스트로 모은다(flatten). 즉, 리스트의 리스트가 있을 때 중첩된 리스트의 원소를 한 리스트로 모을 때 사용한다.
  - toSet(): 컬렉션의 중복을 제거리
  - flatten(): 변환할 내용 없이 펼치기만 하는 경우 사용

##### 지연 계산(lazy) 컬렉션 연산
  - 콜렉션의 연산자(e.g. map, filter)는 결과 컬렉션을 즉시 생성한다. 이는 컬렉션 함수를 연쇄하면 매 단계마다 계산 중간 결과를 새로운 컬렉션에 임시로 담는다는 말이다.
  - 시퀀스(sequence)를 사용하면 중간 임시 컬렉션을 사용하지 않고도 컬렉션 연산을 연쇄할 수 있다.
  ```kotlin
  poeple.map(Person::name).filter { it.startsWith("A") }
  ```
  - map과 filter는 둘 다 리스트를 반환한다. 즉 위 코드에서 연쇄 호출로 인해 리스트를 2개 만들어졌다.
  ```kotlin
  people.asSequence() // 원본 컬렉션을 시퀀스로 변환
      .map(Person::name).filter { it.startsWith("A")}
      .toList() // 결과 시퀀스를 다시 리스트로 변환
  ```
  - 시퀀스의 원소는 필요할 때 비로소 계산되기 때문에 중간 처리 결과를 저장하지 않고도 연산을 연쇄적으로 적용해 효율적으로 계산을 수행할 수 있다.
  - asSequence(): 어떤 컬렉션이든 시퀀스로 바꿀 수 있다.
  - toList(): 시퀀스를 리슷트로 바꿀 때 사용한다.

- 시퀀스 연산 실행: 중간 연산과 최종 연산
  - 중간 연산: 다른 시퀀스를 반환하며 최초 시퀀스의 원소를 변환하는 방법을 알고 있다.
    - 항상 지연 계산된다. 즉, 최종 연산을 하지 않으면 계속 지연이 되어 결과를 반환하지 않는다.
  - 최종 연산: 최초 컬렉션에 대해 변환을 적용한 시퀀스로부터 일련의 계산을 수행해 얻을 수 있는 컬렉션이나 원소, 숫자, 객체이다.
    - 즉시 계산의 수행 순서와 지연 계산의 수행 순서
  ```kotlin
  listOf(1, 2, 3, 4).map { println("eagerly map($it)"); it * it }
                  .filter { println("eagerly filter($it)"); it % 2 == 0 }
  listOf(1, 2, 3, 4).asSequence()
                  .map { println("lazy map($it)"); it * it}
                  .filter { println("lazy filter($it)"); it % 2 == 0 }
                  .toList()
  ```
    - 즉시 계산의 경우 모든 원소에 대해 먼저 map을 끝낸 후 이후 filter를 수행하게 된다.
    - 시퀀스(지연 계산)의 경우 각 원소에 대해 순차적으로 적용이 된다.

- map과 filter 호출 순서에 따른 성능 차이의 발생
  ```kotlin
  val list = listOf(Person("Alice", 27), Person("hzoou", 25), Person("txxbro", 28), Person("iyj", 28), Person("WooVictory", 27))

  list.asSequence().map(Person::name) // map 먼저 실행
      .filter { it.length < 4 }.toList()

  list.asSequence().filter { it.length < 4 } // filter 먼저 실행
      .map(Person::name).toList()
  ```
  - filter 보다 map을 호출할 경우 map은 모든 원소를 변환하므로 더 많은 이터레이션이 발생하게 된다.

- 자바 스트림과 코틀린 시퀀스 비교
  - 자바 8의 스트림과 코틀린의 시퀀스는 개념적으로 같다.
    - 다만, 자바 8일 경우 코틀린 컬렉션과 시퀀스에서 제공하지 않는 스트림 연산(map과 filter)을 여러 CPU에서 병렬적으로 실행하는 기능이 존재한다. 
    - 그렇기 때문에 자바 버전에 따라서 시퀀스와 스트림 중에 적절한 것을 사용하면 된다.
    - 자바 8에 대해서는 다른 개발자의 블로그의 글인 자바 8 스트림 이란? 을 참고하자.

- 시퀀스 만들기
  ```kotlin
  val numbers = generateSequence(0) { it + 1 } // 시퀀스 생성
  val numbersTo100 = numbers.takeWhile { it <= 100 } // while loop 시퀀스 생성
  println(numbersTo100.sum()) // 위의 모든 시퀀스는 sum의 결과를 계산할 때 수행된다.

  fun File.isInsideHiddenDirectory() = generateSequence(this) { it.parentFile }.any { it.isHidden }
  val file = File("/Users/svtk/.HiddenDir/a.txt")
  println(file.isInsideHiddenDirectory())
  ```
  - generateSequence: 이전의 원소를 인자로 받아 다음 원소를 계산하는 시퀀스를 만드는 함수
  - 최종 연산인 sum() 을 호출 하기 전에는 계산되지 않다가 최종 연산이 호출될 때에 계산이 수행된다.
  
##### 자바 함수형 인터페이스 활용
- 함수형 인터페이스
  - 추상 메소드가 단 하나 있는 인터페이스를 함수형 인터페이스 또는 SAM(단일 추상 메소드, Single Abstract method) 인터페이스라고 한다.
  ```java
  button.setOnClickListener(new OnClickListener() {
      @Override
      public void onClick(View v) {
          /* TODO */
      }
  })
  
  // java 8 이후 함수형 인터페이스를 람다로 표현
  button.setOnClickListener {view -> /* TODO */}
  ```
    - 자바에서는 함수형 인터페이스 즉, SAM 인터페이스인 경우 자바 8버전 이후 람다를 이용하여 더 간결하게 표현할 수 있다. (코틀린도 너무나 당연하게 사용 가능하다.)

- 자바 메소드에 람다를 인자로 전달
```java
// java 함수형 인터페이스를 인자로 전달
void postponeComputation(int delay, Runnable computation);
```
```java
// 위의 자바 코드에 코틀린에서 람다를 전달하여 호출
postponeComputation(1000) { println(42) } // 함수형 인터페이스에 람다를 전달

// 객체 식을 전달
postponeComputation(1000, object: Runnable {
    override fun run() {
        println(42)
    }
})
```
  - 컴파일러는 자동으로 람다를 Runnable 인스턴스(Runnable을 구현한 익명 클래스 인스턴스)로 변환하여 전달한다.
  - Runnable을 구현하는 무명 객체를 명시적으로 만들어서 사용하는 것도 가능하다.
  
- 람다를 넘길 때와 무명 객체를 생성하여 넘길 때의 차이점
  - 무명 객체를 생성하여 넘기는 경우, 메소드를 호출할 때마다 새로운 인스턴스가 생성된다.
  - 생성된 Runnable 인스턴스는 단 하나만 생성되며 메소드 호출 시 반복 사용된다.
  - 단, 람다 내에서 람다 외부의 변수를 포획하는 경우에는 무명 객체처럼 새로운 인스턴스가 생성된다.

- SAM 생성자: 람다를 함수형 인터페이스로 명시적으로 변경
  - 컴파일러가 자동으로 람다를 함수형 인터페이스 익명 클래스로 바꾸지 못하는 경우 SAM 생성자를 사용한다.
  ```kotlin
  val listener = OnClickListener { view -> 
      val text = when (view.id) {
          R.id.button1 -> "First button"
          R.id.button2 -> "Second button"
          else -> "Unknown button"
      }
      toast(text)
  }

  button1.setOnClickListener(listener)
  button2.setOnClickListener(listener)
  ```

- 람다와 리스너 등록/해제
  - 람다는 코드 블럭이기 때문에 this 가 없다. 즉, 객체처럼 익명 클래스의 인스턴스를 참조할 수 없다.
  - 람다 내에서 this는 그 람다를 둘러싼 클래스의 인스턴스를 가르킨다. 주의하자.
  - 리스너를 가르키고 싶다면 람다가 아닌 무명 객체를 사용해야 한다.
  - 무명 객체 내에서 this는 객체 인스턴스 자신을 가르킨다.

##### 수신 객체 지정 람다: with와 apply

- 자바의 람다에는 없는, 코틀린 람다만의 독특한 기능인 수신 객체 지정 람다는 수신 객체를 명시하지 않고 람다의 본문 안에서 다른 객체의 메소드를 호출할 수 있게 하는 것이다.

- with 함수
  - with 함수는 파라미터가 2개인 메소드로 첫 번째 인자는 객체를 두 번째 인자는 람다를 받는다.
  - 첫 번째 인자로 받은 객체를 두 번째 인자로 받은 람다의 수신 객체로 만든다.
  ```kotlin
  fun alphabet(): String {
      val sb = StringBuilder()
      return with(sb) {
          for (letter in 'A'..'Z') {
              this.append(letter) // this를 통해 수신 객체에 접근
          }
          append("\nNow I know alphabet!") // this 없이 수신 객체의 메소드 호출
          this.toString() // 람다에서 값 반환
      }
  }
  ```
    - 수신 객체 지정 람다는 확장 함수와 비슷한 동작을 정의하는 한 방법이다.
    - <T, R> with(receiver: T, block: T.() ‐> R): block 함수의 수신 객체는 T

- apply 함수
  - apply 함수는 with 함수와 동일한 동작이지만 항상 자신에게 전달된 객체(수신 객체)를 반환한다.
  ```kotlin
  fun alphabet() = StringBuilder().apply {
      for (letter in 'A'..'Z') {
          append(letter)
      }
      append("\nNow I know the alphabet!")
  }.toString()
  ```
    - fun <T> T.apply(block: T.() ‐> Unit): T: apply 함수는 확장 함수로 정의되어 있다.
    - 객체의 인스턴스를 만들면서 즉시 프로퍼티 중 일부를 초기화 해야 하는 경우 유용하다.

- buildString 함수
  - buildString 함수는 StringBuilder 객체를 만들어 toString()을 호출해주는 작업을 해준다.

  ```kotlin
  fun alphabet() = buidlString {
      for (letter in 'A'..'Z') {
          append(letter)
      }
      append("\nNow I know the alphabet!")
  }
  ```

#### 고차함수
  
##### 고차 함수 정의
- 고차 함수는 다른 함수를 인자로 받거나 함수를 반환하는 함수다. 
  - 코틀린에서는 람다나 함수 참조를 사용해 함수를 값으로 표현할 수 있다. 
  - 고차 함수는 람다나 함수 참조를 인자로 넘길 수 있거나 람다나 함수 참조를 반환하는 함수다.

##### 함수 타입
- 함수 타입을 정의하려면 함수 파라미터의 타입을 괄호 안에 넣고, 그 뒤에 화살표(→)를 추가한 다음, 함수의 반환 타입을 지정하면 된다.
- Unit 타입은 의미 있는 값을 반환하지 않는 함수 반환 타입에 쓰는 특별한 타입이다. 
  - 그냥 함수를 정의한다면 함수의 파라미터 목록 뒤에 오는 Unit 반환 타입 지정을 생략해도 되지만, 함수 타입을 선언할 때는 반환 타입을 반드시 명시해야 하므로 Unit을 빼먹어서는 안 된다.

- 인자로 받은 함수 호출
  - 인자로 받은 함수를 호출하는 구문은 일반 함수를 호출하는 구문과 같다.
  ```kotlin
  fun twoAndThree(operation: (Int, Int) -> Int) {
      val result = operation(2, 3)
      println("The result is $result")
  }

  fun main(args: Array<String>) {
      twoAndThree { a, b -> a + b }
      twoAndThree { a, b -> a * b }
  }
  ```
  - 디폴트 값을 지정한 함수 타입 파라미터나 널이 될 수 있는 함수 타입 파라미터
    - 파라미터를 함수 타입으로 선언할 때도 디폴트 값을 정할 수 있다.
  ```kolin
  fun <T> Collection<T>.joinToString(
          separator: String = ", ",
          prefix: String = "",
          postfix: String = "",
          transform: (T) -> String = { it.toString() } // 함수 타입 파라미터를 선언하면서 람다를 디폴트 값으로 지정한다. 
  ): String {
      val result = StringBuilder(prefix)

      for ((index, element) in this.withIndex()) {
          if (index > 0) result.append(separator)
          result.append(transform(element)) // "transform" 파라미터로 받은 함수를 호출한다. 
      }

      result.append(postfix)
      return result.toString()
  }

  fun main(args: Array<String>) {
      val letters = listOf("Alpha", "Beta")
      println(letters.joinToString()) // 디폴트 변환 함수를 사용한다. -> Alpha, Beta
      println(letters.joinToString { it.toLowerCase() }) // 람다를 인자로 전달한다. -> alpha, beta
      println(letters.joinToString(separator = "! ", postfix = "! ",
             transform = { it.toUpperCase() })) // 이름 붙은 인자 구문을 사용해 람다를 포함하는 여러 인자를 전달한다. -> ALPHA! BETA!
  }
  ```

- 함수를 함수에서 반환
  - 다른 함수를 반환하는 함수를 정의하려면 함수의 반환 타입으로 함수 타입을 지정해야 한다. 함수를 반환하려면 return 식에 람다나 멤버 참조나 함수 타입의 값을 계산하는 식 등을 넣으면 된다.

  ```kotlin
  data class Person(
      val firstName: String,
      val lastName: String,
      val phoneNumber: String?
  )
  
  class ContactListFilters {
      var prefix: String = ""
      var onlyWithPhoneNumber: Boolean = false

      fun getPredicate(): (Person) -> Boolean { // 함수를 반환하는 함수를 정의한다. 
          val startsWithPrefix = { p: Person ->
              p.firstName.startsWith(prefix) || p.lastName.startsWith(prefix)
          }
          if (!onlyWithPhoneNumber) {
              return startsWithPrefix // 함수 타입의 변수를 반환한다. 
          }
          return { startsWithPrefix(it)
                      && it.phoneNumber != null } // 람다를 반환한다. 
      }
  }

  fun main(args: Array<String>) {
      val contacts = listOf(Person("Dmitry", "Jemerov", "123-4567"),
                            Person("Svetlana", "Isakova", null))
      val contactListFilters = ContactListFilters()
      with (contactListFilters) {
          prefix = "Dm"
          onlyWithPhoneNumber = true
      }
      println(contacts.filter(
          contactListFilters.getPredicate()))
  }
  ```

- 람다를 활용한 중복 제거
  - 함수 타입과 람다 식은 재활용하기 좋은 코드를 만들 때 쓸 수 있는 훌륭한 도구다. 웹 사이트 방문 기록을 분석하는 예를 살펴보자.
```kotlin
data class SiteVisit(
    val path: String,
    val duration: Double,
    val os: OS
)
enum class OS { WINDOWS, LINUX, MAC, IOS, ANDROID }
val log = listOf(
    SiteVisit("/", 34.0, OS.WINDOWS),
    SiteVisit("/", 22.0, OS.MAC),
    SiteVisit("/login", 12.0, OS.WINDOWS),
    SiteVisit("/signup", 8.0, OS.IOS),
    SiteVisit("/", 16.3, OS.ANDROID)
)

val averageWindowsDuration = log
    .filter { it.os == OS.WINDOWS }
    .map(SiteVisit::duration)
    .average()
```
  - 이는 함수를 사용하여 일반 함수를 통해 중복을 줄일 수 있다.
```kotlin
fun List<SiteVisit>.averageDurationFor(os: OS) =
        filter { it.os == os }.map(SiteVisit::duration).average()

fun main(args: Array<String>) {
    println(log.averageDurationFor(OS.WINDOWS))
    println(log.averageDurationFor(OS.MAC))
}
```
  - 이 함수는 충분히 강력하지 않다. 
    - 만약 모바일 디바이스(IOS, 안드로이드)의 평균 방문 시간을 구하고 싶다거나 IOS 사용자의 /signup 페이지 평균 방문 시간을 구하고 싶을 경우는 어떻게 해야 할까?
  - 이는 고차 함수를 이용하여 함수를 확장할 수 있다.
```kotlin  
fun List<SiteVisit>.averageDurationFor(predicate: (SiteVisit) -> Boolean) =
        filter(predicate).map(SiteVisit::duration).average()

fun main(args: Array<String>) {
    println(log.averageDurationFor {
        it.os in setOf(OS.ANDROID, OS.IOS) })
    println(log.averageDurationFor {
        it.os == OS.IOS && it.path == "/signup" })
}
```
##### 인라인 함수: 람다의 부가 비용 없애기
- 반복되는 코드를 별도의 라이브러리 함수로 빼내되 컴파일러가 자바의 일반 명령문만큼 효율적인 코드를 생성하게 만들 수는 없을까? 
  - inline 변경자를 어떤 함수에 붙이면 컴파일러는 그 함수를 호출하는 모든 문장을 함수 본문에 해당하는 바이트코드로 바꿔치기 해준다.

- 인라이닝이 작동하는 방식
  - 어떤 함수를 inline으로 선언하면 그 함수의 본문이 인라인된다. 
  - 다른 말로 하면 함수를 호출하는 코드를 함수를 호출하는 바이트코드 대신에 함수 본문을 번역한 바이트 코드로 컴파일한다는 뜻이다.

- 컬렉션 연산 인라이닝
  - 코틀린 표준 라이브러리의 컬렉션 함수는 대부분 람다를 인자로 받는다. 
  - filter와 map은 인라인 함수다.
    - 그 두 함수의 본문은 인라이닝되며, 추가 객체나 클래스 생성은 없다.
    - 이 코드는 리스트를 걸러낸 결과를 저장하는 중간 리스트를 만든다. 
    - 처리할 원소가 많아지면 중간 리스트를 사용하는 부가 비용도 걱정할 만큼 커진다. 
    - asSequence를 통해 리스트 대신 시퀀스를 사용하면 중간 리스트로 인한 부가 비용은 줄어든다. 
    - 이때 각 중간 시퀀스는 람다를 필드에 저장하는 객체로 표현되며, 최종 연산은 중간 시퀀스에 있는 여러 람다를 연쇄 호출한다.

- 함수를 인라인으로 선언해야 하는 경우
  - inline 키워드의 이점을 배우고 나면 코드를 더 빠르게 만들기 위해 코드 여기저기에서 inline을 사용하고 싶어질 것이다. 
    - inline 키워드를 사용해도 람다를 인자로 받는 함수만 성능이 좋아질 가능성이 높다.
  - 일반 함수 호출의 경우 JVM은 이미 강력하게 인라이닝을 지원한다. 
    - JVM은 코드 실행을 분석해서 가장 이익이 되는 방향으로 호출을 인라이닝한다. 
    - 이런 과정은 바이트코드를 실제 기계어 코드로 번역하는 과정(JIT)에서 일어난다. 
    - 이런 JVM의 최적화를 활용한다면 바이트코드에서는 각 함수 구현이 정확히 한 번만 있으면 되고, 그 함수를 호출하는 부분에서 따로 함수 코드를 중복할 필요가 없다. 
    - 반면 코틀린 인라인 함수는 바이트 코드에서 각 함수 호출 지점을 함수 본문으로 대치하기 때문에 코드 중복이 생긴다. 게다가 함수를 직접 호출하면 스택 트레이스가 더 깔끔해진다.
- inline 변경자를 함수에 붙일 때는 코드 크기에 주의를 기울여야 한다.
  - 인라이닝하는 함수가 큰 경우 함수의 본문에 해당하는 바이트코드를 모든 호출 지점에 복사해 넣고 나면 바이트코드가 전체적으로 아주 커질 수 있다.

- 자원 관리를 위해 인라인된 람다 사용
  - 자바 7부터는 자원을 관리하기 위한 특별한 구문인 try-with-resource문이 생겼다.
```java
static String readFirstLineFromFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    }
}
```
- 코틀린 언어는 이와 같은 기능을 언어 구성 요소로 제공하지는 않는다. 
  - 자바 try-with-resource와 같은 기능을 제공하는 use라는 함수가 코틀린 표준 라이브러리 안에 들어있다.
```kotlin
fun readFirstLineFromFile(path: String): String {
    BufferedReader(FileReader(path)).use { br ->
        return br.readLine()
    }
}
```
  - use 함수는 닫을 수 있는(closeable) 자원에 대한 확장 함수며, 람다를 인자로 받는다. use는 람다를 호출한 다음에 자원을 닫아준다.

- 고차 함수 안에서 흐름 제어
  - 람다 안의 return문: 람다를 둘러싼 함수로부터 반환
    - 다음 코드의 실행 결과를 보면 이름이 Alice인 경우에 lookForAlice 함수로부터 반환된다는 사실을 분명히 알 수 있다.
```kotlin
data class Person(val name: String, val age: Int)

val people = listOf(Person("Alice", 29), Person("Bob", 31))

fun lookForAlice(people: List<Person>) {
    for (person in people) {
        if (person.name == "Alice") {
            println("Found!")
            return
        }
    }
    println("Alice is not found")
}
```
    - 이 코드를 forEach로 바꿔 써도 괜찮을까?
```kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach {
        if (it.name == "Alice") {
            println("Found!")
            return
        }
    }
    println("Alice is not found")
}
```
    - 람다 안에서 return을 사용하면 람다로부터만 반환되는 게 아니라 그 람다를 호출하는 함수가 실행을 끝내고 반환된다. 
      - 그렇게 자신을 둘러싸고 있는 블록보다 더 바깥에 있는 다른 블록을 반환하게 만드는 return 문을 넌로컬(non-local) return이라 부른다.
    - 이렇게 return이 바깥쪽 함수를 반환시킬 수 있는 때는 람다를 인자로 받는 함수가 인라인 함수인 경우뿐이다. 
      - forEach는 인라인 함수이므로 람다 본문과 함께 인라이닝된다. 따라서 return 식이 바깥쪽 함수(여기서는 lookForAlice)를 반환시키도록 쉽게 컴파일할 수 있다.

  - 람다로부터 반환: 레이블을 사용한 return
    - 람다 식에서도 로컬 return을 사용할 수 있다. 
    - 람다 안에서 로컬 return은 for루프의 break와 비슷한 역할을 한다. 로컬 return과 넌로컬 return을 구분하기 위해 레이블(label)을 사용해야 한다.
```kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach label@{
        if (it.name == "Alice") return@label
    }
    println("Alice might be somewhere")
}
```
    - 람다에 레이블을 붙여서 사용하는 대신 람다를 인자로 받는 인라인 함수의 이름을 return 뒤에 레이블로 사용해도 된다.
```kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach {
        if (it.name == "Alice") return@forEach
    }
    println("Alice might be somewhere")
}
```
  - 무명 함수: 기본적으로 로컬 return
    - 무명 함수는 코드 블록을 함수에 넘길 때 사용할 수 있는 다른 방법이다.
```kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach(fun (person) {
        if (person.name == "Alice") return
        println("${person.name} is not Alice")
    })
}
```
  - 무명 함수는 일반 함수와 비슷해 보인다. 
    - 차이는 함수 이름이나 파라미터 타입을 생략할 수 있다는 점뿐이다. 
    - 무명 함수 안에서 레이블이 붙지 않은 return 식은 무명 함수 자체를 반환시킬 뿐 무명 함수를 둘러싼 다른 함수를 반환시키지 않는다. 
    - 사실 return에 적용되는 규칙은 단순히 return은 fun 키워드를 사용해 정의된 가장 안쪽 함수를 반환시킨다는 점이다. 
    - 람다 식의 구현 방법이나 람다 식을 인라인 함수에 넘길 때 어떻게 본문이 인라이닝 되는지 등의 규칙을 무명 함수에도 모두 적용할 수 있다.

#### 
