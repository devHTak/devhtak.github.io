---
layout: post
title: (Javascript ES6) Spread, Rest
summary: 인프런 강좌_자바스크립트 ES6+ 기본
author: devhtak
date: '2020-13-03 23:41:00 +0900'
category: Javascript ES6+
---

### Spread

- let, const 사용 기준
  - let과 const 사용 기준
    - let: 변경할 수 있다. && const: 변경할 수 없다.
    - 값이 변경되면 let, 초깃값이 변경되지 않으면 const
      ```javascript
      const list = [10, 20]; // 초깃값을 변경하지 않으면 const 사용 - 안에 값은 변경 가능 list[0] = 20;
      
      let values = [10, 20];
      value.push(30, 40); // [10, 20, 30, 40] 초기값 변경 let 사용
      
      for(let k = 0; k < values.length; k++) {...} // for문안에서 k는 변경될 수 있기 때문에 let 사용
      
      const get = (param) => param + 100; // 함수는 const 사용
      ```

-spread
  - Syntax: [...iterable]
  - [...iterable]
    - [...] 처럼 [] 안에 점(.) 3개를 작성하고 이어서 이터러블 오브젝트 작성
  - 이터러블 오브젝트를 하나씩 전개
  - {key: value}의 object는 Iterable Object는 아니지만 전개 가능하다.
    ```javascript
    const list = [10, 20];
    console.log([11, ...list, 22]); //11, 10, 20, 22
    
    const obj = {one: 10};
    console.log({...obj}); // {one:10, two: 20}
    ```

- Array Spread
  - spread 대상 배열을 작성한 위치에 엘리먼트 단위로 분리(전개)
  - Array spread 작성 형태
    ```javascript
    const one = [21, 22];
    const two = [23, 24];
    const result = [20, ...one, ...two, 25];
    console.log(result); // [20, 21, 22, 23, 24, 25];
    console.log(result.length); // 6
    ```
    - 엘리먼트 단위로 전개한다.
  - 값이 대체되지 않고 전개
    ```javascript
    const one = [11, 12];
    const result = [11, 12, ...one];
    console.log(result); // [11, 22, 11, 22];
    ```
    - 값을 대체하는 것이 아닌 전개한다.

- Spring Spread
  - spread 대상 문자열을 작성한 위치에 문자 단위로 전개
  - String spread 작성 형태
    ```javascript
    const target = "ABC";
    console.log([...target]); //["A", "B", "C"];
    ```
    
- Object Spread
  - spread 대상 Object를 작성한 위치에 프로퍼티 단위로 전개
  - Object spread 작성 형태
    ```javascript
    const one = {key1: 11, key2: 22};
    const result = {key3: 33, ...one};
    console.log(result); {key3: 33, key1: 11, key2: 22};
    ```
    - 오브젝트의 프로퍼티를 전개한다.
  - 프로퍼티 이름이 같으면 값 대체
    ```javascript
    const one = {key1: 11, key2: 22};
    const result = {key1: 33, ...one};
    console.log(result); {key1: 11, key2: 22};
    ```
    - {key1: 33}과 {key1: 11}에서 프로퍼티 이름이 같으므로 33 뒤에 작성한 11로 대체된다.
    - Object는 iterable object가 아니므로 [...one] 형태로 작성하면 에러가 발생한다. {} 형태

- push(...spread)
  - push() 파라미터에 spread 대상 작성
  - 배열 끝에 대상을 분리하여 첨부
    ```javascript
    let result = [21, 22];
    const five = [51, 52];
    result.push(...five);
    console.log(result); // [21, 22, 51, 52];
    
    result.push(..."abc");
    console.log(result); // [21, 22, 51, 52, "a","b", "c"];
    ```
    
### Rest Parameter

- Function spread
  - 호출하는 함수의 파라미터네 spread 대상 작성
    ```javascript
    function add(one, two, three) {
        for(const a in arguments) {
            console.log(a, arguments[a]);
        }
        console.log(one, two, three);
    }
    const values = [ 10, 20, 30, 40 ];
    add(...values); // 0 10, 1 20, 2 30, 3 40, 10 20 30
    ```
    - one에 10, two에 20, three에 30이 매핑된다.
  - 처리 순서 방법
    - 함수가 호출되면 우선, 파라미터의 배열을 엘리먼트 단위로 전개
    - 전개한 순서대로 파라미터 값으로 넘겨줌
    - 호출받는 함수의 파라미터에 순서대로 매핑됨
    
- Rest parameter
  - Syntax : function(param, paramN, ..name) 
  - 분리하여 받은 파라미터를 배열로 결합
    - spread: 분리, rest: 결합
    ```javascript
    function point(...param) {
        console.log(param);
        console.log(Array.isArray(param));
    }
    const values = [ 10, 20, 30 ];
    point(...values); // [10 20 30], true
    ```
    - one, two, three에 10, 20, 30이 매핑된다.
  - 작성 방법
    - 호출받는 함수의 파라미터에 ...에 이어서 파라미터 이름 작성
    - 호출한 함수에서 보낸 순서로 매핑
  - 파라미터와 Rest 파라미터 혼합 사용
    ```javascript
    function point(ten, ...rest) {
        console.log(ten);
        console.log(rest);
    }
    const value = [10, 20, 30];
    point(...value); // 10, [20, 30]
    ```
    - ten에 10이 설정되고 설정되지 않은 나머지 값 전체가 파라미터 rest에 설정된다. 그래서 rest 파라미터이다.
    - 나머지는 시맨틱을 나타내기 위해 파라미터 이름을 rest로 사용하기도 한다.
    
- Array-like
  - Object 타입이지만 배열처럼 이터러블 가능한 오브젝트
    - for()문으로 전개할 수 있음
    ```javascript
    const values = { 0: "가", 1: "나", 2: "다", length: 3};
    for(let k = 0; k < values.length; k++){
        console.log(values[k]);  // 가, 나, 다
    }    
    ```
    - length 프로퍼티는 전개되지 않는다.
    - for in 문으로 적상하면 length 프로퍼티도 전개된다.
  - 작성 방법
    - 프로퍼티 key값을 0부터 1씩 증가하면서 프로퍼티 값을 수정
    - length에 전체 프로퍼티 수 작성
    
- rest와 arguments 차이
  - arguments object
    - 파라미터 작성에 관계없이 전체를 설정
    - Array-like 형태로 Array method를 사용할 수 없음
    - __proto__ 가 Object
  - rest 파라미터
    - 매핑되지 않은 나머지 파라미터만 설정
    - Array Object 형태로 Array 메서드를 사용할 수 있음
    - __proto__가 Array
    
** 출처 1. 인프런 강좌_자바스크립트 ES6+ 기본
