---
layout: post
title: (Javascript ES6) Default Value
summary: 인프런 강좌_자바스크립트 ES6+ 기본
author: devhtak
date: '2020-12-06 22:41:00 +0900'
category: Javascript ES6+
---

### Default Value

- Array, Object 등에서 값 할당
  - 값을 할당하지 않으면 사전에 정의된 값을 할당
  - 할당할 값이 없으면 디폴트 값을 할당
    ```javascript
    const [one, two, five = 50] = [10, 20];
    console.log(five); // 50
    ```
    - one, two에 10, 20을 분할 할당한다.
    - five에 할당할 값이 없으며 이때, five = 50에서 50을 five에 할당
    - 이것을 default value라 한다.
    - = 왼쪽에 이름을 작성하고 오른쪽에 값을 작성
  - 할당할 값이 있으면 디폴트 값을 무시
    ```javascript
    const [one, two, five = 50] = [10, 20, 70];
    console.log(five); // 70
    ```
    - 왼쪽과 오른쪽 모두 값이 3개이다.
    - 값(70)이 있으므로 five에 70을 할당한다. default value를 할당하지 않는다.    
  - Object는 프로퍼티 이름으로 체크
    ```javascript
    const { one, two = 20} = {one: 10};
    console.log(two); // 20
    ```
    - 오른쪽 one의 값인 10을 왼쪽의 one 프로퍼티 값으로 분할 할당
    - two에 할당할 값이 없으며 two = 20에서 20을 two에 할당
  - 디폴트 값 적용 순서
    - 왼쪽에서 오른쪽으로 적용
      ```javascript
      const [one, two = one + 20, five = two + 50] = [10];
      console.log(two); // 30
      console.log(five); // 80
      ```
      - 오른쪽 one의 값인 10을 왼쪽의 one 프로퍼티 값으로 분할 할당
      - 오른쪽에 값이 없으므로 디폴트 값을 할당. 왼쪽에서 오른쪽으로 할당
      - two = one + 20 : one의 값이 10이므로 30이 two에 할당
      - five = two + 50 : two의 값이 30이므로 80이 five에 설정

- 함수의 파라미터에 디폴트 값 적용
  - 넘겨 받은 파라미터 값이 없으면 디폴트 값 할당
    ```javascript
    const add = (ten, two = 20) => ten + two;
    const result = add(10);
    console.log(result); // 30
    ```
    - add(10); // 호출하는 함수의 파라미터 수는 하나
    - (ten, two = 20) ten에 넘겨 받은 값이 10이 설정되고 two에 디폴트 값인 20이 할당
    - 디폴트 값을 작성하지 않으면 two에 undefined가 설정
  - 넘겨받은 파라미터 값이 있으면 디폴트 값을 무시
    ```javascript
    const add = (ten, two = 20) => ten + two;
    const result = add(10, 30);
    console.log(result); // 40
    ```
    - add(10, 30);  두번째 파라미터에 30 작성
    - 호출하는 함수의 파라미터 수와 호출 받는 함수의 파라미터 수가 같으면 디폴트 값이 적용되지 않음
  - 호출함 함수의 파라미터 값이 undefined인 경우
    ```javascript
    const point = () => 20;
    const add = (one, two = point()) => one + two;
    const result = add(10, undefined);
    console.log(result); // 30
    ```
    - add(10, undefined);  // undefined도 값이지만 파라미터 값을 넘겨주지 않은 것과 같다.
    - point() 함수를 호출하고 반환된 값을 디폴트 값으로 사용한다.

** 출처1. 인프런 강좌_자바스크립트 ES6+ 기본
