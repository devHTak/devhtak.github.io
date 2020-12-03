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
** 출처 1. 인프런 강좌_자바스크립트 ES6+ 기본
