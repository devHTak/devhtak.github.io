---
layout: post
title: (Javascript ES6) 연산자, Getter-Setter
summary: 인프런 강좌_자바스크립트 ES6+ 기본
author: devhtak
date: '2020-12-06 22:41:00 +0900'
category: Javascript ES6+
---

### Trailing Commas
```javascript
const obj = {
  one: 1,
  two: 2,
};
const arr = [1, 2,];
```
- Object 끝에 콤마 사용
  - } 앞에 콤마 사용 가능
    
- 배열 끝에 콤마 사용
  - ] 앞에 콤마 사용 가능

### 거듭 제곱
```javascript
console.log(2 ** 3); // 8
console.log(3 ** 2); // 9
console.log(2 ** 3 ** 2); // 512 = 2 ** (3 ** 2)
console.log( ( 2 ** 3) ** 2 ) // 64
```
  - 좌결합성
    - 왼쪽에서 오른쪽으로 계산 -> 1 + 2 + 3 은 1 + 2를 계산하고 + 3을 계산한다.
  - 우결합성
    - 오른쪽에서 왼쪽으로 계산 -> A ** B ** C는 B ** C를 먼저 하고 A**를 계산 (A ** (B ** C))
    
### try-catch
- try-catch의 catch(error)에서 catch 처럼 (error)를 생략 가능
  - ES2019
  ```javascript
  const sports = "스포츠"
  try {
      sports = "축구";
  } catch(error) {
      console.log("(error) 작성");
  }
  
  try {
      sports = "축구";
  } catch {
      console.log("(error) 생략");
  }
  ```
- (error)에서 메시지를 받아 사용하지 않을 때 편리하다.
- 타이핑 실수를 방지할 수 있다.

### 함수 작성 형태
- Object에 함수를 작성할 때 function 키워드를 작성하지 않는다.
  ```javascript
  const sports = {
      point: 100,
      getValue: function() { return this.point; },
      getPoint() { return this.point; }
  }
  console.log(sport.getPoint()); // 100
  ```
- 참고: Object에 함수를 작성하는 이유
  - 함수에서 this로 Object 전체 참조
  - new 연산자로 인스턴스를 생성하지 않음, 메서드가 아닌 함수로 접근
  - Object 전체가 하나의 묶음. 접근성, 가독성이 좋음
  - sports 시멘틱을 부여할 수 있으며 다른 오브젝트와 이름과 프로퍼티 이름이 충돌되지 않음
  
### Getter / Setter

- Getter
  - getter로 선언된 함수를 자동으로 호출
    - 값을 반환하는 시맨틱을 갖고 있으므로 getter 함수에서 값을 반환
  - ES5
    ```javascript
    var book = {};
    boo.defineProperty(book, "title", {
        get: function() { return "책"; }
    });
    console.log(book.title); // 책
    ```
    - book.title을 실행하면 title 프로퍼티에서 get 속성의 존재를 체크한다.
    - 있으면 get()함수를 호출
    - book.title.get() 처럼 함수로 호출하면 에러가 발생한다.
    - ES5의 Descriptor 참조
    - Object.defineProperty(obj, prop, descriptor);
      - 객체에 직접 새로운 속성을 정의하거나 이미 존재하는 속성을 수정한 후 그 객체를 반환
      - obj: 프로퍼티를 정의할 대상 객체,
      - prop: 새로 정의하거나 수정하려는 프로퍼티의 이름,
      - descriptor: 새로 정의하거나 수정하려는 속성을 쓰는 객체이다.
      - 여기서 속성 서술자(property descriptors)는 데이터 서술자(data descriptors)와 접근자 서술자(accessor descriptors)로 나뉜다. 데이터 서술자는 값을 가지는 속성이고, 접근자 서술자는 접근자(getter)와 서술자(setter) 한 쌍을 가지는 속성이다.
  - ES6
    ```javascript
    const book = {
        point: 100,
        get getPoint() {
            return this.point;
        }
    };
    console.log(book.getPoint); // 100
    ```
    - get getPoint(){}처럼 getPoint() 앞에 get을 작성하면 getter로 선언된다.
    - getPoint() 함수가 자동으로 호출된다.
    - ES6 장점
      - ES5처럼 프로퍼티의 속성 구조가 아니다.
        - ES5는 defineProperty를 활용하여 property 단위별로 속성을 선언한다. property에 종속적이다.
      - 작성이 편리하며 다수의 getter 사용이 가능
        ```javascript
        const book = {
            get getPoint() { return "포인트"; }
            get getTitle() { return "제목"; }
        };
        console.log(book.getPoint); // 포인트
        console.log(book.getTitle); // 제목
        ```

- Setter
  - 프로퍼티에 값을 할당하면
    - setter로 선언된 함수 자동 호출
    - 값을 설저하는 시맨틱을 갖고 있으므로 setter 함수에서 값을 설정해야 합니다.
  - ES5 형태
    ```javascript
    var book = {title: "HTML" };
    Object.defineProperty(book, "change", {
        set: function(param) { this.title = param; }
    });
    book.change = "자바스크립트";
    console.log(book.title); // 자바스크립트
    ```
    - book.change = "자바스크립트"; 를 실행하면 change 프로퍼티에서 set 속성의 존쟁 여부를 체크한다.
    - 있으면, set() 함수를 호출
    - "자바스크립트"를 파라미터 값으로 넘겨준다.
  - ES6 형태
    ```javascript
    const book = {
        point: 100,
        set setPoint(param) {
            this.point = param;
        }
    };
    book.setPoint = 200;
    console.log(book.point); // 200
    ```
    - setPoint() 앞에 set을 작성하면 setter로 선언
    - book.setPoint = 200; setPoint에 값을 할당하면 setPoint()가 자동으로 호출
    - 파라미터 값으로 200을 넘겨준다.
  - 변숫값을 함수 이름으로 사용
    ```javascript
    const name = "setPoint";
    const book = {
        point: 100,
        set [name](param) {
            this.point = param;
        }
    };
    book[name] = 200; // book.setPoint = 200;
    console.log(book.point); // 200
    ```
    - name 변숫값인 setPoint 가 함수 이름으로 사용 가능하다.
    - getter도 같은 방법으로 사용할 수 있다.
  - setter 삭제
    ```javascript
    const name= "setPoint";
    const book = {
        set [name](param) {
            this.point = param;
        }
    };
    delete book[name];
    console.log(book[name]); // undefined
    ```
    - delete book[function_name] 을 활용하여 setter를 삭제할 수 있다.

** 출처1. 인프런 강좌_자바스크립트 ES6+ 기본
