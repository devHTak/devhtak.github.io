---
layout: post
title: React, props와 state
summary: React
author: devhtak
date: '2021-02-11 21:50:00 +0900'
category: Frontend
---

#### props

- 부모 component가 자식 component에 데이터를 전달할 때 사용한다.
  - 자식 component에서 props 사용
    ```javascript
    import React, {Component} from 'react';

    class MyName extends Component {
        render() {
            return (
                <div>
                    안녕하세요! 제 이름은 <b>{this.props.name}</b>입니다.
                </div>
            );
        }
    }

    export default MyName;
    ```
  - 부모 copmponent에서 props 넘겨주기
    ```javascript
    import React, {Component} from 'react';
    import MyName from './MyName;;
    
    class App extends Component {
        render() {
            return (
                <MyName name="react" />
            );
        }
    }
    ```

- props가 넘어오지 않을 때 default값을 설정할 수 있다.
  - render() 함수 안에 static 변수 생성하기 
    ```javascript
    import React, {Component} from 'react';

    class MyName extends Component {
        static defaultProps= {
            name: 'default react'
        };
        render() {
            return (
                <div>
                    안녕하세요! 제 이름은 <b>{this.props.name}</b>입니다.
                </div>
            );
        }
    }
    export default MyName;
    ```
  - class 하단에 static 변수 설정
    ```javascript
    import React, {Component} from 'react';

    class MyName extends Component {
        render() {
            return (
                <div>
                    안녕하세요! 제 이름은 <b>{this.props.name}</b>입니다.
                </div>
            );
        }
    }
    MyName.defaultProps = {
        name: 'default react';
    };
    export default MyName;
    ```
  
- 함수형 Copmonet 작성
  - 구조 분해 할당을 활용하여 props를 받아준다.
    - 구조 분해 할당: 구문은 배열이나 객체의 속성을 해체하여 그 값을 개별 변수에 담을 수 있게 하는 JavaScript 표현식입니다.
      ```javascript
      [a, b, ...rest] = [10, 20, 30, 40, 50];
      console.log(rest); // [30, 40, 50]     
      ```
  - 함수형 Component 예시
    ```javascript
    import React from 'react';

    const MyName = ({name}) => {
        return (
            <div>
                안녕하세요. 제 이름은 {name}입니다.
            </div>
        );
    };

    MyName.defaultProps = {
        name: 'default react'
    };
    export default MyName;
    ```
    
#### state

- state는 Component 내에 있다.
- state는 내부에서 변경할 수 있다. setState()를 사용하여 state값을 변경 시킬 수 있다.

```javascript
import React, {Component} from 'react';

class Counter extends Component {
    state = {
        number: 0
    }
    handleIncrease = () => {
        // this.state.number = this.state.number + 1;
        // 절대 하면 안된다. Component에서 변화를 인식하지 못한다.
        this.setState({
            number: this.state.number + 1;
        });
    }
    
    handlerDecrease = () => {
        this.setState({
            number: this.state.number - 1;
        });
    }
    
    render() {
        return (
            <div>
                <h1>Counter</h1>
                <div>값: {this.state.number}</div>
                <button onClick={this.handleIncrease}>+</button>
                <button onClick={this.handlerDecrease}>-</button>
            </div>
        );
    }
}
export default Counter;
```
  - 주의점 1. this.setState()를 활용해야 Component에서 변화를 감지할 수 있다.
  - 주의점 2. this.setState()를 사용하는 handler 함수는 arrow 함수로 사용해야 한다.
    - arrow함수가 아닌 function 키워드를 사용하는 경우 this를 직접 binding 해서 알려주어야 한다.
      ```javascript
      constructor(props) {
          super(props); // Component 클래스 constructor 호출
          this.handlerIncrease = this.hanlderIncrease.bind(this);
          this.handlerDecrease = this.hanlderDecrease.bind(this);
      }
      ```
