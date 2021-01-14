---
layout: post
title: (Javascript ES6) let, const 변수
summary: 인프런 강좌_자바스크립트 ES6+ 기본
author: devhtak
date: '2020-11-30 22:41:00 +0900'
category: Javascript ES6+
---

### 변수 구분

- 로컬(지역) 변수, 글로벌(전역) 변수
  - 변수를 구분하는 이유는 기능과 목적이 다르기 때문이다.
  - 글로벌 변수의 기능과 목적
    - 다른 js 파일에서 변수값 공유
    - 파일에서 공통 변수 개념으로 사용
    - 의도는 좋으나 처리 속도가 떨어진다.
  - 로컬 변수의 기능과 목적
    - 빠르게 식별자를 해결하기 위해 가까운 스코프의 변수를 사용하려는 것
  - var 키워드의 문제
    - var vs const, let
      - var 변수는 scope 기준을 function scope
        ```javascript
        for(var i = 0; i < 10; i++) {
            console.log('i :' + i); // i: 0 ~ 9
        }
        console.log(i); // 10
        ```
          - i는 for문 block에 선언하여 있지만, 사용할 수 있다.
        ```javascript
        function count() {
            for(var i = 0; i < 10; i++) {
                console.log('i :' + i); 
            }
        }
        count(); // i: 0 ~ 9
        console.log(i); // i is not defined.
        ```
          - count()에 선언되어 있는 i를 다시 사용할 수 없다.
      - const, let 변수는 block scope
        ```javascript
        for(let i = 0; i < 10; i++) {
            console.log('i :' + i); // i: 0 ~ 9
        }
        console.log(i); // i is not defined.
        ```
  
- 글로벌 변수 오해
  - 글로벌 변수
    - 글로벌 오브젝트의 로컬 변수
    - var value = 100 처럼 글로벌 변수도 var 키워드 사용이 정상
    - var 키워드를 작성하지 않으면 글로벌 변수로 간주한다.
      ```javascript
      value = 100;
      function point() {
          value = 300;
          console.log(value);
      };
      point(); // 300
      console.log(value); // 300
      ```
      - var 키워드를 사용하지 않고 value를 글로벌 변수로 선언하고 100 할당
      - point() 함수 안에서 value 변수에 300을 할당. value 변수가 로컬 변수가 아니므로 글로벌 오브젝트의 value 변수에 300할당
      - 함수 안에서 글로벌 변수에 값을 설정하는 것은 좋지 않다. 로컬 변수와 글로벌 변수를 구분한 목적을 생각해야 한다.
      
- use strict 사용(ES5)
  - 함수안에서 var 키워드를 사용하지 않으면 에러 발생
  - ES5에서 도입했으나 근본적인 접근이 아니다.
  ```javascript
  "use strict"
  function point(){
      try { 
          value = 300;
      } catch(e) {
          console.log("글로벌 변수 사용 불가");
      };
  };
  point(); //글로벌 변수 사용 불가
  ```
  - ES6+ 에서는 default로 "use strict"가 사용된다. 전체는 아니다.

### let 변수

- let 변수 개요
  - let book = "책";
    - block scope를 가진 변수
    - 변수가 선언된 블록이 스코프
  - 스코브 적용 기준
    - {} 블록, 문, 표현식
    ```javascript
    let sports = "축구";
    if(sports) {
        let sports = "농구";
        console.log(sports); // 농구
    }
    console.log(sports); // 축구
    // if 문 블록 안과 밖의 sports 변수를 각각 갖게 된다.
    ```
  - 블록{} 안과 밖이 스코프가 다르다
    - 변수 이름이 같아도 값이 대체되지 않음
  - 블록 안에서 밖에 변수를 접근할 수는 있지만, 밖에서 안에 변수를 접근할 수는 없다.
    ```javascript    
    let sports = "축구";
    if(sports) {
        sports = "농구";
        console.log(sports); // 농구
    }
    console.log(sports); // 농구
    ```
          
- let 변수 선언
  - syntax
    - let name1 [= value1] [, name2 [=value2]]
  - name1, name2 에 변수 이름 작성
    - 식별자로 사용
    - [] 는 생략 가능
    - 값을 할당하지 않아도 된다. 
    - 할당하지 않으면 undefined가 할당된다. var 변수에 경우 undefined 변수도 사용할 수 있지만, let 변수는 사용할 수 없다.
  - value1, value2에 초깃값 작성
    - 표현식 작성 가능, 평가 결과 사용

- 블록 스코프
  - 블록 기준
    - 중괄호 { 코드 }
    - function name() { 코드 }
    - if( a === 1 )  { 코드 }
  - 블록 안과 밖이 스코프가 다르다.
    - 변수 이름이 같아도 값이 대체되지 않는다.
  - 스코프에 같은 이름의 변수 사용 불가
  

### 스코프
- function 블록
  - function name(){} 블록 스코프
  - 스코프가 다르기 때문에 function 안과 밖에 같은 이름에 let 변수 선언 가능하다
    ```javascript
    let sports = "축구";
    function show() {
        let sports = "농구";
        console.log(sports); 
    };
    show(); // 농구
    console.log(sports); // 축구
    ```
  - function 밖의 let 변수를 function 안에서 사용 가능(클로저)
    ```javascript
    let sports = "축구";
    function show() {
        console.log(sports); 
    }
    show(); // 축구
    ```

- try-catch
  - try-catch 문도 블록 스코프
  - try 블록 {} 기준으로 안과 밖에 같은 이름의 let 변수 선언 가능
  - catch() 에서 try 블록 안에 변수를 사용하지 않고 try 밖의 변수 사용
    ```javascript
    let sports = "축구";
    try {
        let sports = "농구";
        console.log(sports); // 농구
        abc = error;
    } catch(e) {
        console.log("catch: " + sports); // "cathch: 축구"
    };
    console.log(sports); // 축구
    ```
    - try 블록 {} 또한 블록스코프이기 때문에 새롭게 변수 선언이 가능하며 log를 찍는다.
    - abc = error; 에서 error가 발생한다.
    - catch 문 안에서는 try에 선언한 sports가 아닌 블록 밖에 sports를 접근한다.

- switch-case
  - switch 문도 블록 스코프이다.
  - case, default는 블록 스코프가 아니다.
  
### let 변수와 var 변수 차이

- for 문
  - var 변수는 스코프를 갖지 않는다.
    ```html
    <ul class="sports">
        <li>축구</li>  
        <li>농구</li>
        <li>야구</li>
    </ul>
    ```
    ```javascript
    var node = document.querySelector(".sports");
    for(var i = 0; i < node.children.length; i++) {
        node.children[i].onclick = function(event) {
            event.target.style.backgroundColor = "yellow";
            console.log(k);                                
        }
    }
    ```
    - 어떤 것을 클릭하더라도 항상 for() 문이 끝났을 때의 값인 3을 출력한다.
    - var k = 0; 에서 k 변수의 스코프는 함수다.
  - let 변수는 스코프를 갖는다.
    ```javascript
    var node = document.querySelector(".sports");
    for(let i = 0; i < node.children.length; i++) {
        node.children[i].onclick = function(event) {
            event.target.style.backgroundColor = "yellow";
            console.log(k);                                
        }
    }
    ```
      - let 변수는 for문 안에서 스코프가 결정되기 때문에 이벤트를 설정할 때의 k 값을 출력한다.
      
### let 변수와 this

- 글로벌 오브젝트에서 let 변수를 this로 참조 불가
  ```javascript
  var music = "음악";
  let sports = "축구";
  console.log(this.music + " " + this.sports); //음악 undefined
  ```
   - 현재 위치는 글로벌 오브젝트이다.
   - var music = "음악"; 은 window object에 설정되지만 let sports = "축구"; 은 window object에 설정되지 않는다.
   - this.music 에서 this는 window object를 참조하지만 this.sports에서 sports가 window object에 설정되지 않으므로 undefined
   - global object에 선언된 let 변수는 script 블록에 속하게 된다.
- 글로벌 오브젝트에서 var와 let 변수가 설정되는 위치 구조

### JS 파일 다수 사용

- 다수의 JS 파일 사용
  - 모든 JS 파일에서 글로벌 오브젝트에 작성한 var 변수와 let 변수를 공유
    - 글로벌 오브젝트, script 블록에 있는 변수들을 공유할 수 있다.
  - let 변수는 블록 안에 작성하면 공유되지 않는다.
    - 새로운 블록 안에 작성한 let 변수는 공유할 수 없다.

- let 변수의 scope
  - debugging 모드로 살펴보면 let 변수는 script(global), block({}), local(function(){})에 선언된 블록에 따라 설정된다.

- 정리
  - 글로벌 오브젝트 작성 
    ```javascript
    var globalVar = "var 변수";
    let globalLet = "let 변수";
    {
        let globalBlock = "block 변수";
    };
    ```
    - var 변수: window 객체에 설정, 공유
    - let 변수: script에 설정, 공유
      - window.sports = {} 처럼 의도적으로 작성하지 않아도 된다.
    - 블록안 let 변수: Block에 설정, 공유하지 않음
      - 글로벌 오브젝트에서만 사용하는 로컬 변수로 사용
  - 함수에 작성
    ```javascript
    function showLocal() {
        var localVar = "var 변수";
        let localLet = "let 변수";
        {
            let localBlock = "block 변수";
        };
    }
    ```
    - var 변수, let 변수 : Local에 설정
    - { let localBlock } : Block에 설정

### Hoisting

- ES5의 실행 컨텍스트 처리 순서
  - 함수 선언문 설정
  - 변수 이름 바인딩 변수값은 undefined (+함수 표현식)
  - 소스 코드 실행
    ```javascript
    console.log(music); // undefined
    var music="음악";
    ```
    - music 변수를 아래에 선언했지만, 먼저 undefined로 바인딩 하였기 때문에 식별자 해결을 할 수 있다.
    - 이 것이 호이스팅이다.
    - 식별자 해결을 하지 못하면 에러가 발생한다.

- let 변수는 호이스팅이 되지 않는다.
  - 즉, let 변수는 앞에서 변수 사용일 불가능하다.
  ```javascript
  try { 
      console.log(sports);
  } catch(e) {
      console.log('호이스팅 불가');
  };
  let sports = "축구";
  // 호이스팅 불가 출력
  ```

- let 변수를 인식하는 시점
  - let 변수는 선언을 한 후에야 변수를 인식할 수 있다. 즉, 식별자를 해결할 수 있다.
  - let globalList; // 엔진이 undefined를 할당한다.
  
- block 안에 let 변수 작성
  - 초기화 단계(Initializer)
    - 정적 환경의 선언적 환경 레코드에 변수 이름을 바인딩 한다.
    - 이때 var 변수는 undefined를 초기값으로 설정하고 let 변수는 초기값을 설정하지 않는다.
  - 변수에 값을 넣으면 변수로 인식하고, 변수에 값이 없으면 변수로 인식하지 않는다. (Binding List)
    - let 변수 선언을 실행하면 그 때 값이 설정되며, 값을 할당하지 않고 변수를 선언만 하면 엔진이 undefined를 할당
    - 따라서, 변수 선언을 실행한 후에는 변수가 값을 갖고 있으므로 변수를 인식할 수 있다.
  - 엔진에서는 Initializer와 Binding List 메커니즘을 사용
  - 즉, var 변수는 초기화 단계에서 undefined 값을 갖게 되고 let 변수는 값을 갖지 않는다. 그래서 엔진이 값을 갖고 있는 변수를 인식하여 var 는 hoisting이 가능하고 값을 갖고 있지 않는 let 변수는 hoisting이 불가능하다.
  
### const 변수

- const 변수
  - 구문: const name1 = value1 [, name2 = value2]
  - 값을 바꿀 수 없는 변수 선언
  - name1에 변수 이름 작성, 식별자로 사용
    ```javascript
    const sports = "운동";
    try {
        sports = "농구";
    } catch(e) {
        console.log("const 할당 불가");
    }
    ```
  - value1, value2에 초기값 작성
    - 반드시 값을 작성, 변수 선언만 불가
    - 표현식 작성 가능, 평가 결과 사용
  - 우선 let이 아닌 const 사용 가능을 검토
  
- const 변수는 값을 바꿀 수 있는가?
  - const 변수 전체를 바꿀수는 없다.
  - Object의 프로퍼티는 변경할 수 있다.
    ```javascript
    const book = {title: "책"};
    try {
        book = {title: "음악 책"};
    } catch(e) {
        console.log("const 전체 할당 불가");
    };
    book.title = "미술 책";
    console.log(book.title); // 미술 책
    ```
    - book 객체 전체를 변경할 수는 없지만, book.title과 같이 프로퍼티 값을 변경하는 것은 가능하다.
  - 배열의 엘리먼트 값도 바꿀 수 있다.
    ```javascript
    const book = ["책"];
    try {
        book = ["음악 책"];
    } catch(e) {
        console.log("const 전체 할당 불가");
    }
    book[0] = "미술 책";
    console.log(book[0]); // 미술 책
    ```
    - book 배열 전체를 변경할 수는 없지만, 배열 index 요소는 변경할 수 있다.
    
** 출처1. 인프런 강좌_자바스크립트 ES6+ 기본
