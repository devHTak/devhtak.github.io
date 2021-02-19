---
layout: post
title: React Hook
summary: React
author: devhtak
date: '2021-02-11 21:41:00 +0900'
category: Frontend
---

#### React Hook

- Class Component처럼 Functional Component에서 사용하지 못하는 Lifecycle을 사용하도록 도와준다.
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

- 출처: John Ann님의 노드 리액트 기초 강의
