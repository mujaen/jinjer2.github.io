---
layout: post
title: "React Router Dom으로 페이지 전환 처리하기"
description: "페이지 전환 처리"
category: blog
tags: react
---

<!--more-->

* this unordered seed list will be replaced by the toc
{:toc}

## Intro

이번 포스팅에서는 React 16.8에 새로 추가된 Hook의 종류와 사용규칙을 알아보도록 하겠습니다!  
Hook은 기존의 class 바탕의 코드를 작성하지 않고 상태 값과 여러 React 기능을 사용할 수 있도록 고안되었죠    
함수형 컴포넌트에 라이프사이클(Hook)을 적용하면서 함수가 어떻게 작동하는지 하나씩 알아봅시다.       

## Hooks

### 1. useState

'useState'는 state변수와 해당 변수를 갱신할 수 있는 두 개의 쌍을 반환하며 인자로 초기 값을 넘겨주게 됩니다.     
그럼 state 변수를 하나 선언하도록 해보죠     

```javascript
import React, { useState } from "react";

const [userState, setUserState] = useState({
    _id: '',
    name: '',
    auth: false
});

return (
    <div>
        <p>{userState.name}</p>
        <button onClick={() => setUserState({
            _id: '1',
            name: 'jinjer',
            auth: true
        })}>Click</button>
    </div>
);
```

> 'useState' Hook을 사용해서 userState, setUserState 두 개의 쌍을 이용하여 초기 값을 갱신할 수 있습니다.



### 2. useEffect

'useState' 만큼 자주 사용하는 Hook이죠 'useEffect'를 사용하면 함수 컴포넌트에서 사이드 이펙트를 수행할 수 있습니다.  
버튼을 클릭하면 state변수의 값을 변경해주는 예시를 보면서 어떻게 동작되는지 알아보도록 하죠  

```javascript
import React, { useEffect } from "react";

const App = () => {
    const [count, setCount] = useState(0);
    const [state, setState] = useState(0);

    useEffect(() => {
        setState(() => state + 1);
    }, [count]);

    return (
        <div>
            <p>{state}</p>
            <button onClick={() => setCount(count + 1)}>Click</button>
        </div>
    );
};
```

> 'useEffect'는 2개의 인자를 가집니다. 첫 번째 인자는 사이드 이펙트를 실행할 화살표 함수이며 두 번째 인자는 의존성 배열입니다.

두 번째 의존성 배열을 선언하지 않는다면 무한루프에 빠질 수 있습니다. 첫 번째 함수가 두 번째 배열의 원소에 따라 의존성을 가지기 때문이죠 보통은 빈 배열을 선언해도 무방하지만 위의 예시처럼 count에 따라 사이드 이펙트가 수행되도록 설정할 수 있습니다

### 3. useContext

React가 처음 알려졌을 당시 props를 하위 컴포넌트에서 사용하려면 전부 인자로 전달하고  
받아서 사용하고 또 다시 넘겨주는 번거로움이 존재했습니다  
이러한 문제점을 해결하기 위해 전역으로 props를 관리할 수 있는 방법을 모색했고 Redux와 Context가 만들어 졌죠
'useContext'는 Context에서 현재 값을 반환 받을 수 있습니다 하위 컴포넌트 어디에서든 단 한번의 선언으로 사용이 가능하게 된거죠  

```javascript
import React, { useContext } from "react";
import ReactDOM from "react-dom";
import {PageContext, PageProvider} from "./context/PageContext";

const App = () => {
    const {page} = useContext(PageContext);
    
    return (
        <div>{page}</div>    
    );
};

ReactDOM.render(
    <PageProvider>
        <App />
    </PageProvider>,
    document.getElementById("app"));
```

> 'PageProvider'를 최상위에서 렌더링해야 context에 설정된 props를 가져올 수 있습니다!

### 4. useRef

'useRef'는 .current 프로퍼티로 전달된 인자로 초기화된 변경 가능한 객체를 반환합니다.       
반환된 객체는 컴포넌트의 전 생애주기를 통해 유지되지만 렌더링에 곧바로 반영되진 않습니다

```javascript
import React, { useRef } from "react";
import ReactDOM from "react-dom";

const App = () => {
    const countRef = useRef(0);

    const clickHandler = () => {
        countRef.current++;
        console.log(`Clicked ${countRef.current} times`);
    };

    return (
        <div>
            <button onClick={clickHandler}>Click</button>
            <p>{countRef.current}</p>
        </div>
    );
};
```


### 5. useSelector, useDispatch

'useSelector'와 'useDispatch'는 React-Redux에서 제공되는 Hooks입니다 useSelector는 저장소 상태를 유일한 인수로  
사용하여 호출합니다. useDispatch는 저장소에서 함수에 대한 참조를 반환합니다. 아래의 예시를 통해 적용하는 방법을 간단히 알아보도록 하죠  

```javascript
import {useSelector, useDispatch} from "react-redux";

const App = () => {
    const dispatch = useDispatch();
    const user = useSelector(store => store.user);

    const signoutHandler = () => {
        dispatch(actions.signout());
    };

    return (
        <React.Fragment>
            <Button onClick={signoutHandler}>
                Signout
            </Button>
        </React.Fragment>
    );
};
```

## 사용 규칙

1. 반복문, 조건문, 중첩되는 함수 내에서 실행하면 안됩니다! 
2. 일반적인 자바스크립트 함수가 아닌 React 함수 내에서만 Hook을 호출해야 합니다
3. 최상위에서만 Hook을 호출해야 합니다

최상위에서만 Hook을 호출해야 한다는게 무엇일까요? 아래의 예시로 알려드리겠습니다!  
React는 렌더된 순서대로 state를 인식합니다 만약 아래 코드와 같이 조건문으로 인하여  
state에 영향을 준다면 조건에 따라 state 순서에 혼동을 줄 수 있습니다 그렇기에 만약 조건문을 사용해야 한다면  
함수 내부에서 사용하는 것이 좋습니다.

```javascript
const App = () => {
    const [name, setName] = useState('Jinjer');
    
    // Good!
    useEffect(() => {
        if(name !== '') {
            setName('Jinjer Kim');   
        }
    }, []);

    // Bad..
    if(name !== '') {
        useEffect(() => {
            setName('Jinjer Kim');
        }, []);
    }
};
```


## Conclusion

오늘 포스팅에서는 React의 Hook을 하나씩 살펴보면서 자주 사용되는 Hook이 어떤 방식으로 동작하는지 알아보았습니다    
기존 class 방식의 생명주기 활용과 함수형으로 개발된 Hook을 쓰면서 직접 경험해보고 때에 따라 사용하는게 좋을 듯 하네요  
단 사용 규칙을 꼭 지켜야 state가 순서대로 안전하게 동작할 수 있으니 유의하세요! 
  
