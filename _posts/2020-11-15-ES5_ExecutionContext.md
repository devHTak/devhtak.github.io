---
layout: post
title: (Javascript ES5) Execution Context
summary: 인프런 강좌_자바스크립트중고급_근본 핵심 이해
author: devhtak
date: '2020-11-15 19:41:00 +0900'
category: javascript
---

### Execution Context

  - 실행 컨텍스트란 자바스크립트 코드가 실행되고 연산되는 범위를 나타내는 추상적인 개념으로 코드가 실행된다면 실행 컨텍스트 내부에서 실행되고 있는 것이다.
  - 자바스크립트 엔진은 코드를 실행하기 위해서 여러가지 정보를 알아야 한다.
    - 변수(전역, 지역, 매개 변수, 객체의 프로퍼티), 함수 선언, 변수의 유효범위, this
  - 실행 컨텍스트의 타입
    - Global Execution Context: window 오브젝트인 전역 컨텍스트를 생성하고, this를 global object로 할당한다.
    - Functional Execution Context : 함수가 호출될때마다 실행 컨텍스트가 생성된다.
    - Eval Functional Execution Context: eval 함수 또한 자신 만의 실행 컨텍스트를 가진다.
  - Execution Stack(호출 스택)과 함수 실행 순서
    - 자바스크립트 엔진이 script tag를 처음 만나면 전역 컨텍스트를 만들고 현재 실행되고 있는 호출 스택에 이를 push한다.
  - Execution Context 형태
    - 환경 (Creation Pashe)
      - LEC(Lexical Environment Component): Identifier-variable mapping 되는 곳으로 아래 3가지 일을 진행한다.
        - Environment Records: 함수와 변수를 기록한다. (ER)
          - DER(Declarative Environment Record) : 변수와 함수 선언을 저장하는 곳이다.
          - OER(Object Environment Record): 전역 코드에 대한 LE는 OER에 포함된다. 
        - Reference to the outer environment: 외부 lexical 환경으로 접근할 수있다는 의미이다. (OER)
      - VEC(Variable Environment Component): LEC와 function, 변수 식별자가 binding 되는 점을 포함해 동일하다.
      - TBC(This Environment Component): Thid binding, this의 값이 여기서 결정된다.
    ```javascript
    /*
    show 실행 콘텍스트(EC): {
        렉시컬 환경 컴포넌트(LEC): {
            환경 레코드(ER): {
                선언적 환경 레코드(DER):{
                    title: 'Javascript book'
                },
                오브젝트 환경 레코드(OER): {}
            },
            외부 렉시컬 환경 참조(OLER):{
                point: 123,
                getPoint: function(){}
            }
        },
        변수 환경 컴포넌트(VEC): {}
        this 바인딩 컴포넌트(TBC):{
            글로벌 오브젝트(window)
        }
    }
    */
    function book() {
        var point = 123;
        function show() {
            var title ="Javascript book";
            // getPoint();
            // this.bookAmount;
        }
        function getPoint() {return point;}

        show();
    };

    book();
    ```
    - book() 함수가 호출되면
      - show function 오브젝트 생성
      - show의 [[Scope]] 에 스코프 설정 (대괄호 2개는 엔진이 설정하는 프로퍼티를 뜻한다.)
      - 스코프는 book() 함수 내로 설정한다.
    - show() 함수가 호출되면 EC 생성
      - 엔진 컨트롤이 show()함수로 이동하기 전에 함수 실행을 위한 Context 환경 구축
      - LEC, VEC, TBC 생성 첨부
      - LEC에 ER, OLER 첨부
      - ER에 DER, OER 첨부
    - DER에 show()의 변수, 함수 기록
    - OLER에 show()의 [[Scope]] 를 설정
    - this 바인딩 컴포넌트에 this 참조 설정
  
- 식별자 해결 (Identifier Resolution)
  - 식별자 해결
    - 사용할 변수/함수를 결정하는 것 예) point 변수
    - 신속, 정확한 검색을 위해 스코프 필요
  - 스코프에서 이름을 찾기 위해
    - 스코프 이름을 설정
    - 값은 변경되지만 이름은 변경되지 않음
    - 식별자 해결 대상은 이름
  - resolutation 의 사전적 의미: 해결, 결정
    - 결정도 시맨틱적으로 맞음
    - 시멘틱: 코드 조각의 의미를 나타낸다.
  ```javascript
  var point = 100;
  function getPoint() {
    var point = 200;
    return point;
  }
  var result = getPoint(); // scope이 getPoint 내부에 있기 때문에 point/200값을 return 한다.
  console.log(result); // 200
  ```
- 스코프 용도
  - 식별자 해결을 위한 수단, 방법
    - 스코프가 목적이 아니다.
  - 식별자가 유일하면 스코프는 필요하지 않다. 하지만 유일하게 작성하는 것은 불가능하기 때문에 스코프가 필요하다.
- Scope Chain(ES3)
  - 실행 콘텍스트와 관련이 있으며 식별자 해결을 위해 사용
  - 식별자를 검색하기 위한 {name: value} 리스트
    - 함수가 호출되면 scope를 생성하고 함수의 변수와 함수를 {name: value} 형태로 설정
    - 생성한 scope를 scope chain에 연결하고 scope chain에서 식별자를 해결
    - 동적 처리
    - ES3의 실행 콘텍스트 환경 (scope chain, Artivation Object)
  - ES5에는 LEC의 DERd에 함수의 변수와 함수 이름을 바인드. ES5부터 Scope chain을 사용하지 않으며 DER에서 변수와 함수이름을 검색하여 실행
  - ES3에서는 함수가 호출될 때마다 scope chain을 생성하여 동적으로 처리하는 데, ES5는 정적으로 미리 설정하기 때문에 엔진 처리가 ES5가 빠르다.
- Lexical Environment
  - 정적 환경
    - function 키워드를 만나면 function 오브젝트를 생성하고 스코프를 FO의 [[Scope]에 설정한다.
      - 이것은 함수 밖의 스코프가 설정되는 것. 아직 함수 내부를 들어가지 않았기 때문에 함수 밖의 스코프가 결정된다.
      - 그래서 동적이 아닌 정적이다.
    - 이 시점에서 스코프가 결정된다. -> 이것이 Lexical Environment (정적 환경) (함수가 호출될 때 생성되는 것이 아니다!!)
    - 함수가 호출되면 FO의 [[Scope]]를 실행 콘텍스트의 렉시컬 환경 컴포넌트의 외부 렉시컬 환경 참조에 설정
  - var 키워드 문제
    - 함수에서 var 키워드를 사용하지 않고 변수를 선언하면 글로벌 오브젝트에 설정된다. 이는 렉시컬 환경 구조에 맞지 않다.
    - ES5에 해결 방법으로 use strict 사용한다.
      - Strict mode: ES5 버전에 새로운 기능으로 엄격한 운용 콘텍스트 안에서 실행하게 끔 만들어 준다.
        - 장점 1. 흔히 발생하는 코딩 실수를 잡아내어 예외를 발생시킨다.
        - 장점 2. 상대적으로 안전하지 않은 액션이 발생하는 것을 방지하거나 그럴 때 예외를 발생시킨다.
        - 장점 3. 혼란스럽거나 제대로 고려되지 않은 기능들을 비활성화 시킨다.
    - ES6 해결 방법으로는
      - let 변수, const 변수를 사용한다. 변수 자체에 스코프 제약을 둔다.
      - ES6는 기본으로 Strict mode를 사용한다.
  - 동적 환경
    - 실행 시점에 스코프 결정
      - with 문 : strict 모드에서 에러 발생
      - eval() 함수: 동적변수를 선언할때 eval 함수를 사용한다. 보안에 문제가 있다.
 
** 출처1. 인프런 강좌_자바스크립트 중고급: 근본 핵심 이해

** 출처2. https://velog.io/@imacoolgirlyo/JS-%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8%EC%9D%98-Hoisting-The-Execution-Context-%ED%98%B8%EC%9D%B4%EC%8A%A4%ED%8C%85-%EC%8B%A4%ED%96%89-%EC%BB%A8%ED%85%8D%EC%8A%A4%ED%8A%B8-6bjsmmlmgy
