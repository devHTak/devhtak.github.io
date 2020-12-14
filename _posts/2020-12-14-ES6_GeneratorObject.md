---
layout: post
title: (Javascript ES6) Generator Object
summary: 인프런 강좌_자바스크립트 ES6+ 기본
author: devhtak
date: '2020-12-14 22:41:00 +0900'
category: Javascript ES6+
---

### Generator function

- Generator Function
  - function* 키워드를 사용한 함수
  - Generator Function 형태
    - function* 선언문, function* 표현식, GeneratorFunction
      ```javascript
      function* sports(one) {}
      const book = function*(one) {}
      const music = Object.getPrototypeOf(function* (){}).constructor;
      const gen = new music();
      ```
  - 작성 방법
    - function* 다음에 소괄호() 작성, 이어서 작성해도 되고 하나 이상 띄어도 된다.

- function* 선언문
  - 형태: functoin* fName() {}
  - 파라미터: [param[,...]] (option)
  - 반환: Generator Object
  
  - function* 다음에 함수 이름 작성
  - 제너레이터 함수를 호출하면 함수 블록{}을 실행하지 않고 Generator Object를 생성하여 반환
    ```javascript
    function* sports(one, two) {
        yield one + two;
    };
    console.log(typeof sports); // function
    const obj = sports(1, 2);
    console.log(typeof obj); // object
    console.log(obj.next()); // {value:3, done: false}
    ```
    - Generator Function의 타입은 function이다.
    - const obj = sports91, 2); 
      - sports 함수를 호출하면 Generator Object를 생성하여 반환
      - 이 때, 함수 코드를 실행하지 않는다. 파라미터 값은 생성한 오브젝트에 설정된다.
      - new 연산자를 사용할 수 없다. 단일 함수로 사용
    - typeof obj
      - 생성한 Generator Object 타입은 object
    - obj.next()
      - Generator Object가 iterator object이므로 next() 함수를 호출할 수 있으며 이 때 함수코드가 실행된다.
  - Generator Object는 iterator object
  - 함수 코드 실행
    - Generator object의 메소드를 호출할 때

- function* 표현식
  - 형태: const name = function*() {}
  - 파라미터: [param [, ...]]; (option)
  - 반환: Generator Object
  
  - function* 다음에 함수 이름 작성은 선택
    - 일반적으로 함수 일므은 작성하지 않음
    - function* 왼쪽에 변수를 선언하며 변수 이름이 함수 이름이 된다.
      ```javascript
      const sports = function* (one) {
          yield one;
      };
      const obj = new sports(100);
      console.log(obj.next()); // {value: 100, done: false}
      ```
  - 함수를 선언하는 형태만 다를 뿐 다른 것은 function* 선언문과 같다.
  
- GeneratorFunction
  - 형태: new GenerationFunction()
  - 파라미터: [param, [, ...]], functionBody(option)
  - 반환: Generator Object
  
  - GeneratorFunction.constructor를 사용하야 제너레이터 함수를 생성
    - parameter를 문자열로 작성
      ```javascript
      const fn = new Function("one", "return one");
      console.log(fn(100)); // 100
      
      const create = Object.getPrototypeOf( function*(){}).constructor;
      console.log(create); // function GeneratorFunction() { [native code] }
      
      const sports = new create("one", "yield one");
      console.log(typeof sports); // function
      
      const obj = sports(100);
      console.log(obj.next()); // {value: 100, done: false}
      ```
      - const create = Object.getPrototypeOf( function*(){}).constructor;
        - 제너레이터 함수를 생성하는 cosntructor(생성자)를 할당
        - constructor가 할당되므로 new 연산자로 생성자 함수를 호출할 수 있다.
      - sports = new create(param)
        - GeneratorFunction을 사용하여 제너레이터 함수를 생성하고 sports 변수 할당
        - param에 파라미터와 함수 코드 작성
      - const obj = sports(100);
        - Generator Function을 호출
        - Generator Object 생성, 반환
        - 함수 코드를 실행하지는 않고, 100이 one에 매핑된다.
      - obj.next()
        - Generator Object는 Iterator Object이며 obj에 iterator object가 할당되어 있으므로 next()를 호출할 수 있다.
        
    - 마지막 파라미터가 함수 코드가 되고 앞은 파라미터 이름이 된다.
  - Generator Function 구조
    - generator function을 펼치면 prototype이 있다.
      - 이것을 펼치면 cosntructor가 있어야 하는데 없고, 메서드 또한 없다.
    - __proto__가 있으며 이것을 펼치면 constructor가 있다.
      - __proto__에 다른 오브젝트의 prototype에 연결된 프로퍼티를 인스턴스 개념으로 생성하여 첨부한 것이 표시된다.
    - 즉, GeneratorFunction의 constructor가 첨부된 것
  
### yield 키워드

- syntax: [returnValue] = yield[표현식];
- yield 키워드 사용 형태
  - next()로 호출할 때마다 하나씩 실행
    ```javascript
    function* sports(one) {
        yield one + 10;
        yield;
        const value = yield one + 50;
    };
    const obj = sports(30);
    console.log(obj.next()); // {value: 40, done: false} 
    console.log(obj.next()); // {value: undefined, done: false}
    console.log(obj.next()); // {value: 80, done: false}
    console.log(obj.next()); // {value: undefined, done: true}
    ```
- yield 키워드는 
  - 제너레이터 함수 실행을 멈추거나 다시 실행할 때 사용
  - yield 오른쪽의 표현식을 평가하고 결과 반환
  - 표현식을 작성하지 않으면 undefined 반환
- [returnValue] 
  - 오른쪽의 평가 결과가 설정되지 않고 다음 next()에서 파라미터로 넘겨준 값이 설정
- yield 표현식을 평가하면 호출한 곳으로 {value: 값, done: true/false}를 반환
  ```javascript
  function* sports(one) {
      yield one;
      const check = 20;
  };
  const obj = sports(10);
  console.log(obj.next()); // {value: 10, done: false}
  console.log(obj.next()); // {value: undefined, done: true}
  ```
  - 두 번째 obj.next()에서는 그 다음 yield 키워드가 없으므로 {value: undefined, done: true} 를 반환
- value 값
  - yield 표현식의 평가 결과 설정
  - yield를 실행하지 못하면 undefined
- done 값
  - yield를 실행하면 false
  - yield를 실행하지 못하면 true

```javascript
function* sports(one) {
    const two = yield one;
    let param = yield one + two;
    yield param + one;
}
const obj = sports(10);
console.log(obj.next(); // {value: 10, done: false}
console.log(obj.next(); // {value: NaN, done: false}
console.log(obj.next(20); // {value: 30, done: false}
console.log(obj.next(); // {value: undefined, done: true}
```
- yield 옆에 있는 returnValue에 할당하지 않는다. 그래서 NaN 출력
- param은 next함수에 파라미터를 사용할 수 있다.

### next()

- 형태: generatorObject.next()
- 파라미터: 제너레이터로 넘겨줄 파라미터 값(option)
- 반환: {value: 값, done: true/false}

- next()는 yield 단위로 실행
  -yield 수만큼 next()를 작성해야 yield 전체를 실행
- next()를 호출하면 이전 yield 다음 yield까지 실행
  ```javascript
  function* sports(value) {
      value += 20;
      const param = yield ++value;
      value = param + value;
      yield ++value;
  };
  const obj = sports(10);
  console.log(obj.next()); // {value: 31, done: false}
  console.log(obj.next(20)); // {value: 52, done: false}
  ```
- yield를 작성하지 않았을 때
  ```javascript
  function* sports(value) {
      ++value;
      console.log(value);
  };
  const obj = sports(10);
  console.log(obj.next()); // 11, {value: undefined, done: true}
  ```
- 제너레이터 함수에 return 문을 작성했을 때
  ```javascript
  function* sports(value) {
      return ++value;
  };
  const obj = sports(10);
  console.log(obj.next()); // {value: 11, done: true}
  console.log(obj.next()); // {value: undefined, done: true}
  ```
  - yield가 없고 return이 있기 때문에 값을 반환은 하지만 done은 false가 아닌 true가 된다.

- 함수는 호출할 때마다 변수에 초기값을 설정
- 제너레이터 함수는
  - 제너레이터 오브젝트를 생성할 때 초깃값을 설정
  - next()로 실행할 때마다 초깃값을 설정하지 않음
  - 변숫값을 그대로 유지
    ```javascript
    const sports = function* (param) {
        const one = param + 10;
        yield one;
        var two = 2;
        yield one + two;
    };
    const obj = sports(10);
    console.log(obj.next()); // {value: 20, done: false} 
    console.log(obj.next()); // {value: 22, done: false}
    ```
    - Generator function에 2개의 yield가 있고, const one과 two가 있다.
    - obj의 [[Scope]]를 펼치면 0: local, one: undefined, param: 10, two: undefined
    - param에 10이 있다는 것은 sports 함수 안으로 들어간 것이며, sports 함수가 호출되어 실행 컨텍스트의 초기화 단게에서 초깃값을 설정한것이다.
      - 단지, 함수 안의 코드를 실행하지 않은 것이다.
    - obj.next()를 호출하면 sports 제너레이터 함수 안으로 이동
    - const one = param + 10; 에서 멈추면, one: undefined, param: 10, two: undefined 이다.
      - 이 값은 제너레이터 오브젝트를 만들 때 설정한 값
    - const one = param + 10;
      - one의 변수값이 20으로 변경
    - yield one; 에서 {value: 20, done: false} 반환
    - 함수를 빠져 나온 후 다시 obj.next()를 호출하면 함수 안으로 이동, 함수 안의 변수에 초깃값을 설정, 앞의 obj.next()로 one 변수에 할당한 값이 그대로 남는다.
    - 이 것이 Generator function의 특징
      - 제너레이터 오브젝트를 생성할 때 초깃값을 설정하고 next()를 호출할 때마다 초깃값을 설정하지 않는다.

** 출처1. 인프런 강좌_자바스크립트 ES6+
