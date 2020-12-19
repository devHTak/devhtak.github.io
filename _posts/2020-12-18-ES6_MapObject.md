---
layout: post
title: (Javascript ES6) Map Object
summary: 인프런 강좌_자바스크립트 ES6+ 기본
author: devhtak
date: '2020-12-18 22:41:00 +0900'
category: Javascript ES6+
---

### Map Object

- Map Object
  - key와 value의 컬렉션
  - 형태
    - [key, value] 형태처럼 대괄호 안에 key와 value를 작성
    - 다양한 타입을 key로 사용할 수 있다.
      ```javascript
      const obj = new Map([ 
          ["key", "value"], [{book: 200}, "오브젝트"], [100, "Number"]
      ]);
      for(let keyValue of obj) {
          console.log(keyValue); // [key, value], [{book: 200}, 오브젝트], [100, Number]
      }
      ```
  - Map의 key 처리
    - for - of 문에서 작성한 순서대로 읽혀진다.
    - 순서 보장
    
- new Map()
  - 형태: new Map()
  - 파라미터: \[이터러블 오브젝트](option)
  - 반환: 생성한 Map 인스턴스
  
  - Map 인스턴스를 생성하여 반환
  - 파라미터에 이터러블 오브젝트 작성
    ```javascript
    const obj = new Map([
        ["key": "value"], [100, "Number"]
    ]);
    console.log(obj); // {}
    console.log(typeof obj); // object
    ```
  - Same-Value-Zero 비교 알고리즘
    - key 값을 비교
      ```javascript
      const obj = new Map([
          [100, "Number"], ["100", "String"]
      ]);
      for(let [key, value] of obj) {
          console.log(`${key}: ${value}`); // 100: Number, 100: String
      }
      ```
    - key 값이 같으면 value가 대체됨
      ```javascript
      const obj = new Map([
          [100, "Number1"], [100, "Number2"]
      ]);
      for(let [key, value] of obj) {
          console.log(`${key}: ${value}`); // 100: Number2
      }
      ```
      - key 값이 타입까지 같으면 value가 대체된다.
  - 잘못 작성한 형태
    ```javascript
    try {
        new Map(["one", 1]);
    } catch {
        console.log("[[one, 1]]");
    }
    // [[one, 1]] 출력 - 대괄호가 1개 빠졌다.
    const obj = new Map([{five: 5}]);
    for(let [key, value] of obj) {
        console.log(`${key}: ${value}`); // undefined: undefined
    }
    ```
    - new Map(["one", 1]): 대괄호 2개를 작성해야 한다.
    - const obj = new Map([{five: 5}]): key 만 작성하면, 에러가 발생하지 않지만, key와 value에 undefined가 설정된다.

- Map과 Object 비교
  - Map Object 구조
    ```javascript
    const map = Map;
    const list = [1, 2]; 
    const obj = new Map([ ["one", "첫 번째"], ["two", "두 번째"] ]);
    ```
    - const map = Map;
      - Map Object에 get Symbol(Symbol.species)가 있다. 따라서 constructor를 오버라이드 할 수 있다.
      - prototype을 펼치면 Symbol.iterator가 있다. iterator 오브젝트이기 때문에 for-of 가능
    - const list = [1, 2]; const obj = new Map([ ["one", "첫 번째"], ["two", "두 번째"] ]);
      - 오른쪽 obj를 펼치면 [[Entries]]가 있다. 대괄호 2개는 엔진에서 설정하는 것을 뜻한다.
      - [[Entries]]를 펼치면 0: {"one" => "첫 번째"} 형태
      - 인덱스를 부혀하여 key로 사용하고 {"one": "첫 번째"}를 value로 설정
      - 이것은 배열 형태와 구조가 비슷, size가 length 기능
      - 인덱스를 부여하여 저장하므로 작성한 순서로 읽혀진다. 
    
  - key
    - Map: 타입 제약 없음
    - Object: String, Symbol    
  - {key: value} 수
    - Map: size 프로퍼티로 구함
    - Object: 전체를 읽어 구해야 함
  - 처리시간: MDN에 의하면 빈번하게 key, value를 추가/삭제할 때는 Map이 Object보다 좋은 경우가 있다고 한다.
  
### Map 사용 메서드

- set()
  - 형태: Map.prototype.set()
  - 파라미터: key, value
  - 반환: key, value가 설정된 인스턴스
  
  - Map 인스턴스에 key, value 설정
    - key, value 순서로 파라미터 작성
    - key, value를 설정한 인스턴스 반환
      ```javascript
      let obj = new Map();
      obj.set("one", 100);
      obj.set({}, "오브젝트");
      obj.set(function() {}, "Function");
      obj.set(new Number("100"), "인스턴스");
      obj.set(NaN, "Not a Number");
      
      for(let [key, value] of obj) {
          console.log(`${key}: ${value}`); // one: 100, [object Object]: 오브젝트, function() {}: Function, 100: 인스턴스, NaN, Not a Number
      }
      ```
  - key 값이 같으면 value가 바뀐다.
    ```javascript
    let obj = new Map();
    const book = {};
    obj.set(book, "첫 번째");
    obj.set(book, "두 번째");
    
    for(let [key, value] of obj) {
        console.log(`${key}: ${value}`); // [object Object]: 두 번째
    }
    ```
  
- get()
  - 형태: Map.prototype.get()
  - 파라미터: key 값
  - 반환: [key, value] 에서 value, undefined
  
  - Map에서 key 값이 같은 value 반환
    - key 값이 같지 않거나 타입이 다르면 undefined 반환
      ```javascript
      let obj = new Map([
          ["one", 100], ["200", "String 타입"]
      ]);
      console.log(obj.get("one")); // 100
      console.log(obj.get("two")); // undefined
      console.log(obj.get(200)); // undefined
      ```
    - 오브젝트 설정과 추출
      ```javascript
      let obj = new Map();
      const like = {sports: "스포츠"};
      obj.set(like, {value: "농구"});
      console.log(obj.get(like)); // {value: 농구}
      ```
      - 같은 메모리 주소를 사용
      
  - has()
    - 형태: Map.prototype.has()
    - 파라미터: key 값
    - 반환: key가 존재하면 true, 아니면 false
    
    - Map 인스턴스에서 key의 존재 여부를 반환
      - key가 존재하면 true, 아니면 false
        ```javascript
        const obj = new Map([ ["one", 100] });
        console.log(obj.has("one")); // true
        console.log(obj.has("two")); // false
        ```
        
### Map의 Iterator object

- entries()
  - 형태: Map.prototype.entries()
  - 파라미터: 파라미터 없음
  - 반환: 생성한 이터레이터 오브젝트 생성
  
  - Map 인스턴스로 이터레이터 오브젝트 생성, 반환
    - Map 인스턴스에 설정된 순서로 반환
    - next()로 [key, value] 반환
      ```javascript
      const obj = new Map([ ["one", 100], ["two", 200] ]);
      const iter = obj.entries();
      console.log(iter.next()); // {value: [one, 100], done: false}
      console.log(iter.next()); // {value: [two, 200], done: false}
      console.log(iter.next()); // {value: undefined, done: true}
      ```
  
- keys()
  - 형태: Map.prototype.keys()
  - 파라미터: 파라미터 없음
  - 반환: 생성한 이터레이터 오브젝트
  
  - Map 인스턴스의 key로 iterator object 생성, 반환
    - value는 포함하지 않음
    - Map 인스턴스에 설정된 순서로 반환
  - next()로 key 반환
    ```javascript
    const obj = new Map([ ["one", 100], ["two", 200] ]);
    const iter = obj.keys();
    console.log(iter.next()); // {value: one, done: false}
    console.log(iter.next()); // {value: two, done: false}
    console.log(iter.next()); // {value: undefined, done: true}
    ```
  
- values()
  - 형태: Map.prototype.values()
  - 파라미터: 파라미터 없음
  - 반환: 생성한 이터레이터 오브젝트
  
  - Map 인스턴스의 value로 iterator object 생성, 반환
    - key는 포함하지 않음
    - Map 인스턴스에 설정된 순서로 반환
  - next()로 key 반환
    ```javascript
    const obj = new Map([ ["one", 100], ["two", 200] ]);
    const iter = obj.values();
    console.log(iter.next()); // {value: 100, done: false}
    console.log(iter.next()); // {value: 200, done: false}
    console.log(iter.next()); // {value: undefined, done: true}
    ```

- Symbol.iterator()
  - 형태: Map.prototype[Symbol.iterator]
  - 파라미터: 파라미터 없음
  - 반환: {done: true/false, value: 값]
  
  - Map 인스턴스로 iterator object 생성, 반환
    - Map.prototype.entries() 와 같음
    - next()로 [key, value] 반환
    ```javascript
    const obj = new Map([ ["one", 100], ["two", 200] ]);
    const iter = obj[Symbol.iterator]();
    console.log(iter.next()); // {value: [one, 100], done: false}
    console.log(iter.next()); // {value: [two, 200], done: false}
    console.log(iter.next()); // {value: undefined, done: true}
    ```
    
### 콜백함수, 삭제, 지우기

- forEach()
  - 형태: Map.prototype.forEach()
  - 파라미터: callback 함수, this로 참조할 object(option)
  - 반환: undefined
  
  - Map 인스턴스를 반복하면서 callback 함수 호출
    - map(), filter() 등의 callback 함수가 동반되는 메서드 사용 불가.
  - callback 함수에 넘겨주는 파라미터
    - value, key, Map 인스턴스 - key, value 순서가 아니다.
      ```javascript
      const obj = new Map([ ["one", 100], ["two", 200] ]);
      const callback = (value, key, map) => {
          console.log(`${key}: ${value}`);
      };
      obj.forEach(callback); // one: 100 / two: 200
      ```
    - 콜백 함수에서 this 사용
      ```javascript
      const obj = new Map([ ["one", 100], ["two", 200] ]);
      function callback(value, key, map) {
          console.log(`${key}, ${value}, ${this.check}`);
      }
      obj.forEach(callback, {check: 50}); // one, 100, 50 / two, 200, 50
      ```
      - 콜백 함수를 Arrow function으로 작성하면 this는 window object로 두번째 파라미터를 사용하지 못한다.
      
- delete()
  - 형태: Map.prototype.delete()
  - 파라미터: key 값
  - 반환: 삭제 성공: true / 실패: false
  
  - Map 인스턴스에서 파라미터 값과 같은 entry 삭제
    - 같은 key가 있으면 삭제 후 true 반환, 없으면 false 반환
      ```javascript
      const obj = new Map([ ["one", 100], [{}, "오브젝트"] ]);
      
      console.log(obj.delete("one")); // true
      console.log(obj.delete({})); // false
      
      const sports = {};
      obj.set(sports, "스포츠");
      console.log(obj.delete(sports)); // true
      ```
      - obj.delete({}) 에서 [{}, "오브젝트"]에 {}는 다르다 - 참조하는 메모리 주소가 다르다.
      
- clear()
  - 형태: Map.prototype.clear()
  - 파라미터: 없음
  - 반환: 없음
  
  - Map 인스턴스의 모든 entry를 지움
    - Map 인스턴스를 삭제하는 것은 아니다.
    - 따라서 [key, value]를 추가할 수 있다.
      ```javascript
      const obj = new Map([ ["one": 100, "two": 200] ]);
      console.log(obj.size); // 2
      
      obj.clear();
      console.log(obj.size); // 0
      
      obj.set("add", "추가");
      console.log(obj.size); // 1
      ```
  - size 프로퍼티
    - Map 인스턴스의 entry 수를 반환
    - 개발자 코드로 수정할 수 없다.
    
    
    
    
    
    
    
    
    
    
    
** 출처1. 인프런 강좌_자바스크립트 ES6+
