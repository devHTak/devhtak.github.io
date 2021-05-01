  
#### Reactive stream
   
- non-blocking backPressure(배압) 을 이용하여 비동기 서비스를 할 때 기본이 되는 스펙
- java의 RxJava, Spring의 WebFlux의 Core에 있는 Project Reactor 프로젝트 모두 해당 스펙을 따르고 있다.
- java 1.9 버전에 추가된 Flow 역시 reactive stream 스펙을 채택하여 사용하고 있다.
   
- blocking과 non-blocking
  - blocking: 자신의 수행 결과가 끝날 때 까지 제어권을 갖고 있는 것을 의미
  - non-blocking: 자신이 호출되었을 때 제어권을 바로 자신을 호출한 쪽으로 넘기며, 자신을 호출한 쪽에서 다른 일을 할 수 있도록 하는 것을 의미
   
- BackPressure(배압)
  - 한 컴포넌트가 부하를 이겨내기 힘들 때, 시스템 전체가 합리적인 방법으로 대응해야 한다. 
  - 과부하 상태의 컴포넌트에서 치명적인 장애가 발생하거나 제어 없이 메시지를 유실해서는 안 된다. 
  - 컴포넌트가 대처할 수 없고 장애가 발생해선 안 되기 때문에 컴포넌트는 상류 컴포넌트들에 자신이 과부하 상태라는 것을 알려 부하를 줄이도록 해야 한다. 
  - 이러한 배압은 시스템이 부하로 인해 무너지지 않고 정상적으로 응답할 수 있게 하는 중요한 피드백 방법이다.
  - 배압은 사용자에게까지 전달되어 응답성이 떨어질 수 있지만, 이 메커니즘은 부하에 대한 시스템의 복원력을 보장하고 시스템 자체가 부하를 분산할 다른 자원을 제공할 수 있는지 정보를 제공할 것이다.
   
#### Reactive Programming

```
In computing, reactive programming is a declarative programming
paradigm concerned with data streams and the propagation of change.
(변화의 전파와 데이터 흐름과 관련된 선언적 프로그래밍 패러다임이다.)
```
- 변화의 전파와 데이터 흐름: 데이터가 변경될 때마다 이벤트를 발생시켜 데이터를 계속적으로 전달한다.
- 선언적 프로그래밍: 실행할 동작을 구체적으로 명시하는 명령형 프로그래밍과 달리 선언형 프로그래밍은 단순히 목표를 선언한다.
  ```java
  // 선언형 프로그래밍 방법
  List<Integer> numbers = Arrays.asList(1, 3, 21, 10, 8, 11);
  int sum1 = numbers.stream().filter(number -> number > 6 && (number % 2 != 0)).mapToInt(number -> number).sum();
  System.out.println("선언형 프로그래밍 사용: " + sum1);
  
  int sum2 = 0;
  for(Integer i : numbers) {
      if(i > 6 && (i % 2 != 0)) {
          sum2 += i
      }
  }
  System.out.println("명령형 프로그래밍 사용: " + sum2);
  ```
  - 명령형은 for, if문등을 활용하여 구체적인 알고리즘을 명시

- 리액티브의 개념이 적용된 예
  - PUSH 방식: 데이터의 변화가 발생했을 때 변경이 발생한 곳에서 데이터를 보내주는 방식
    - 리액티브 방식
  - PULL 방식: 변경된 데이터가 있는지 요청을 보내 질의하고 변경된 데이터를 가져오는 방식

#### 마블 다이어그램

- 리액티브 프로그래밍을 통해 발생하는 비동기적인 데이터의 흐름을 시간의 흐름에 따라 시각적으로 표시한 다이어그램
- 마블 다이어그램 알아야 하는 이유
  - 문장으로 적혀 있는 리액티브 연산자(Operators)의 기능을 이해하기 어려움
  - 리액티브 연산자의 기능이 시각화되어 있어서 이해하기 쉬움
  - 리액티브 프로그래밍의 핵심인 연산자를 사용하기 위한 핵심 도구

![image](https://user-images.githubusercontent.com/42403023/116775768-b948a580-aa9f-11eb-99c0-8091ea5b5ba5.png)

  - map operator: Observable의 값을 다니며 선언된 함수에 대한 연산을 진행하여 Observable을 return한다.
    ```java
    Observable observable = Observable.just(1, 25, 9, 15, 7, 30)
       .map(x -> x * 10)
       .subscribe(x - > System.out.println(x)); // 10, 250, 90, 150, 70, 300 출력
    ```
  - filter operator: Observable의 값을 다니며 선언된 조건에 부합한 데이터만 Observable을 return한다.
    ```java
    Observable observable = Observable.just(1, 25, 9, 15, 7, 30)
       .filter(x -> x > 10)
       .subscribe(x - > System.out.println(x)); // 25, 15, 30 출력
    ```
  - just 함수: 데이터 가공, 변화하는 함수가 아닌 Observable, Flowable을 생성하는 함수이다.


#### 출처

- Kevin의 RxJava 강의

- 참고: [Java] Reactive Stream 이란? 
  (URL: https://sabarada.tistory.com/98)
