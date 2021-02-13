---
layout: post
title: Redux
summary: React
author: devhtak
date: '2021-02-12 11:41:00 +0900'
category: Frontend
---

#### Redux 설명

- What is Redux?
  - Redux is a predictable state container for JS apps.
  - 상태 관리 라이브러리
  
- react에서 state란?
  - props
    - properties의 줄임말
    - props flow downwards from the parent component
      - 부모 component에서 자식 component에게 설정등을 보낼 수 있다.
    - props are immutable from the child perspective if you want to change that value? the parent should just change its internal state.
      - 부모 component에서 보낸 값은 자식 component에서 변경할 수 없다. 다시 부모 component에서 변경해야 한다.
    ```
    <ChatMessages message={message} currentMember={member} />
    ```
  - state
    - parent component에서 child component로 data를 보내는 것이 아닌 그 component 안에서 data를 전달하려면 state를 사용한다.
      - 부모 자식 관계가 아닌 component 안에서 데이터를 전달할 수 있다.
    - 예를 들어 검색 창에 글을 입력할 때 글이 변하는 것은 state로 바꾸는 것이다.
    - stats is mutable
      - state는 props와 달리 부모, 자식 component에서 수정할 수 있다.
    - state가 변하면 re-render 된다.
    ```
    state={
        message: '',
        attachFile: undefined,
        openMenu: false,
    };
    ```
  - redux는 state를 관리한다.
    - redux가 없으면 상태를 공유하기 위해 component hirachy에서 데이터 이동이 계속 필요하다.
    - redux를 사용하면 component에서 store에 바로 접근하여 가져다 사용할 수 있다.
  
- Redux Data Flow (strict unidirectional data flow)
  - Action -> Reducer -> Store --(Subscribe)--> React Component --(Dispatch-action)--> Action
    - 철저하게 한방향으로 흐른다.
    
  - Action
    - a plain object describing what happend.
    ```
    { type: 'LIKE_ARTICLE', articleId: 42}
    { type: 'FETCH_USER_SUCCESS', response: {id: 3, name: 'Mary'}}
    { type: 'ADD_TODO', text: 'Read the Redux docs.'}
    ```
    - Mary liked Article which id is 42
  
  - Reducer
    - a function describing how the application's state changes.
    ```
    (previousState, action) => nextState
    ```
    - 이전 state와 action object를 받은 후에 next state를 return한다.
    - Rducer는 pure function이기 때문에 reducer 내부에서 주의할 점이 있다.
      - Mutate its arguments.
      - Perform side effets like API calls and routing transitions.
      - Call non-pure functions, e.g. Date.now() or Math.random().

- Store
  - The object that brings them together.
  - A store holds the whole state tree of your application.
    - 모든 state를 감싼 상위 객체  
  - The only way to change the state inside it is to dispatch an action on it.
  - A store is not a class. It's just an object with a few methods on it.
  
- <Provider /> is the higher-order component provided by React Redux that lets you bind Redux to React

#### Redux 사용하기

- setting
  - dependency download
    - redux
    - react-redux
    - redux-promis    
    - redux-thunk
    ```
    npm install redux react-redux redux-promise redux-thunk --save
    ```
  - redux-promise와 redux-thunk
    - middleware 이다.
    - Redux store는 항상 plain object action을 받아들이는 데 항상 plain object일 수 없다.
      - promise, function
    - 그래서, middleware는 "tech" dispatch()를 할 수 있다.
      - plain object가 아닌 객체를 interceptor를 함으로써 promise, function도 accept할 수 있도록 한다.
    - redux-promise(promise)
      - "teaches" dispatch how to accept promises, by intercepting the promise and dispatching actions when the promise resolves or rejects.
    - redux-thunk(function)
      - "teaches" dispatch how to accept funcions, by interception the function and calling it instead of passing it on to the reducers.
      
- Promise 란?
  - 참조: https://joshua1988.github.io/web-development/javascript/promise-for-beginners/
  - Promise는 자바스크립트 비동기 처리에 사용되는 객체다.
  - 여기서 자바스크립트의 비동기 처리란 '특 코드의 실행이 완료될 때까지 기다리지 않고 다음 코드를 먼저 수행하는 자바스크립트의 특성을 의미한다.
  
#### Redux 연결하기

- index.js
  ```javascript
  import {Provider} from 'react-redux';
  import { applyMiddleware, createStore } from 'redux';
  import promiseMiddleware from 'redux-promise';
  import ReduxThunk from 'redux-thunk';
  import Reducer from './_reducers';

  const createStoreWithMiddleware = applyMiddleware(promiseMiddleware, ReduxThunk)(createStore);

  ReactDOM.render( 
      <Provider store={createStoreWithMiddleware(Reducer, 
            window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__()
          )}>
          <App />
      </Provider>
      , document.getElementById('hello')
  );

  ```
  - Provider로 전체 App을 감싸야 한다. 여기에 store를 지정할 수 있다.
  - store는 ReduxThunk(redux-thunk), promiseMiddleware(redux-promise) moddleware를 적용하여 사용한다.
  - window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__() 는 크롬의 redux extension을 사용하기 위해 추가하였다.
  - Reducer는 개발자가 직접 개발하여 사용할 reducer를 설정하여 지정한다.
    - index.js는 생략이 가능하다 (import Reducer from './_reducers/index.js')
  
- _reducer/index.js
  ```javascript
  import { combineReducers } from 'redux';
  import user from './user_redux';

  const rootReducer = combineReducers({
      user,
  })

  export default rootReducer;
  ```
  - CombindReducers
    - Root Reducer에서 여러 state를 묶어서 사용하도록 한다.
  - 주석처리한 것 처럼 하위 reducer를 넣어주면 된다.

- 예제: 로그인 페이지
  - LandingPage.js
    ```javascript
    import React, { useEffect } from 'react';
    import {withRouter} from 'react-router-dom';
    import axios from 'axios';

    function LandingPage(props) {

        const onLogoutHandler = (event) => {
            axios.get('/api/users/logout')
                .then((response)=>{
                    console.log(response.data);
                    if(response.data.success) {
                        props.history.push('/');
                    } else {
                        alert('Error');
                    }
                })
        }

        return (
            <div style={{ display:'flex', justifyContent:'center', alignItems: 'center', width: '100%', heidht:'100vh'}}>
                <h2>시작 페이지</h2>
                <br />
                <button onClick={onLogoutHandler}>Logout</button>
            </div>
        );
    }

    export default withRouter(LandingPage);
    ```
    
  - LoginPage.js
    ```javascript
    import React, {useState} from 'react';
    import {useDispatch} from 'react-redux';
    import {loginUser} from '../../../_actions/user_action';
    import {withRouter} from 'react-router-dom';

    function LoginPage(props) {

        const [Email, setEmail] = useState("");
        const [Password, setPassword] = useState("");

        const onEmailHandler = (event) => {
            setEmail(event.currentTarget.value);
        }

        const onPasswordHandler = (event) => {
            setPassword(event.currentTarget.value);
        }

        const dispatch = useDispatch();


        const onSubmitHandler = (event) => {
            event.preventDefault();

            let body = {
                email: Email,
                password: Password,
            };

            dispatch(loginUser(body))
                .then(response => {
                    if(response.payload.loginSuccess) {
                        props.history.push('/');
                    } else {
                        alert('Error');
                    }
                })



        }

        return (
            <div style={{display:'fleax', justifyContent:'center', alignItems:'center', width:'100%', height:'100vh'}}>
                <form style={{display: 'flex', flexDirection: 'column'}} onSubmit={onSubmitHandler}>
                    <label>Email</label>
                    <input type="email" value={Email} onChange={onEmailHandler}/> 
                    <label>Password</label>
                    <input type="password" value={Password} onChange={onPasswordHandler}/>
                    <br />
                    <button>
                        Login                    
                    </button>
                </form>
            </div>
        )
    }

    export default withRouter(LoginPage);
    ```
  - RegisterPage.js
    ```javascript
    import React, {useState} from 'react';
    import {useDispatch} from 'react-redux';
    import {registerUser} from '../../../_actions/user_action';
    import {withRouter} from 'react-router-dom';

    function RegisterPage(props) {

        const dispatch = useDispatch();

        const [Email, setEmail] = useState("");
        const [Name, setName] = useState("");
        const [Password, setPassword] = useState("");
        const [ConfirmPassword, setConfirmPassword] = useState("")

        const onEmailHandler = (event) => {
            setEmail(event.currentTarget.value);
        }

        const onNameHandler = (event) => {
            setName(event.currentTarget.value);
        }

        const onPasswordHandler = (event) => {
            setPassword(event.currentTarget.value);
        }

        const onConfirmPasswordHandler = (event) => {
            setConfirmPassword(event.currentTarget.value);
        }

        const onSubmitHandler = (event) => {
            event.preventDefault();
            if(Password !== ConfirmPassword)
                return alert('비밀번호가 다릅니다.');

            let data = {
                email: Email,
                name: Name,
                password: Password,
            };

            dispatch(registerUser(data))
                .then((response)=>{
                    if(response.payload.success)
                        props.history.push('/');
                    else
                        alert('Error');
                })

        }

        return (
            <div style={{display:'fleax', justifyContent:'center', alignItems:'center', width:'100%', height:'100vh'}}>
                <form style={{display: 'flex', flexDirection: 'column'}} onSubmit={onSubmitHandler}>
                    <label>Email</label>
                    <input type="email" value={Email} onChange={onEmailHandler} />

                    <label>Name</label>
                    <input type="text" value={Name} onChange={onNameHandler} />

                    <label>Password</label>
                    <input type="password" value={Password} onChange={onPasswordHandler} />

                    <label>Confirm Password</label>
                    <input type="password" value={ConfirmPassword} onChange={onConfirmPasswordHandler} />

                    <br />
                    <button>
                        Register
                    </button>
                </form>
            </div>
        )
    }

    export default withRouter(RegisterPage);
    ```
  - \_action/type.js
    ```javascript
    export const LOGIN_USER = 'login_user';
    export const REGISTER_USER = 'register_user';
    export const AUTH_USER = 'auth_user';
    ```
  - \_action/user_actions.js
    ```javascript
    import axios from 'axios';
    import {
        LOGIN_USER,
        REGISTER_USER,
        AUTH_USER
    } from './type.js';


    export function loginUser(dataToSubmit) {
        const request = axios.post('/api/users/login', dataToSubmit)
            .then(response => response.data);
        return {
            type: LOGIN_USER,
            payload: request
        };
    }

    export function registerUser(dataToSubmit) {
        const request = axios.post('/api/users/register', dataToSubmit)
            .then(response => response.data);

            return {
                type: REGISTER_USER,
                payload: request,
            }
    }

    export function auth() {
        const request = axios.get('/api/users/auth')
            .then(response => response.data)

        return {
            type: AUTH_USER,
            payload: request
        }
    }
    ```
  - \_reducers.js/user_redux.js
    ```javascript
    import {
        LOGIN_USER,
        REGISTER_USER,
        AUTH_USER
    } from '../_actions/type';

    export default function(state={}, action) {
        switch (action.type) {
            case LOGIN_USER:
                return {...state, loginSuccess: action.payload};
            case REGISTER_USER:
                return {...state, register: action.payload};
            case AUTH_USER:
                return {...state, userData: action.payload};
            default:
                return state;
        }
    }
    ```
  - App.js
    ```javascript
    import React from 'react';
    import {
      BrowserRouter as Router,
      Switch,
      Route,
      Link
    } from 'react-router-dom';

    import Auth from './hoc/auth';
    import LandingPage from './components/views/LandingPage/LandingPage';
    import LoginPage from './components/views/LoginPage/LoginPage';
    import RegisterPage from './components/views/RegisterPage/RegisterPage';


    function App() {
      return (
        <div className="App">
          <Router>
            <div>
              <Switch>
                <Route exact path="/" component={Auth(LandingPage, null)}></Route>
                <Route exact path="/login" component={Auth(LoginPage, false)}></Route>
                <Route exact path="/register" component={Auth(RegisterPage, false)}></Route>
              </Switch>          
            </div>
          </Router>
        </div>
      );
    }
    export default App;
    ```
  - index.js
    ```javascript
    import React from 'react';
    import ReactDOM from 'react-dom';
    import './index.css';
    import App from './App';
    import reportWebVitals from './reportWebVitals';
    //import 'antd/dist/antd.css';
    import {Provider} from 'react-redux';
    import { applyMiddleware, createStore } from 'redux';
    import promiseMiddleware from 'redux-promise';
    import ReduxThunk from 'redux-thunk';
    import Reducer from './_reducers';

    const createStoreWithMiddleware = applyMiddleware(promiseMiddleware, ReduxThunk)(createStore);

    ReactDOM.render( 
      <Provider store={createStoreWithMiddleware(Reducer, 
          window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__()
        )}>
        <App />
      </Provider>
      , document.getElementById('hello')
    );

    // If you want to start measuring performance in your app, pass a function
    // to log results (for example: reportWebVitals(console.log))
    // or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
    reportWebVitals();

    ```
  
- 참고: JohnAnn 님의 리액트 기초 강의
