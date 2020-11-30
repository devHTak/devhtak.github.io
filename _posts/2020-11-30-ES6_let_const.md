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
  
- 글로벌 변수 오해
  - 글로벌 변수
    - 글로벌 오브젝트의 로컬 변수
    - var value = 100 처럼 글로벌 변수도 var 키워드를 사용이 정상
    - var 키워드를 작성하지 않으도 글로벌 변수로 간주한다.
      ```javascript
      value = 100;
      function point() {
          value = 300;
          console.log(value);
      }
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
      }
  }
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
    }
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
    }
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
        node.children[k].onclick = function(event) {
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
        node.children[k].onclick = function(event) {
            event.target.style.backgroundColor = "yellow";
            console.log(k);                                
        }
    }
    ```
      - let 변수는 for문 안에서 스코프가 결정되기 때문에 이벤트를 설정할 때의 k 값을 출력한다.
      
