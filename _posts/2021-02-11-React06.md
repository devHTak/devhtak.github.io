---
layout: post
title: React 배열 데이터 렌더링 및 관리
summary: React
author: devhtak
date: '2021-02-11 23:41:00 +0900'
category: Frontend
---

#### 배열 데이터 삽입하기

- 자식 컴포넌트가 부모한테 값 전달하기
  - App.js
    - data를 인자로 받는 handlCreate 함수 생성
      ```javascript
      handleCreate = (data) => {
          console.log(data);
      };
      ```
    - 자식 컴포넌트에 onCreate 이벤트로 handleCreate 지정
      ```javascript
      <PhoneForm onCreate={this.handleCreate} />
      ```
  - 자식컴포넌트
    - state로 지정하고자 하는 데이터 생성
      ```javascript
      state= { name: '', phone: ''};
      ```
    - 특정 이벤트에 대하여 props로 넘겨준 onCreate를 넘겨준다.
      ```javascript
      handleSubmit = (e) => {
          e.preventDefault(); // form의 submit했을 때 페이지 리로딩 중지
          this.props.onCreate(this.state);
      }
      ```
    
  - 자식 컴포넌트에서 값을 변경하고 이벤트를 발생시키면, 부모 컴포넌트의 handleCreate가 실행된다.

- 예제
  - 자식 컴포넌트에서 이름과 전화번호를 입력하면 부모 컴포넌트에 특정 배열에 key값과 함께 저장하자.
  - App.js
    ```javascript
    class App extends Component {
        id = 0;
        state = {
            information: [],
        }

        handleCreate = (data) => {
            // this.state.information.push(data);
            // this.setState({
            //   information: this.state.information
            // })
            // 위와 같은 방법은 안된다. - React에서 불변성을 유지해주어야 한다.
            // setState사용, 배열이나 객체를 변경할 때에는 새로운 객체, 배열을 만들어야 한다.
            const { information } = this.state;
              this.setState({
                // information: information.concat({
                //   id: this.id++,
                //   ...data,
                // })
                information: information.concat(Object.assign({}, data, {id: this.id++}))
            })
        }
        render() {
            return (
                <div className="App">
                    <PhoneForm onCreate={this.handleCreate} />
                    {JSON.stringify(this.state.information)}
                </div>
            );
        }  
    }
    export default App;
    ```
    - this.setState()와 배열에서 this.states.information.concat()을 사용한 이유?
      - React는 불변성을 유지해야 한다.
      - this.state.information을 직접 넣어주어도 변화를 감지하지 못한다. 값을 변경할 때에는 항상 this.setState()!
      - 기존의 객체 또는 배열을 사용하지 않고 그것을 기반으로 새로운 객체, 배열을 만들어 주입해야 한다.
    - 내부 데이터
      - spread를 활용하여 데이터를 편리하게 넣을 수 있다.
      - Object.assign으로 새롭게 할당한 객체를 넘겨주어도 된다.
          
#### 배열 렌더링하기

- JavaScript 배열 내장함수: Map

  - map 함수 활용
    - map() 메서드는 배열 내의 모든 요소 각각에 대하여 주어진 함수를 호출한 결과를 모아 새로운 배열을 반환합니다.
    - arr.map(callback(currentValue[, index[, array]])[, thisArg])
      - callback: 새로운 배열 요소를 생성하는 함수. 다음 세 가지 인수를 가집니다.
        - currentValue: 처리할 현재 요소
        - index(Optional): 처리할 현재 요소의 인덱스
        - array(Optional): map()을 호출한 배열
        - thisArg(Optional): callback을 실행할 때 this로 사용되는 값
      - 반환값: 배열의 각 요소에 대해 실행한 callback의 결과를 모은 새로운 배열.
      
    ```javascript
    const numbers = [1, 2, 3, 4, 5];
    const squared = numbers.map(n => n * n );
    console.log(squared); // [1, 4, 9, 16, 25]
    ```
    
  - information으로 PhoneInfoList를 만들어 보자
    - PhoneInfo.js
      ```javascript
      class PhoneInfo extends Component {
          render(){
              const {name, phone, id} = this.props.info;
              const style={
                  border: '1px solid black',
                  padding: '8px',
                  margin: '8px'
              }
              return (
                  <div style={style}>
                      <div><b>{name}</b></div>
                      <div><b>{phone}</b></div>
                      <div><b>{id}</b></div>
                  </div>
              )
          }
      }
      export default PhoneInfo;
      ```
      - information이 비어있을 수 있으니 분기 처리 필요 
    - PhoneInfoList.js
      ```javascript
      class PhoneInfoList extends Component {
          static defaultProps = {
              data: [],
          }
          render() {
              const {data} = this.props;
              const list = data.map(
                  info => (<PhoneInfo info={info} key={info.id} />)
              )
              return (
                  <div>
                      {list}
                  </div>
              );
          };
      }
      export default PhoneInfoList;
      ```
      - key를 입력해주어야 배열을 rendering할 때 성능을 보장할 수 있다.
      - data를 static defaultProps 로 초기화하지 않으면 빈 배열이 넘어올 때 오류가 발생할 수 있다.
    - App.js
      - props로 PhoneForm으로 입력받은 
      ```javascript
      return (
          <div className="App">
              <PhoneForm onCreate={this.handleCreate} />
              <PhoneInfoList data={this.state.information} />
          </div>
      );
      ```     
    
  - 배열 렌더링에 사용되는 key값
    - 리액트에서 key가 없다면 배열 내에 CRUD가 발생할 때 많은 부분의 배열이 다시 랜더링 된다.
      - [1, 2, 3]
        ```html
        <div>1</div>
        <div>2</div>
        <div>3</div>        
        ```
      - [1, 2, 4, 3] : 배열에 4를 추가했다
        ```html
        <div>1</div>
        <div>2</div>
        <div>4</div>
        <div>3</div>        
        ```
        - 결과는 같으나 3이 있던 부분에서 4로 변하고, <div>3</div>가 추가된다.
    - 만약 key가 있다면, 인덱스 있는 부분만 추가하거나 삭제하고, 다른 값에 영향을 주지 않는다.
      - [ {key:0, value:1}, {key: 1, value: 2}, {key: 2, value: 3}]
        ```html
        <div key={0}>1</div>
        <div key={1}>2</div>
        <div key={2}>3</div>        
        ```

#### 배열에서 데이터 제거

- array.prototype.slice
  - array.slice(s, e); : s에서 e까지 포함된 배열을 가져온다.
    ```javascript
    const numbers= [ 1, 2, 3, 4, 5];
    numbers.slice(1, 3); // [2, 3]
    numbers.slice(0,2).concat(numbers.slice(3,5)); // [1, 2, 4, 5]
    [...numbers.slice(0, 2), 10, numbers.slice(3, 5)]; // [1, 2, 10, 4, 5]
    ```
- array.prototype.filter를 사용
  - callback을 넣어서 조건에 맞는 배열을 가져올 수 있다.
    ```javascript
    const numbers= [ 1, 2, 3, 4, 5];
    numbers.filter(n => n > 3); // [4, 5]
    ```

- 제거 예제
  - App.js
    - handler를 생성하여 props로 전달
    ```javascript
    handleRemove = (id) => {
      const {information} = this.state;
      this.setState({
        information: information.filter(info => info.id !== id)
      });
    }
    ```
    ```javascript
    <PhoneInfoList data={this.state.information} 
          onRemove={this.handleRemove}/>
    ```
  - PhoneInfoList.js
    - 그대로 PhoneInfo에게 props 전달
      ```javascript
      const list = data.map(
          info => (<PhoneInfo 
                      info={info} 
                      key={info.id}
                      onRemove={this.props.onRemove}/>)
      )
      ```
  - PhoneInfo.js
    - 버튼을 생성하여 props로 받은 onRemove 호출
    ```javascript
    class PhoneInfo extends Component {
        handleRemove = () => {
            const {info, onRemove} = this.props;
            onRemove(info.id);
        }
        render(){
            const {name, phone} = this.props.info;

            const style={
                border: '1px solid black',
                padding: '8px',
                margin: '8px'
            }

            return (
                <div style={style}>
                    <div><b>{name}</b></div>
                    <div><b>{phone}</b></div>
                    <button onClick={this.handleRemove}>삭제</button>
                </div>
            )
        }
    }
    ```

#### 배열에서 데이터 수정

- slice, map 활용
  - numbers 배열에 3을 9로 수정하라
    ```javascript
    const numbers = [1, 2, 3, 4, 5];
    // slice활용
    [
        ...numbers.slice(0, 2), 
        9, 
        ...numbers.slice(3, 5)
    ];
    // map 활용
    numbers.map(n => {
        if(n === 3)
          return 9;
        return n;
    });
    ```
    
- 예제
  - PhoneInfo에 수정버튼을 클릭하여 수정할 수 있도록 하자
  - App.js
    - onUpdate 핸들러를 props로 자식 컴포넌트에 전달
    ```javascript
    handleUpdate = (id, data) => {
      const {information} = this.state;
      this.setState({
        information: information.map(info => {
          if(info.id === id)
            return {id: id, ...data};
          return info;
        })
      });
    }
    ```
    ```javascript
    <PhoneInfoList data={this.state.information} 
          onRemove={this.handleRemove}
          onUpdate={this.handleUpdate}/>
    ```
  - PhoneInfoList.js
    - 그대로 PhoneInfo.js로 전달
    ```javascript
    const list = data.map(
        info => (<PhoneInfo 
                    info={info} 
                    key={info.id}
                    onRemove={this.props.onRemove}
                    onUpdate={this.props.onUpdate}/>)
    )
    ```
  - PhoneInfo.js
    - 수정 버튼 클릭하면 editing값을 토글하여 input 태그를 보여주고, 적용하면 값이 넘어가도록 수정하자
    ```javascript
    state = {
        editing: false,
        name: '',
        phone: '',
    }
    handleToggleEdit = () => {
        // true->false: updating
        const {info, onUpdate} = this.props;
        if(this.state.editing) {
            onUpdate(info.id, {
                name: this.state.name,
                phone: this.state.phone
            });
        } else {
            this.setState({
                name: info.name,
                phone: info.phone
            })
        }
        // false-> true: input 값을 state에 넣어주기
        this.setState({
            editing: !this.state.editing
        })
    }
    handleChange = (e) => {
        this.setState({
            [e.target.name]: e.target.value
        })
    }

    render(){
        const {name, phone} = this.props.info;
        const {editing} = this.state;

        const style={
            border: '1px solid black',
            padding: '8px',
            margin: '8px'
        }

        return (
            <div style={style}>
                {
                    editing ? (
                        <div>
                            <div><input 
                                onChange={this.handleChange} 
                                name="name" 
                                value={this.state.name}/>
                            </div>
                            <div><input 
                                onChange={this.handleChange} 
                                name="phone" 
                                value={this.state.phone}/>
                            </div>
                        </div>
                    ) : (
                        <Fragment>
                            <div><b>{name}</b></div>
                            <div><b>{phone}</b></div>
                        </Fragment>
                    )
                }                
                <button onClick={this.handleRemove}>삭제</button>
                <button onClick={this.handleToggleEdit}>{
                    editing ? '적용' : '수정' 
                }</button>
            </div>
        )
    }
    ```
- 출처: velopert님의 누구든지 하는 리액트: 초심자를 위한 react 핵심 강좌
