---
layout: post
title: "Context + Hooks로 사용자 인증하기"
description: "Context로 전역 데이터 관리하기"
category: blog
tags: react
---

<!--more-->

* this unordered seed list will be replaced by the toc
{:toc}

## Install

이번 포스팅에서는 React Context를 적용해서 사용자 인증 처리를 하여 로그인/비로그인 화면을 각각 보여줄 겁니다.       
기본적인 환경 설정을 위해 프로젝트 폴더를 하나 만들고 터미널을 열어 아래의 명령어를 입력해 줍니다.  

```shell
npm install react react-dom
or
yarn add react react-dom
```

> babel도 같이 설치해 줍니다!

## App

React 서버가 정상적으로 구동되었다는 전제하에 Context를 만들어 하위 컴포넌트에서 데이터를 조작할 수 있도록 
파일을 하나씩 만들면서 알아보도록 하죠  

### 1. Context

루트 경로에 'src' 폴더를 하나 생성하고 'context' 폴더를 하나 더 만들어 AuthContext.js 파일을 생성해 주세요          
context를 만들기 위해 'createContext'를 import 합니다.     

```javascript
// src/context/AuthContext.js
import React, { useState, createContext } from "react";

export const AuthContext = createContext();

export const AuthProvider = ({children}) => {
    const [userState, setUserState] = useState({
        _id: '',
        name: '',
        auth: false
    });

    const isAuthenticated = async (token) => {
        return await fetch(apiURL, {
            method: "GET",
            headers: {
                "Accept": "application/json",
                "Authorization": `Token ${token}`
            }
        })
        .then(response => response.json())
        .catch(err => console.log(err))
    };

    return (
        <AuthContext.Provider value={ {userState, setUserState, isAuthenticated} }>
            {children}
        </AuthContext.Provider>
    );
};
```

> 'useState' Hook을 사용해서 사용자의 인증여부에 따라 'auth'의 값을 바꿔줄 겁니다

Provider는 Context를 구독하는 컴포넌트들에게 context의 변화를 알리는 역할을 맡는데요 'AuthContext'에 value로    
useState Hooks로 설정한 userState, setUserState와 isAuthenticated 함수를 하위 컴포넌트로 전달합니다.  

### 2. App

루트 경로에 'src' 폴더를 하나 생성하고 app.js 파일을 하나 만들어 주세요  
React와 React DOM을 포함하여 아래와 같이 import 합니다

```javascript
// src/app.js
import React, {useContext, useEffect} from "react";
import ReactDOM from "react-dom";
import {AuthContext, AuthProvider} from "./context/AuthContext";

const App = () => {
    const {userState, setUserState} = useContext(AuthContext);
    
    return userState.auth ? <div>true</div> : <div>false</div>;
};

ReactDOM.render(
    <AuthProvider>
        <App />
    </AuthProvider>,
    document.getElementById("app"));
```

> 'AuthProvider'를 최상위에서 렌더링해야 context에 설정된 props를 가져올 수 있습니다!

'useContext' Hooks을 사용하여 위에서 만들었던 AuthContext의 현재 값을 반환 받을 수 있습니다  
그럼 이제 fetch를 사용해서 인증을 해보고 userState의 auth값을 변경해 보도록 하죠   

```javascript
// src/app.js
const App = () => {
    const {userState, setUserState, isAuthenticated} = useContext(AuthContext);
    const token = 로그인 후 저장된 토큰

    const authenticationUser = () => {
        isAuthenticated(token)
        .then(data => {
            if(data.error) {
                console.error('Error:', data.error)
            } else {
                setUserState({
                    _id: data.user._id,
                    name: data.user.name,
                    auth: data.user.auth
                });
            }
        });
    };

    useEffect(() => {
        authenticationUser();
    }, []);
    
    return userState.auth ? <div>true</div> : <div>false</div>;
};
```

눈치가 빠르신분들은 단번에 알아채셨을 겁니다 로그인 후에 쿠키나 세션에 저장된 토큰을 context의 'isAuthenticated' 함수를   
통해 인증 받고 결과를 'setUserState'로 변경해주므로 인증여부에 따라 다른 화면이 보이는 것을 확인할 수 있습니다.  

## Conclusion

오늘 포스팅에서는 React Context와 Hooks를 이용해서 간단하게 사용자 인증을 구현해 보았는데요  
전역에서 데이터를 취급하려면 Redux나 Context를 사용해서 데이터를 관리하게 됩니다  
개인적으로 둘 다 같은 의도를 가지고 만들어졌기 때문에 크게 다르다고 생각하진 않아요!   
상황에 따라 자신에게 맞는 방법으로 개발하시면 될 것 같습니다:)
  
