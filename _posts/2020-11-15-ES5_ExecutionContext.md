---
layout: post
title: (Javascript ES5) Execution Context
summary: 인프런 강좌_자바스크립트중고급_근본 핵심 이해
author: devhtak
date: '2020-11-15 19:41:00 +0900'
category: javascript
---

### 들어가기

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
    - LEC(Lexical Environment Component): Identifier-variable mapping 되는 곳으로 아래 3가지 일을 진행한다.
      - Environment Records: 함수와 변수를 기록한다. (ER)
        - DER(Declarative Environment Record) : 변수와 함수 선언을 저장하는 곳이다.
        - OER(Object Environment Record): 전역 코드에 대한 LE는 OER에 포함된다. 
      - Reference to the outer environment: 외부 lexical 환경으로 접근할 수있다는 의미이다. (OLER)
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
      - show() 함수가 호출되면 functional EC 생성
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
  - ES5에는 LEC의 DER에 함수의 변수와 함수 이름을 바인드. ES5부터 Scope chain을 사용하지 않으며 DER에서 변수와 함수이름을 검색하여 실행
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

### Execution Context
  
  - Execution Context
    - 함수가 실행되는 영역, 묶음
    - 함수 코드를 실행하고 실행 결과를 저장
    - 스펙 상의 사양 (엔진 처리 과정)
    - 실행 콘텍스트 스펙
      ```javascript
      function music(title) {
          var musicTitle = title;
      }
      music("음악");
      ```
      - music("음악"); 으로 함수가 호출하면 엔진은 실행 콘텍스트를 생성하고 실행 컨텍스트 안으로 이동합니다.
      - 실행 컨텍스트 실행 단계
        - 준비 단계, 초기화 단계, 코드 실행 단계
      - Execution Context 생성 시점: 실행 가능한 코드를 만났을 때
      - 실행 가능한 코드 유형: 함수 코드, 글로벌 코드, eval 코드
      - 코드 유형을 분리한 이유
        - Execution Context에서 처리 방법과 실행 환경이 다르기 때문이다.
        - 함수 코드 -> Lexical 환경, 글로벌 코드 -> Global 환경 , eval 코드 -> 동적 환경
- Execution Context 상태 컴포넌트
  - Execution Context 상태를 위한 오브젝트 -> Execution Context 안에 생성
  - 상태 컴포넌트 유형
    - Lexical Execution Component(LEC)
    - Variable Execution Component(VEC)
    - This Binding Component(TBC)
      ```
      Execution Context(EC): {
          Lexical Execution Component(LEC): {
              Environment Record(ER): { point: 100 },
              Object Lexical Environment Reference(OLER): {
                  title: "book", getTitle: function() {}
              }
          }
          Variable Execution Component(VEC): {}
          This Binding Component(TBC): {}
      }
      ```
      
  - Lexical Execution Component
    - 함수와 변수의 식별자 해결을 위한 환경 설정
    - 함수 초기화 단계에서 해석한 함수와 변수를 {name: value} 형태로 저장, 이름으로 함수와 변수를 검색하게 된다.
      - 초기화 단계에서 함수 선언문은 {name: function object} 형태 / 변수, 함수 표현식은 {name: undefined} 형태  
    - 함수 밖의 함수와 변수 참조 환경을 설정한다. 함수 밖의 함수와 변수를 사용할 수 있게끔 한다.
    - function, with, try-catch를 만났을 때 Lexical Execution Component 생성 
    - 컴포넌트 구성
      - Environment Record
        - Environment Record에 함수안의 함수와 변수를 기록
        - Object Lexical Environment Reference에 function object의 \[\[Scope]]를 설정
        - 따라서 함수 안과 밖의 함수와 변수를 사용할 수 있게 됨        
      - Object Lexical Environment Reference
        - Scope와 실행중인 함수가 Context 형태이므로 스코프의 변수와 함수를 별도의 처리 없이 즉시 사용할 수 있다.
        - Execution Context에서 함수 안과 밖의 함수, 변수를 사용할 수 있으므로 함수와 변수를 찾기 위해 Execution Context를 벗어 나지 않아도 된다.
  - Variable Execution Context
    - Execution Context 초기화 단계에서 LEC와 함께 설정, 초기값 복원할 때 사용하기 위해 함께 설정된다. 
    - 코드 실행되며 변경되는 값들은 LEC에 저장되고 초기값은 VEC에 저장된다. VEC에 있는 초기값은 초기화할 때 사용되며 with 문에서 사용된다.
    - 함수 코드가 실행되면 실행 결과를 LEC에 설정하며 초기값이 변하게 되므로 이를 유지하기 위한 것이 된다.

- Execution Context 실행 과정
  
```javascript
var base = 200;
function getPoint(bonus) {
    var point = 100;
    return point + base + bonus;
}
console.log(getPoint(50)); // 350 출력
```
  - getPoint function object 생성
    - 오브젝트의 \[\[Scope]]에 Global Object 설정
  - base 선언
  - base에 200 할당 한 후, getPoint()함수 호출
  - 엔진은 EC를 생성하고, EC 안으로 이동합니다.
    - 준비단계 1. Component를 생성하여 EC에 첨부. LEC, VEC, TBC
    - 준비단계 2. ER을 생성하여 LEC에 첨부 - 함수 안의 함수, 변수를 바인딩한다.
    - 준비단계 3. OLER을 생성하여 LEC에 첨부하고, function object의 [[Scope]]를 설정
      ```
      EC: {
        LEC= { ER: {}, OLER: { base: 200 } }, VEC: { }, TBC: {}
      }
      ```
      
    - 초기화 단계 1. 호출한 함수의 파라미터 값을 호출된 함수의 파라미터 이름에 매핑. 환경 레코드에 작성 (변수 선언 전 파라미터가 먼저 설정된다.)
    - 초기화 단계 2. 함수 선언문을 function object로 생성
    - 초기화 단계 3. 함수 표현식과 변수에 초기값 설정. 현재까지는 외부에 실행 상태를 제공하지 않는다.
      ``` 
      EC: {
        LEC= { ER: {bonus: 70, point: undefined}, OLER: { base: 200 } }, VEC: { }, TBC: {}
      }    
      ```
      
    - 실행 단계 1. 함수 안의 코드를 실행합니다. var point = 100;
    - 실행 단계 2. EC 안에서 관련된 함수와 변수를 사용할 수 있습니다.
  - 해당 과정은 메모리에서 진행한다.

- Environment Record(환경 레코드)
  - 기록 대상에 따라 다르기 때문에 Environment Record를 구분한다.
  - Declarative Environment Record(선언적 환경 레코드)
    - function, 변수, catch 문에서 사용한다.
    - 앞 절에서 환경 레코드에 설정한다고 했는데 실제로 DER에 설정된다.
  - Object Environment Record(오브젝트 환경 레코드)
    - 글로벌 함수와 변수, with 문에서 사용
    - 정적이 아니라 동적이기 때문
- Global Environment
```
EC:{
    GE: {
        ER: { OER: Global Object },
        OLER: null
    }
}
```
  - Global Object에서 사용
  - Lexical Environment Component와 형태가 같음
  - 동적으로 함수와 변수 바인딩
    - 함수에서 var 키워드를 사용하지 않고 변수를 선언하면 글로벌 오브젝트에 설정되기 때문
    - 이런 이유로 오브젝트 환경 레코드 사용
  - 외부 렉시컬 환경 참조 값은 null

- This Binding Component
  - 목적
    - this로, 함수를 호출한 오브젝트의 프로퍼티에 액세스한다.
    - 예) this.propertyName
  - Access Mechanism
    - obj.book() 형태에서 this로 obj를 참조할 수 있도록 This Binding Component에 obj 참조를 설정
    - obj의 프로퍼티가 변경되면 동적으로 참조 (설정이 아닌 참조)
  - 실행 과정
    ```javascript
    var obj = {point: 100};
    obj.getPoint = function() {
        return this.point;
    }
    console.log(obj.getPoint()); // 100
    ```
      - 마지막 줄에서 getPoint() 함수 호출
      - Execution Context 생성
      - 3개의 Component 생성( LEC, VEC, TBC )
      - This Binding Component에 getPoint()에서 this로 obj의 프로퍼티를 사용할 수 있도록 바인딩
        - 초기화 단계. 파라미터, 함수 선언문, 변수 선언이 없다.
        - 실행 단계 1. return this.point 실행
        - 실행 단계 2. TBC에서 point 검색 - getPoint() 함수를 호출한 오브젝트가 TBC에 참조된 상태
        - 실행 단계 3. TBC에 point 프로퍼티가 있으므로 100을 반환
        - 추가 설명: obj.getPoint()에서 obj의 프로퍼티가 TBC에 바인딩되도록 의도적으로 설계해야 한다.
        
    ```
    EC: {
        LEC= { ER: { DER: {}, OER: {}}, OLER: {},
        VEC: {},
        TBC: {
          point: 100, getPoint: function(){}
        }
    }
    ```
    
- 호출 스택 (Call Stack)
  - Execution Context의 논리적 구조
  - FILO 순서
    - 함수가 호출되면 스택의 가장 위에 EC가 위치하게 된다.
    - 다시 함수 안에서 함수를 호출하면 호출된 함수의 EC가 스택의 가장 위에 놓이게 된다.
    - 함수가 종료되면 스택에서 빠져 나옴(FILO)
  - 가장 아래는 Global Object의 함수가 위치한다.
    ```javascript
    function one() { two(); console.log(1); }
    function two() { three(); console.log(2); }
    function three() { console.log(3); }
    one();
    ```
      - 3 2 1 순으로 출력된다.

- 파라미터 매핑
  - 함수 호출
    - 함수가 호출되면 3개의 파라미터 값을 EC로 넘겨 줍니다.
      - 함수를 호출한 오브젝트
      - 함수 코드
      - 호출한 함수의 파라미터 값
    - 함수를 호출한 오브젝트를 TBC에 설정하여 this로 참조
    - 함수 코드
      - function object의 \[\[Code]]에 설정되어 있음
    - 호출한 함수의 파라미터 값
      - 호출된 함수의 Argument object에 설정
  - 파라미터 값 매핑
    - 호출한 함수에서 넘겨 준 파라미터 값을 호출된 함수의 파라미터 작성 순서에 맞추어 값을 매핑하는 것
  - 엔진 처리 관점
    - EC로 넘겨준 파라미터 값과 function object의 [[FormalParameters]]에 작성된 이름에 값을 매핑하고 결과를 DEC에 설정하는 것
- 파라미터 이름에 값 매핑 방법
  ```javascript
  var obj = {};
  obj.getTotal = function(one, two) {
      return one + two;
  }
  console.log(obj.getTotal(11, 22, 77));
  ```
    - getTotal 오브젝트의 \[\[FormalParameters]]에서 호출된 함수의 파라미터 이름을 구합니다. 파라미터 이름은 \["one", "two"] 형태로 \[\[FormalParameters]]는 function object를 생성할 때 설정한다.
    - 파라미터 이름 배열을 하나씩 읽는다.
    - 파라미터에서 index 번째의 값을 구한다. 인덱스에 값이 없으면 undefined 반환 (오류가 발생하는 것은 아니다.)
    - 파라미터 이름의 이름과 값을 DER에 {one: 11, two: 22} 형태로 설정한다. 같은 이름이 있으면 값이 대체된다.
    - 파라미터 이름을 전부 읽을 때까지 배열을 읽고, 값을 구하는 작업을 반복한다.
    - 여기서 77(값)은 매핑되는 파라미터 이름이 없어 DER에 저장되지는 않지만 함수에 Argument Object에는 들어가게 된다.

** 출처1. 인프런 강좌_자바스크립트 중고급: 근본 핵심 이해

** 출처2. https://velog.io/@imacoolgirlyo/JS-%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8%EC%9D%98-Hoisting-The-Execution-Context-%ED%98%B8%EC%9D%B4%EC%8A%A4%ED%8C%85-%EC%8B%A4%ED%96%89-%EC%BB%A8%ED%85%8D%EC%8A%A4%ED%8A%B8-6bjsmmlmgy
