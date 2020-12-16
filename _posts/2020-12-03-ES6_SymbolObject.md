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
    - const obj = list[Symbol.iterator] ();
      - 위 형태로 호출하면 이터레이터 오브젝트를 생성하여 반환한다.
      - obj의 __proto__를 펼치면 next()가 있다. next()가 있으므로 obj는 iterator object이다. 
  
### Symbol Object

- primitive 값
  - 자바스크립트에서 Primitive 값은 오브젝트가 아니라 값이며 함수를 갖고 있지 않음
  - const num = 100; 을 할당하면 num 변수에 100만 할당되며 아무것도 첨부되지 않음
    - 100이 primitive 값
  - ES5의 primitive 값 타입
    - string, number, boolean, null, undefined
  - ES6의 symbol 타입 추가
 
- Wrapper object
  - wrapper object
    - primitive 값이 포함된 오브젝트
    - wrapper object에는 메소드가 있음
  - wrapper object가 있는 primitive 값 ㅇ타입
    - string: String, number: Number object, boolean: Boolean, symbol: Symbol object
  - const obj = new String(100);
    - obj 인스턴스의 [[PrimitiveValue]]에 100이 설정
    - [[PrimitiveValue]] 형태
      ```javascript
      const sports = new String(100);
      const sym = Symbol("ABC");
      ```
      - debugger 모드에서 sports를 펼치면 [[PrimitiveValue]]: "100"이 있다.
      - [[PrimitiveValue]]가 프리미티브 값을 나타내는 프로퍼티 이름이며 "100"이 프로퍼티 값이다.
      - sports는 wrapper object이다.
      - sym은 펼칠 수가 없으며 [[PrimitiveValue]]가 표시되지 않는다.
      - 그렇다고 Symbol에 Primitive 값이 없는 것은 아니며 이 Symbol은 Primitive 값을 외부에 노출시키지 않는 특성 때문이다.      
  - undefined, null은 wrapper 오브젝트 없다

- Symbol() 함수
  - 형태: Symbol()
  - 파라미터: 설명, 주석(option)
  - 반환: 유일한 Symbol 값
  
  - Symbol() 함수는 값을 생성하여 반환
    - 반환된 값을 볼 수 없다.
      ```javascript
      const sym = Symbol();
      console.log(sym);
      console.log(typeof sym);
      ```
      - const sym = Symbol(); // Symbol 오브젝트가 아닌 Symbol 값을 생성하여 반환
      - console.log(sym); // 생성한 Symbol 값이 출력되지 않고 Symbol 값을 생성한 코드 형태가 표시
      - console.log(typeof sym); // Symbol로 생성한 값 타입은 symbol이다.
    - new 연산자를 사용할 수 없다.
  - 프로그램 전체를 통해 유일한 값 제공
    ```javascript
    const one = Symbol(); const two = Symbol();
    console.log(one === two); // false
    ```
    - Symbol() 을 실행할 때마다 프로그램 전체에서 하나만 있는 값을 생성
  - Symbol 값으로 연산 불가
    ```javascript
    let sym = Symbol();
    try {
        const add = sym + 5;
    } catch(e) {
        console.log("연산 불가");
    }
    ```
    - Symbol은 값이지만 연산할 수 없다.
  - Symbol 타입 변경 불가 
    ```
    let sym = Symbol();
    try {
      +sym
    } catch(e) {
        console.log("값 타입 변경 불가");
    }
    ```
    - +sym: 단항 +연산자는 Number 타입으로 변경. Symbol 타입을 바꿀 수는 없다.
    - 외부에 Symbol 값이 노출되는 처리(계산, 변환 등) 을 할 수 없다.
  - 파라미터에 주석, 설명을 작성
    ```javascript
    const sym = Symbol("주석, 설명");
    console.log(sym); // Symbol(주석, 설명);
    ```
    - 파라미터에 작성한 문자를 생성한 값의 문자열로 작성한다.
    - 생성한 Symbol 값을 볼 수 없으므로 값 설명이 필요할 때 사용한다.
    - Symbol() 실행에 영향을 미치지 않는다.
  - Symbol 값을 문자열로 바꿔서 연결
    ```javascript
    const sym = Symbol("설명");
    console.log(sym.toString() + "연결"); // Symbol(설명)연결
    ```
    - Symbol 값을 toString()으로 변환하면 에러가 발생하지 않지만 값이 변환되지 않고 값을 만든 형태에 문자열을 연결
    - new String(sym) 형태는 에러가 발생
  - Template에 사용
    ```javascript
    const sym = Symbol("주석, 설명");
    try {
        `${sym}`
    } catch(e) {
        console.log("`${sym}` 불가");
    }
    ```
    - Symbol 값을 Template에 사용할 수 없다.
    - `${sym}` 불가

- Symbol 사용 형태
  - Object의 프로퍼티 키로 사용
    - Symbol 값이 유일하므로 중복되지 않는다.
    - symbol-keyed property라고 부른다.
      ```javascript
      const sym = Symbol("설명");
      const obj = {[sym]: 100};
      ```
      - const obj = {[sym]: 100};
        - Symbol 값을 Object의 프로퍼티 키로 사용
        - [sym] 처럼 대괄호 안에 Symbol()로 할당한 변수 이름을 작성
        - 이를 symbol-keyed property라고 부른다.
    - 프로퍼티 값 추출 방법
      ```javascript
      const sym = Symbol("설명");
      const obj = {[sym]: 100};
      console.log(obj[sym]); // 100
      console.log(obj.sym); // undefined
      ```
      - obj[sym]: Symbol() 결과를 할당한 sym을 프로퍼티 키로 사용하여 값을 구한다.
        - 프로퍼티 값인 100 출력
      - obj.sym : undefined가 출력되며 obj[sym] 형태로 사용해야 한다. 
  - Object에서 함수 이름으로 사용
    ```javascript
    const sym = Symbol("함수 이름");
    const obj = {
        [sym](param) {
            return param;
        }
    };
    console.log(obj[sym](200)); // 200
    ```
    - [sym](param) { } // 형태로 함수 정의
    - obj[sym](param); // 형태로 함수 호출

  - for-in 문에서 사용
    - Symbol이 열거되지 않음
    - [[Enumerable]]: false이기 때문
      ```javascript
      const obj = {
          [Symbol("100")]: 100,
          two: 200
      };
      for(let key in obj) {
          console.log(key); // two
      }
      ```
      - Object에 Symbol-keyed 프로퍼티를 사용하여 프로퍼티 값을 작성
      - for-in 문으로 열거되지 않고, 에러가 발생하지 않는다.
  - Object.getOwnPropertySymbols()로 열거 가능
  - for-of 문에서 사용
    - 배열 안에 Symbol() 작성
      ```javascript
      const list = [Symbol(100)];
      for(let value of list) {
          console.log(value); // Symbol(100)
      }
      ```
  - JSON.stringify()에서 사용
    - Symbol 값이 문자열로 변환되지 않음
      ```javascript
      const sym = Symbol("JSON");
      const result = JSON.stringify({[sym]: "ABC"});
      console.log(result); // {}
      ```
      - Symbol은 변환에서 제외된다. 이것은 Symbol 값을 외부에 노출하지 않기 위해서이다.

### Symbol Property

- Well-Known Symbols
  - 스펙에서 @@iterator 형태를 볼 수 있음
  - @@은
    - Well-Known Symbol을 나타내는 기호
    - @@match 와 Symbol.match가 같다
    - 스펙에서는 @@match 형태를 사용하고, 개발자는 Symbol.match를 사용
  - Well-Known Symbol 이란?
    - 스펙에서 알고리즘에 이름을 부여하고 이름으로 참조하기 위한 빌트인 Symbol 값
  - 개발자 코드 우선 실행
    - match()를 실행하면 Symbol.match 를 오버라이딩하지 않았으면 디폴트로 @@match 실행
    - 소스 코드에서 Symbol.match를 작성하면 @@match가 실행되지 않고, Symbol.match가 실행
    - 개발자 코드로 디폴트 기능을 오버라이딩할 수 있다.
  - Symbol 12개(ES2019 기준)
    |Symbol|대응|Symbol|대응|
    |------|---|---|---|
    |Symbol.asyncIterator|for-await-of|Symbol.hasInstance|instanceof|
    |Symbol.isConcatSpreadable|Array.prototype.concat()|Symbol.iterator|for-of|
    |Symbol.match|String.prototype.match()|Symbol.replace|String.prototype.replace()|
    |Symbol.search|String.prototype.search()|Symbol.species|cosntructor|
    |Symbol.split|String.prototype.split()|Symbol.toPrimitive|ToPrimitive()|
    |Symbol.toStringTag|Object.prototype.toString()|Symbol.unscopables|with|
    
    - Symbol.toStringTag
      - Object.prototype.toString()의 확장(overriding)
      - toString()으로 인스턴스 타입을 구하면 [object Object] 형태로 반환
        - 인스턴스 타입을 명확하게 구할 수 없다.
          ```javascript
          const Book = function() {}
          const obj = new Book();
          console.log(obj.toString()); // [object Object]
          console.log({}.toString()); // [object Object]
          ```
      - Symbol.toStringTag로 구분 가능
        - [object Object] 에서 두 번째에 표시될 문자열을 작성
        - 예: "ABC" 지정, [object "ABC"]로 반환
          ```javascript
          const Sports = function() {}
          console.log(Sports.toString()); //[object Object]
          
          Sports.prototype[Symbol.toStringTag] = "농구";
          console.log(obj.toString()); // [object 농구]
          ```
          - Sports.prototype[Symbol.toStringTag] = "농구";
            - prototype에 Symbol.toStringTag를 연결하고 Object에 작성할 문자를 농구로 작성했다.
            - 표시될 문자를 임의로 작성할 수 있으며 function 마다 지정할 수 있다.
    - Symbol.isConcatSpreadable
      - Array.prototype.concat()은 배열의 엘리먼트를 전개하여 반환
        ```javascript
        const one = [10, 20], two = ["A", "B"];
        const show = () => {
            console.log(one.concat(two));
        };
        show(); // [10, 20, A, B]
        two[Symbol.isConcatSpreadable] = true;
        show(); // [10, 20, A, B]
        two[Symbol.isConcatSpreadable] = false;
        show(); // [10, 20, [A, B]]
        ```
        - 대상이 Array이면, 전개하는 것이 default이다.
        - [Symbol.isConcatSpreadable] = true
          - one 배열 끝에 two 배열의 엘리먼트를 하나씩 연결
        - [Symbol.isConcatSpreadable] = false
          - 전개하지 않고 two 배열 자체를 연결
      - Array-like 전개
        ```javascript
        const one = [10, 20];
        const like = {0: "A", 1: "B", length: 2};
        const show = () => {
            console.log(one.concat(like));
        };
        show(); // [10, 20, {0: A, 1: B, length: 2}]
        two[Symbol.isConcatSpreadable] = true;
        show(); // [10, 20, A, B]
        two[Symbol.isConcatSpreadable] = false;
        show(); // [10, 20, {0: A, 1: B, length: 2}]
        ```
        - Array-Like는 전개하지 않는 것이 default로, Array와 반대이다.
        - Symbol.isConcatSpreadable이 true면 그때 전개한다.
    
    - Symbol.species
      - Symbol.species 는 cosntructor를 반환
        - constructor를 실행하면 인스턴스를 생성하여 반환하므로 결국, 인스턴스를 반환        
      - Symbol.species 를 오버라이드하면 다른 인스턴스를 반환할 수 있다는 의미
      - 메서드를 실행한 후의 결과 형태
        ```javascript
        const obj = [1, 2, 3];
        const one = obj.slice(1, 3);
        const two = one.slice(1, 2); 
        ```
        - [1, 2, 3]으로 Array Object를 생성하여 obj에 할당
          - debugger 모드에서 obj 구조를 보면 prototype은 없고 __proto__만 있으므로 obj는 빌트인 Array Object가 아니라 Array.prototype에 연결된 메서드로 생성한 인스턴스.
        - const one = obj.slice(1, 3);
          - 구조는 차이가 없으며 값 [2, 3]만 다르다. 이 것은 인스턴스에 있는 메서드를 호출하면 메서드 실행 결괏값을 반환하지 않고 결괏값이 설정된 인스턴스를 반환하기 때문이다.
        - const two = one.slice(2, 3); 
          - 바로 앞에서 반환된 one으로 메서드를 호출할 수 있다는 것은 one이 인스턴스이기 때문, two 또한 인스턴스로 구조 또한 같다
      - Symbol.species 기능
        ```javascript
        class Sports extends Array {};
        const obj = new Sports(10, 20, 30);
        const one = obj.slice(1, 2);
        console.log(one); // [20]
        ```
        - class Sports extends Array{}
          - 빌트인 Array object를 상속(확장, 연결) 받습니다.
        - const obj = new Sports(10, 20, 30);
          - 인스턴스 생성
        - const one = obj.slice(1, 2);
          - obj 인스턴스의 slice()를 호출하면 slice() 처리 결과를 인스턴스에 설정하여 인스턴스를 반환
          - 즉, obj 안에 prototype - constructor 가 없는데, 인스턴스를 반환해주는 것이 Symbol.species 기능이다.
        - 이렇게 인스턴스의 메서드를 호출했을 때 인스턴스를 반환하도록 하는 것이 Symbol.species 기능
      - Symbol.species 오버라이드
        ```javascript
        class Sports extends Array {
            static get [Symbol.species]() {
                return Array;
            }
        };
        const one = new Sports(10, 20, 30);
        console.log(one instanceof Sports); // true
        
        const two = one.slice(1, 2);
        console.log(two instanceof Sports); // false
        console.log(two instanceof Array); // true
        // Symbol.species가 Array를 return하기 때문에 Array에서 true다
        ```
        - Symbol.species 는 static Accessor property 이고, getter만 있고, setter가 없다.
        - Symbol.species를 사용할 수 있는 빌트인 오브젝트
          - Array, Map, Set, RegExp
          - Promise, ArrayBuffer, TypedArray
        - 빌트인 오브젝트를 상속받은 class에 Symbol.species를 작성하면 빌트인 오브젝트의 @@species가 오버라이드 된다.
    - Symbol.toPrimitive
      - 오브젝트를 대응하는 Primitive 값으로 변환
      - 대응, 기대하는 타입
        - number, string, default
      - 오브젝트를 문자열에 대응
        ```javascript
        const point = {bonus: 100};
        console.log(point.toString()); // [object Object]
        const book = {
            toString() {
                return "책";
            }
        };
        console.log(`${book}`); // 책
        ```
        - 문자열은 toString()을 오버라이딩한다.
      - 오브젝트를 숫자에 대응
        ```javascript
        const point = {bonus: 100};
        console.log(point.valueOf()); // {bonus: 100}
        
        const book = {
            toString() { return 70 },
            valueOf() { return 30}
        };
        console.log(book * 20); // 600
        ```
        - 숫자는 valueOf()를 오버라이딩한다.
      - Symbol.toPrimitive() 사용
        ```javascript
        const obj = {
            [Symbol.toPrimitive](hint) {
                return hint === "number" ? 30 : hint === "string" ? "책" : "default";
            }
        };
        console.log(20 * obj); // 600
        console.log(`${obj}` + 100); //책100
        console.log(obj + 50); // default50
        console.log("default" == obj); // true
        ```
        - 20* obj
          - *에 맞는 Primitive로 변환하므로 number가 된다.
        - `${obj}`
          - template에 경우 string으로 판단한다
        - obj + 50
          - obj에 대한 값이 없으므로 default로 판단한다
        - "default" == obj
          - == 와 같은 비교에는 default가 설정된다.
    - Symbol.iterator
      - @@iterator가 있는 빌트인 오브젝트
        - String, Array, Map, Set, TypedArray
      - 빌트인 Object에는 @@iterator가 없지만, 구현할 수 있다.
      - Array Object
        - Array object의 [Symbol.iterator]()를 호출하면 iterator object 반환
          - next()로 배열 엘리먼트 값을 구할 수 있다.
            ```javascript
            const list = [10, 20];
            const obj = list[Symbol.iterator]();
            console.log(obj.next()); // {value: 10, done: false}
            console.log(obj.next()); // {value: 20, done: false}
            console.log(obj.next()); // {value: undefined, done: true}
            ```
      - String Object
        - String object의 [Symbol.iterator]()를 호출하면 iterator object 반환
          - next()로 배열 엘리먼트 값을 구할 수 있다.
            ```javascript
            const list = "1A";
            const obj = list[Symbol.iterator]();
            console.log(obj.next()); // {value: 1, done: false}
            console.log(obj.next()); // {value: A, done: false}
            console.log(obj.next()); // {value: undefined, done: true}
            ```
      - 빌트인 Object
        - 빌트인 Object에는 Symbol.iterator가 없다.
          - Symbol.iterator가 반복을 처리하므로 Object에 Symbol.iterator를 작성하면 반복할 수 있다.
            ```javascript
            const obj = {
                [Symbol.iterator]() {
                    return {
                        count: 0,
                        maxCount: this.maxCount,
                        next() {
                            if(this.count < this.maxCount) {
                                return {value: this.count++, done: false};
                            }
                            return {value: undefined, done: true};
                        }
                    };
                }
            };
            obj.maxCount = 2;
            for(const value of obj) {
                console.log(value); // 0, 1
            }
            ```
            - 엔진이 for-of 문을 시작하면 먼저 obj에서 [Symbol.iterator]를 검색, 이를 위해 obj에 [Symbol.iterator] 작성
      - 제너레이터 함수 연결
        - Object{}에 Symbol.iterator를 작성하고 generator 함수를 연결하면 반복할 때마다 yield를 수행
          ```javascript
          const obj = {};
          obj[Symbol.iterator] = function*() {
              yield 1; 
              yield 2; 
              yield 3;
          };
          console.log([...obj]); // [1, 2, 3]
          ```
          - [...obj]를 실행하면 [Symbol.iterator]를 검색하고 있는 경우 yield가 끝날 때까지 반복하여 배열을 만든다
        - 연결 구조
          - Symbol.iterator의 __proto__에 제너레이터 오브젝트가 있는 구조
        - 제너레이터 오브젝트에 이터레이터 오브젝트를 연결하여 값을 공유하는 형태
          - 제너레이터 오브젝트에 이터레이터 오브젝트가 포함된 구조
            ```javascript
            const gen = function*() {
                yield 10;
                yield 20;
            };
            const genObj = gen();
            console.log(genObj.next()); // {value: 10, done: false}
            
            const obj = genObj[Symbol.iterator]();
            console.log(obj.next); // {value: 20, done: false}
            ```
            - const obj = genObj[Symbol.iterator]();
              - iterator 객체를 반환하기 때문에 다음 yield가 나온다.
          

        
** 출처 1. 인프런 강좌_자바스크립트 ES6+ 기본
