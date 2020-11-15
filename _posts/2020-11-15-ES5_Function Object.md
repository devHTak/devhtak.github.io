---
layout: post
title: Javascript ES5. Function Object
summary: 자바스크립트 중고급: 근본 핵심 이해 from 인프런 강좌
author: devhtak
date: '2020-11-15 13:41:00 +0900'
category: javascript
---

### Function Object

- Function Object 형태
  - Build-in function 오브젝트
    - Function.prototype.call()
  - function object
    - function book(..) {..}
    - var book = function(...) {...}
    - 인스턴스지만, new 연산자로 생성한 인스턴스와 구분하기 위해 강좌에서는 function 오브젝트로 표기
  - function 인스턴스
    - new Book() 처럼 new 연산자를 사용하여 Book.prototype에 연결된 메소드로 사용
- Function Object 생성
  - var book = function(...) {...}
  - 엔진이 function이란 키워드를 만나면 빌트인 Function Object의 prototype에 연결된 메소드(Function.prototype.call())로 function object 생성
  - 생성한 오브젝트를 book 변수에 할당
  - book() 형태로 호출 - function object이므로 호출할 수 있음
- Function Object 저장
  - 함수를 호출하려면 생성한 function object를 저장해야 한다.
  - function object 저장 형태로는
    - {name: value} 형태로 저장 - {book: 생성한 object} 형태
  - 함수를 호출하면
    - 저장된 object에서 함수 이름(book)으로 검색
    - value 값을 구하고
    - value가 function object이면 호출
- 함수가 호출되면 엔진은 {name: value} 형태로 실행환경을 설정하고 함수 코드를 실행한다. {name: value} 형태로 생각을 전환해야 JS의 아키텍처와 메커니즘을 쉽게 이해할 수 있다. 즉, function(){} 코드를 보면 함수의 변수와 함수가 {name: value} 형태로 연상되어야 한다.

- function object 생성 과정
  - function sports(..) {...} 형태에서 function 키워드를 만나면
  - 오브젝트를 생성하고 저장
    - {sports: {...}}
    - sports는 function 오브젝트의 이름이 되고 오브젝트에 프로퍼티가 없는 상태로 저장된다. (빈 오브젝트)
      - 이제부터 빈 오브젝트를 채운다.
  - sports 오브젝트에 prototype오브젝트 첨부
  - prototype에 constructor 프로퍼티 첨부
    - property.constructor가 sports 오브젝트 참조
  - prototype에 __proto__ 오브젝트 첨부
    - ES5 스펙에 __proto__가 기술되어 있지 않으며 ES6스펙에 기술
    - 엔진이 사용한다는 뉘앙스로 정의
  - 빌트인 Object.prototype의 메소드로 Object 인스턴스를 생성하여 prototype.__proto__에 첨부
  - sports Object에 __proto__ 오브젝트 첨부. sports.__proto__ 구조가 된다.
  - 빌트인 Function.prototype의 메서드로 function 인스턴스를 생성하여 sports.__proto__에 첨부
  - sports object property에 초기값 설정
    - arguments, caller, length, name 프로퍼티
  ```
  sports = {
    arguments: {},
    caller: {},
    length: 0,
    name: "sports",
    prototype: {
      constructor: sports,
      __proto__: Object.prototype
    },
    __proto__: Function.prototype
  }
  ```  
- Function Object 구조
  - function 오브젝트에 prototype이 있으며 
    - constructor가 연결된다.
    - __proto__가 연결된다.
    - Object 인스턴스가 연결된다.
  - function 오브젝트에 __proto__가 있으며
    - Function 인스턴스가 연결된다.
    - Array면 Array 인스턴스가 연결되고, String이면 String 인스턴스가 연결된다.
- Function 실행 환경
  - 함수 실행 환경 인식이 필요한 이유?
    - 함수가 호출되었을 때 실행될 환경을 알아야 실행 환경에 맞추어 실행할 수 있기 때문이다.
  - 실행환경 설정 시점
    - function 키워드를 만나 function object 를 생성할 때 (정적 환경 - Lexical environment)
  - 설정하는 것
    - 실행 영역(함수가 속한 scope)
    - 파라미터, 함수 코드 등
- 함수 실행 환경 저장
  - function 오브젝트를 생성하고 바로 실행하지 않으므로 함수가 호출되었을 때 사용할 수 있도록 환경을 저장한다.
  - 어디에 저장? 생성한 function object에 저장한다.
  - 인식한 환경을 function object의 내부 프로퍼티에 {name : value} 형태로 설정한다. 
  
- 내부 프로퍼티란?
  - 엔진이 내부 처리에 사용하는 프로퍼티로 스펙 표기로 외부에서 사용 불가한다.
  - [[...]] 형태. ex) [[Scope]]
  - 내부 프로퍼티 분류
    - 공통 프로퍼티
      - 모든 오브젝트에 공통으로 설정되는 프로퍼티
    - 선택적 프로퍼티
      - 오브젝트에 따라 선택적으로 설정되는 프로퍼티
      - 해당되는 오브젝트에만 설정
  - 공통 내부 프로퍼티
  |프로퍼티 이름|값 형태|개요|
  |------|---|---|
  | [[Prototype]] | Object 또는 Null | 오브젝트의 prototype |
  | [[Class]] | String | 오브젝트 유형 구분 |
  | [[Extensible]] | Boolean | 오브젝트에 프로퍼티 추가 가능 여부 |
  | [[Get]] | any | 이름의 프로퍼티 값 (getter) |
  | [[GetOwnProperty]] | 프로퍼티 디스크립터 | 오브젝트 소유의 프로퍼티 디스크립터 속성 |
  | [[GetProperty]] | 프로프터 디스크립터 | 오브젝트의 프로퍼티 디스크립터 속성 |
  | [[Put]] | - | 프로퍼티 이름으로 프로퍼티 값 설정 |
  | [[CanPut]] | Boolean | 값(value) 설정 가능 여부 |
  | [[HasProperty]] | Boolean | 프로퍼티의 존재 여부 |
  | [[Delete]] | Boolean | 오브젝트에서 프로퍼티 삭제 가능 여부|
  | [[DefaultValue]] | any | 오브젝트의 디폴트 값 |
  | [[DefinedOwnProperty]] | Boolean | 프로퍼티 추가, 프로퍼티 값 변경 가능 여부 |
  
  - 선택적 내부 프로퍼티: 오브젝트에 따라 선택적 설정
  |프로퍼티 이름|값 형태|개요|
  |------|---|---|
  | [[PrimitiveValue]] | 프리미티브 값 | Boolean, Date, Number, String 오브젝트에서 제공 |
  | [[Construct]] | Object | new 연산자로 호출되며 인스턴스를 생성 |
  | [[Call]] | any | 함수 호출 |
  | [[HasInstance]] | Boolean | 지정한 오브젝트로 생성한 인스턴스 여부 |
  | [[Scope]] | 렉시컬 환경 | Function 오브젝트가 실행되는 렉시컬(정적) 환경 |
  | [[FormalParameters]] | 문자열 리스트 | 호출된 함수의 파라미터 이름 리스트 |
  | [[Code]] | JS Code | 함수에 작성한 JS 코드 설정, 함수가 호출되었을 때 실행 | 
  | [[TargetFunction]] | Object | Function 오브젝트의 bind()에 생성한 타깃 함수 오브젝트 설정 |
  | [[BoundThis] | any | bind()에 바인딩된 this object |
  | [[BoundArguments]] | list | bind()에 바인딩된 아규먼트 리스트 |
  | [[Match]] | 매치 결과 | 정규 표현식의 매치 결과 |
  | [[ParameterMap]] | Object | 아규먼트 오브젝트와 함수의 파라미터 매핑 |
  
- 함수 정의 형태
  - 함수 정의
    - 함수 코드가 실행될 수 있도록 JS 문법에 맞게 함수를 작성하는 것
  - 함수 정의 형태
    - 함수 선언문
       |구분|타입|데이터(값)|
       |------|---|---|
       |function|Function|function 키워드|
       |식별자|String|함수이름|
       |파라미터|Any|파라미터 리스트 opt|
       |함수 블록|Object|실행 가능 코드 opt|
       |반환| Funcion|생성한 Function Object|
       - 엔진이 function 키워드를 만나면 function object를 생성하고, 함수 이름을 functino object 이름으로 사용
       ```javascript
       function book(value1, value2) {
        return value1 + ", " + value2;
       }
       console.log(book("JS", "DOM")); // JS, DOM 출려
       ```       
    - 함수 표현식
       |구분|타입|데이터(값)|
       |------|---|---|
       |function|Function|function 키워드|
       |식별자|String|함수이름|
       |파라미터|Any|파라미터 리스트 opt|
       |함수 블록|Object|{실행 가능 코드 opt}|
       |반환| Funcion|생성한 Function Object|
       
       ```javascript
       var getBook = function(title) {
        return title;
      }
      console.log(getBook("JS Book"); // JS Book 출력
       ```
    - new Function(param, body) 문자열로 작성
  - 엔진 해석 방법
    - 엔진 해석 순서
      - 자바스크립트는 스크립팅 언어
        - 스크립팅 언어는 작성된 코들르 위에서부터 한줄씩 해석(환경설정)하고 실행
        - 하지만, 자바스크립트는 다르다.
      - 중간에 있는 코드가 먼저 해석될 수도 있다.
      - 기준1. 함수 선언문을 순서대로 해석한다. 작성한 순서대로 해석한다.
        - function sports(){};
      - 기준 2. 표현식을 순서대로 해석한다. 작성한 순서대로 해석한다.
        - var value = 123;
        - var book = function() {};
    ```javascript
    <script type="application/javascript">
      function book() {
        console.log(title); // undefined
        console.log(readBook); // undefined
        console.log(getBook); // function object
        debugger;
        var title = "JS Book";
        function getBook() {
          return title;
        };
        var readBook = function(){};
        getBook();
      }
      book();
    </script>
    ```
      - 엔진 처리 상태
        - 마지막줄에 book() 함수를 호출하면 엔진이 book 함수를 바라본다. debugger에서 실행이 멈춘다.
        - title, readBook은 undefined, getBook은 function objec
        - getBook이 function object라는 것은 function getBook(){}을 해석했기 때문이다.
        - title, readBook에 설정된 undefined도 값이며 값이 있다는 것은 엔진이 해석하는 것을 뜻합니다. 해석하지 않았다면 title, readBook 값이 표현되지 않습니다. 하지만 아직 value가 세팅되지 않았다는 것이다.
      - 함수 코드 해석 순서
        - 함수 선언문 해석: function getBook(){};
        - 변수초기화: var title = undefined; var readBook = undefined;
        - 코드 실행: var title = "JS Book"; var readBook = function() {}; getBook();
    
    
    
    
    
    
    
    
    
  


