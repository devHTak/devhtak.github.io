---
layout: post
title: React - Back과 Front 구성(Axios와 Proxy)
summary: React
author: devhtak
date: '2021-02-12 10:41:00 +0900'
category: Frontend
---

#### Data Flow

- Database <-> Server <-> Client
  - 로그인을 한다면
    - Client에서 id, pw 정보를 담아 Server로 request를 보낸다.
    - Server는 해당 id, pw 정보를 확인하기 위해 Database에 저장되어 있는 id에 대한 pw를 가져와 확인한다.
    - Server는 확인 여부에 따라 성공 또는 실패 response를 생성하여 Client에게 보낸다.

- Client에서 Server에 보내기 위해 Axios를 사용한다.

#### Axios

- jQuery의 Ajax와 같다
- Promise API를 활용하는 HTTP 비동기 통신 라이브러리
- 특징
  - 운영환경에 따라 브라우저의 XMLHttpRequest 객체 또는 http api 사용
  - Promise(ES6) API 사용
  - 요청과 응답 데이터의 변형
  - HTTP 요청 취소
  - HTTP 요청과 응답을 JSON 형태로 자동 변경  
- dependency download
  ```
  $ npm install axios --save
  ```
- 사용방법
  - GET
    - 입력한 url에 존재하는 자원에 요청을 합니다.
    ```
    axios.get(url,[,config])	
    ```
  - POST
    - 새로운 리소스를 생성(create)할 때 사용합니다.
    ```
    axios.post("url주소",{data객체},[,config])
    ```
  - DELETE
    - REST 기반 API 프로그램에서 데이터베이스에 저장되어 있는 내용을 삭제하는 목적으로 사용합니다.
    ```
    axios.delete(URL,[,config]);
    ```
  - PUT
    - REST 기반 API 프로그램에서 데이터베이스에 저장되어 있는 내용을 갱신하는 목적으로 사용됩니다.
    ```
    axios.put(url[, data[, config]])
    ```
  - method chaining
    ```javascript
    axios.post('/user', {
        firstName: 'Fred',
        lastName: 'Flintstone'
    })
    .then(function (response) {
        console.log(response);
    })
    .catch(function (error) {
        console.log(error);
    });
    ```
    - then()은 요청에 대한 응답이 정상적으로 왔을 경우 실행하도록 되어 있다.
    - catch()는 요청에서 오류가 발생할 경우 실행하도록 되어 있다.
    - finally()는 try-catch 문에서 finally처럼, then 또는 catch가 완료되는 시점에서 실행되도록 구성되어 있다.

- Request Config
  ```
  {
    // `url` is the server URL that will be used for the request
    url: '/user',

    // `method` is the request method to be used when making the request
    method: 'get', // default

    // `baseURL` will be prepended to `url` unless `url` is absolute.
    // It can be convenient to set `baseURL` for an instance of axios to pass relative URLs
    // to methods of that instance.
    baseURL: 'https://some-domain.com/api/',

    // `transformRequest` allows changes to the request data before it is sent to the server
    // This is only applicable for request methods 'PUT', 'POST', 'PATCH' and 'DELETE'
    // The last function in the array must return a string or an instance of Buffer, ArrayBuffer,
    // FormData or Stream
    // You may modify the headers object.
    transformRequest: [function (data, headers) {
      // Do whatever you want to transform the data

      return data;
    }],

    // `transformResponse` allows changes to the response data to be made before
    // it is passed to then/catch
    transformResponse: [function (data) {
      // Do whatever you want to transform the data

      return data;
    }],

    // `headers` are custom headers to be sent
    headers: {'X-Requested-With': 'XMLHttpRequest'},

    // `params` are the URL parameters to be sent with the request
    // Must be a plain object or a URLSearchParams object
    params: {
      ID: 12345
    },

    // `paramsSerializer` is an optional function in charge of serializing `params`
    // (e.g. https://www.npmjs.com/package/qs, http://api.jquery.com/jquery.param/)
    paramsSerializer: function (params) {
      return Qs.stringify(params, {arrayFormat: 'brackets'})
    },

    // `data` is the data to be sent as the request body
    // Only applicable for request methods 'PUT', 'POST', 'DELETE , and 'PATCH'
    // When no `transformRequest` is set, must be of one of the following types:
    // - string, plain object, ArrayBuffer, ArrayBufferView, URLSearchParams
    // - Browser only: FormData, File, Blob
    // - Node only: Stream, Buffer
    data: {
      firstName: 'Fred'
    },

    // syntax alternative to send data into the body
    // method post
    // only the value is sent, not the key
    data: 'Country=Brasil&City=Belo Horizonte',

    // `timeout` specifies the number of milliseconds before the request times out.
    // If the request takes longer than `timeout`, the request will be aborted.
    timeout: 1000, // default is `0` (no timeout)

    // `withCredentials` indicates whether or not cross-site Access-Control requests
    // should be made using credentials
    withCredentials: false, // default

    // `adapter` allows custom handling of requests which makes testing easier.
    // Return a promise and supply a valid response (see lib/adapters/README.md).
    adapter: function (config) {
      /* ... */
    },

    // `auth` indicates that HTTP Basic auth should be used, and supplies credentials.
    // This will set an `Authorization` header, overwriting any existing
    // `Authorization` custom headers you have set using `headers`.
    // Please note that only HTTP Basic auth is configurable through this parameter.
    // For Bearer tokens and such, use `Authorization` custom headers instead.
    auth: {
      username: 'janedoe',
      password: 's00pers3cret'
    },

    // `responseType` indicates the type of data that the server will respond with
    // options are: 'arraybuffer', 'document', 'json', 'text', 'stream'
    //   browser only: 'blob'
    responseType: 'json', // default

    // `responseEncoding` indicates encoding to use for decoding responses (Node.js only)
    // Note: Ignored for `responseType` of 'stream' or client-side requests
    responseEncoding: 'utf8', // default

    // `xsrfCookieName` is the name of the cookie to use as a value for xsrf token
    xsrfCookieName: 'XSRF-TOKEN', // default

    // `xsrfHeaderName` is the name of the http header that carries the xsrf token value
    xsrfHeaderName: 'X-XSRF-TOKEN', // default

    // `onUploadProgress` allows handling of progress events for uploads
    // browser only
    onUploadProgress: function (progressEvent) {
      // Do whatever you want with the native progress event
    },

    // `onDownloadProgress` allows handling of progress events for downloads
    // browser only
    onDownloadProgress: function (progressEvent) {
      // Do whatever you want with the native progress event
    },

    // `maxContentLength` defines the max size of the http response content in bytes allowed in node.js
    maxContentLength: 2000,

    // `maxBodyLength` (Node only option) defines the max size of the http request content in bytes allowed
    maxBodyLength: 2000,

    // `validateStatus` defines whether to resolve or reject the promise for a given
    // HTTP response status code. If `validateStatus` returns `true` (or is set to `null`
    // or `undefined`), the promise will be resolved; otherwise, the promise will be
    // rejected.
    validateStatus: function (status) {
      return status >= 200 && status < 300; // default
    },

    // `maxRedirects` defines the maximum number of redirects to follow in node.js.
    // If set to 0, no redirects will be followed.
    maxRedirects: 5, // default

    // `socketPath` defines a UNIX Socket to be used in node.js.
    // e.g. '/var/run/docker.sock' to send requests to the docker daemon.
    // Only either `socketPath` or `proxy` can be specified.
    // If both are specified, `socketPath` is used.
    socketPath: null, // default

    // `httpAgent` and `httpsAgent` define a custom agent to be used when performing http
    // and https requests, respectively, in node.js. This allows options to be added like
    // `keepAlive` that are not enabled by default.
    httpAgent: new http.Agent({ keepAlive: true }),
    httpsAgent: new https.Agent({ keepAlive: true }),

    // `proxy` defines the hostname, port, and protocol of the proxy server.
    // You can also define your proxy using the conventional `http_proxy` and
    // `https_proxy` environment variables. If you are using environment variables
    // for your proxy configuration, you can also define a `no_proxy` environment
    // variable as a comma-separated list of domains that should not be proxied.
    // Use `false` to disable proxies, ignoring environment variables.
    // `auth` indicates that HTTP Basic auth should be used to connect to the proxy, and
    // supplies credentials.
    // This will set an `Proxy-Authorization` header, overwriting any existing
    // `Proxy-Authorization` custom headers you have set using `headers`.
    // If the proxy server uses HTTPS, then you must set the protocol to `https`. 
    proxy: {
      protocol: 'https',
      host: '127.0.0.1',
      port: 9000,
      auth: {
        username: 'mikeymike',
        password: 'rapunz3l'
      }
    },

    // `cancelToken` specifies a cancel token that can be used to cancel the request
    // (see Cancellation section below for details)
    cancelToken: new CancelToken(function (cancel) {
    }),

    // `decompress` indicates whether or not the response body should be decompressed 
    // automatically. If set to `true` will also remove the 'content-encoding' header 
    // from the responses objects of all decompressed responses
    // - Node only (XHR cannot turn off decompression)
    decompress: true // default

  }
  ```
- Response Schema
  - then으로 받을 때 response를 사용할 수 있다.
  ```
  {
    // `data` is the response that was provided by the server
    data: {},

    // `status` is the HTTP status code from the server response
    status: 200,

    // `statusText` is the HTTP status message from the server response
    statusText: 'OK',

    // `headers` the HTTP headers that the server responded with
    // All header names are lower cased and can be accessed using the bracket notation.
    // Example: `response.headers['content-type']`
    headers: {},

    // `config` is the config that was provided to `axios` for the request
    config: {},

    // `request` is the request that generated this response
    // It is the last ClientRequest instance in node.js (in redirects)
    // and an XMLHttpRequest instance in the browser
    request: {}
  }
  ```
  
#### COSR 이슈, Proxy 설정

- CORS
  - Cross Origin Resource Sharing(CORS)
    - 보안을 목적으로 서로 다른 origin에서 자원을 공유할 때 오류가 발생하도록 한다.
  - origin?
    - 비슷한 개념으로 도메인이 있는데, 오리진은 프로토콜://도메인:port 까지 포함한다.
    - 도메인(domain): naver.com
    - 오리진(origin): https://www.naver.com:port/
  - Server와 Client에 Port 번호 등에 origin이 다른 경우 CORS 에러가 발생한다.
    - Server: localhost:5000, Client: localhost:3000 이면 오류가 발생한다.

- CORS 오류 해결하기
  - proxy를 활용한 해결방법 
    - Configuring the Proxy Manually 
    - https://create-react-app.dev/docs/proxying-api-requests-in-development
    - http-proxy-middleware 설치
      ```
      $ npm install http-proxy-middleware --save
      ```
    - src/setupProxy.js 생성
      ```javascript
      const {createProxyMiddleware} = require('http-proxy-middleware')
      module.exports = function(app) {
          app.use(
              '/api',
              createProxyMiddleware({
                  target: 'http://localhost:5000',
                  changeOrigin: true
              })
          );
      };
      ```
  
#### Proxy Server

- HTTP 완벽 가이드 참고
- Client <-> Proxy <-> Server
  - IP를 Proxy Server에서 임의로 바꿔 버릴 수 있다. 그래서 인터넷에서는 접근하는 사람의 IP를 모르게 한다.
  - 보내는 데이터도 임의로 변경할 수 있다.
  
- Proxy Server 사용 이유!
  - 방화벽 기능, 웹 필터 기능
    - 회사에서 직원들이나 집안에서 아이들 인터넷 사용 제어
    - 더 나은 보안 제공
    - 이용 제한된 사이트 접근 기능
  - 캐쉬 데이터, 공유 데이터 제공
    - 캐쉬를 이용해서 더 빠른 인터넷 이용 제공

#### Concurrently를 이용해 프론트, 백 한번에 켜기

- Concurrently란?  
  - 여러개의 commands를 동시엑 작동시킬 수 있게 해주는 Tool
  
- Usage
  - dependency download
    ```
    $ npm install concurrently --save
    ```
  - Remember to surround seperate commands with quotes:
    ```
    concurrently "command1 arg" "command2 arg"
    ```
  - Otherwise concurrently would try to run 4 seperate commands: command1, arg, command2, arg.
    - in package.json, escape quotes:
    ```
    "start": "concurrently \"command1 arg\" \"command2 arg\""
    ```
- 예제
  ```
  "scripts": {
    "start": "node ./server/index.js",
    "backend": "nodemon ./server/index.js",
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "concurrently \"npm run backend\" \"npm run start --prefix client\""
  },
  ```
  - 여기서 dev에 적용했다.
  - npm run backend로 node.js를 올리고, npm run start --prefix client로 client 디렉토리에 있는 package.json의 npm run start를 실행

- 참고: JohnAnn 님의 리액트 기초 강의
