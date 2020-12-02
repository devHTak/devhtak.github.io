---
layout: post
title: (Javascript ES6) Arrow Function
summary: 인프런 강좌_자바스크립트 ES6+ 기본
author: devhtak
date: '2020-11-30 22:41:00 +0900'
category: Javascript ES6+
---

### Arrow Function

- Arrow Function
  - 코드 형태
    - (param) => { 함수 코드 }
    ```javascript
    const add = function(one, two) {
        return one + two;
    }
    console.log(add(1, 2)); // 3
    const total = (one, two) => {
        return one + two;
    }
    console.log(total(1, 2)); // 3
    ```
  - function(){}의 축약 형태지만 고려할 사항도 있다
    - this 참조가 다르다

- 함수 블록 사용
  - 함수 블록과 return 작성 생략 가능
    ```javascript
    const total = (one, two) => one + two;
    console.log(totla(one, two)); // 3
    ```
    - => 앞에서 줄을 분리하면 syntax error 발생하지만 => 뒤에서는 줄을 분리할 수 있다.
  - 함수 블록{}만 작성한 형태
    ```javascript
    const total = (one) => {}
    console.log(total(1)); // undefined
    ```
    - 함수 블록{}만 작성하면 undefined 반환
    - 함수 블록에 return을 작성하지 않은 것과 같은 형태로 블록안에 return이 없으면 undefined를 리턴한다.
    - ES3에 지정된 Javascript 문법
  - {key: value}를 반환하는 형태
    ```javascript
    const point = (param) => ({book: param});
    const result = point("책");
    for(const key in result) {
        console.log(key + ": " + result[key]); // book: 책
    }
    ```
    - {key: value} 를 소괄호()로 감싸면 {key: value}를 반환한다.
    - const point = (param) => { return {book: param}; }; 도 가능하다.
    - 소괄호()를 작성하지 않으면 undefined를 반환한다.
- 파라미터 사용
  - 파라미터가 하나일 때
    ```javascript
    const get = param => param + 20;
    console.log(get(30)); // 50
    ```
  - 파라미터가 없으면 소괄호만 작성
    ```javascript
    const get = () => 10 + 20;
    console.log(get()); // 30
    ```

### 화살표 함수 구조

- 화살표 함수 구조
  - 화살표 함수는 일반 함수와 구조가 다르다
    - 일반함수는 prototype에 constructor와 __proto__ 이 설정된다.
    - Arrow Function은 prototype과 constructor가 없다. 
      - prototype에 메소드를 연결하여 확장할 수 없지만 prototype이 없기 때문에 가볍다.
      - constructor가 없기 때문에 new 연산자로 인스턴스를 생성할 수 없다.

- arguments 사용불가
  ```javascript
  const point = () => {
      try {
          const args = arguments;
      } catch(e) {
          console.log("arguments 사용 불가");
      }      
  }
  point(10, 20); // arguments 사용 불가
  ```
  - arguments 대신 rest 파라미터 사용

### Arrow Function과 this

- 화살표 함수와 this
  - strict 모드에서 함수를 호출할 때
    - 함수 앞에 오브젝트 작성은 필수
      ```javascript
      "use strict"
      function book() {
          function getPoint() { console.log(this); }
          getPoint();
          // window.getPoint();
      }
      window.book();
      ```
      - strict 모드에서는 window.book() 처럼 호출하는 함수 앞에서 오브젝트를 작성해야 한다. 이렇게 하지 않으면 book() 함수 안에서 this 값이 undefined이다.
      - 또한, getPoint() 처럼 window를 앞에 작성하지 않으면 getPoint() 안에서 this 값이 undefined이다.
      - 이를 피하기 위해 window.getPoint()를 호출하면 window 오브젝트에 getPoint()가 없으므로 에러가 난다.
        - window.getPoint()는 오류가 발생한다.
      - strict 모드의 함수에서 this를 참조하기 위해서는 this를 별도로 저장한 후 사용해야 하는데 번거롭다.
    - 화살표 함수로 해결
      ```javascript
      "use strict"
      var point = 100;
      function sports() {
          const getPoint = () => {
              console.log(this.point);
          };
          getPoint();
      }
      window.sports();
      ```
    
  - 화살표 함수에서 this가 글로벌 오브젝트 참조
    - this 값이 undefined










** 출처 1. 인프런 강좌_자바스크립트 ES6+ 기본
