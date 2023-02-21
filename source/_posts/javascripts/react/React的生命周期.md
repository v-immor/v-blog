---
title: React 的生命周期
date: 2021-03-14 14:16
categories:
    - [JavaScript, React]
tags:
- React
- 原理
---

## 生命周期介绍
组件的生命周期可分成三个状态：

- Mounting：已插入真实 DOM
- Updating：正在被重新渲染
- Unmounting：已移出真实 DOM

生命周期的方法有：

- **componentWillMount** 在渲染前调用,在客户端也在服务端。

- **componentDidMount** : 在第一次渲染后调用，只在客户端。之后组件已经生成了对应的DOM结构，可以通过this.getDOMNode()来进行访问。 如果你想和其他JavaScript框架一起使用，可以在这个方法中调用setTimeout, setInterval或者发送AJAX请求等操作(防止异步操作阻塞UI)。

- **componentWillReceiveProps** 在组件接收到一个新的 prop (更新后)时被调用。这个方法在初始化render时不会被调用。

- **shouldComponentUpdate** 返回一个布尔值。在组件接收到新的props或者state时被调用。在初始化时或者使用forceUpdate时不被调用。
可以在你确认不需要更新组件时使用。

- **componentWillUpdate**在组件接收到新的props或者state但还没有render时被调用。在初始化时不会被调用。

- **componentDidUpdate** 在组件完成更新后立即调用。在初始化时不会被调用。

- **componentWillUnmount**在组件从 DOM 中移除之前立刻被调用。

## 生命周期注意项:
需要注意**componentWillMount、componentWillReceiveProps、componentWillUpdate**这三个生命周期函  数，React官方有准备移除的计划，并且在最新的版本中已经更名为**UNSAFE_XXX **。故，这些函数在开发项目时应避免使用。
移除原因：

1. **componentWillMount：**
> componentWillMount() 在挂载之前被调用。它在 render() 之前调用，因此在此方法中同步调用 setState() 不会触发额外渲染。通常，我们建议使用 constructor() 来初始化 state。
> 避免在此方法中引入任何副作用或订阅。如遇此种情况，请改用 componentDidMount()。
> 此方法是服务端渲染唯一会调用的生命周期函数。

2. **componentWillReceiveProps：**
> componentWillReceiveProps() 会在已挂载的组件接收新的 props 之前被调用。如果你需要更新状态以响应 prop 更改（例如，重置它），你可以比较 this.props 和 nextProps 并在此方法中使用 this.setState() 执行 state 转换。
> **请注意，如果父组件导致组件重新渲染，即使 props 没有更改，也会调用此方法。**如果只想处理更改，请确保      进行当前值与变更值的比较。
> 在挂载过程中，React 不会针对初始 props 调用 componentWillReceiveProps()。组件只会在组件的 props 更    新时调用此方法。调用 this.setState() 通常不会触发 componentWillReceiveProps()。


3. **componentWillUpdate：**
> 当组件收到新的 props 或 state 时，会在渲染之前调用 componentWillUpdate()。使用此作为在更新发生之前执行准备更新的机会。初始渲染不会调用此方法。
> **注意，你不能此方法中调用 this.setState()；**在 componentWillUpdate() 返回之前，你也不应该执行任何其他操作（例如，dispatch Redux 的 action）触发对 React 组件的更新
> 通常，此方法可以替换为 componentDidUpdate()。如果你在此方法中读取 DOM 信息（例如，为了保存滚动位置），则可以将此逻辑移至 getSnapshotBeforeUpdate() 中。


## 官方详细介绍
[https://react.docschina.org/docs/react-component.html](https://react.docschina.org/docs/react-component.html)

## 版本生命周期图
### 16.0前
![](https://cdn.nlark.com/yuque/0/2019/jpeg/412560/1573024477334-cdc75266-1c03-4456-8884-636590eb38e7.jpeg#align=left&display=inline&height=761&originHeight=900&originWidth=740&size=0&status=done&width=626)
### 16.3
![image.png](https://cdn.nlark.com/yuque/0/2019/png/412560/1573024330391-de1ae95f-5765-4fc4-87f3-d65562021717.png#align=left&display=inline&height=673&name=image.png&originHeight=673&originWidth=1175&size=71548&status=done&width=1175)
### 16.4
![image.png](https://cdn.nlark.com/yuque/0/2019/png/412560/1573024364754-4a3fc8cc-4e13-447e-87f0-88db049d1b8c.png#align=left&display=inline&height=665&name=image.png&originHeight=665&originWidth=1169&size=70468&status=done&width=1169)
