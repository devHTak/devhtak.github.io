---
layout: post
title: (Javascript ES6) Number Object
summary: 인프런 강좌_자바스크립트 ES6+ 기본
author: devhtak
date: '2020-12-07 22:41:00 +0900'
category: Javascript ES6+
---

### IEEE 754

- 자바스크립트는 IEE 754에 정의된 64비트 부동 소수점으로 수를 처리한다.
  - double-precision floating-point format numbers
  - 64비트로 최솟값과 최댓값을 처리
- 정수와 실수를 구분하지 않는다.
  - 1을 1.0으로 처리
  - 1과 1.2를 더할 수 있다.
  
### 64 비트 구성

- sign 비트 (1비트)
  - 63: 1비트
  - 값이 0이면 양수, 1이면 음수
- 지수(exponent) (11비트)
  - 52 ~ 62 : 11비트
- 가수(fraction) (52비트)
  - 0~51: 52비트 + 1(사인 비트): 53비트

- 값을 구하는 방법
  - 비트 값은 0 아니면 1
  - 2^X 승 값을 더해 값을 구하면
    - ex) 0비트부터 1, 1, 1이면, 1 * (2^0) + 1 * (2^1) + 1 * (2^2) = 7이 된다.
    
### Number 상수

- safe integer란
  - 지수(e)를 사용하지 않고 나타낼 수 있는 값(가수 부분 활용)
  - 2의 64 승이 아닌 2의 53승(fraction)
- Number.MAX_SAFE_INTEGER
  - 2^53 - 1 (9007199254740991)
- Number.MIN_SAFE_INTEGER
  - -(2^53) + 1 (-9007199254740991)

### Number.EPSILON

- Number.EPSILON
  - 아주 작은 값
  - 2.2204460492503130808472633361816E-16
  - 또는 2^-52

- 사용 사례
  - 미세한 값 차이 형태
    ```javascript
    const total = 0.1 + 0.2;
    console.log(total == 0.3); //false
    console.log(total); // 0.30000000000000004
    ```
    - 이처럼 소수점을 활용할 때 발생하는 미세한 값의 차이를 처리할 때 EPSILON을 사용한다.
  - 미세한 값 차이를 같은 값으로 간주
    ```javascript
    const value = Math.abs(0.1 + 0.2 - 0.3);
    console.log(value < Number.EPSILON); // true
    ```
    - 값 차이가 Number.EPSILON 보다 작으면 true를 반환
  - 0/0으로 NaN가 되는 것을 방지
    ```javascript
    console.log(0/0); // NaN
    const value = 0 / (0 + Number.EPSILON);
    console.log(value); // 0
    ```
    - 아주 작은 값(Number.EPSILON)을 더해 나누면 0이 된다. 0이므로 후속처리가 가능하다.
    
### 진수

- Binary (2진수)
  - 0b0101, 0B0101 형태로 작성
  - 숫자 0 다음에 b/B 작성하고 이어서 0 또는 1로 값을 작성
    ```javascript
    cosnt value = 0B111;
    console.log(value); // 7 (1+2+4)
    ```
- Octal (8진수)
  - 0O0105 형태로 작성
  - 숫자 0 다음에 영문 o/O 작성하고 이어서 0~7로 값을 작성
  - ES3는 첫자리에 영문 o/O 작성
    ```javascript
    const value = 0O111;
    console.log(value); // 73 (1+8+64)
    ```
 
 ### Number 함수
 
 - isNaN()
  - 형태: Number.isNaN(), 파라미터: 비교 대상, 반환: NaN이면 true, 아니면 false
  - NaN 값의 여부를 체크
    - NaN이면 true, 아니면 false
      ```javascript
      const value = (0/0);
      console.log(Number.isNaN(value), isNaN(value)); // true true
      console.log(Number.isNaN("ABC"), isNaN("ABC")); // false, true
      ```
  - NaN 체크 방법
    - NaN === NaN
      - 결과가 false이므로 사용 불가
    - isNaN(), 글로벌 오브젝트
      - is Not a Number에 약자로, 숫자 객체가 아닌지를 반환한다.
      - String 이더라도, 숫자로 변환한 다음 그 결과를 반환한다.
        ```javascript
        console.log(Number.isNaN("200"), isNaN("200")); // false, false
        ```
    - Number.isNaN()
    - Object.is(NaN, NaN) : true

- isInteger()
  - 형태: Number.isInteger(), 파라미터: 비교 대상, 반환: 정수이면 true, 아니면 false
  - 파라미터 값이 정수면 true, 아니면 false 반환
  - 정수로 인식
    ```javascript
    console.log(Number.isInteger(0)); // true
    console.log(Number.isInteger(1.0)); // true
    console.log(Number.isInteger(1.01)); // false
    ```
    - 1.0 또한 정수로 판단한다.
  - 정수가 아닌 것으로 인식
    ```javascript
    console.log(Number.isInteger("12")); // false
    console.log(Number.isInteger(true)); // false
    ```
    - Number.isInteger는 액면 그대로(변수에 타입까지) 구분하여 확인한다.

- isSafeInteger()
  - 형태: Number.isSafeInteger(), 파라미터: 비교 대상, 반환: safe 정수이면 true, 아니면 false
  - 파라미터 값이 safe 정수이면 true, 아니면 false
  - -(2^53 - 1) ~ (2^53 - 1): true / 아니면 false
  - true로 인식
    ```javascript
    console.log(Number.isSafeInteger(7.0)); // true
    console.log(Number.isSafeInteger(Number.MAX_SAFE_INTEGER)); // true
    console.log(Number.isSafeInteger(Number.MIN_SAFE_INTEGER)); // true
    ```
  - false로 인식
    ```javascript
    console.log(Number.isSafeInteger(7.1)); // false
    console.log(Number.isSafeInteger("100")); // false - 타입또한 검사한다.
    console.log(Number.isSafeInteger(NaN)); // false
    console.log(Number.isSafeInteger(Infinity)); // false
    ```

- isFinite()
  - 형태: Number.isFinite(), 파라미터: 비교 대상, 반환: 유환 값이면 true, 아니면 false를 반환
  - 파라미터 값이 유한 값이면 true 아니면 false 반환
  - 글로벌 오브젝트의 isFinite()와 체크 결과가 다르다.
    ```javascript
    console.log(Number.isFinite(100), isFinite(200)); // true, true
    console.log(Number.isFinite("70"), isFinite("80")); // false, true - Global은 타입을 신경쓰지 않는다.
    console.log(Number.isFinite(true), isFinite(true)); // false, true - Global은 타입을 신경쓰지 않아 true/false를 1/0으로 구분한다.
    
    console.log(Number.isFinite(NaN), isFinite(NaN)); // false, false
    console.log(Number.isFinite(undefined), isFinite(undefined)); // false, false
    ```
    - 함수는 오브젝트에 속해야 하므로 Number와 관련된 것은 Number 오브젝트의 함수 사용
    - 글로벌 오브젝트의 함수는 글로벌 사용이 목적 - 파라미터를 Number로 변환하여 체크한다.
    
** 출처1. 인프런 강좌_자바스크립트 ES6+
