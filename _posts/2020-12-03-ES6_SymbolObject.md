---
layout: post
title: (Javascript ES6) Symbol Object
summary: 인프런 강좌_자바스크립트 ES6+ 기본
author: devhtak
date: '2020-13-03 22:41:00 +0900'
category: Javascript ES6+
---

### Iteration

- Iteration
  - Iteration의 사전적 의미: 반복
    - for() 문의 반복 개념과는 차이가 있다.
      ```javascript
      const list = [10, 20];
      for( let value of list ) {
          console.log(value); // 10, 20
      }
      const obj = list[Symbol.iterator]();
      console.log(obj.next()); // {value: 10, done: false}
      console.log(obj.next()); // {value: 20, done: false}
      console.log(obj.next()); // {value: undefined, done: true}
      ```
  - Iteration을 위한 프로토콜 필요
    - 데이터 송수신 프로토콜 정의
    - 즉, 이터레이션은 프로토콜을 갖고 있으며 프로토콜에 따라 이터레이션 수행
    - 프로토콜이 구문과 빌트인이 아니므로 프로토콜에 맞으면 이터레이션 가능
  
- Iteration Protocol
  - Iteration Protocol은 오브젝트가 iteration할 수 있는 구조여야 한다.
    - [10, 20]은 가능(배열), 100은 불가능(값)
  - Iteration 함수를 갖고 있어야 한다.
  - Iteration Protocol 구분
    - Iterable protocol
    - Iterator protocol
  - 개발자 코드로 프로토콜을 맞추면 iteration할 수 없은 오브젝트를 iteration할 수 있도록 할 수 있다.
  
### Iterable Object
  
- Iterable Protocol
  - Iterable Protocol이란?
    - 오브젝트가 반복할 수 있는 구조여야 한다.
      - Symbol.iterator를 작고 있어야 한다.
        ```javascript
        const list = [10, 20];
        console.log(list[Symbol.iterator]); // function values() { [native code] }
        ```
  - 아래의 빌트인 오브젝트는 디폴트로 이터러블 프로토콜을 할 수 있다.
    - 즉, Symbol.iterator를 갖고 있다.
    - Array, Arguments, String, TypedArray, Map, Set, DOM NodeList
      
- Iterable Object
  - Iterable Object란?
    - Iterable Protocol을 갖고 있는 Object
    - 반복 구조, Symbol.iterator()
      ```javascript
      const list = [10, 20];
      console.log(list[Symbol.iterator]); // function values() { [native code] }
      
      const obj = {one: 10, two: 20};
      console.log(obj[Symbol.iterator]); // undefine
      ```
      - [] 리터럴(Array Object)로 생성한 list에 Symbol.iterator가 있으므로 Array Iterable Object입니다.
      - {} 리터럴(prototype으로 이뤄진 object)로 생성한 obj에 Symbol.iterator가 없으므로 Object는 Iterable Object가 아니다.
        - obj는 property로 구성된 object
      - for 문의 반복과 iteration과는 차이가 있고, for-in 문의 반복과 iteration 또한 차이가 있다.
        ```javascript
        const obj = {one: 10, two: 20};
        for(let key in obj) { console.log(key, obj[key]); } // one 10, two : 20 
        ```
    - 스펙에서는 @@iterator()로 표기된다.
    
  - Iterable Object 구조
    - list의 구조를 보면 안에 __proto__가 있다. __proto__를 펼치면 Array Object의 메소드가 표시된다.
    - 아래로 내려가면 Symbol(Symbol.iterator)가 있다. 따라서 Array Object는 Iterable Object이다.
    - Symbol(Symbol.iterator)를 펼치면 __proto__가 존재하며 Function object의 메서드가 연결되어 있다. 즉, 함수이다.
    - Symbol.iterator가 함수이므로 호출할 수 있다.
  
  - 자체 오브젝트에는 없지만 이터러블 오브젝트를 상속받아도 된다.
    - 즉, prototype chain(__proto__)에 있으면 된다.
    - 예를 들어, Array Object를 상속받으면 Iterable Object가 된다.
    - 즉, Iterable Object가 없더라도 상속받으면 사용할 수 있다.
    
### Iterator Object

- Iterator Protocol
  - Iterator Protocol
    - 값을 순서대로 생성하는 방법(규약)
  - Iterator Object
    - Symbol.iterator()를 호출하면 Iterator Object를 생성하여 반환
    - Iterator Object에 next()가 있음
    - 생성한 오브젝트를 Iterator라고 부름
      ```javascript
      const list = [10, 20];
      const obj = list[Symbol.iterator]();
      console.log(obj.next()); // {value: 10, done: false}
      console.log(obj.next()); // {value: 20, done: false}
      console.log(obj.next()); // {value: undefined, done: true}
      ```
      - iterator object의 next()를 호출하면 iterator를 호출한다고 한다.
      - 첫번째 next() 호출 {value: 10, done: false}를 반환 한다. value는 list의 첫번째 값이고 done: false는 iterator 상태입니다.
      - 두번째 next() 호출 {value: 20, done: false}를 반환 한다. value는 list의 두번째 값이고 done: false는 iterator 상태입니다.
      - 세번째 next() 호출 {value: undefined, done: true}를 반환 한다. undefined는 처리할 값이 없다는 뜻이며 done:true는 이터레이터의 종료를 뜻한다.
  - Iterator Object 구조
    - const obj = list[[Symbol.iterator]] ();
      - 위 형태로 호출하면 이터레이터 오브젝트를 생성하여 반환한다.
      - obj의 __proto__를 펼치면 next()가 있다. next()가 있으므로 obj는 iterator object이다. 
  
** 출처 1. 인프런 강좌_자바스크립트 ES6+ 기본
