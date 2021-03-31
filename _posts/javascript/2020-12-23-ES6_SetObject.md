---
layout: post
title: (Javascript ES6) Set Object
summary: 인프런 강좌_자바스크립트 ES6+ 기본
author: devhtak
date: '2020-12-23 22:41:00 +0900'
category: Javascript ES6+
---

### Set Object

- Set Object는 value 값의 컬렉션
- [value1, ... valueN] 형태로 작성 
  - Set은 대괄호[] 가 하나
    ```javascript
    const obj = new Set([
        1, 2, "ABC"
    ]);
    console.log(`size: ${obj.size}`); // size: 3
    for(let value of obj) {
        console.log(value); // 1, 2, ABC
    }
    ```
  - 작성한 순서로 전개된다.
  
- new Set()
  - 형태: new Set()
  - 파라미터: 값(option)
  - 반환: 생성한 Set 인스턴스
  - Set 이스턴스 생성, 반환
    - 파라미터에 값을 작성
    - 프리미티브, 오브젝트 타입 사용 가능
      ```javascript
      const obj = new Set([
          1, 2, 1, [], {}
      ]);
      console.log(`size: ${obj.size}`); // size: 4
      for(let value of obj) {
          console.log(value); // 1, 2, [], {}
      }
      ```
      - 같은 값이 있으면, 첫 번째의 1을 유지하며 세 번째의 1을 설정하지 않는다.
      - same-value-zero 비교 알고리즘으로 비교, size에서는 세 번째 1이 설정되지 않으므로 값은 4가 된다.
  - size 프로퍼티
    - Set 인스턴스의 엘리먼트 수를 반환
  - Set 오브젝트 구조
    ```javascript
    const set = Set;
    const obj = new Set([
        "one", "two"
    ]);
    ```
    - Set 오브젝트에 Symbol(Symbol.species)가 있다. 즉 constructor를 오버라이드 할 수 있다.
    - prototype에 Symbol.iterator가 있다.
    - obj를 펼치면 [[Entries]]가 있다.
      - [[Entries]]를 펼치면 0: "one" 형태로 인덱스를 부여하여 key로 사용하고, "one", "two"를 value로 설정한다.
      - 인덱스를 부여하는 구조는 Map과 같으며 인덱스를 부여하여 저장하므로 작성한 순서로 읽혀진다.
      
- Set과 Map 비교
  - 저장 형태
    - Map: Key, Value로 작성, Key를 Key로 사용하여 [key, value]로 저장
    - Set: Value만 작성, value를 key로 사용하여 [value]로 저장
  - 값을 구하는 형태
    - Map: get(key) 형태로 value를 구할 수 있다.
    - Set: get() 메소드가 없다. 값 하나를 따로 구할 수 없으며 반복으로 값을 구하거나 iterator를 사용해야 한다.
    
### Set의 메서드

- add()
  - 형태: Set.prototype.add()
  - 파라미터: value
  - 반환: value가 설정된 인스턴스
  - Set 인스턴스 끝에 value 추가
    ```javascript
    let obj = new Set();
    obj.add("축구").add("농구");
    obj.add("축구");
    for(let value of obj) {
        console.log(value); // 축구, 농구
    }
    ```
  - 사용 형태
    - 함수를 생성하여 value로 사용
      ```javascript
      let obj = new Set();
      obj.add(function sports() { return 100; });
      obj.add(function sports() { return 200; });
      
      for(let value of obj) {
          console.log(value()); // 100, 200
      }
      ```
        - 같은 이름의 function을 사용했지만 참조하는 메모리 주소가 다르므로 설정된다.
    - value에 생성한 함수 이름 작성
      ```javascript
      const sports = () => 100;
      let obj = new Set();
      obj.add(sports);
      obj.add(sports);
      
      for(let value of obj) {
          console.log(value()); // 100
      }
      ```
        - Set에 메모리도 같은 function을 넣었기 때문에 하나만 설정된다.
    - object를 value로 사용
      ```javascript
      const sports = {
          "축구": 11, "야구": 9
      };
      let obj = new Set();
      obj.add(sports);
      for(let value of obj) {
          console.log(value); //{축구: 11, 야구: 9}
      }
      ```

- has()
  - 형태: Set.prototype.has()
  - 파라미터: key값
  - 반환: key가 존재하면 true / 아니면 false
  - Set 인스턴스에서 값의 존재 여부를 반환
    ```javascript
    const sports = () => {}
    const obj = new Set([
        sports
    ]);
    console.log(obj.has(sports)); // true
    console.log(obj.has("book")); // false
    ```
  - get() 메서드가 없으므로 has()로 값의 졵 여부를 체크한 후 존재하면 체크한 값을 값으로 사용
  
- forEach()
  - 형태: Set.prototype.forEach()
  - 파라미터: callback 함수 / this로 참조할 object(option)
  - 반환: undefined
  - Set 인스턴스를 반복하면서 callback 함수 호출
    - map(), filter() 등의 callback 함수가 동반되는 메소드 사용 불가
  - callback 함수에 넘겨주는 파라미터
    - value, key(value), Set 인스턴스
      ```javascript
      const obj = new Set([
          "one", ()=>{}
      ]);
      function callback(value, key, set) {
          console.log(value);
      }
      obj.forEach(callback); // one, () => {}
      ```
    - 콜백 함수에서 this 사용
      ```javascript
      const obj = new Set([
          "one", "two"
      ]);
      function callback(value, key, set) {
          console.log(`${value}, ${this.check]`);
      }
      obj.forEach(callback, {check: "ABC"}); // one, ABC / two, ABC
      ```
      
- delete()
  - 형태: Set.prototype.delete()
  - 파라미터: key 값
  - 반환: 삭제성공: true / 실패: false
  - Set 인스턴스에서 파라미터 값과 같은 엘리먼트 삭제
  - 같은 value가 있어 삭제에 성공하면 true 반환, 삭제에 실패하면 false 반환
    ```javascript
    const obj = new Set([
        "one", "two"
    ]);
    console.log(obj.delete("one")); // true
    console.log(obj.delete("one")); // false
    ```

- clear()
  - 형태: Set.prototype.clear()
  - 파라미터: 파라미터 없음
  - 반환: 없음
  - Set 인스턴스의 모든 엘리먼트를 지움
    - Set 인스턴스를 삭제하는 것은 아님 따라서 value를 추가할 수 있다.
      ```javascript
      const obj = new Set([
        "one", "two"
      ]);
      console.log(obj.size()); // 2
      obj.clear();
      console.log(obj.size()); // 0
      obj.add("one");
      console.log(obj.size()); // 1
      ```
      
### Set과 Iterator Object

- entries()
  - 형태: Set.prototype.entries()
  - 파라미터: 파라미터 없음
  - 반환: 생성한 이터레이터 오브젝트 생성
  - Set 인스턴스로 iterator object 생성, 반환
    - Set 인스턴스에 설정된 순서로 반환, next()로 [value, value] 반환
    - [value, value] 형태인 이유는 Map 오브젝트와 같은 모듈을 사용학 때문이다.
      ```javascript
      const obj = new Set([
          "one", ()=>{}
      ]);
      const iterObj = obj.entries();
      console.log(iterObj.next()); // {value: [one, one], done: false}
      console.log(iterObj.next()); // {value: [()=>{}, ()=>{}], done: false}
      console.log(iterObj.next()); // {value: undefined, done: ture}
      ```
  
- keys()
  - 형태: Set.prototype.keys()
  - 파라미터: 파라미터 없음
  - 반환: 생성한 이터레이터 오브젝트
  - value가 key가 되므로 keys()는 의미가 없다. Map Object와 맞추기 위한 것
  - Set 인스턴스의 value를 key로 사용하여 이터레이터 오브젝트 생성, 반환
    - Set 인스턴스에 설정된 순서로 반환
  - next()로 value(key) 반환
    ```javascript
    const obj = new Set([
        "one", () => {}
    ]);
    const iterObj = obj.keys();
    console.log(iterObj.next()); // {value: one, done: false}
    console.log(iterObj.next()); // {value: ()=>{}, done: false}
    console.log(iterObj.next()); // {value: undefined, done: ture}
    ```
    
- values()
  - 형태: Set.prototype.values()
  - 파라미터: 파라미터 없음
  - 반환: 생성한 이터레이터 오브젝트
  - Set 인스턴스의 value로 이터레이터 오브젝트 생성, 반환
    - Set 인스턴스에 설정된 순서로 반환
  - next()로 value 반환
    ```javascript
    const obj = new Set([
        "one", () => {}
    ]);
    const iterObj = obj.values();
    console.log(iterObj.next()); // {value: one, done: false}
    console.log(iterObj.next()); // {value: ()=>{}, done: false}
    console.log(iterObj.next()); // {value: undefined, done: ture}
    ```
  
- Symbol.iterator()
  - 형태: Set.prototype[Symbol.iterator]
  - 파라미터: 파라미터 없음
  - 반환: {done: true/false, value: 값}
  - Set 인스턴스로 이터레이터 오브젝트 생성, 반환
    - Set.prototype.values()와 같다.
    - next()로 value 반환
      ```javascript
      const obj = new Set([
          "one", () => {}
      ]);
      const iter = obj[Symbol.iterator]();
      console.log(iterObj.next()); // {value: one, done: false}
      console.log(iterObj.next()); // {value: ()=>{}, done: false}
      console.log(iterObj.next()); // {value: undefined, done: ture}
      ```

### WeakSet Object

- Set Object와 차이
  - 오브젝트만 value 값으로 사용할 수 있다.
  - number 등의 프리미티브 타입 사용 불가
- 개념은 WeakMap과 같다
  - value만 작성하는 것이 다르고 value의 참조가 바뀌면 GC 대상

- new WeakSet()
  - 형태: new WeakSet()
  - 파라미터: 오브젝트(option)
  - 반환: 생성한 WeakSet 인스턴스
  - WeakSet 인스턴스 생성, 반환
  - 파라미터
    - 대괄호[] 안에 오브젝트 작성
      ```javascript
      const empty = new WeakSet();
      const sports = {};
      const obj = new WeakSet([
         sports
      ]);
      ```
      
- has()
  - 형태: WeakSet.prototype.has()
  - 파라미터: 오브젝트
  - 반환: true(존재)/false(존재X)
  - WeakSet 인스턴스에 value의 존재 여부 반환
    ```javascript
    const fn = () => {}
    const obj = new WeakSet([
        fn
    ]);
    console.log(obj.has(fn)); // true
    ```
  
- add()
  - 형태: WeakSet.prototype.add()
  - 파라미터: value, object
  - 반환: value가 설정된 인스턴스
  - WeakSet 인스턴스에 value 설정
    - 파라미터에 value로 설정될 오브젝트 작성
      ```javascript
      const obj = new WeakSet();
      const fn = () => {};
      obj.add(fn); 
      console.log(obj.has(fn)); // true
      ```
      
- delete()
  - 형태: WeakSet.prototype.delete()
  - 파라미터: key, object
  - 반환: true(성공)/false(실패)
  - WeakSet 인스턴스에서 value와 일치하는 엘리먼트삭제
    ```javascript
    const fn = () => {}
    const obj = new WeakSet([
        fn
    ]);
    console.log(obj.delete(fn)); // true
    console.log(obj.has(fn)); // false
    ```
  
** 출처1. 인프런 강좌_자바스크립트 ES6+
