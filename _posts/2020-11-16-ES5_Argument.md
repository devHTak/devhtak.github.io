---
layout: post
title: (Javascript ES5) Argument
summary: 인프런 강좌_자바스크립트중고급_근본 핵심 이해
author: devhtak
date: '2020-11-16 22:41:00 +0900'
category: javascript
---

### Argument

- Argument 처리 구조
  - 파라미터를 {key: value} 형태로 저장
    ```javascript
    function get() {
        return arguments;
    }
    console.log(get("A", "B")); // {0: A, 1: B}
    ```
    - 파라미터 수만큼 0부터 인덱스 부여하여 key로 사용
    - 파라미터로 받은 값을 value에 설정
    - {0: param1, 1: param2}
  - Array-like
    - {0: param1, 1: param2}와 같은 형태를 Array-like 라고 부른다.
    - key 값이 0부터 1씩 증가
    - length 프로퍼티가 있어야 함 -> length라고 하는 것은 반복문을 사용할 수 있다.
    
- 엔진의 파라미터 처리
  ```javascript
  var get = function(one) {
      return one;
  }
  get(77, 100);
  ```
  - get() 함수를 호출하면서 77, 100을 파라미터 값으로 넘겨 줍니다.
  - 넘겨받은 값을 함수의 파라미터 이름에 설정합니다.
    - 정적 환경의 선언적 환경 레코드에 설정 (one: 77 형태로 지정)
  - Argument Object를 생성합니다. 파라미터로 받은 값을 Array-like 형태로 만든다.
  - 넘겨받은 파라미터 수를 Argument object의 legnth 프로퍼티에 설정
  - 넘겨받은 파라미터 수만큼 반복하면서 0부터 key로 하여 순서대로 파라미터 값을 설정 {0: 77}, {1: 100} 형태가 된다.
  - 함수의 초기화 단계에서 실행한다.
  
  
