---
layout: post
title: React 기본
summary: React
author: devhtak
date: '2021-02-11 21:41:00 +0900'
category: Frontend
---

#### React란?

- Frontend Library로 Facebook에서 2013년에 처음 release됐다.
- 특징1. Components
  - module과 비슷하게 컴포넌트로 이뤄져 있어서 reusable이 뛰어나다.
- 특징2. Virtual Dom
  - We built React to solve one problem: building large applications with data that changes over time.
  - Mutation(변화)
    - React: Mutation을 하지 말고, 데이터에 변화가 생기면 뷰를 새로 만들어버리면 어떨까?
    - 하지만 Browser에서 데이터 변화에 대한 새로운 뷰를 만드는 것은 성능에 문제가 발생한다.
    - 그래서 생각한 것이 Virtual DOM
      - 변화가 생기면 DOM에 새로운 것을 추가하는 것이 아닌 JS로 이뤄진 Virtual DOM에 한번 Rendering을 한 후에 기존의 DOM과 비교한 후에 필요한 부분에 update한다.
  - Real DOM
    - 만약 10개의 리스트가 있고, 그 중 한가지의 리스트만 update되었더라도 전체 리스트를 다시 reload 해야된다.
    - super expensive한 작업
  - Virtual DOM    
    - 만약 10개의 리스트가 있고, 그 중 한가지의 리스트만 update되면, 그 바뀐 한가지 아이템만 DOM에서 바꿔준다.
    - JSX을 렌더링한다. 그러면 Virtual DOM이 Update된다.
    - Virtual DOM이 이전 Virtual DOM에서 찍어둔 Snapshot과 비교를 해서 바뀐 부분을 찾는다.
    - 과정을 diffing이라고 부른다.
    - 그 바뀐 부분만 Real DOM에서 바꿔준다.

#### React 시작하기(create-react-app)

- 원래 리액트 앱을 처음 실행하기 위해서 webpack이나 babel같은 것을 설정해야 했다.
  - Babel
    - 최신 자바스크립트 문법을 지원하지 않는 브라우저들을 위해서 최신 자바스크립트 문법을 구형 브라우저에서도 돌수있게 변환
  - Webpack
    ```
    As its core, webpack is static module bundle for modern JavaScript application.
    When webpack processes your applicatoin, in internally builds a dependency graph 
    which maps every module your project needs and generates one or more bundles.
    ```
    - 많은 모듈을 합하여(bundle) 간단하게 해준다.

- 이런 설정을 create-react-app Command로 바로 시작할 수 있다.
  ```
  $ npx create-react-app . // 현재 폴더에 create-react-app 을 실행하도록 한다.
  $ npm install -g create-react-app // 이전에는 이렇게 했다. global 디렉토리에 다운로드
  ```
  - npm vs npx
    - 현재는 npx를 이용하여 npm으로 다운받지 않고 사용가능하다.
    
#### npm 과 npx

- NPM (Node Package Manager)
  - It is an online repository for the publishing of open-source Node.js projects.
  - It is a command-line utility for interacting with the said repository that aids in package installation, version management, and dependency management.
  - package.json에 dependency, build 등이 정의되어 있다.
  
  - npm install Locally
    - Links created at the ./node_modules/.bin directory
    
  - npm install Globally!
    - Links created from the global bin/ directory
    - ex) /usr/local/bin on Linux
    - ex) %AppData%/npm on Windows
  
  - 만약 install하는 NPM을 다른 프로젝트에서 쓰지 않는 다면 Global로 Install할 필요가 없으므로 Disk Space를 낭비하지 않을 수 있다.

- 원래 create-react-app을 할때 npm install -g create-react-app으로 global 디렉토리에 다운받았다.
  - 이제는 npx를 이용하여 그냥 create-react-app을 이용할 수 있다.
  - npx가 npm registry에서 create-react-app을 찾아서(look up) 다운로드 없이 실행 시켜준다.
  - Disk Space를 낭비하지 않을 수 있고, 항상 최신 버전을 사용할 수 있다는 장접이 있다.
  
#### React App 구조(Structure)

- my-app/
  - README.md
  - node_modules/
  - package.json/
  - public/
    - index.html
    - favicon.ico
  - src/
    - App.css
    - App.js
    - App.test.js
    - index.css
    - index.js
    - logo.svg
  
- src 디렉터리 하위는 webpack에서 관리하며, public은 webpack이 관리하지 않는다.
  - 이미지 등은 webpack이 관리할 수 있도록 src 하위에 넣도록 한다.
  
- 처음 시작할 때(npm run start) src 밑에 index.js가 실행되며 index.js는 ReactDOM으로 App.js를 로드한다.
  - npm run start command는 package.json에 설정되어 있다.
  ```
  ReactDOM.render(<App />. document.getElementById('root'))
  ```
    - index.html에 <div id="root"></div> 태그에 App.js에서 return하는 div태그 자식으로 들어가도록 되어 있다.

- 참조: https://create-react-app.dev/docs/folder-structure/

#### boiler-plate 예제

- myApp/src/
  - _actions -> Redux(action)를 위한 폴더 
  - _reducer -> Redux(reducer)를 위한 폴더
  - components
    - views -> page들이 저장되어 있다.
      - Sections -> 해당 페이지에 관련된 css, component를 넣는다.
  - App.js -> Routing 관련 일을 처리한다.
  - Config.js -> 환경 변수같은 것들을 정하는 곳
  - hoc -> Higer Order Component의 약자
  - utils -> 여러 군데에서 쓰일 수 있는 것들을 이곳에 넣어서 어디서든 쓸 수 있도록 해준다.
  
- hoc
  - Concretely, a higher-order component is a function that takes a component and returns a new component.
    ```
    const EnhanceComponent = higerOrderComponent(WrappedComponent);
    ```
  - 예제) Auth라는 HOC가 있고, 하위에 Admin Component가 있다.
    - Auth에서 해당 유저가 해당 페이지에 들어갈 자격이 알아 낸 후에 자격이 된다면 Admin Component에 가게 해주고 아니면 다른 페이지로 보낸다.
    - Admin Component는 Admin 유저만 들어올 수 있다.
    
- 출처: John Ann님의 노드 리액트 기초 강의
