---
layout: post
title: React - Router
summary: React
author: devhtak
date: '2021-02-19 11:41:00 +0900'
category: Frontend
---

#### React Router

- 페이지 이동을 할 때 React Router DOM을 사용한다.
- 어떻게 사용하는가?
  - https://reacttraining.com/react-router/web/example/basic
- dependency donwload
  ```
  $ npm install react-router-dom --save
  ```
- 예제
  ```javascript
  import React from 'react';
  import {
      BrowserRouter as Router,
      Switch,
      Route,
      Link
  } from 'react-router-dom';


  function App() {
      return (
          <div className="App">
              <Router>
                  <div>
                      <ul>
                          <li><Link to="/">HOME</Link></li>
                          <li><Link to="/about">ABOUT</Link></li>
                          <li><Link to="/dashboard">DASHBOARD</Link></li>
                      </ul>
                      <hr />
                      <Switch>
                          <Route exact path="/" component={Home} />
                          <Route exact path="/about"><About /></Route>
                          <Route exact path="/dashboard"><Dashboard /></Route>
                      </Switch>
                  </div>
              </Router>
          </div>
      );
  }

  function Home() {
      return (
          <div>
              <h2>Home</h2>
          </div>
      );
  }

  function About() {
      return (
          <div>
              <h2>About</h2>
          </div>
      );
  }

  function Dashboard() {
      return (
          <div>
              <h2>Dashboard</h2>
          </div>
      );
  }
  export default App;
  ```
  
- 참고: John Ann님의 React 기본 강의
