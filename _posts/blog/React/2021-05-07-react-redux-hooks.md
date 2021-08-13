---
layout: post
title: "React Rudux로 성능 및 코드 최적화 하기"
description: "Express에 Socket.io 연결하기"
category: blog
tags: react
---

<!--more-->

* this unordered seed list will be replaced by the toc
{:toc}

## Install

이번 포스팅에서는 React Rudux를 적용해서 저장소에서 특정 데이터를 추출하고 조작할 겁니다.      
기본적인 환경 설정을 위해 프로젝트 폴더를 하나 만들고 터미널을 열어 아래의 명령어를 입력해 줍니다.  

```shell
npm install react react-dom redux react-redux
or
yarn add react react-dom redux react-redux
```

> React Rudux 설정은 [공식 문서](https://react-redux.js.org/){: target="_blank"}을 참고해 주세요!

## Counter

React 서버가 정상적으로 구동되었다는 전제하에 Number를 Count 하는 동안 저장소에 특정 데이터를 두고 조작해 보면서    
React Redux가 어떤 형태로 데이터를 추출할 것인지 같이 만들면서 보겠습니다  

### 1. App

루트 경로에 'src' 폴더를 하나 생성하고 app.js 파일을 하나 만들어 주세요  
React와 React DOM을 포함하여 아래와 같이 import 합니다     

```javascript
// src/app.js
import React from "react";
import * as ReactDOM from "react-dom";
import * as actions from "./actions/actions";
import { Provider, useDispatch, useSelector} from "React-redux";
import configureStore from "./store/index";
import reducer from "./reducer/index";

const store = configureStore(reducer);

const App = () => {
	const dispatch = useDispatch();
	const count = useSelector(store => store.count);
	
	return (
		<div>
			<span>{count}</span>
			<button onClick={() => dispatch(actions.increment(count))}>+</button>
			<button onClick={() => dispatch(actions.decrement(count))}>-</button>
		</div>
	);
}

ReactDOM.render(
	<Provider store={store}>
		<App/>
	</Provider>,
	document.getElementById('app'));
```

> 'Provider'를 최상위에서 렌더링해야 Hooks가 저장소 인스턴스에 액세스할 수 있습니다!

#### (1) useDispatch

'useDispatch'는 Redux 저장소에서 dispatch 함수에 대한 참조를 반환할 수 있습니다.      
아래의 dispatch 함수는 actions의 설정된 타입을 참조해서 Reducer로 보내게 됩니다

```javascript
const dispatch = useDispatch();

<button onClick={() => dispatch(actions.increment(count))}>+</button>
<button onClick={() => dispatch(actions.decrement(count))}>-</button>
```


#### (2) useSelector

'useSelector'는 원하는 데이터를 저장소에서 추출할 수 있습니다.  
store에 reducer를 지정하여 count 상태를 조작하기 위해 변수로 설정합니다
 
```javascript
const store = configureStore(reducer);
const count = useSelector(store => store.count);
```

### 2. Store

루트 경로에 'store' 폴더를 하나 생성하고 index.js 파일을 하나 만들어 주세요      
저장소를 만들기 위해 'createStore'를 redux에서 import 합니다.     

```javascript
import {createStore} from "redux";


const configureStore = (reducer) => {
	return createStore(reducer);
};

export default configureStore;
```

> 'configureStore' 함수가 리턴하는 저장소에는 이전에 만든 reducer가 담길 겁니다!  

### 3. Actions

루트 경로에 'actions' 폴더를 하나 생성하고 actions.js 파일을 하나 만들어 주세요      
Count 하기 위한 Increment 와 Decrement 함수를 추가하고 타입을 지정해 줄 겁니다.    

```javascript
// src/actions/actions.js
export const increment = count => ({
	type: 'INCREMENT_COUNT'
});

export const decrement = count => ({
	type: 'DECREMENT_COUNT'
});
```

### 4. Reducer

루트 경로에 'reducer' 폴더를 하나 생성하고 count.js 파일을 하나 만들어 주세요      
action의 타입에 따라 초기 state 값의 상태를 변경해 줄 겁니다  

```javascript
// src/reducer/count.js
const count = (state = 0, action) => {
	switch(action.type){
		case 'INCREMENT':
			return state + 1;
		case 'DECREMENT':
			return state - 1;
		default:
			return state
	}
};

export default count;
```

같은 경로에 index.js 파일을 하나 더 만들어 주세요          
생성한 count.js 파일을 import 하고 'combineReducers'를 사용하여 내보낼 겁니다           

```javascript
// src/reducer/index.js
import {combineReducers} from "redux";
import count from "./count"

export default combineReducers({
	count
});
```

> 눈치가 빠르신분들은 'combineReducers'가 하나가 아닌 여러개의 리듀서 함수를 결합할 수 도 있다는 것을 아시겠죠? 


## Conclusion

서버를 실행해서 정상적으로 Counter 동작이 제대로 수행되는지 확인해 봅니다  
오늘 포스팅에서는 React Redux로 저장소에서 특정 데이터의 상태를 조작해 보    

  
