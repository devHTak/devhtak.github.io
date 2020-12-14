---
layout: post
title: (Javascript ES6) RegExp Object
summary: 인프런 강좌_자바스크립트 ES6+ 기본
author: devhtak
date: '2020-12-14 22:41:00 +0900'
category: Javascript ES6+
---

### RegExp Object

- last index
  - 정규 표현식 사용 형태
    ```javascript
    const value="ABC";
    const obj = new RegExp("A", "g");
    console.log(obj.test(value)); // true
    const result = /A/g;
    console.log(reg.test(value)); // true
    ```
    - /A/g
      - 정규 표현식 리터럴 사용 형태로 new 연산자를 사용하지 않을 뿐 new RegExp("A", "g");와 같다.
      
  - 매치 시작 위치를 lastIndex 프로퍼티에 설정, 디폴트 값: 0    
  - g 플래그를 사용하면 lastIndex 프로퍼티 위치부터 매치
    ```javascript
    const value="ABABA", obj=/B/g
    console.log(obj.test(value) + ": " + obj.lastIndex); // true: 2
    console.log(obj.test(value) + ": " + obj.lastIndex); // true: 4
    console.log(obj.test(value) + ": " + obj.lastIndex); // false: 0
    ```
    - lastIndex가 1, 3이 아닌 2, 4를 나타내는 겂은 매치된 인덱스 다음부터 확인한다는 뜻이다.
    - 다음 매치되는 값이 없으면 0을 return 한다.
    
  - g 플래그를 사용하지 않으면 lastIndex 프로퍼티 값이 바뀌지 않는다.
    - lastIndex 값을 지정해도 적용되지 않고 0번 인덱스부터 매치
      ```javascript
      const value = "ABABA", obj=/B/;
      console.log(obj.test(value) + ": " + obj.lastIndex); // true: 0
      console.log(obj.test(value) + ": " + obj.lastIndex); // true: 0
      ```
      - g 플래그를 사용하지 않으면 lastIndex는 0을 리턴한다.
      - lastIndex가 0이라는 것은 적용되지 않는다는 것을 의미한다.
      
      ```javascript      
      const value = "ABABA", obj=/B/;
      console.log(obj.test(value) + ": " + obj.lastIndex); // true: 0
      obj.lastIndex=2;
      console.log(obj.test(value) + ": " + obj.lastIndex); // true: 2
      console.log(obj.test(value) + ": " + obj.lastIndex); // true: 2
      ```
      - obj.lastIndex = 2;
        - 강제로 lastIndex를 설정했으므로 2번 인덱스부터 매치를 해야 하지만 0번 인덱스부터 매치
        - 2번 인덱스부터 매치하면 마지막 console.log()에 대한 값이 false를 반환해야 하지만 위에 이유로 true를 반환한다.
        
### y flag

- lastIndex 위치에 매치
  - lastIndex 부터가 아니라 lastIndex 위치에 매치
  - 매치되면 lastIndex 값이 1 증가
    ```javascript
    const value="AABBA", obj=/A/y;
    console.log(obj.test(value) + ": " + obj.lastIndex); // true: 1
    console.log(obj.test(value) + ": " + obj.lastIndex); // true: 2
    console.log(obj.test(value) + ": " + obj.lastIndex); // false: 0
    ```
    - y flag 도 매치되면 1을 더한다.
    - 4번 인덱스에 A가 있지만 2번 인덱스에 매치하며 2번 인덱스 값이 B이므로 매치되지 않는다.
    - lastIndex에 존재하는 경우만 true를 넘겨준다.
    
- lastIndex 값을 지정할 수 있다.
  - sticky 프로퍼티에 true 설정
    ```javascript
    const value = "AABBA", obj=/A/y;
    console.log(obj.sticky); // true
    obj.lastIndex = 4;
    console.log(obj.test(value) + ": " + obj.lastIndex); // true: 5
    ```
    - obj.sticky
      - y flag를 사용하면 sticky 프로퍼티에 true가 설정
    - obj.test(value)
      - lastIndex를 4로 설정했기 때문에 마지막 A가 매치되어 true 출력, lastIndex는 1을 더한 5가 된다.
      
### u flag

- 정규 표현식의 패턴을 유니코드의 코드 포인트로 변환하여 매치
  - unicode 프로퍼티에 true 설정
    ```javascript
    const obj = new RegExp("\u{31}\u{32}", "u");
    console.log(obj.test("12")); // true
    console.log(obj.unicode); // true
    ```
- u flag를 사용하지 않으면 코드 포인트를 문자로 매치
  ```javascript
  const obj = /u{31}\u{32}/;
  console.log(obj.test("12")); // false
  ```
  
### s flag

- 정규 표현식에서 dot(점, .)은 모든 문자를 매치하지만 줄 바꿈 문자는 매치하지 않는다.
- s flag를 사용하면 줄 바꿈 문자를 매치 (ES2018)
  - dotAll 플래그에 true 설정
    ```javascript
    const text = `line
    줄을 바꿈`;
    // 이전 방법
    console.log(/[\s\S]+/.test(text)); // true
    console.log(/[^]+/.test(text)); // true
    
    // ES2018 이후 방법 - s flag
    const obj = new RegExp(".+", "s");
    console.log(obj.test(text)); // true
    console.log(obj.dotAll); // true
    ```
    - s flag를 사용하면 줄 바꿈 문자를 매치할 수 있다.
- 줄 바꿈 문자

** 출처1. 인프런 강좌_자바스크립트 ES6+
