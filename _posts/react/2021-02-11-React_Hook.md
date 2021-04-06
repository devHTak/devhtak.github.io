---
layout: post
title: React Hook
summary: React
author: devhtak
date: '2021-02-11 21:41:00 +0900'
category: Frontend
---

#### React Hook

- 16.8 버전 이후에 새롭게 추가되었으며 Class Component 작성 없이 Function Component로만 React의 기능을 사용할 수 있게 해준다.
- React Component
  - Class Component
    - provide more features
    - longer code
    - more complex code
    - slower performance
    ```javascript
    import React, {Component} from 'react'
    export default class Hello extends Component {
        render() {
            return (
                <div>Hello Class Component</div>
        )};
    };
    ```
    
  - Functional Component
    - provides less features
    - shorter code
    - simpler code
    - faster performance
    ```javascript
    import React from 'react'
    export default function Hello() {
        return (
            <div>Hello Functional Component</div>
        );
    }
    ```
  - Class Component에서만 사용할 수 있는 것
    - show more common lifecycle
      
      |Pase|Mounting|Updating|Unmounting|
      |---|---|---|---|
      |Render Phase|constructor|-|-|
      |Render Phase|render|render(new props, setState(), forceUpdate())|-|
      |Commit phase|react updates DOM and refs|react updates DOM and refs|-|
      |Commit phase|componentDidMount|componentDidUpdate|componentWillUnmount|
      
      - Render phase: 순수하고 부작용이 없습니다. React에 의해 일시 중지, 중단 또는 재시작 될 수 있습니다
        - constructor: 생성자로 객체가 처음 생성될 때 init을 해준다. state 등에 값을 미리 세팅해둔다.
        - render: HTML DOM에 알맞게 넣어주어 rendering을 한다.        
      - Commit phase: DOM을 사용하여 부작용을 실행하고 업데이트를 예약 할 수 있습니다.
        - componentDidMount: Mount과 완료된 상태에서 데이터를 가져오는 등을 진행할 수 있다.
        - update 등에서도 비슷한 순서에서 할 수 있다.
      - Mounting -> Updating -> Unmounting 순으로 lifecycle이 정해진다.
      - 참고: http://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/
      
    - Functional Component 에서도 lifecycle 사용하기 위해서 react 16.8에서 Hook을 update했다.
      ```javascript
      import React, {Component} from 'react';
      import axios from 'axios';
      
      export default class Hello extends Component {
          constructor(props) {
              super(props);
              this.state = {name: ""};
          }
          
          componentDidMount() {
              axios.get('/api/user/name')
                  .then(response => {
                      this.setState({name: response.data.name});
                  });
          }
          
          render() {
              return (
                  <div>My name is {this.state.name}</div>
              );
          }
      };
      ```
      
      ```javascript
      import React, {useEffect, useState} from 'react'
      import axios from 'axios';
      
      export default function Hello() {
          const [Name, setName] = useState("");
          useEffect(()=> {
              Axios.get('/api/user/name')
                  .then(response=> {
                      setName(response.data.name);
                  });
          }, []);
          
          return (
              <div>My name is {Name}</div>
          );
      }
      ```
      - constructor 에서 state 세팅 <-> useState
      - componentDidMount <-> useEffect    

#### State Hook

```javascript
import React, {useState} from 'react';
function Example() {
    // 새로운 state 변수로 count와 변경하는 setCount 메서드 선언 
    // count를 0으로 초기화
    const [count, setCount] = useState(0); 
    
    return (
        <div>
            <button onClick={()=>setCount(count+1)} >Click</button>
        </div>
    );
}
export default Example;
}
```
- useState
  - state 변수를 컴포넌트 안에서 사용할 수 있게 해준다.
  - state를 추가하고 싶을 때 사용한다.
  - parameter로 state의 초기 값을 넘겨줄 수 있다.
  - 리턴은 변수와 갱신 함수 2가지 쌍을 반환한다.
 
- state 가져오기
  - 클래스 컴포넌트
    ```javascript
    {this.state.count}
    ```
  - Hook를 사용한 함수 컴포넌트
    ```javascript
    {count}
    ```
    
- state 갱신하기  
  - 클래스 컴포넌트에서는 this.setState를 활용하여 state 변수객체를 대입할 수 있었다.
  - 함수 컴포넌트는 useState로 생성한 갱신함수(setCount)로 변경 값을 넣어 갱신할 수 있다.

#### Effect Hook 사용하기

```javascript
import React, {useState, useEffect} from 'react';

function Example() {
    const [count, setCount] = useState(0);
    
    // componentDidMount, componentDidUpdate와 같은 방식
    useEffect(() => {
        document.title = `You cliecked ${count} times`;
    });
    
    return (
        <div>
            <button onClick={()=> setCount(count + 1)}>Click</button>
        </div>
    );
}
```

- useEffect Hook을 componentDidMount와 componentDidUpdate, componentWillUnmount가 합쳐진 것으로 생각해도 좋습니다.

- Class Component
  - 리액트의 class 컴포넌트에서 render 메서드 그 자체는 side effect를 발생시키지 않습니다. 
  - 이때는 아직 이른 시기로서 이러한 effect를 수행하는 것은 리액트가 DOM을 업데이트하고 난 이후입니다.
  - 리액트 class에서 side effect를 componentDidMount와 componentDidUpdate에 두는 것이 바로 이 때문입니다.

- useEffect
  - 컴포넌트가 렌더링한 이후에 다양한 side effects를 표현할 수 있다.
  - useEffect를 함수 컴포넌트 안에 둠으로써 state 변수를 접근할 수 있게 해준다.
  - useEffect는 렌더링 초기화, 모든 업데이트에 수행된다.

#### Hook 규칙

- Hook은 컴포넌트에 최상위(top level)에서만 호출해야 한다.
  - 반복문, 조건문, 중첨된 함수 내에서 Hook을 호출하면 안된다.
  - React가 useState, useEffect가 여러번 호출되는 중에도 Hook의 상태를 올바르게 유지하도록 해준다.
  - React는 Hook이 호출되는 순서에 의존하여 state가 어떤 useState가 호출되는 지 알 수 있다.

- 오직 React 함수 내에서 Hook을 호출해야 한다.
  - 일반적인 JavaScript 함수에서 호출하면 안된다.

- 출처: John Ann님의 노드 리액트 기초 강의
- 출처: https://ko.reactjs.org/docs/hooks-intro.html
