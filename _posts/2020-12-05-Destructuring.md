---
layout: post
title: (Javascript ES6) Destructuring
summary: 인프런 강좌_자바스크립트 ES6+ 기본
author: devhtak
date: '2020-12-05 23:41:00 +0900'
category: Javascript ES6+
---

### Destructuring

- Destructuring
  - Destructuring Assignment => 분할 할당
  - 작성 형태
    ```javascript
    let one, two, three;
    const list = [ 1, 2, 3 ];
    [one, two, three] = list;
    console.log(one); // 1
    console.log(two); // 2
    console.log(three); // 3
    console.log(list); // [ 1, 2, 3];
    ```  
  - 사전적 의미
    - ~ 구조를 파괴하다
    - 파괴, 해체는 있는 것이 없어지는 뉘앙스
    - 원 데이터는 변경되지 않음
  - 이 관점에서 보면 분할 / 분리가 더 가깝다

### Array Destructuring Assignment
  - 배열의 엘리먼트를 분할하여 할당
    - 인덱스에 해당하는 변수에 할당
      ```javascript
      let one, two, three;
      const list = [ 1, 2, 3 ];
      [one, two, three] = list;
      console.log(one); // 1
      console.log(two); // 2
      console.log(three); // 3
      ```
      - 왼쪽부터 인덱스에 해당하는 오른쪽 배열의 값을 변수에 할당한다.
      - one, two, three에 1, 2, 3이 할당된다.
    - 할당 받을 변수 수가 적은 경우
      ```javascript
      let one, two;
      const list = [ 1, 2, 3 ];
      [one, two] = list;
      console.log(one); // 1
      console.log(two); // 2
      ```
      - 왼쪽에 할당받을 변수가 2개이고 오른쪽에 분할할당할 값이 3개이다.
      - 왼쪽 인덱스에 맞추어 값을 할당하여 3은 할당되지 않는다.
    - 할당 받을 변수 수가 많은 경우
      ```javascript
      let one, two, three, four;
      const list = [ 1, 2, 3 ];
      [one, two, three, four] = list;
      console.log(three); // 3
      console.log(four); // undefined
      ```
      - 왼쪽에 할당받을 변수가 4개이고 오른쪽에 분할할당할 값이 3개이다.
      - 왼쪽에 값을 할당할 수 없는 변수에 undefined가 설정된다.
    - 배열 차원에 맞추어 분할 할당
      ```javascript
      let one, two, three, four;
      [one, two, [three, four]] = [1, 2, [3, 4]];
      console.log([one, two, three, four]); //[1, 2, 3, 4]
      ```
      - [three, four]와 [3, 4]가 배열이다.
      - 배열 차원이 변환된다.
    - 매치되는 인덱스에 변수가 없으면 값을 할당하지 않음
      ```javascript
      let one, two, three, four;
      [one, , , four] = [1, 2, 3, 4];
      console.log([one, two, three, four]); //[1, undefined, undefined, 4]
      ```
      - [one, , , four] 형태에서 콤마로 구분하고 변수를 작성하지 않았다.
      - 인덱스를 건너 띄어 할당
      - one에 1을 할당, 2와 3은 건너 띄고 four에 4를 할당
  - spread와 같이 사용
    - 나머지를 전부 할당
      ```javascript
      let one, rest;
      [one, ...rest] = [ 1, 2, 3, 4];
      console.log(one); // 1
      console.log(rest); //[ 2, 3, 4];
      ```
      - one에 1을 할당하고 나머지 2, 3, 4를 나머지 rest에 [2, 3, 4] 배열로 할당
      - rest 파라미터를 호출받는 함수의 파라미터에 작성하지만, 나머지라는 시맨틱이 강해서 코드처럼 사용하기도 한다.
      - 분리하지 않고 결합된 상태를 설정하므로 어긋나지 않는다.
    - 인덱스를 반영한 나머지 할당
      ```javascript
      let one, three, rest;
      [one, , three, ...rest] = [ 1, 2, 3, 4, 5];
      console.log(one); // 1
      console.log(three); // three
      console.log(rest); // [4, 5]
      ```
      - one에 1을 할당하고 2는 건너띄고 three에 3을 할당한다.
      - 나머지 값 [4, 5]는 rest에 할당

### Object Dstructuring Assignment

  - Object의 프로퍼티를 분할하여 할당
  - 프로퍼티 이름이 같은 프로퍼티에 값을 할당
    ```javascript
    const {one, two} = {one: 10, two: 20};
    console.log(one); // 10
    console.log(two); // 20
    ```
    - 왼쪽의 object가 {name: value}형태가 아니라 프로퍼티 이름만 작성했다.
    - 프로퍼티 이름이 같은 오른쪽 프로퍼티 값을 왼쪽의 프로퍼티 값으로 할당
    - one에 10, two에 20을 할당하여 {one: 10, two: 20} 형태가 된다.
  - 프로퍼티 이름을 별도로 작성
    ```javascript
    let one, two;
    ({one, two} = {one: 10, two: 20});
    console.log(one); // 10
    console.log(two); // 20
    ```
    - let one, two; 프로퍼티 이름을 별도로 작성 
    - ({one, two} = {one: 10, two: 20}); 전체를 소괄호 ()안에 작성
  - 프로퍼티 값 위치에 변수 이름 작성
    ```javascript
    let five, six;
    ({one: five, two: six} = {one: 10, two: 20});
    console.log(five); // 10
    console.log(six); // 20
    ```
    - 이름을 별도로 선언하였으므로 소괄호() 안에 작성
    - 오른쪽 one 프로퍼티 값 10를 five에 할당, 오른쪽 two 프로퍼티 값 20을 six에 할당
    - console.log(one); 을 하면 ReferenceError 프로퍼티 이름으로 값을 구할 수 없다.
    - five와 six 변수값을 구하는 것이 목적이다.
  - Object 구조에 맞추어 값 할당
    ```javascript
    const {one, plus: {two, three}} = {one: 10, plus: {two: 20, three: 30}};
    console.log(one); // 10
    console.log(two); // 20
    console.log(three); // 30
    ```
    - plus: {two: 20, three: 30} plus는 구조(경로)를 만들기 위한 것
    - 왼쪽에 plus가 있고, two가 있으면 two 프로퍼티 값에 20 할당
    - 구조가 같지 않으면 실행할 때 에러 발생
    - console.log(plus); plus는 구조를 만들기 위한 것으로 실제 존재하지 않는다. ReferenceError 발생
    - 할당한 후, 이름으로 값을 구할 수 있다.
  - 나머지를 Object로 할당
    ```javascript
    const {one, ...rest} = {one: 10, two: 20, three: 30};
    console.log(rest); // {two: 20, three: 30}
    ```
    - rest에 나머지 Object를 할당

### Parameter Destructuring Assignment

  - 호출하는 함수에서 Object 형태로 넘겨준 파라미터 값을 호출받는 함수의 파라미터에 맞추어 할당.
    ```javascript
    function add({one, two}) {
        console.log(one + two);
    }    
    add({one: 10, two: 20}); // 30
    ```
    - 호출하는 함수에서 넘겨준 one과 two를 호출받는 함수의 프로퍼티 이름에 맞추어 프로퍼티 값을 분할 할당
  - Object 구조에 맞추어 값 할당
    ```javascript
    function add({one, plus: {two, three}) {
        console.log(one + two + three); 
    }
    add({one: 10, plus: {two: 20, three: 30}}); // 60
    ```
    - 호출하는 함수에서 넘겨준 Object 구조와 프로퍼티에 맞추어 프로퍼티 값을 할당

### Object Oepration

  - 같은 프로퍼티 이름 사용
    ```javascript
    const value = {book: 10, book: 20};
    console.log(value); // {book: 20}
    ```
    - {book: 10, book: 20} 에서 프로퍼티 이름인 book이 같다
    - ES5 strict 모드에서 프로퍼티 이름이 같으면 에러가 난다.
    - ES6 에서는 strict 모드에 관계없이 에러가 발생하지 않으며, 뒤에 프로퍼티 값으로 대체
  - Shortand property names
    ```javascript
    let one = 10, two = 20;
    const values = {one, two};
    console.log(values); //{one: 10, two: 20};
    one = 30;
    console.log(values); //{one: 10, two: 20};
    ```
    - one과 two 변수에 값을 작성하였으며 {one, two} 형태로 values에 할당
    - one이 프로퍼티 값으로 할당된다. 만약 one에 값이 변경된다 하더라도 values에 one이 변경되지 않는다. (깊은 복사)
    - "Shortand property names"는 MDN에 작성된 것으로 스펙에 정의된 것은 아니다.

### Property 이름 조합
  
  - 문자열을 프로퍼티 이름으로 사용
    ```javascript
    const value = {
        ["one"+"two"]: 12
    }
    console.log(value.onetwo); // 12
    ```
    - [] 안에 문자열로 이름을 작성
    - "one"과 "two"를 연결하여 onetwo를 프로퍼티 이름으로 사용
  - 변숫값을 프로퍼티 이름으로 사용
    ```javascript
    const item= "World";
    const sports = {
        [item]: 100,
        [item + " Cup"]: "월드컵",
        [item + " Sports"]: function() {
            return "스포츠";
        }
    };
    console.log(sports[item]); // 100
    console.log(sports[item + " Cup"]); // 월드컵
    console.log(sports[item + " Sports"]()); // 스포츠    
    ```
    - 변숫값을 프로퍼티 이름으로 사용
    - 변숫값과 문자열을 연결할 수 있다.
    - 프로퍼티 이름에 공백이 있는 것이 어색하지만 가능하다.
    - 함수 호출도 가능하다. 변수값에 따라 함수 이름을 정의할 수 있다.
  - 분할 할당을 조합한 형태
    ```javascript
    const item = "book";
    const result = {[item]: title} = {book: "책"};
    console.log(result); // {book: "책"}
    ```
    - 변숫값을 프로퍼티 이름으로 사용하고 분할 할당한 형태
    - { [item] : title} { book: title } 형태가 된다.
    - { book : "책"} {book: title} "cor"이 할당된다.
    
** 출처 1. 인프런 강좌_자바스크립트 ES6+ 기본
