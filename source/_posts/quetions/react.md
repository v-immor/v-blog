---
title: React
date: '2023-02-23 22:07'
quiz: true
hide: true
categories:
  - Quetions
tags:
  - 面试
abbrlink: 4a322b5b
---

## React 原理

1.  `React` 是什么？ {.quiz .fill}

    > 1. `React` 是一个构建用户界面的 JS 库，具有`声明式编程`、`组件化`、`跨平台`的特点。
    > 2. 声明式编程可以让开发者降低操作 DOM 的负担，组件化利于视图层的开发复用。
    > 3. `React` 通过 `虚拟 DOM` 来描述真实 DOM 的结构，所以 `React` 具有跨平台的能力。

2.  `React` 的生命周期 {.quiz .fill}

    > - `construtor` -> `getDerivedStateFromProps` -> `shouldComponentUpdate` -> `render` -> `getSnapshotBeforeUpdate` -> `componentDidMount` -> `componentDidUpdate` -> `componentWillUnMount`
    > - 挂载时：`construtor` -> `getDerivedStateFromProps` -> `render` -> `componentDidMount`
    > - 更新时：`getDerivedStateFromProps` -> `shouldComponentUpdate` -> `render` -> `getSnapshotBeforeUpdate` -> `componentDidUpdate`
    > - 卸载时：`componentWillUnMount`
    > - 从整个运行阶段来看，可分为 `Render` 阶段和 `Commit` 阶段

3.  `react` 和 `react-dom` 的区别 {.quiz .fill}

    > `react` 包中主要是导出应用开发所需要的函数，`react-dom` 是 `react` 的 web 平台的渲染器，正常应用开发只需要导入 `render` 函数即可

4.  描述一下 `React 的 Fiber 架构` {.quiz .fill}

    > - `Fiber 架构`是 React 团队主要是为了解决旧架构中同步更新不可中断导致阻塞渲染进程的问题提出的架构，它主要是重构 React 架构中的 `Reconcile` 层，并且增加了 `Schedule` 层来做任务的调度。
    > - 在 Fiber 的运行机制里，Fiber 能够让 React 将同步更新中断，把主线程的控制权交回给浏览器控制，从而达到不阻塞浏览器渲染的目的；
    >   React 中的 Fiber 是一个`多环的链表结构`，链表的结构让 Fiber 在遍历中实现中断和恢复，通过`子节点`、`父节点`、`同级节点`的三个指针，来描述 DOM 的树结构。
    > - 从 React 的运行逻辑来看，React 大体可以分为两个阶段： 1. 进行 `DIFF`，`任务调度`的 `Render` 阶段。 2. 将 `Render` 阶段产出的 `Fiber` 渲染到页面的 `Commit` 阶段。
    > - 在 **Render** 阶段又分成 `Reconcile` 协调部分和 `Schedule` 任务调度部分。
    >
    > * `Reconcile` 主要是将 JSX 产出的 DOM 结构和当前 Fiber 结构进行深度优先遍历，判断是否要新增、删除、移动节点，并打上对应的操作标记 `effectTag`。
    > * `Schedule` 则是用来控制主线程让出机制的。通过判断在一帧中还剩余的时间是否足够执行一个宏任务，来决定是否进入到下一个更新工作或者让出主线程，等待主线程的下一次的空闲。
    >   > 当有 Fiber 发生更新时，`Reconcile` 和 `Schedule` 会穿插执行，`Reconcile` 每一次循环前会通过 `Schedule` 的 `shouldYield()` 函数判断剩余时间是否足够执行任务，若足够就继续进行遍历工作，若不够就让出主线程进入下一个事件循环有足够时间时再执行任务。
    >
    > - **Commit** 阶段主要的工作是将 `Render` 产出的有 `effectTag` 标记的 Fiber 根据不同的操作标记执行不同的 DOM 操作，并将 Fiber 上的所有的`副作用函数`进行处理。

5.  描述一下 `React Diff` 的原理 {.quiz .fill}

    > - React 的 Diff 使用深度优先遍历算法，从上到下，从左到右进行递归遍历找出可复用的节点。
    > - 和原本的 Diff 算法不同的地方是，React 设定了三种限制，将 Diff 算法的复杂度从 `O(n^3)` 降低至 `O(n)` 。
    > - 这三种限制分别是
    >   1. 只对同级元素进行遍历，如果元素发生了跨级变动，则不会复用该元素。
    >   2. 如果元素的类型发生变化，则直接销毁当前元素及其子孙元素，并新建元素及子孙元素。
    >   3. 开发者可以通过 Key 让元素在渲染的时候保持稳定复用。
    > - 根据上面三种限制，React 会对 JSX 语法产生的 DOM 结构和当前渲染的 Fiber 结构经过两次遍历来完成 `fiber` 的更新。
    > - 遍历的逻辑
    >   1. 一一对比 `vdom` 和 `fiber`，如果可以复用则处理下一个节点的对比，否则结束遍历。
    >      - 如果所有的 `vdom` 遍历完了，那就把 `fiber` 剩余的节点删掉。
    >      - 如果 `vdom` 有剩余，那就进行第二次遍历。
    >   2. 第二次遍历就是将剩余的 `fiber` 放到 `map` 里，继续遍历剩下的 `vdom`。
    >      - 如果可以复用就进行移动
    >      - 第二次遍历完后，将剩余的 `vdom` 进行新增、将剩余的 `fiber` 进行删除。

6.  React 的`合成事件`是什么？ {.quiz .fill}

    > - `React 合成事件` 是 React 为了抹平各平台`事件 API`的差异，根据 W3C 的`事件机制规范`实现的抽象跨平台事件机制。
    > - 除了能够磨平不同平台的差异达到跨平台的能力外，还能够通过事件委托机制优化了内存开销，并且能够干预事件的分发来提升用户的体验。
    > - 它和原生事件不同的主要区别是
    >   1.  命名方式不同，原生事件是全小写，合成事件是小驼峰
    >   2.  原生事件可以指定绑定在具体元素上，合成事件是统一绑定在 `Root 元素` 上，在具体执行时才进行派发执行。
    >   3.  原生事件组织冒泡的形式可以通过 `e.stopPropogation()` 或者 `return false`，合成事件只能通过 `e.stopPropogation()`

7.  `React` 什么时候挂载的合成事件 {.quiz .fill}

8.  `React V17` 中为什么要移除事件池？ {.quiz .fill}

    > - 在旧版中，`事件池`是为了复用不同事件的事件对象，用来提升性能，并且在所有的事件对象在真正使用前都置为 `null`，所以在旧版中，要在异步任务里访问到正确的事件对象就必须先执行 `e.persist()`。
    > - 在 17 的版本里，删除了事件池的优化操作，开发者就可以按照预期访问到正确的事件对象。
    > - 为了保持兼容，17 版本的 `e.persist()` 也可以继续使用，不过没有效果而已。

9.  多次调用 `click` 事件，在 document 上事件监听会执行几次？ {.quiz .fill}

    > - 执行一次。因为 React 在注册监听函数不是单一的回调函数，而是 `dispatchEvent` 分发函数，所以即使是调用了多次也只会注册一次监听函数

10. React 的合成事件主要做了什么？ {.quiz .fill}

    > 1. 事件绑定（注册、监听）
    > 2. 事件触发（合成、分发）

11. React Hooks 的原理 {.quiz .fill}

    > - hooks 是 React 为了使函数组件也能够使用 React 的 state 或其他特性而设计的`钩子函数`。
    > - hook 在 Fiber 中存放于 `memoizedState` 属性中，是一个链表结构，通过 `next` 指针指向下一个 hook。
    > - 如果 `memoizedState` 发生变化，会创建一个 `Update` 对象到 `updateQueue` 里。

12. React 是如何处理副作用的 {.quiz .fill}

13. React `class 组件` 和 `Hook 组件` 的区别 {.quiz .fill}

    > - 从语法来看，`class` 需要继承 `React.Component`，`Hook` 则不需要。
    > - 从代码执行角度来看，`class` 的运行逻辑没有 `Hook` 清晰，`class` 通过生命周期钩子实现，需要了解生命周期钩子的作用及含义，还需要关注 `this` 指针问题。`Hook` 则是纯函数实现方式，不需要关注生命周期的执行，不需要考虑 `this`。
    > - 从 `React` 的设计理念来看，`Hook` 更加符合 `UI=Fn(data)` 的理念。
    > - 从 `React` 运行逻辑来看，`class` 需要进行实例化，`Hook` 运行完便可回收。`Hook` 在内存上会更加节省一些。
    > - 从 `React` 的特性来看，`class` 组件使用 `state` 只需要定义在 `this` 上即可使用，`Hook` 则需要通过 `useState` 钩子来实现状态。
    > - **函数式组件天然可以捕获渲染时的所使用的值**。

14. setState 是同步的还是异步的 {.quiz .fill}

    > - 没有启用 `Fiber` 架构时，在合成事件和生命周期钩子中是异步的，在 settimeout 、原生事件中是同步的。
    > - 启用 `Fiber` 架构后都是异步更新的。

15. setState 的批量更新和 useState 的批量更新如何实现 {.quiz .fill}

16. React 如何更新视图？(`dispatch` 如何实现视图更新) {.quiz .fill}

17. `useCallback` 和 `useMemo` 的区别 {.quiz .fill}

    > - `useCallback` 和 `useMemo` 都是用于缓存的钩子，让函数和值具有稳定性。
    > - 两个钩子第一个参数都是回调函数，第二个参数是一个依赖对象的数组，`useMemo` 的回调函数会在 render 函数执行时同步执行并返回值，`useCallback` 则是将回调函数进行缓存，等待调用。第二个参数就是决定回调函数是否更新的依赖对象数组，只要数组内的对象发生更新，回调函数就会更新。
    > - `useMemo` 常用于缓存复杂的计算结果
    > - `useCallback` 则是用于需要稳定函数的处理，比如`防抖`、`节流`的回调函数，滚动事件的监听函数。
    > - `useMemo` 和 `useCallback` 都会存在闭包陷阱，若在内部使用了非依赖数组中的对象，则会无法获取最新的对象值。

18. `useEffect` 和 `useLayoutEffect` 的区别 {.quiz .fill}

    > - 执行时机不一样，`useLayoutEffect` 在 DOM 变更后`同步触发`，`useEffect` 在 DOM 变更前 `异步触发`。
    > - `useLayoutEffect` 同步触发会同步提交 DOM 的变化，所以在 `useLayoutEffect` 中修改状态只会触发一次 `render`，`useEffect` 则会触发两次。
    > - `useLayoutEffect` 会阻塞浏览器渲染。
    > - `useLayoutEffect` 在服务端是不会执行的。

19. React 为什么要使用虚拟 DOM？虚拟 DOM 的原理？ {.quiz .fill}

    > - 虚拟 DOM 就是 JS 对象，用 JS 对象去描述 DOM 的结构。
    >
    > 1. 为了实现跨平台能力。
    > 2. 通过操作数据控制 UI 的渲染逻辑具有更强的表达能力，降低开发者的负担。
    > 3. 预先在内存中找出更新的元素，再进行统一渲染可以降低频繁操作 DOM 消费。
    >
    > - 虚拟 DOM 的缺点是无法做到极致的性能优化，声明式驱动视图的更新和命令式让视图更新的中间过程还是具有一定消耗的。

20. `React.lazy` 和 `Suspense` 的原理和实现 {.quiz .fill}

    > - `lazy` 是借助 `import` 的动态加载实现的，`import` 的动态加载实现了 `Promise` 的规范。
    > - `Suspense` 通过 `componentDidCatch` 来捕获 `lazy` 未加载成功时的渲染问题。
    > - `lazy` 加载的组件必须使用 `Suspense` 包裹起来才能够正常使用。

    > ```tsx
    > function lazy(fn: () => Promise<{ default: any }>) {
    >   const fetcher = {
    >     status: 'pending',
    >     result: null,
    >     promise: null,
    >   }
    >
    >   return function Component() {
    >     const promise = fn()
    >     fetcher.promise = promise
    >
    >     promise.then((res) => {
    >       if (fetcher.status === 'pending') {
    >         fetcher.status = 'resolve'
    >         fetcher.result = res.default
    >       }
    >     })
    >
    >     React.useEffect(() => {
    >       if (fetcher.status === 'pending') {
    >         throw fetcher
    >       }
    >     }, [])
    >
    >     if (fetcher.status === 'resolved') {
    >       return fetcher.result
    >     }
    >
    >     return null
    >   }
    > }
    >
    > class Suspense extends React.PureComponent {
    >   /**
    >    * isRender 异步组件是否就绪，可以渲染
    >    */
    >   state = {
    >     isRender: true,
    >   }
    >
    >   componentDidCatch(event) {
    >     this.setState({ isRender: false })
    >
    >     event.promise.then(() => {
    >       /* 数据请求后，渲染真实组件 */
    >       this.setState({ isRender: true })
    >     })
    >   }
    >
    >   render() {
    >     const { fallback, children } = this.props
    >     const { isRender } = this.state
    >
    >     return isRender ? children : fallback
    >   }
    > }
    > ```

21. `React.Fragment` 的原理 {.quiz .fill}

22. `React` 如何做`时间切片` {.quiz .fill}

    > - 判断当前循环中已经执行了多少秒，如果低于 `5ms` 就继续下一个任务，不切片，如果大于 `5ms` 则进行切片，中断任务执行；

23. `useSyncExternalStore` 的作用及原理 {.quiz .fill}

## React 应用

1. `useReducer` 和 `Redux` 的区别和优缺点 {.quiz .fill}

2. 如何解决 `useState`、`useEffect` 的闭包问题 {.quiz .fill}

3. hook 为什么不能放在条件语句中 {.quiz .fill}

   > - 因为 `Hooks` 基于链表数据结构实现的，在 React 进行遍历时，当前处理的 hook 指针是按序依次更新的，如果 hook 在条件语句中使用就会出现指针错误，找不到或找到错误的 hook 进行处理；

## React 技术生态

1.  `Redux` 的核心概念是？ {.quiz .fill}

    > - `Redux` 的本质是`发布-订阅`模式的实现。
    > - `Redux` 的核心思想是不产生副作用，数据状态`可追踪`、`可回溯`、`可预测`。
    > - `Redux` 的基本流程是
    >   1. 将应用的状态放到一个 `Store` 集中管理。
    >   2. 视图层通过不同的 `Action` 触发不同 `Reducer` 函数去更新 `Store` 中对应的 `state`。
    >   3. 视图层通过订阅 `Store` 的变化来更新视图层。
    > - `Redux` 的基本原则是
    >   1. 一个应用只能有一个数据源（store 唯一）。
    >   2. state 只读，只能通过 `action` 对象驱动 `reducer` 来修改 state。
    >   3. 只能通过纯函数 `reducer` 来修改 store。

2.  `Redux` 中的 `reducer` 为什么必须要返回一个新的对象？ {.quiz .fill}

    > - 纯函数的基本概念是`相同的输入始终输出始终相同`，也就是说需要做到`不修改输入参数`、`不调用不纯的函数`、`不执行有副作用的操作`。根据这三个原则可以解释 `reducer` 为什么需要返回一个新对象，而不是在原来的对象中进行修改。

3.  `Redux` 为什么要把 `reducer` 设计为纯函数？ {.quiz .fill}

    > 1. `Redux` 的定位是一个`可追踪`、`可回溯`、`可预测`的状态管理库，如果 `reducer` 不纯，异步的任务将会破坏 `Redux` 的三个基本原则。

4.  `Redux` 如何处理副作用/异步任务 {.quiz .fill}

    > - `Redux` 处理异步任务主要通过中间件来完成。`Redux` 的中间件是一个`洋葱模型`用来增强 `dispatch` 的能力。
    > - `Redux` 处理异步的整体流程是：
    >   1. 异步任务开始前，通过 `action` 通知 `redux`。
    >   2. 异步任务结束后，再通过 `action` 通知 `redux`。
    >   3. 视图层通过订阅不同的状态来更新视图。

5.  `useSelector` 的原理 {.quiz .fill}

    > 1. `useSelector` 直接订阅了 `store` 的变化，在 `Redux` 更新中，`store` 是不断变化的，但内部 state 的引用只会在相应 `state` 发生变化时才会更新，所以 `useSelector` 的第一个参数是一个筛选函数，用于订阅具体的 `state`，仅当订阅的旧 `state` 和筛选函数返回的新 `state` 不同时才会进行触发视图的更新，第二个参数则是一个更具体的比较函数，当返回 true 才会更新，类似于 `shouldComponentUpdate` 的作用。
    > 2. `useSelector` 的更新，早期使用 `forceRender` 实现，现在使用 `useSyncExternalStore` 实现，避免因为 `React` 自身的调度更新导致状态不一致；

6.  `redux-thunk` 和 `react-saga` 的区别 {.quiz .fill}

    > - `thunk` 使用的是高阶函数的形式通过 `Promise` 来增强 `dispatch`。
    > - `saga` 则使用 `generator` 函数的形式增强 `dispatch`。
    > - 两者在写法上有较大的不同，`thunk` 会比较贴近业务开发，`saga` 的语义会更加清晰，耦合度会更加宽松，但写法相对比较复杂。

7.  `Redux` 和 `Mobx` 的区别是什么，使用场景 {.quiz .fill}

    > 1. `Redux` 基于 `发布-订阅` 模式，走的是单向数据流，store 唯一，数据不可变；
    > 2. `Mobx` 基于 `观察者` 模式，采用的是双向数据流，store 可以多个，数据是可变的；
    > 3. `Redux` 需要手动追踪数据的变化，也能对数据变化进行追踪和追溯，`Mobx` 则只需对数据变化进行观察，在变化时触发监听，数据变化不好追溯；
    > 4. `Redux` 处理异步时，需要借助中间件增强 `Action`，`Mobx` 则天然可支持异步处理；

8.  原子状态库的原理是什么？ {.quiz .fill}

9.  原子类状态库和 `Redux`、`Mobx` 等状态库有什么区别 {.quiz .fill}

    > 1. 原子类状态库和 `Redux` 的核心原理都是 `发布-订阅` 模式，`Mobx` 则采用`观察者`模式实现；
    > 2. 原子类状态库的基本理念是细粒度更新、扁平化分散管理、可追溯，`Redux` 的基本理念是单向数据流、集中管理、可追溯，`Mobx` 的基本理念是衍生、响应式、可观察；

10. `React-Query` 的原理是什么? {.quiz .fill}

    > > `React-Query` 是一个以 `hook` 的方式管理请求的请求管理库（Server State）。
    >
    > 1. `React-Query` 的本质是一个外部的状态管理器，类似于 `Redux`、`Mobx`，不同的是它集成了与服务端请求的状态处理，并提供给开发者使用；
    > 2. `React-Query` 使用了`观察者`模式去桥接 `React` 和状态库的数据变化 `useState(() => new Observer(queryClient, options))`；
    > 3. `React-Query` 通过 `cache` 的变化通知订阅者执行回调来进行更新，在 v3 版本使用的是 forceUpDate 强制更新，v4 使用 `useSyncExternalStore` 来订阅 `Observer` 的变化

11. 什么是`状态持久化`（数据持久化）？ {.quiz .fill}

12. `React SSR` 的原理？ {.quiz .fill}

    > 1. 在服务端调用 `renderString` 将组件和数据输出为 `HTML` 字符串，将这部分视图先交给浏览器进行渲染。
    > 2. 在服务端通过在全局变量挂载服务端发生的状态变化，在客户端进行 `hydrate - 水合`后更新页面，让客户端和服务端的状态保持一致。

13. `React-Route` 的原理？ {.quiz .fill}

14. `Taro2` 的原理？ {.quiz .fill}

15. `Taro3` 的原理？ {.quiz .fill}

## 对比其他框架

1. `Vue Diff` 和 `React Diff` 有什么不同？ {.quiz .fill}

   > - Vue 的 Diff 采用的是双端 Diff，通过双端四次比较来确定节点是否可复用。

2. 为什么没有 `Vue Fiber`？ {.quiz .fill}

3. `React` 和 `Vue` 的区别 {.quiz .fill}
