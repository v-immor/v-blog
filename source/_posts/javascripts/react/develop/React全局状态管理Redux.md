---
title: React全局状态管理 - Redux
date: '2022-06-01 11:12'
categories:
  - - React
    - 开发实践
abbrlink: 33bad1c2
---

# Redux组成
1. redux由action、reducer、store三个部分组成
![](https://cdn.nlark.com/yuque/0/2019/png/412560/1575014804335-fb1bce0e-4b5f-4d54-9c28-dfa70aaef82a.png)
2. redux工作流程

![](https://cdn.nlark.com/yuque/0/2019/webp/412560/1575014603014-685c44b6-483f-4e93-ae3a-13dabba7de01.webp#align=left&display=inline&height=368&originHeight=368&originWidth=1240&size=0&status=done&style=none&width=1240)

3. redux分发理解

![](https://cdn.nlark.com/yuque/0/2019/gif/412560/1575014755198-d0be1ffb-ae30-40c7-af49-b359ea99dd2f.gif#align=left&display=inline&height=475&originHeight=475&originWidth=700&size=0&status=done&style=none&width=700)
## action
是把数据从应用传到 store 的有效载荷。它是 store 数据的唯一来源。一般来说，就是通过 store.dispatch() 将 action 传到 store。
action 本质上是 JavaScript 普通对象。我们约定，action 内使用一个字符串类型的 type 字段来表示将要执行的动作。多数情况下，type 会被定义成字符串常量。当应用规模越来越大时，建议使用单独的模块或文件来存放 action。
action:{type:'TODO',payload:{name:'lisi'}}
```javascript
// action 类型
const ADD_TODO = 'ADD_TODO';

// 通过dispatch将action递交到reducer中处理
dispatch({
	type: ADD_TODO,
  payload:{name:'lisi'}
})
```
在具体使用过程中，通常会使用一个函数来生成相应的action对象，使得代码更容易测试和移植

```javascript
// 生成action
function addTodo(text) {
  return {
    type: ADD_TODO,
    text
  };
}
// 通过dispatch将action递交到reducer中处理
dispatch(addTodo(text));
```

## reducer
action 只是描述了有事情发生了这一事实，并没有指明应用如何更新 state。那么该如何去更新state或执行其他动作，这就是reducer需要关注的事情了。
通常一个reducer里存在一个switch，根据action类型来执行不同的state更新操作，同时包涵一个initState,用来初始化当前reducer中的state，其中返回的对象将会覆盖整个state对象，如果不希望变动其他对象，则需要对之前的state进行扩展赋值，只将更新的state进行覆盖
reducer(preSttate,action)
```javascript
import { QUERY, SUBMIT } from "../actions/homeActions";

// 初始state
const initState = {};

// reducer 函数 reducer(preSttate,action)
const home = (state = initState, { type, payload }) => {
  switch (type) {
    case QUERY:
      return { ...state, loading: false, ...payload };
    case SUBMIT:
      return { ...state, loading: false, ...payload };
    default:
      return { ...state, loading: true };
  }
};

export default home;

```

## store
使用 action 来描述“发生了什么”、使用 reducer 来根据 action 更新 state ，那么store就是负责将它们联系到一起的对象。
store的职责：

- 维持应用的 state；
- 提供 getState() 方法获取 state；
- 提供 dispatch(action) 方法更新 state；
- 通过 subscribe(listener) 注册监听器。

**Redux 应用只有一个单一的 store！Redux 应用只有一个单一的 store**！**Redux 应用只有一个单一的 store！**
当需要拆分处理数据的逻辑时，使用 reducer 组合（combineReducers()） 而不是创建多个 store。

```javascript
import { createStore } from "redux";
// 多个reducers处理后的结果
import reducers from "../reducers";

// store通过createStore获取，Redux 应用只有一个单一的 store！
const store = createStore(reducers);

export default store;
```

综上，一个完整的redux应当包含actions、reducers、store三个部分。

# 将Redux使用在React应用中
## 将store挂载到dom中
使用Provider组件即可让整个dom树中的节点都能够通过connect去访问store的内容
```javascript
import React from "react";
import ReactDOM from "react-dom";
import "./global.less";
import App from "./Route";
// 引入Provider
import { Provider } from "react-redux";
import store from "./redux/store";

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById("root")
);
```

## 通过connect去访问store
```javascript
import React, { Component } from "react";
import { connect } from "react-redux";

// 如需使用装饰器模式配置@babel/plugin-proposal-decorators
@connect(({ home }) => {
  return { home };
})
class BaseLayout extends Component {
  componentDidMount() {
    // 可使用dispatch去更新store
    console.log(this.props); // {children,dispatch,home}
  }

  render() {
    const { children } = this.props;
    return (
      <div>
        {children}
      </div>
    );
  }
}

export default BaseLayout;
```

## 通过dispatch去更新store

```javascript
// BaseLayout.js
import React, { Component } from "react";
import { connect } from "react-redux";
import { updateStore } from "../../redux/actions/homeActions";

@connect(({ home }) => {
  return { home };
})
class BaseLayout extends Component {
  componentDidMount() {
    console.log(this.props);
  }

  render() {
    const { children, dispatch, home } = this.props;
    return (
      <div>
        <button
          onClick={() => {
      		// 按钮点击后将会把home的state更为{name:'张三'}，updateStore为生成action的函数
            dispatch(updateStore({ name: "张三" }));
          }}
        >
          更新store
        </button>
        {home.name}
        {children}
      </div>
    );
  }
}

export default BaseLayout;


// homeActions.js (action)
export const QUERY = "QUERY";
export const SUBMIT = "SUBMIT";

export const updateStore = payload => {
  return {
    type: QUERY,
    payload
  };
};


// home.js (reducer)
import { QUERY, SUBMIT } from "../actions/homeActions";

const initState = {};

const home = (state = initState, { type, payload }) => {
  switch (type) {
    case QUERY:
      return { ...state, loading: false, ...payload };
    case SUBMIT:
      return { ...state, loading: false, ...payload };
    default:
      return { ...state, loading: true };
  }
};

export default home;
```

