---
layout: post
title: (Javascript ES6) Array Object
summary: 인프런 강좌_자바스크립트 ES6+ 기본
author: devhtak
date: '2020-12-13 22:41:00 +0900'
category: Javascript ES6+
---

### from(), of()

- from()
  - 형태: Array.from()
  - 파라미터: 변환 대상, 이터러블 오브젝트 | 전개할 때마다 호출할 함수(option) | 호출된 함수에서 this로 참조할 object(option)
  - 반환: Array object

  - 첫 번째 파라미터의 오브젝트를 Array Object로 반환
    ```javascript
    const like = {0: "zero", 1: "one", length: 2}
    const list = Array.from(like);
    console.log(list); // [zero, one]

    console.log(Array.from("ABC")); // [A, B, C]
    ```
    - Array-like 오브젝트를 Array Object로 변환하여 반환'
    - "ABC" 문자 단위로 분리하여 배열로 반환

    ```javascript
    function args() {
        return Array.from(arguments);
    };
    console.log(args(1, 2, 3)); // [1, 2, 3]  
    ```
    - arguments 객체를 배열로 반환

    ```javascript
    // <li class="sports">농구</li>
    // <li class="sports">축구</li>
    const nodes = document.querySelectorAll(".sports");
    const show = (node) => {
        console.log(node.textContent);
    };
    Array.from(nodes).forEach(show); // 축구, 농구
    ```
    - NodeList가 이터러블 오브젝트이므로 Array.from()으로 읽을 수 있다.

  - 두 번째 파라미터에 함수 작성(option)
    - 이터러블 오브젝트를 전개할 때마다 호출
      ```javascript
      const like = {0: "zero", 1: "one", length: 2}
      console.log(Array.from( (like, value) => {
          return value +" 변경";
      }); // [zero 변경, one 변경]
      ```
      - 이터러블 오브젝트를 하나씩 읽는다.
      - 읽은 값을 넘겨주면서 콜백 함수 호출
      - 콜백 함수에서 반환된 값을 배열에 첨부하여 반환

  - 세 번째 파라미터에 오브젝트 작성(option)
    - 호출된 함수에서 this로 참조
      ```javascript
      const like = {0: 10, 1: 20, length: 2}
      console.log(Array.from( value => {
          return value + this.point;
      }, {point: 7}); // [17, 27]
      ```
      - 콜백 함수에서 this로 3번째 파라미터의 오브젝트를 참조
      - 화살표 함수를 사용하면 콜백 함수에서 3번째 파라미터의 오브젝트를 참조하지 않는다.
    
- of()
  - 형태: Array.of()
  - 파라미터: 변환 대상 값, 다수 작성 가능
  - 반환: Array object
  
  - 파라미터 값을 Array로 변환하여 반환
    ```javascript
    const result = Array.of(1, 2, 3);
    console.log(result); // [1, 2, 3]
    console.log(Array.of()); // []
    ```
    - 파라미터 값을 1, 2, 3을 Array object에 첨부하여 반환
    - 파라미터를 작성하지 않으면 빈 Array object 반환
    
  - 파라미터에 변환 대상 값을 작성 
    - 콤마로 구분하여 다수 작성 가능
    
### 배열 엘리먼트 복사

- copyWithin()
  - 형태: Array.prototype.copyWithin()
  - 파라미터: 복사한 값을 설정할 시작 인덱스 | 복사 시작 인덱스(option) | 복사 끝 인덱스(option)
  - 반환: 변경된 Array Object (얕은 복사로 반환)
  
  - 범위 값을 복사하여 같은 오브젝트에 설정
  - 두 번째 파라미터의 인덱스부터 복사하여
    - 첫 번째 파라미터 인덱스부터 순서대로 설정(대체)
      ```javascript
      const list = ["A", "B", "C", "D", "E"];
      const copy = list.copyWithin(1, 3);
      console.log(list); // [A, D, E, D, E]
      console.log(copy); // [A, D, E, D, E]
      ```
      - list 배열이 대상
      - 두 번째 파라미터의 3번 인덱스부터 배열의 끝까지 복사하여 1번 인덱스부터 차례로 설정
      - D와 E를 복사하므로 엘리먼트가 2개이며
      - 1번 인덱스부터 2개를 대체하므로 B->D, C->E로 대체
      - 복사 대상에 대체하므로 반환된 Array object와 복사 대상이 같다.
      
  - 세 번째 파라미터의 인덱스 직전까지 복사
    ```javascript
    const list = ["A", "B", "C", "D", "E"];
    list.copyWithin(0, 2, 4);
    console.log(list); // [C, D, C, D, E]
    ```
    - 두 번째 파라미터의 2번 인덱스부터 세번째 파라미터의 4번 인덱스 직전까지 복사하여 list 배열의 0번 인덱스부터 설정
    - 2번 인덱스, 3번 인덱스를 복사하므로 C와 D를 복사하게 ㅗ딘다.
    - A->C, B->D로 대체
  - 복사 시작 인덱스와 끝 인덱스를 작성하지 않으면 배열 전체 복사
    ```javascript
    const list = ["A", "B", "C", "D", "E"];
    list.copyWithin(3);
    console.log(list); // [A, B, C, A, B]
    ```

- copyWithin() 특징
  - shallow copy(얕은 복사)
  - 같은 배열 안에서 이동하는 개념
    ```javascript
    const list = ["A", {B: "가"}, "C"];
    console.log(list.copyWithin(0, 1)); // [{B: 가}, C, C]
    ```
    - {B: 가}를 복사할 때 새로운 객체를 생성하지 않고 현재의 메모리 주소를 복사한다.
    - shallow copy이다.
    - 연동되지 않으려면 deep copy를 해야 한다.
  - 배열의 엘리먼트 수가 변동되지는 않는다.  
  - 배열 안에서 엘리먼트 이동은 엘리먼트를 왼쪽, 오른쪽으로 이동, 처리 속도가 빠르다.
  
- generic
  - copyWithin function is intentionally generic
  - generic 사용 형태
    ```javascript
    const like = {0: 10, 1: 20, 2: 30, length: 3};
    console.log(Array.prototype.copyWithin.call(like, 1, 0)); // {0: 10, 1: 10, 2: 20, length: 3}
    ```
    - call()의 첫 번째 파라미터에 Array-like를 작성했으며 Array-like 타입은 object이다.
    - copyWithin()이 Array method이므로 Array를 넘겨 주어야 하는 데, Array-like를 넘겨주어도 처리가 된다.
    - 이 것이 제네릭이다. copyWithin() 함수는 제네릭 함수
    - 배열로 반환하지 않고 대상 오브젝트 형태(Array-lie)로 반환한다.
  - generic이 뜻하는 것은 copyWithin()이 Array 메소드로 Array object가 처리 대상이지만, generic 은 Array object가 아닌 array-like, iterable object 또한 처리할 수 있다는 것이다.
  
### 같은 값, 인덱스 검색

- find()
  - 형태: Array.prototype.find()
  - 파라미터: 콜백 함수 | 콜백 함수에서 this로 참조할 object(option)
  - 반환: 배열 엘리먼트 또는 undefined
  
  - 배열에 엘리먼트를 하나씩 읽어가면서 콜백 함수 호출
    - 콜백 함수에서 true를 반환하면 find()를 종료
    - 현재 처리중인 엘리먼트 값을 반환
      ```javascript
      const list = ["A", "B", "C"];
      const callback = (value, index, all) => value === "B";
      console.log(list.find(callback)); // B
      ```
      - ["A", "B", "C"]를 반복하면서 콜백 함수 호출
      - 콜백 함수에서 엘리먼트 값이 B이면 true를 반환
      - 콜백 함수에서 true를 반환하면 현재 처리중인 엘리먼트 값인 B를 반환하고 find() 실행 종료
      - 조건에 맞으면 find() 실행을 종료하므로 배열 앞에서 true가 되면 효율이 높다.
      
      ```javascript
      const list = ["A", "B", "C"];
      const callback = (value, index, all) => value === 77;
      console.log(list.find(callback)); // undefined
      ```
      - 콜백 함수에 조건에 맞는 값이 없으면 undefined를 반환
      
      ```javascript
      const list = ["A", "B", "C"];
      function callback(value, index, all) {
          return value === this.check;
      };
      console.log(list.find(callback), {check: "A"); // A
      ```
      - 두 번째 파라미터에 콜백 함수에서 this로 참조할 오브젝트를 작성한 형태
      - 콜백 함수를 화살표 함수로 작성하면 콜백 함수에서 this가 window를 참조하므로 두 번째 파라미터의 오브젝트를 참조하지 못한다.
      - 일반 함수를 작성해야 한다.      
  - 파라미터: 엘리먼트, 인덱스, 배열 전체

- findIndex()
  - 형태: Array.prototype.findIndex()
  - 파라미터: 콜백 함수 | 콜백 함수에서 this로 참조할 오브젝트(option)
  - 반환: 배열 인덱스 또는 -1
  
  - 배열의 엘리먼트를 하나씩 읽어가면서 콜백 함수 호출
    - 콜백 함수에서 true 반환하면 findIndex() 종료하면서 현재 처리중인 엘리먼트의 인덱스를 반환
      ```javascript
      const list = ["A", "B", "C"];
      const callback = (value, index, all) => value === "B";
      console.log(list.find(callback)); // 1
      ```
      - ["A", "B", "C"]를 반복하면서 콜백 함수 호출
      - 콜백 함수에서 엘리먼트 값이 B이면 true를 반환
      - 콜백 함수에서 true를 반환하면 현재 처리중인 엘리먼트의 인덱스를 반환하고 findIndex()를 종료
      
      ```javascript
      const list = ["A", "B", "C"];
      const callback = (value, index, all) => value === 77;
      console.log(list.find(callback)); // -1
      ```
      - 콜백 함수에서 조건에 맞는 값이 없으면 -1을 반환
      - indexOf(searchValue, fromIndex)는 값을 지정할 수 있으며 검색을 시작할 인덱스를 지정할 수 있다.
        - 콜백 함수가 없으므로 다양한 조건으로 체크 불가
        - 단, 값만으로 인덱스를 찾을 때는 indexOf()가 효율적
      - includes(searchValue, fromIndex)는 true/false 반환
  - 파라미터: 엘리먼트, 인덱스, 배열 전체

### 대체, 포함 여부

- fill()
  - 형태: Array.prototype.fill()
  - 파라미터: 설정할 값 | 시작 인덱스(option) | 끝 인덱스(option)
  - 반환: 변경된 Array object
  
  - 범위 값을 지정한 값으로 설정, 반환
  - 설정 방법
    - 시작 인덱스부터 끝 인덱스 직전까지
    - 첫 번째 파라미터 값으로 설정(대체)
      ```javascript
      const list = ["A", "B", "C"];
      list.fill("책", 1);
      console.log(list); // [A, 책, 책];
      ```
      - 시작 인덱스를 작성하고 끝 인덱스를 작성하지 않으면 시작 인덱스부터 끝까지가 대체 대상
      - 첫 번째 파라미터 값인 "책"으로 대체
      
      ```javascript
      const list = ["A", "B", "C", "D", "E"];
      list.fill("책", 1, 3);
      console.log(list); // [A, 책, 책, D, E];
      ```
      - 끝 인덱스를 설정하면, 시작 인덱스부터 끝 인덱스 직전까지 대체
      
      ```javascript
      const list = ["A", "B", "C"];
      list.fill("책");
      console.log(list); // [책, 책, 책];
      ```
      - 시작 인덱스와 끝 인덱스를 작성하지 않으면 전체가 대체 대상
  - Generic 함수
    ```javascript
    const like = {0: "A", 1: "B", 2:"C", length: 3};
    console.log(Array.prototype.fill.call(like, "책", 1)); //{0:A, 1: 책, 2: 책, length: 3} 
    ```
    - Array-like와 같은 iterable object도 사용 가능

- includes()
  - 형태: Array.prototype.includes()
  - 파라미터: 비교하려는 값 | 비교 시작 인덱스(option) - 디폴트 0
  - 반환: true(있음), false(없음)
  
  - 대상 배열에
    - 첫 번째 파라미터 값이 있으면 true, 없으면 false를 반환
      ```javascript
      const list = [10, 20, 30];
      console.log(list.includes(10)); // true
      console.log(list.includes(10, 1)); // false
      ```
      - 10이 있지만 비교 인덱스가 1부터 이므로 false
    - 두 번째 파라미터는 선택이며 비교 시작 인덱스 작성
  - Generic 함수
    ```javascript
    const like = {0: 10, 1: 20, 2: 30, length: 3};
    console.log(Array.prototype.includes.call(like, 10)); // true
    ```

### 배열 차원 변환

- flat()
  - 형태: Array.prototype.flat() // ES2019
  - 파라미터: 대상 깊이(option) - 디폴트 1
  - 반환: 새로운 배열
  
  - 배열 차원을 반환하고 새로운 배열로 설정하여 반환
    - 파라미터의 대상 깊이에 따라 변환이 다르다.
      ```javascript
      const list = [1, 2, [3, 4]];
      const result = list.flat();
      console.log(list); // [1, 2, [3, 4]];
      console.log(result); // [1, 2, 3, 4];
      ```
      - flat() 파라미터에 값을 작성하지 않았으며 디폴트는 1이다.
      - 파라미터에 1을 더하면 2차원이 되며 2차원까지를 엘리먼트로 변환
      - [1, 2]는 1, 2 가 되며, [[3, 4]] 도 3, 4,가 된다.
      - 변환한 엘리먼트를 새로운 배열에 설정하여 반환, 따라서 1차원 배열의 엘리먼트로 설정
      - flat() 대상인 list는 바뀌지 않는다.
  - 파라미터에 0을 작성한 경우
    ```javascript
    const list = [1, 2, [3, 4]];
    console.log(list.flat(0)); // [1, 2, [3, 4]];
    ```
    - 파라미터에 값 0에 1을 더하면 1이다.
    - [1, 2]는 1, 2가 되며 배열에 설정하여 반환하므로 [1, 2]가 된다.
    - [[3, 4]] 는 [3, 4]가 되며 배열에 설정하여 반환하므로 [[3, 4]]가 된다.
  - 파라미터에 1보다 큰 값을 작성한 경우
    ```javascript
    const list = [1, 2, [3, 4, [5, [6]]]];
    console.log(list.flat(2)); // [1, 2, 3, 4, 5, [6]]
    ```
    - 파라미터에 1을 더한 3차원까지 엘리먼트로 변환하므로 [5]까지 변환
    - 4차원인 6은 3차원을 빼어 [6]으로 변환하여 배열에 설정되기 때문에 [[6]]이 된다.    
  - 빈 엘리먼트 삭제
    ```javascript
    const list = [1, 2, , , , [3, 4]];
    console.log(list.length); // 6
    const change = list.flat();
    console.log(change); // [1, 2, 3, 4]
    console.log(change.length); // 4
    ```

- flatMap()
  - 형태: Array.prototype.flatMap(); // ES2019
  - 파라미터: 콜백 함수 | 콜백 함수에서 this로 참조할 object(option)
  - 반환: 새로운 배열
  
  - flat()과 기능은 같음
  - 배열을 반복하면소 콜백 함수 호출
    - 파라미터: 엘리먼트, 인덱스, 배열 전체
    - 콜백 함수에서 반환한 값을 배열로 반환
      ```javascript
      const list = [10, 20];
      const callback = (element, index, all) => {
          return element + 5;
      };
      console.log(list.flatMap(callback)); // [15, 25]
      console.log(list.map(callback)); // [15, 25]
      ```
      - 콜백 함수에서 파라미터로 넘겨준 값을 단지 값만 변경하여 반환하면 map()과 flatMap()의 차이가 없다.
  - map()과의 차이
    ```javascript
    const list = [10, 20];
    const callback = (element, index, all) => {
        return [element + 5];
    };
    console.log(list.map(callback)); // [[15], [25]]
    console.log(list.flatMap(callback)); // [15, 25]
    ```
    - 콜백 함수에서 배열로 반환하는 경우
    - map()은 반환된 배열을 다시 새로운 배열에 설정하여 반환하므로 2차원 배열이 된다.
    - flatMap()에 경우 반환된 값을 1차원으로 줄서 반환한다.

### Array iterator object 생성

- entries()
  - 형태: Array.prototype.entries()
  - 반환: Array iterator object

  - Array object를 Array iterator object로 생성하여 반환
  - 배열의 엘리먼트를 [key, value] 형태로 변환
    - array iterator object 구조
      - Iterator: ArrayIterator
        - __proto__: Array Iterator
        - function next() {}
      ```javascript
      const iterator = [10, 20].entries();
      console.log(iterator.next().value); // [0, 10]
      console.log(iterator.next().value); // [1, 20]
      ```
      - entries() 메서드를 통해 Array iterator object 생성
      - iterator.next()를 하면 value에 array-like로 생성하게 된다 {0: 0, 1: "A", length: 2}, {0: 1, 1: "B", length: 2}
      - array iterator object는 [key, value] 형태로, 배열의 인덱스가 key가 되고, 엘리먼트 값이 value가 된다.
  - for-of 문으로 전개
    ```javascript
    const iterator = ["A", "B"].entries();
    for(const property of iterator) {
        console.log(property); // [0, A], [1, B]
    }
    ```
    - 전개할 때에는 next() 보다 for-of가 편리하다.
    
    ```javascript
    const iterator = ["A", "B"].entries();
    for(const [key, value] of iterator) {
        console.log(`${key}: ${value}`); // 0: A, 1: B
    }
    ```
    - 분할 할당으로 key, value로 나눌 수 있다.
  - 이터레이터는 다시 반복할 수 없다.
    ```javascript
    const iterator = ["A", "B"].entries();
    for(const [key, value] of iterator) {
        console.log(`${key}: ${value}`); // 0: A, 1: B
    }
    for(const property of iterator) {
        console.log("다시 전개"); // 출력되지 않는다.
    }
    console.log(iterator.next()); // {value: undefined, done: true}
    
- keys()
  - 형태: Array.prototype.keys()
  - 반환: Array iterator object
  
  - Array object를 Array iterator object로 생성하여 반환
    - entries()와 같으며 [key, value]의 형태가 아닌 key만 반환
  - 배열 인덱스가 key가 된다.
    ```javascript
    const iterator = ["A", "B"].keys();
    console.log(iterator.next()); // {value: 0, done: false}
    console.log(iterator.next()); // {value: 1, done: false}
    console.log(iterator.next()); // {value: undefined, done: true}
    ```
  
    ```javascript
    const iterator = ["A", "B"].keys();
    for(const property of iterator){
        console.log(iterator.next()); // 0, 1
    }
    ```

- values()
  - 형태: Array.prototype.values()
  - 반환: Array iterator object
  
  - array object를 array iterator object로 생성, 반환
  - [key, value] 형태에서 key는 반환하지 않고 value 만 반환
  - 배열의 엘리먼트 값이 value가 된다.
    ```javascript
    const iterator = ["A", "B"].values();
    console.log(iterator.next()); // {value: A, done: false}
    console.log(iterator.next()); // {value: B, done: false}
    console.log(iterator.next()); // {value: undefined, done: true}
    ```
    
    ```javascript
    const iterator = ["A", "B"].keys();
    for(const property of iterator) {
        console.log(property); // A, B
    }
    ```
  - Symbol.iterator]() 사용과 같다
    ```javascript
    const check = Array.property.values === Array.prototype[Symbol.iterator];
    console.log(check); // true
    ```
    - values() 대신 [Symbol.iterator]()를 사용해도 결과가 같다.
  - 값이 연동된다.
    ```javascript
    let list = ["A", "B"];
    let iterator = list.values();
    
    list[0] = "연동";
    console.log(iterator.next()); // {value: 연동, done: false}
    console.log(iterator.next()); // {value: B, done: false}
    ```
    - Array Iterator Object에서 배열의 메모리 주소를 참조하므로 값이 연동된다.

** 출처1. 인프런 강좌_자바스크립트 ES6+
