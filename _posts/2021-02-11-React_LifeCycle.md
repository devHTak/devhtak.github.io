---
layout: post
title: React Lifecycle API
summary: React
author: devhtak
date: '2021-02-11 22:41:00 +0900'
category: Frontend
---

#### LefeCycle API

- Component가 나타날 때(mounting), 업데이트 될 때(updating), 사라질 때(unmounting)를 기준으로 작업이 필요할 때 사용한다.
- LifeCycle API
  - Mounting
    - 컴포넌트가 브라우저 상에 나타날 때
    - Render Phase
      - constructor
      - getDerivedStateFromProps
      - render
    - Pre-Commit Phase
    - Commit Phase
      - componentDidMount
  - Updating - New props, setState(), forceUpdate()
    - 컴포넌트의 props, state가 변경할 때
    - Render Phase
      - getDerivedStateFromProps (mounting에서 new props 까지)
      - shouldComponentUpdate (new props + setState() 까지)
      - render
    - Pre-Commit Phase
      - getSnapshotBeforeUpdate
    - Commit Phase
      - componentDidUpdate
  - Unmounting
    - 컴포넌트가 브라우저에서 사라질 때
    - Commit Phase
      - componentWillUnmount
      
- Phase
  - Render Phase: Pure and has no side effects, May be paused, aborted or restarted by React.
  - Pre-Commit Phase: Can read the DOM
  - Commit Phase: Can work with DOM, run side effects, schedule updates.
      
- 컴포넌트 초기 생성
  - 컴포넌트가 브라우저에 나타나기 전, 후에 호출하는 API가 들어 있다.

  - constructor
    - 컴포넌트 생성자 함수
    - 컴포넌트가 새로 만들어질 때마다 해당 함수가 호추로딘다.
    ```javascript
    constructor(props) {
        super(props);
    }
    ```

  - componentDidMount
    - 컴포넌트가 화면에 나타나게 됐을 때 호출된다.
    - DOM을 사용해야 하는 외부 라이브러리를 연동
    - 해당 컴포넌트에서 필요로 하는 데이터를 요청하기 위해 axios, fetch 등을 통해 ajax 요청
    - DOM의 속성을 읽거나 직접 변경하는 작업을 진행
    ```javascript
    componentDidMount() {
        console.log('componentDidMount');
    }
    ```

- 컴포넌트 업데이트
  - 컴포넌트 업데이트는 props의 변화, 그리고 state의 변화에 따라 결정된다.

  - \[deprecate] componentWillReceiveProps
    - Component가 새로운 props를 받게됐을 때 호출
    - state와 props에 따라 변해야 하는 로직을 작성한다.
    - 새로 받게될 props는 nextProps로 조회할 수 있으며, 이 때 this.props를 조회하면 업데이트 되기 전의 API이다.
    - 16.3부터 getDerivedStateFromProps()로 대체
    ```javascript
    componentWillReceiveProps(nextProps) {
        //this.props는 아직 바뀌지 않은 상태
    }
    ```

  - getDerivedStateFromProps
    - v16.3 이후에 만들어진 라이프사이클 API
    - setState를 하는 것이 아니라 리턴하는 형태
    - componentWillReceiveProps와 다르게 초기화 과정에서도 실행된다.
    ```javascript
    static getDerivedStateFromProps(nextProps, prevState) {
        // 여기서는 setState 를 하는 것이 아니라
        // 특정 props 가 바뀔 때 설정하고 설정하고 싶은 state 값을 리턴하는 형태로
        // 사용됩니다.
        /*
        if (nextProps.value !== prevState.value) {
          return { value: nextProps.value };
        }
        return null; // null 을 리턴하면 따로 업데이트 할 것은 없다라는 의미
        */
    }
    ```
    - 예제
      - props와 state의 value값이 다른 경우, state의 value값을 props와 맞춰주자
      - App.js
      ```java
      class App extends Component {
          state = {
              counter: 1,
          };

          handleIncrease = () => {
              this.setState({
                  counter: this.state.counter + 1,
              });
          }

          handleDecrease = () => {
              this.setState({
                  counter: this.state.counter - 1,
              });    
          }
          render() {
              return (
                  <div className="App">
                      <MyComponent name="react" value={this.state.counter}/>
                      <button onClick={this.handleIncrease}>+</button>
                      <button onClick={this.handleDecrease}>-</button>
                  </div>
              );
          }  
      }
      ```
      - MyComponent.js
      ```javascript
      class MyComponent extends Component {
          state = {
              value: 1
          }

          static defaultProps = {
              value: 0
          }

          static getDerivedStateFromProps(nextProps, prevState) {
              if(prevState.value != nextProps.value) {
                  return {
                      value: nextProps.value
                  };
              }
              return null;
          }

          render() {
              return(
                  <Fragment>
                      <h2>{this.props.name}</h2>
                      <div>props: {this.props.value}</div>
                      <div>state: {this.state.value}</div>
                  </Fragment>
              )
          }
      }
      ```
      - getDerivedStateFromProps로 update하기 전에 nextProps와 prevState를 비교하여 nextState를 setState하는 것이 아닌 return해줄 수 있다.

  - shouldComponentUpdate
    ```javascript
    shouldComponentUpdate(nextProps, nextStae) {
        if(nextProps.value == 10)
            return false;
        return true;
    }
    ```
    - 해당 컴포넌트는 최적화하는 작업에서 매우 유용하게 사용된다.
    - 변화가 발생한 부분을 감지해내기 위해 Virtual DOM에 한번 그려줘야 한다.
    - 즉, 현재 컴포넌트의 상태가 업데이트 되지 않아도, 부모 컴포넌트가 리렌더링되면, 자식 컴포넌트도 렌더링되어야 한다.
      - 여기서 렌더링이란 render()함수가 호출된다는 의미
    - Virtual DOM에 리렌더링 하는 것보 불필요한 경우에는 방지하기 위해서 shouldComponentUpdate 를 작성한다.

  - \[deprecate]componentWillUpdate
    ```javascript
    componentWillUpdate(nextProps, nextState) {
        // ...
    }
    ```
    - 애니메이션 효과를 초기화하거나, 이벤트 리스너를 없애는 작업을 한다.
    - 이 API는 16.3 이후 deprecate 되었다.

  - getSnapshotBeforeUpdate
    ```javascript
    getSnapshotBeforeUpdate(prevProps, prevState) {
        // DOM 업데이트가 일어나기 직전의 시점입니다.
        // 새 데이터가 상단에 추가되어도 스크롤바를 유지해보겠습니다.
        // scrollHeight 는 전 후를 비교해서 스크롤 위치를 설정하기 위함이고,
        // scrollTop 은, 이 기능이 크롬에 이미 구현이 되어있는데, 
        // 이미 구현이 되어있다면 처리하지 않도록 하기 위함입니다.
        if (prevState.array !== this.state.array) {
            const {
                scrollTop, scrollHeight
            } = this.list;

            // 여기서 반환 하는 값은 componentDidMount 에서 snapshot 값으로 받아올 수 있습니다.
            return {
                scrollTop, scrollHeight
            };
        }
    }

    componentDidUpdate(prevProps, prevState, snapshot) {
        if (snapshot) {
            const { scrollTop } = this.list;
            if (scrollTop !== snapshot.scrollTop) return; // 기능이 이미 구현되어있다면 처리하지 않습니다.
            const diff = this.list.scrollHeight - snapshot.scrollHeight;
            this.list.scrollTop += diff;
        }
    }
    ```
    - 해당 예제는 props 또는 state가 변하는 이벤트가 발생하여도 기존 보고있던 위치로 scroll이 움직이지 않는다.
    - 발생 시점
      - render()
      - getSnapshotBeforeUpdate()
      - 실제 DOM에 변화 발생
      - componentDidUpdate
    - 해당 API를 통해 DOM 변화가 일어나기 직전의 DOM 상태를 가져오고 여기서 리턴하는 값은 componentDidUpdate에서 3번째 파라미터로 받아올 수 있게 된다.

  - componentDidUpdate
    ```javascript
    componentDidUpdate(prevProps, prevState, snapshot) {
        //...
    }
    ```
    - render()를 호출하고 난 다음에 발생한다.
    - 이 시점에는 이미 this.state와 this.props가 바꿔어있다.
    - 그리고 getSnapshotBeforeUpdate에서 return한 값이 snapshot으로 받아온다.

- Component 제거
  - 컴포넌트가 더 이상 필요하지 않게 되면 단 하나의 API가 호출된다.

  - componentWillUnmount
    - MyComponent.js
    ```javascript
    componentWillUnmount() {
        // 이벤트, setTimeout, 외부 라이브러리 인스턴스 제거
        console.log("Good Bye");
    }    
    ```
    - App.js
    ```javascript
    return (
        <div className="App">
            {
                this.state.counter < 10 && <MyComponent name="react" value={this.state.counter}/>
            }
            <button onClick={this.handleIncrease}>+</button>
            <button onClick={this.handleDecrease}>-</button>
        </div>
    );
    ```
    - 예제는 counter가 10이상이 될 때 MyComponent가 사라진다. 그 때 componentWillUnmount() 가 실행된다.
    - 주로 등록했던 이벤트를 제거한다. (setTimeout -> clearTimeout)
    - 추가적으로, 외부 라이브러리를 사용한게 있고, 해당 라이브러리에 dispose 기능이 있다면 여기서 호출한다.

- Component 에러 발생
  - componentDidCatch
    ```javascript
    componentDidCatch(error, info) {
        this.setState({
            error: true,
        });
        // API를 통해서 서버에 오류 내용 아리기
    }
    ```
    - 에러가 발생할 수 있는 컴포넌트의 부모 컴포넌트에 작성한다.
    - error를 state로 등록해서, 해당 값에 따라 오류 화면을 비춰줄 수 있다.


 - 출처: https://react-anyone.vlpt.us/05.html     
