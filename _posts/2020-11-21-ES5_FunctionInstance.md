---
layout: post
title: (Javascript ES5) Function Instance
summary: 인프런 강좌_자바스크립트중고급_근본 핵심 이해
author: devhtak
date: '2020-11-21 17:41:00 +0900'
category: javascript
---

### function instance
- function instance 기준
  - function 구분
    - 빌트인 function object
    - function object: function 키워드로 생성
    - function isntance: new 연산자로 생성  
  - function object 또한 인스턴스다
    - 빌트인 Function Object로 생성하기 때문이다.
    - 성격적으로는 인스턴스지만 new 연산자로 생성한 인스턴스와 구분하기 위해 강좌에서는 function object로 표기
  - new 연산자로 생성하는 인스턴스는 일반적으로 prototype에 프로퍼티를 작성

- function instance 생성 
```javascript
function Book(point) {
    this.point = point;
}
Book.prototype.getPoint() = funciton() {
    return this.point + 200; 
}
var book = new Book(100);
console.log(book.point); // 100
console.log(book.getPoint()); // 300
```
  - 해당 코드는 funcion instance를 생성하는 전형적인 형태이다.
  - function Book(point) { ... }
    - Book Object를 생성
    - Book.prototype이 만들어진다.
  - Book.prototype.getPoint() = function() { ... } 
    - Book.prototype에 getPoint(프로퍼티)를 연결하고 function object 를 할당
    - Book.prototype이 오브젝트이므로 프로퍼티를 연결할 수 있다.
  - var book = new Book(100);
    - Book() 을 실행하며 인스턴스를 생성하고 생성한 인스턴스에 point 값을 설정
    - Book.prototype에 연결된 프로퍼티를 생성한 인스턴스에 할당
  - console.log(book.point); 
    - obj 인스턴스에 프로퍼티 이름으로 값을 구해 출력
  - console.log(book.getPoint());
    - obj 인스턴스의 메소드를 호출
  - return this.point + 200;
    - this가 book 인스턴스를 참조
  - 강좌의 함수/메소드 사용 기준
    - Book(): 함수
    - getPoint(): 메소드, prototype에 연결되어 있으면 메소드다.

  ** 함수와 메소드의 차이
  - 함수는 객체로부터 독립적이며, 메소드는 객체에 종속적이다.
  - 메소드는 거의 모든 면에서 함수와 동일하지만, 메소드는 호출된 객체에 암시적으로 전달되며, 메소드는 클래스 안에 있는 데이터를 조작할 수 있다는 차이점이 있다.
  
### 생성자 함수
- 생성자 함수
  - new 연산자와 함께 인스턴스를 생성하는 함수
    - new Book()에서 Book()이 생성자 함수
  - new 연산자
    - 인스턴스 생성을 제어하여 생성자 함수 호출
  - 생성자 함수
    - 인스턴스 생성하고 선언된 변수로 반환 한다
    - 인스턴스에 초깃값 설정
  - 코딩 관례로 생성자 함수의 첫 문자는 대문자

- 생성자 함수 실행 과정
```javascript
function Book(point) {
    this.point = point;
}
Book.prototype.getPoint = function() {
    return this.point + 200;
}
var book = new Book(100);
```
  - new 연산자로 인스턴스 생성을 제어하고 생성자 함수인 Book() 으로 인스턴스를 생성하여 반환합니다.
  - 엔진이 new 연산자를 만나면
    - function의 [[Cosntruct]]를 호출하면서 파라미터 값으로 100을 넘겨줍니다.
  - function object를 생성할 때
    - Book() 함수 전체를 참조하도록 [[Construct]]에 설정합니다.
  - [[Construct]]에서 인스턴스를 생성하여 반환
  - 반환된 isntance를 new 연산자가 받아 new 연산자를 호출한 곳으로 반환
  - new 라는 뉘앙스로 인해
    - new 연산자가 인스턴스를 생성하는 것으로 생각할 수 있지만
    - function object의 [[Cosntruct]]가 인스턴스를 생성한다. Book()이 생성자 함수다.

- 인스턴스 생성 과정
  - new Book(100)을 실행
    - Book 오브젝트의 [[Construct]] 호출
    - 파라미터 값을 [[Construct]]로 넘겨준다.
  - [[Constuct]]는 빈 Object를 생성한다.
    - 이것이 인스턴스이며 지금은 빈 오브젝트 { } 이며 하나씩 채워간다.
  - Object에 내부 처리용 프로퍼티를 설정
    - 공통 프로퍼티와 선택적 프로퍼티
  - Object의 [[Class]]에 "Object" 설정
    - 따라서 생성된 인스턴스 타입은 Object
  - Book.prototype에 연결된 프로퍼티(메소드)를 생성한 인스턴스의 [[Prototype]]에 설정, constructor도 같이 설정
  ```
  Book instance: {
      point: 100,
      __proto__ = {
        constructor: Book,
        getPoint: function() {},
        __proto__: Object
      }
  }
  ```

### constructor
- constructor 프로퍼티
```
Book function object: {
    prototype: {
        constructor: Book
    }
}
```
  - 생성하는 function object를 참조
    - function object를 생성할 때 설정
    - prototype에 연결되어 있다.
    - constructor가 없어도 인스턴스가 생성되지만 필요하지 않은 것은 아니다.
  - ES5: constructor 변경 불가 (생성자를 활용할 수 없다)
  - ES6: constructor 변경 가능 (활용성이 높다)

- constructor 비교
```javascript
var Book = function() {};
console.log(Book === Book.prototype.constructor); // true

var obj = new Book();
console.log(Book === obj.constructor); // true

console.log(typeof Book); // function
console.log(typeof obj); // object
```
  - Book === Book.prototype.constructor;
    - 실행결과 true;
    - Book object와 Book.prototype.constructor가 타입까지 같다는 뜻.
    - Book object를 생성할 때 Book.prototype.constructor가 Book Object를 참조하기 때문
  - Book === obj.constructor;
    - obj의 constructor가 Book Object를 참조하므로 true 출력
  - typeof Book : Book Object의 타입은 function
  - typeof obj: obj 인스턴스의 타입은 Object
  - function object를 인스턴스로 생성했더니 Object로 타입이 변경되었다.
    - 이것은 [[Construct]]가 실행될 때 생성한 오브젝트의 [[Class]]에 'Object'를 설정하기 때문
  - Object Type이 바뀐다는 것은 object 성격과 목적이 바뀐 것을 뜻한다.
    - function object로 접근하는 것이 아닌 인스턴스에 개념으로 접근해야 한다.

### prototype, 상속
- prototype object 목적
  - prototype 확장
    - prototype에 프로퍼티를 연결하여 prototype 확장이 가능하다.
    - Book.prototype.getPoint = function() {}
  - property 공유
    - 생성한 인스턴스에서 원본 prototype의 프로퍼티 공유
    - var obj = new Book(123); obj.getPoint(); // Book instance prototype의 getPoint를 사용하는 것
  - 인스턴스 상속
    - function 인스턴스를 연결하여 상속
    - Point.prototype = new Book();
- 인스턴스 상속
```javascript
function Book(title) {
    this.title = title;
}
Book.prototype.getTitle = function() {
    return this.title;
}
function Point(title) {
    Book.call(this, title);
}
Point.prototype = Object.create(Book.prototype, {});
var obj = new Point("자바스크립트");
console.log(obj.getTitle()); // 자바스크립트
```
  - 인스턴스 상속 방법
    - prototype에 연결된 프로퍼티로 인스턴스로 생성하여 상속받을 prototype에 연결
    - 그래서 prototype-based 상속이라고 한다. (Classed-based 상속과 비교)
  - JS에서 prototype은 상속보다 프로퍼티 연결이 의미가 더 크다.
    - 인스턴스 연결도 프로퍼티 연결의 하나이다.
  - ES5 상속은 OOP 의 상속 기능 부족
  - ES6 의 Class로 상속 사용
  ```javascript
  class Point extends Book {}
  ```

### prototype 확장
- prototype 확장 방법
  - prototype에 프로퍼티를 연결하여 작성
    - prototype.name = value 형태
  - name에 프로퍼티 이름 작성, value에 JS 데이터 타입 작성, 일반적으로 function을 사용
  - prototype에 null을 사용하면 확장이 불가능하다.

- 프로퍼티 연결 고려사항
  - prototype에 연결할 프로퍼티가 많을 때
    - Book.prototype.name1, 2, ~ N 형태는 반복해서 작성해야 하므로 번거롭다.
    - Book.prototype = { name1: value1, name2: value2 ... nameN: valueN} 형태로 작성
  - 객체로 연결하는 경우에는 constructor가 지워진다.
    - {name1: value1 ... } 형태로 설정한 후, prototype에 constructor를 다시 연결
  
- constructor 연결
```javascript
function Book() {};
Book.prototype = {
    constructor: Book,
    setPoint: function() {}
};
var obj = new Book();
console.log(obj.constructor); // function Book() {}
```
  - 오브젝트 리터럴 {}을 사용하여 프로퍼티를 연결할 때에는 constructor가 지워지는 것을 고려해야 한다.
  - constructor가 없어도 인스턴스가 생성되지만 constructor가 연결된 것이 정상이므로 코드처럼 constructor에 Book function을 할당하자

- prototype 확장과 인스턴스 형태
```javascript
function Book(point) { this.point = point; };
Book.prototype.getPoint = functino() { return this.point; };
var obj = new Book(100);
console.log(obj.getPoint()); // 100
```
  - function Book(point) { };
    - Book Object 생성
  - Book.prototype.getPoint = function() {}
    - Book.prototype에 getPoint 메소드 연결
  - var obj = new Book(100);
    - 인스턴스를 생성하여 obj에 할당
  - obj.getPoint()
    - obj 인스턴스의 getPoint() 호출
  - 인스턴스 생성
    - prototype에 연결된 메소드를 인스턴스.메소드이름() 형태로 호출
  ```
  obj: {
      point: 100,
      __proto__ = {
          constructor: Book,
          getPoint: function() {},
          __proto: Object
      }
  }
  ```
  
  
  
  
  
  
  
  
  
