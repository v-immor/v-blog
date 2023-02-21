---
title: React 的原理导读
date: 2022-12-04 14:42
categories:
  - [JavaScript, React]
tags:
  - React
  - 原理
---

## ![image.png](https://cdn.nlark.com/yuque/0/2023/png/448234/1675069920704-6f8183a3-1276-4e1d-ab62-39a3ea2eef68.png#averageHue=%23fdfcb9&clientId=u2f044d5c-2bb7-4&from=paste&height=1380&id=ueceaacac&name=image.png&originHeight=1380&originWidth=1848&originalType=binary&ratio=1&rotation=0&showTitle=false&size=243276&status=done&style=none&taskId=u6d8a2d5a-29e2-4b74-8ded-df44326196f&title=&width=1848)
`document.querySelector('#app')._reactRootContainer` 控制台获取整个 root 对象 
## 旧 React 架构（v15）

1. 架构分层：
   - Reconciler（协调器）：负责找出变化的组件（render 阶段）
   - Renderer（渲染器）：负责将变化的组件渲染到页面上（commit 阶段）
2. Reconciler：当使用 this.setState、this.forceUpdate、ReactDOM.render 等 API 触发更新时，Reconciler 会做以下的事情
   - 调用函数组件或 class 组件 render 方法，将返回的 JSX 转换为 VDOM
   - 将 VDOM 和上次更新时的 VDOM 对比，找出本次更新中变化的 VDOM
   - 通知 Renderer 将变化的 VDOM 渲染到页面上
3. 缺点：在  Reconciler 中，mount 组件会调用 mountComponent, update 的组件会调用 updateComponent，这两个方法都会递归更新子组件，而递归更新带来问题如下
   - 浏览器 16.6ms 刷新一次（跑一帧），GUI 渲染线程与 JS 线程是互斥的，不能同时执行
   - 16.6 ms 内需处理：JS 脚本执行 --> 样式布局 --> 样式绘制。react 是采用递归更新，一旦开始，不能中断，当整个递归时间超过 16.6 ms，导致用户交互卡顿
   - 如果说采用可中断的异步更新来代替同步，那么在 16.6 ms 内就会没足够时间进行 Reconciler，导致一帧内 UI 渲染缺失

![image.png](https://cdn.nlark.com/yuque/0/2022/png/448234/1651763990234-f195972f-1eb6-4763-843b-ad140dd818b3.png#averageHue=%23bebebe&clientId=ub18f502b-ca66-4&errorMessage=unknown%20error&from=paste&height=125&id=It4Rv&name=image.png&originHeight=498&originWidth=1464&originalType=binary&ratio=1&rotation=0&showTitle=false&size=378407&status=error&style=none&taskId=ucec65c38-14ba-4cb9-a2b5-26bba8d80f7&title=&width=366)![image.png](https://cdn.nlark.com/yuque/0/2022/png/448234/1651764026163-1091a4ff-fa7f-4f27-ae1f-6b42980b311f.png#averageHue=%23c9c9c9&clientId=ub18f502b-ca66-4&errorMessage=unknown%20error&from=paste&height=129&id=NXx3J&name=image.png&originHeight=528&originWidth=1522&originalType=binary&ratio=1&rotation=0&showTitle=false&size=269550&status=error&style=none&taskId=u556f4b6c-8b81-47aa-9569-48189246bdd&title=&width=373)
## 新 React 架构（v16）

1. 出现背景：为了解决 reconcile 同步不可中断执行带来页面卡顿现象（js 执行时间长，会导致浏览器没有时间绘制 DOM 从而出现卡顿）和应对不同网络状态下获取数据响应问题；
2. 架构分层：
   - Scheduler：用来实现时间切片和任务调度；
      - 任务调度：调度任务优先级，高优先级任务先进入 Reconciler
      - 时间切片：在浏览器空闲时触发回调，中断或继续执行 fiber 任务单元
   - lane：用来细粒度的管理各个任务的优先级，让高优先级的任务先执行，各个 Fiber 工作单元还能比较优先级，相同优先级的任务可以一起更新
   - Reconciler（协调器）：找出变化的节点，并打上不同的 tag
   - Renderer（渲染器）：负责将 Reconciler 中打好标签的节点渲染到视图上
3. Reconciler：对比 v15 版本 Reconciler
   - 从递归处理 VDOM 变成可中断的循环；每次循环都会调用 shouldYield 判断当否有剩余时间
   - 解决中断导致更新 DOM 渲染不完全问题：Reconciler 和 Renderer 不在是交替工作，当 Scheduler 将任务交给 Reconciler 后， Reconciler 会为需变化的 VDOM 打上标记（增/删/更新...），当所有组件都完成 Reconciler 后，统一交给 Renderer
   - 发生在 render 阶段，分别为节点执行 beginWork 和 completeWork，或者计算 state，对比节点差异，为节点赋值相应的 effectTag
3. Renderer：commit 阶段， 根据 Reconciler 打了标记的 VDOM，同步执行对应的 DOM 操作
4. 架构更新流程如下图：
   - 红框中的步骤随时可能被（1）更高优先级任务（2）当前帧没有剩余时间，中断。
   - 红框中的步骤始终在内存中进行，不会更新页面上 DOM, 所以反复中断，也不会导致 React 15 中 DOM 更新不完全问题。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/448234/1651805207302-022099d9-8dea-4333-820e-1c7d36ea6b38.png#averageHue=%23f7f7f7&clientId=ub18f502b-ca66-4&errorMessage=unknown%20error&from=paste&height=493&id=a41wR&name=image.png&originHeight=986&originWidth=2290&originalType=binary&ratio=1&rotation=0&showTitle=false&size=282788&status=error&style=none&taskId=uaef789ea-261d-4411-bdfe-b62bdb81f18&title=&width=1145)

5. React 有 3 种模式
   - legacy（默认）：这模式可能不支持一些新功能
      - 在合成事件有自动批处理功能，仅限于一个浏览器任务，非 React 事件需使用 unstable_batchedUpdates
      - ReactDOM.render(<App/>, rootNode)
   - blocking：开启部分 concurrent 模式特性的中间模式，当前实验中，作为迁移 concurrent 模式的第一个步骤
      - 所有 setState 默认批处理
      - ReactDOM.createBlockingRoot(rootNode).render(<App/>)
   - concurrent：面向未来的开发模式，任务中断/任务优先级都是针对 concurrent 模式
      - 所有 setState 默认批处理
      - ReactDOM.createRoot(rootNode).render(<App/>)
6. 架构运行流程：schedule-->render-->commit
7. 代数效应：除了cpu的瓶颈问题，还有一类问题是和副作用相关的问题，比如获取数据、文件操作等。不同设备性能和网络状况都不一样，react怎样去处理这些副作用，让我们在编码时最佳实践，运行应用时表现一致呢，这就需要react有分离副作用的能力，为什么要分离副作用呢，因为要解耦，这就是代数效应
8. 异步可中断原理：利用浏览器 16.6ms 刷新频率，在每一帧分配一个时间给 js 执行，如果这个事件内还没执行完，就暂停它的执行，等下一帧继续执行，把执行权交给浏览器绘制，从而解决页面卡顿问题
## React v18

1. 扩大批量处理事件：由只限于原生事件扩大为 Promise，setTimeout，native event handlers 等这些非 React 原生的事件内部的更新也会得到合并：
```javascript
const handleClick=()=>{
  setCount(1)
  setName('li') // 原生事件 click 触发两次更新 state批量处理后，只会触发一次render
}

```
## 解析 JSX

1. jsx 经过 babel 编译后的产物：
   - v17 之前用 @babel/babel-preset-react-app 将 jsx 解析成 React.createElement(type, config, ...children) 函数调用 
   - v17 之后用 react/jsx-runtime
```javascript
<div className='box'>
  <h1 style={{color: red}}>第一章</h1>
</div>

// v16.x 版本
"use strict"
React.createElement('div', {
  className: 'box'
},React.createElement('h1', {
  style: {
    'color': 'red'
  }
}, '\u7B2C\u4E00\u7AE0'))

// v17 版本
"use strict"
var _jsxRuntime = require('react/jsx-runtime')
(0, _jsxRuntime.jsxs)('div',{
  className: 'box',
  children: [(0, _jsxRuntime.jsxs)('h1',{
     style: {
    	'color': 'red'
  	},
    children: '\u7B2C\u4E00\u7AE0'
  })]
})
```

2. jsx-runtime：
```javascript
// 处理数据，调用 ReactElement() 创建 VDOM 格式
export function jsx(type, config, maybeKey) {
  let propName;
  const props = {} // 标签上属性集合
  let key = null;
  let ref = null;
  if(maybeKey !== undefined) {
    key = '' + maybeKey
  }
  if(hasValidKey(config)) {
    key = '' + config.key
  }
  if(hasValidRef(config)) {
    ref = '' + config.ref
  }
  // 将属性添加到 props 中
  for (propName in config) {
    if(hasOwnProperty.call(config, propName) &&
      !RESERVED_PROPS.hasOwnProperty(propName)
      ) {
     		props[propName] = config[propName]
      }
  }
  // 处理默认 props
  if (type && type.defaultProps) {
    const defaultProps = type.defaultProps;
    for (propName in defaultProps) {
      if (props[propName] === undefined) {
        props[propName] = defaultProps[propName];
      }
    }
  }
  return ReactElement(
      type,
      key,
      ref,
      undefined,
      undefined,
      ReactCurrentOwner.current,
      props
  )
}

const ReactElement = function(type,key,ref,self,source,owner,props) {
  const element = {
    $$typeof: REACT_ELEMENT_TYPE,	// 是否是 ReactElement
    type: type,
    key: key,
    ref: ref,
    props: props,
    _owner: owner,
  }
  if(_DEV_){
    // 开发环境下将 _store、_self、_source 设置成不可枚举状态
  }
  return element
}
```

3. React.createElement() ：对组件参数进行处理，以及添加一些必要标识，返回 VDOM（过程和 jsx-runtime 差不多，只是传入参数不同，处理不同）
   - 解析 config 参数中是否有合法 key、ref 属性，并将其它属性挂到 props 上
   - 解析第三个参数，并分情况挂载到 props.children 上
   - 处理默认 props，如果存在该属性则挂载到 props，不存在则添加上
   - 开发环境下将 _store、_self、_source 设置成不可枚举状态，为后期 diff 做优化
   - 将 type、key、ref、props 等属性通过调用 ReactElement() 创建 VDOM
4. v17 前，需要用到 babel 将 jsx 转换成 React.createElement() 所以需要导入 `import React from 'react'`，v17 后，babel 编译后会自动导入 `import {jsx as _jsx} from 'react/jsx-runtime'`，所以不需导入 react
## render 阶段

1. Reconciler 工作阶段在 React 内部被称为 render 阶段，render() 函数也是在这阶段调用的
2. render(vdom, container)：协调阶段，可认为这是一个 diff 阶段，这个阶段可中断，会找出所有变更节点等，这些变更 React 称做副作用，
   - 创建每个节点的 fiber，生成一颗有新状态的 workInProgress tree
   - mount 时根据 fiber 生成 真实 DOM，并且将当前子节点进行插入 appendChildren
   - update 时对比新旧 fiber 状态，产生新的 fiber 节点，通过链表的形式，挂载到 RootFiber
3. mount 阶段（root 为 null）：
   - 创建 FiberRoot：当容器内部有节点时，需移除 `container.removeChild(rootSibling)`
   - 进入 update 阶段
3. update 阶段：创建或获取一个队列，将 update 对象入队，进入调度流程
   - updata 对象中包含 setSate 一些信息
   -  expirationTime：任务过期时间，根据任务优先级和当前时间计算出的执行截止时间，当 expirationTime 值大于当前时间可延后执行，让高优先级任务执行，一旦过了截止时间，应立即执行这个任务
```javascript
import ReactDOM from 'react-dom'
ReactDOM.render(<APP />, document.getElementById('root')

// 如果 id='root' 内有子节点，是会被 render() 移除掉的，所以不用写
<div id='root'></div>

// update 对象
var update = {
    eventTime: eventTime, // 等价于 expirationTime
    lane: lane,
    tag: UpdateState,
    payload: null,
    callback: null,
    next: null
  };
```
### performUnitOfWork

1. performUnitOfWork 主要分为“递”和“归”阶段，两个阶段交替执行
2. 递阶段：以 DFS 方式遍历每个节点执行 beginWork()，根据传入 fiberNode 创建下一级 fiberNode
3. 归阶段：调用 completeWork() 处理 fiberNode，直至 HostRootFiber 归阶段停止，render 阶段结束
### beginWork

1. 主要是判断当前属于 mount 还是 update 流程（c fiberNode===null）,从而进入不同流程处理
   - mount：根据 fiber.tag（组件类型） 不同，创建不同的子 fiberNode
   - update：进入 Reconciler 模块进行 diff 判断是否可复用节点，从而克隆/创建 fiberNode

![image.png](https://cdn.nlark.com/yuque/0/2023/png/448234/1673853387457-23015b2d-e99a-4936-bd50-40f78744465a.png#averageHue=%23fefefe&clientId=u0fa7928d-7295-4&from=paste&height=540&id=u93655947&name=image.png&originHeight=1080&originWidth=1920&originalType=binary&ratio=1&rotation=0&showTitle=false&size=151825&status=done&style=none&taskId=ua6bf8aee-f09f-435e-b88a-21db7e450e4&title=&width=960)
### completeWork

1. update 时：Fiber 节点已存在对应的 DOM 节点，不需要生成 DOM 节点，所以主要处理 props
   - onClick、onChange 等回调函数注册
   - 处理 stype prop、DANGROUSLY_SET_INNER_HTML prop、children prop
   - 将处理好的 props 赋值给 wip.updateQueue（更新队列），在 commit 阶段渲染在页面上
2. mount 时：为 Fiber 节点生成对应 DOM 节点，将子孙 DOM 节点插入刚生成 DOM 节点中，形成 DOM Tree，处理 props（类似上述 update）
3. effctList：
   - 背景：为了避免在 commit 阶段再次遍历 Fiber Tree 寻找 effctTag !== null 的节点
   - 原理：每个执行完 completeWork 且存在 effectTag 的 Fiber 节点会被保存在 effctList 单向链表中，形成一条都有 effectTag 的单向链表，commit 阶段只需遍历 effectTag 就可以执行所有 effect 了

![image.png](https://cdn.nlark.com/yuque/0/2023/png/448234/1673858881525-5ed824f6-9adf-40f0-b0bb-4d07499c922c.png#averageHue=%2323262d&clientId=u0fa7928d-7295-4&from=paste&height=85&id=ue9bab766&name=image.png&originHeight=170&originWidth=1190&originalType=binary&ratio=1&rotation=0&showTitle=false&size=34197&status=done&style=none&taskId=udf561d4d-d547-462a-bc4f-74d21e514d2&title=&width=595)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/448234/1673858964893-89b99e03-5e90-47e4-8596-a71663817582.png#averageHue=%23f5f9fb&clientId=u0fa7928d-7295-4&from=paste&height=702&id=u256bbaff&name=image.png&originHeight=1404&originWidth=1594&originalType=binary&ratio=1&rotation=0&showTitle=false&size=402576&status=done&style=none&taskId=u0d566ebe-8346-487c-b1f0-6cd7d1b59d0&title=&width=797)
## commit 阶段

1. 提交阶段（又叫 Renderer 阶段）：同步，不可中断，将 render 阶段段计算出来副作用的 FIber 节点组成的单向链表 effectList 一次性执行了，此 FIber 节点的 updateQueue 中保存了变化的 props
2. 执行生命周期（useLayoutEffect、useEffect...），获取 rootFiber.firstEffect 根据 effectTag 操作页面
3. 主要工作（即 Renderer 工作流程）：三个阶段都会遍历 effectList 单向链表
   - before mutation 阶段（执行 DOM 操作前）：遍历 effectList
      - 处理 DOM 节点渲染/删除后的 autoFocus、blur 逻辑
      - 调用 getSnapshotBeforeUpdate 生命周期钩子
      - 异步调度 useEffect：异步执行原因是防止同步执行阻塞浏览器渲染
   - mutation 阶段（执行 DOM 操作）：会对 Fiber 节点做如下操作
      - 根据 ContentReset effctTag 重置文字节点
      - 更新 ref
      - 根据 effectTag(Placement、Update、Deletion、Hydrating) 分别处理
   - layout 阶段（执行 DOM 操作后）
      - 调用生命周期和 hook 相关操作：调用 setState 第二个参数、useLayoutEffect、useEffect 销毁函数
      - 赋值 ref：获取 DOM 实例，更新 ref
4. before mutation 之前会处理一些变量赋值、状态重置工作；layout 之后会进行 useEffect 相关处理、性能追踪
## Fiber

1. 背景：解决 React 15 递归创建 VDOM 不能中断而导致页面高频刷新卡顿问题
   - 卡顿原因：屏幕以 16.66ms/帧刷新，一帧内如果事件处理时间过长就会导致没时间渲染或渲染的画面被下一帧立即覆盖，肉眼没捕获到
2. 概念：Fiber 是一种纯 JS 可中断的链式数据结构，这数据节点的一些信息，可对应真实 DOM，又能作为分割单元；Fiber 是一个可执行单元，每执行完一个单元，就会检查现在剩余的时间，如果时间不够下一个单元，就把控制权交回给浏览器
   - ❓：如果不把时间交回给浏览器会发生什么？
```javascript
class FiberNode {
  constructor(tag, key, pendingProps) {
    this.tag = tag; // 表示当前fiber的类型
    this.key = key;
    this.type = null // 'div' || 'h1' || Ding
    this.stateNode = null; // 表示当前fiber的实例
    this.child = null; // 表示当前fiber的子节点 每一个fiber有且只有一个指向它的firstChild
    this.sibling = null; // 表示当前节点的兄弟节点 每个fiber有且只有一个属性指向隔壁的兄弟节点
    this.return = null; // 表示当前fiber的父节点
    this.index = 0;
    this.memoizedState = null; // 表示当前fiber的state
    this.memoizedProps = null; // 表示当前fiber的props
    this.pendingProps = pendingProps; // 表示新进来的props
    this.effecTag = null; // 表示当前节点要进行何种更新
    this.firstEffect = null; // 表示当前节点的有更新的第一个子节点
    this.lastEffect = null; // 表示当前节点的有更新的最后一个子节点
    this.nextEffect = null; // 表示下一个要更新的子节点

    this.alternate = null; // 用来连接current和workInProgress的
    this.updateQueue = null; // 一条链表上面挂载的是当前fiber的新状态

    // 其实还有很多其他的属性
    // expirationTime: 0
  }
}
```

1. fiber tree:

![image.png](https://cdn.nlark.com/yuque/0/2023/png/448234/1673246535570-c3be5265-c736-432b-be12-142b99cb94cd.png#averageHue=%23fef0ee&clientId=u004834b4-dea3-4&from=paste&height=600&id=u8563cbe8&name=image.png&originHeight=1954&originWidth=1442&originalType=binary&ratio=1&rotation=0&showTitle=false&size=267800&status=done&style=none&taskId=u0619ac0e-f368-4f14-8eac-20b3d34bca6&title=&width=443)

2. Fiber 架构 = fiber 数据结构 + 算法，Fiber 是一个执行单元，每执行一个就会检查有没有剩余时间执行下一个

![image.png](https://cdn.nlark.com/yuque/0/2023/png/448234/1673247067997-9a58560c-0cb3-4b76-a486-58a143e8ade6.png#averageHue=%23f5f5f5&clientId=u004834b4-dea3-4&from=paste&height=749&id=u8ca523be&name=image.png&originHeight=1498&originWidth=1118&originalType=binary&ratio=1&rotation=0&showTitle=false&size=393193&status=done&style=none&taskId=ud34f649f-de5f-4ff2-8b03-f6bd70e81f8&title=&width=559)

3. 管理的整体数据：
   - Root：整个应用的根，是一个对象，不是 fiber，有个属性指向 current tree，有个属性指向 workInProgess tree
   - current tree：每个节点都是 fiber，保存上一次状态，每个 fiber 对应一个 jsx 节点
   - workInProgess tree：每个节点都是 fiber，保存本次新的状态，每个 fiber 对应一个 jsx 节点
### Build Fiber Tree

1. 流程：先“递” 后 “归”，“归” 过程中遇到兄弟 Fiber 节点，继续对兄弟 Fiber 节点进行 “递” 和 “归”，完成后，继续向父级“归”
2. 例子如下：child 方向箭头是 “递” 过程，return 是 “归” 过程

![image.png](https://cdn.nlark.com/yuque/0/2022/png/448234/1651823405603-8fc74b2d-ac8b-400b-a330-e9555f3965f1.png#averageHue=%23fdfdfd&clientId=ub18f502b-ca66-4&errorMessage=unknown%20error&from=paste&height=335&id=kwVCu&name=image.png&originHeight=1338&originWidth=1618&originalType=binary&ratio=1&rotation=0&showTitle=false&size=279929&status=error&style=none&taskId=u6e593500-f7c9-41e9-b2c6-c226eba2f50&title=&width=405)
```javascript
// 注意“KaSong”阶段没有 beginWork 和 completeWork，是因为单一文本节点，React 会做特殊处理
1. rootFiber beginWork
2. App Fiber beginWork
3. div Fiber beginWork
4. "i am" Fiber beginWork
5. "i am" Fiber completeWork
6. span Fiber beginWork
7. span Fiber completeWork
8. div Fiber completeWork
9. App Fiber completeWork
10. rootFiber completeWork
```

3. benginWork：传入当前 Fiber 节点，创建子 Fiber 节点

![image.png](https://cdn.nlark.com/yuque/0/2022/png/448234/1651843871907-a47b7ade-40ee-48b9-998e-ead938a9e5f9.png#averageHue=%23fefefe&clientId=ub18f502b-ca66-4&errorMessage=unknown%20error&from=paste&height=540&id=NkC7r&name=image.png&originHeight=1080&originWidth=1920&originalType=binary&ratio=1&rotation=0&showTitle=false&size=151825&status=error&style=none&taskId=u35d56ddd-28da-490b-92c9-6f1b846de03&title=&width=960)

4. completeWork：

![image.png](https://cdn.nlark.com/yuque/0/2022/png/448234/1651844488902-650fc418-9815-48a3-8258-9692e6d5bed3.png#averageHue=%23f5f9fb&clientId=ub18f502b-ca66-4&errorMessage=unknown%20error&from=paste&height=702&id=n2M9R&name=image.png&originHeight=1404&originWidth=1594&originalType=binary&ratio=1&rotation=0&showTitle=false&size=402576&status=error&style=none&taskId=ub7d50f8e-6457-4331-8c01-1c2eaaaf703&title=&width=797)

5. 在 completeWork 上层函数 completeUnitOfWork 中，每个执行完 completeWork 且存在 effectTag 的 Fiber 节点会被保存在 effectList 单向链表中；`rootFiber.firstEffect --->(nextEffect) fiber --->(nextEffect) fiber`
## 状态更新

1. 会触发更新的操作：HostRoot 和 ClassComponent 共用一套 Update 结构，FunctionComponent 单独一种
   - React.DOM.render：HostRoot
   - this.setState：ClassComponent
   - this.forceUpdate：ClassComponent
   - useState：FunctionComponent
   - useReducer：FunctionComponent
```javascript
// Update 结构
const update = {
  eventTime,	// 任务过期时间
  lane,		// 优先级相关字段
  suspenseConfig,	// Suspense 相关
  tag: UpdateState,	// 更新类型 UpdateState | ReplaceState | ForceUpdate | CaptureUpdate
  payload: null,	// 更新挂载数据，不同类型组件数据不同
  callback: null,	// 更新的回调函数
  next: null,	// 与其它 Update 连接形成链表
};
```

2. 更新流程：

![](https://cdn.nlark.com/yuque/0/2023/jpeg/448234/1673865563110-945ad7c8-8052-4121-8590-4a6b5501ce09.jpeg)

3. updateQueue：如 Fiber 上有两个上次未处理的 Update，会保存在下次的 baseUpdate，记为 u1、u2，当前触发更新该 Fiber 产生 u3、u4 两个 Update，那么 firstBaseUpdate === u1、lastBaseUpdate === u2、shared.pending === u3-->u4，当进入 render 阶段 pending 就会被剪开并连接在 lastBaseUpdate 后面，那么 fiber.updateQueue.baseUpdate: u1 --> u2 --> u3 --> u4，然后遍历 baseUpdate 以 baseState 为初始值计算新 state，遍历时优先级低的 Update 会被跳过
```javascript
// queue 存储 update
const queue = {
  baseState: fiber.memoizedState,	// 更新前 fiber 的 state
  firstBaseUpdate: null,	// 更新前的 Update 的链表头
  lastBaseUpdate: null,	// 更新前的 Update 的链表尾
  shared: {		// 触发更新时，新产生的 Update 保存在 pending 形成环状单向链表
    pending: null,
  },
  effects: null,	// 数组，保存 update.callback !== null 的 Update
};
```

4. ClassComponent 将 queue 存储在实例中，FunctionComponent 存储在 fiber 中
```javascript
const fiber = {
  // 保存该FunctionComponent对应的Hooks链表
  memoizedState: null,
  stateNode: App
}
// hook 与 update 相似，都通过链表连接，每个 useState 对应一个 hook 对象
hook = {
  // 保存 update 的 queue，即上述介绍的 queue
  queue: {
    pending: null
  },
  // 保存hook对应的state
  memoizedState: initialState,
  // 与下一个Hook连接形成单向无环链表
  next: null
}
```
## Scheduler

1. Scheduler：调度器，调度任务优先级，高优先级任务先进入 Reconciler，在浏览器空闲时触发回调
2. 作用：主要实现了时间切片和优先级调度；异步可中断执行 Fiber 工作单元，类似 requestIdleCallback，可实现 batchedUpdates 批量更新和Suspense
### 时间切片

1. 基本原理：利用 MessageChannel 创建一个宏任务实现，当浏览器不支持 MessageChannel 会使用 setTimeout，默认每个人任务初始剩余时间 5ms。
   - 不选用 requestAnimationFrame：
      - 当第一次任务调度不是 rAF() 触发的会导致本次页面更新前执行两次任务；
      - rAF() 触发间隔时间不确定，由于浏览器是自行判断是否应该更新页面（通常是在宏任务完成之后），如果需要更新则执行 rAF()，否则，执行下一个宏任务，就会浪费浏览器间隔更新的时间
      - rAF()  在当前页面未激活状态下会停止执行
   - 不选用微任务：微任务是在页面更新前全部执行完，达不到将“主线程还给浏览器”的目的（每执行完一个宏任务就会将主线程还给浏览器，但微任务需要将队列清空才会归还）
   - 不首选 setTimeout：
      - `setTimeout(fn,0)/setInterval(fn,0)`创建的宏任务至少有 4ms 执行时差，因为 setTimeout 执行时机和 js 执行有关
      - MassageChannel 通常会先于 setTimeout 执行，但也需要根据 setTimeout 设置的延迟时间相关
   - 不选用 requestIdelCallback： 兼容性不好，且触发不稳定
2. 时间切片对外暴露的方法是 shouldYield()，用来判断浏览器是否有剩余时间，是否需要中断任务
### 任务调度

1. 基本原理：给不同的任务设置过期时间，越先过期优先级越高
2. Scheduler 存在两个队列：timerQueue（保存未就绪任务），taskQueue（保存已就绪任务）；当有新未就绪任务注册时，将其插入 timerQueue 并根据时间重排任务顺序，当 timerQueue 中任务就绪（startTime <= currentTime）就会取出并加入 taskQueue，从 taskQueue 取出任务执行
3. 对外暴露 unstable_runWithPriority() 方法，用来获取任务优先级
### lane

1. 背景：react 之前用 `expirationTime = startTime + timeout` 代表优先级，但由于 io 优先级高于 cpu，所以 expirationTime 不能和 io 很好的搭配工作，于是就有了细粒度更好的 lane
2. 概念：lane 用二进制表示优先级，1 表示位置，同一个二进制可以有多个相同优先级的位（”批“）
3. lane 中如果低优先级的任务一直被高优先级打断，到了它的过期时间，也会编程高优先级
## Reconciler

1. Reconciler：协调器，收集变化的组件，并打上不同的 tag，将新的 Fiber 节点赋值给 workInProgress.child。 当 current === null 时是 mount 组件
   - mount 组件：调用 mountChildFibers() 创建新的 Fiber 节点
   - update 组件：调用 renconcileChildFibers() 将当前组件与该组件上次更新对应的 Fiber 节点比较（Diff 算法），生成新的 Fiber 节点
2. mountChildFibers() 对比 renconcileChildFibers() 的区别是不会为生成的节点带上 effectTag 属性（标记增、删、移...）
   - mountChildFibers() 不带 effectTag 可以使得在 commit 阶段不会对每个节点都执行一次插入操作，只在 rootFiber 带 effectTag ，在 commit 阶段就只会执行一下插入操作，提升性能
### Diff

1. 一个 DOM节点在某一时刻最多会有 3 个节点和他相关，Diff 本质是对比 JSX 对象和 c Fiber 生成 wip Fiber
   - current Fiber（c Fiber）：与视图中的 DOM 对应
   - workInProgress Fiber（wip Fiber）：与更新进程中的 DOM 对应
   - JSX 对象：包含描述 DOM 元素所需的数据
2. Diff 本身会耗费性能，就算最前沿算法比对复杂度也为 O(n3)，n 是树中元素的数量，所以为降低复杂度，React Diff 会预设三个限制
   - 规则 1：只对同级 diff，如果一个 DOM 元素在前后两次更新中跨越了层级，则 React 不会复用它
      - 单节点 diff：newChild === object/number/string，代表更新后同级只有一个元素，直接根据 newChild 创建 wip Fiber 并返回
      - 多节点 diff：newChild === array/iterator，代表更新后同级有多个元素，遍历 newChild 创建 wip Fiber 并返回
   - 规则 2：两个不同类型元素会产生不同的树，如 div-->p，React 会销毁 div 及其子孙元素，并新建 p 及其字段元素
   - 规则 3：开发者可以通过 key 来暗示在不同渲染下能够保持稳定
```javascript
// 当没有 key 时会命中规则 2，使得 div 子元素不能复用
// 当有 key 时会命中规则 3，使得 div 子元素能复用，只需交换顺序
// 更新前
<div>
  <p key='1'>1</p>
  <div key='2'>2</div>
</div>
// 更新后
<div>
  <div key='2'>2</div>
   <p key='1'>1</p>
</div>
```

3. Fiber 是否可复用判断：先判断 key 相同（前后未设 key 也视为相同），再判断类型相同，均相同，Fiber 可复用
4. 单节点 diff：考虑节点是否可复用、多余节点删除

![](https://cdn.nlark.com/yuque/0/2023/jpeg/448234/1673715244252-a0230839-8a0a-4037-8477-c7d568f63b90.jpeg)

5. 多节点 diff：考虑节点复用、增删、移动

![](https://cdn.nlark.com/yuque/0/2023/jpeg/448234/1673714929632-348b128d-1cdf-4ce0-8cb5-32d039a00006.jpeg)

6. 手写一个简单版的 diff
## 双缓存机制

1. 作用：Fiber 树的构建与替换--对应着 DOM 树的创建与更新
2. 定义：在内存中构建并直接替换的技术叫双缓存
3. 双缓存 Fiber 树：
   - React 中最同时存在两颗 Fiber 树：current Fiber tree(当前屏幕上显示的)、workInProgress Fiber tree(内存中正在构建的)
   - 两棵树的节点通过 alternate 属性连接
```javascript
currentFiber.alternate === workInProgressFiber
workInProgressFiber.alternate === currentFiber
```

4. React 应用根节点通过 current 指针指向不同 Fiber 树来切换 current Fiber tree 和 workInProgress Fiber tree 完成 DOM 的创建/替换
5. 创建流程（mount）如下：

![image.png](https://cdn.nlark.com/yuque/0/2022/png/448234/1651820837270-fce97b2d-e410-42c4-b06b-aecb40ed5b2e.png#averageHue=%23fcf7f6&clientId=ub18f502b-ca66-4&errorMessage=unknown%20error&from=paste&height=392&id=ue64b7e8d&name=image.png&originHeight=1568&originWidth=1260&originalType=binary&ratio=1&rotation=0&showTitle=false&size=225868&status=error&style=none&taskId=uff81c02a-fcaf-4133-b567-1240c4196f3&title=&width=315)![image.png](https://cdn.nlark.com/yuque/0/2022/png/448234/1651820962028-2385b298-d8de-4634-a43d-76d8314648c6.png#averageHue=%23fdf9f9&clientId=ub18f502b-ca66-4&errorMessage=unknown%20error&from=paste&height=395&id=u2a5d2b37&name=image.png&originHeight=1576&originWidth=1178&originalType=binary&ratio=1&rotation=0&showTitle=false&size=185379&status=error&style=none&taskId=ucce4b3b5-1b6e-4b33-8874-228276e1657&title=&width=295)

6. 更新流程（update）如下：

![image.png](https://cdn.nlark.com/yuque/0/2022/png/448234/1651821151580-30dfcbcb-5200-4996-bd92-5c275502b142.png#averageHue=%23fbfaf9&clientId=ub18f502b-ca66-4&errorMessage=unknown%20error&from=paste&height=394&id=uf2fecb30&name=image.png&originHeight=1574&originWidth=1242&originalType=binary&ratio=1&rotation=0&showTitle=false&size=251434&status=error&style=none&taskId=u6d8d6128-50dc-404a-b322-06c4dbf00bf&title=&width=311)![image.png](https://cdn.nlark.com/yuque/0/2022/png/448234/1651821193599-25762e22-a2fe-4a14-af89-9c88ccaa39ee.png#averageHue=%23fdfbfb&clientId=ub18f502b-ca66-4&errorMessage=unknown%20error&from=paste&height=394&id=u811762a4&name=image.png&originHeight=1572&originWidth=1082&originalType=binary&ratio=1&rotation=0&showTitle=false&size=140449&status=error&style=none&taskId=ube50ee16-2631-4cc9-84ec-f4730662ce8&title=&width=271)
## 合成事件

1. 原理：react 事件是在  document（17 在 root）监听所有事件，当事件冒泡到 document 处（17 root 捕获阶段）封装事件（不是浏览器原生事件，而是 react 实现的合成事件）内容并处理；

![image.png](https://cdn.nlark.com/yuque/0/2022/png/448234/1665372333600-21ebbf66-9c02-47bb-98e7-946a7552ef38.png#averageHue=%23fdfdfb&clientId=u73f647c3-21b1-4&from=paste&height=235&id=Vq5Jt&name=image.png&originHeight=395&originWidth=878&originalType=binary&ratio=1&rotation=0&showTitle=false&size=103873&status=done&style=none&taskId=uffb3c5b1-cbf2-4fa9-9593-c37447ad9c6&title=&width=522)

2. 目的：
- 进行浏览器兼容：抹平不同浏览器事件对象的差异，将不同平台事件模拟合成事件
- 节省内存：React 引入事件池（React17后被废弃），在事件池中获取或释放事件对象。当事件触发，从事件池弹出，避免频繁创建和销毁（垃圾回收）
- 方便事件统一管理和事物机制
3. 合成事件与原生事件对比
   - 命名不同：原生全小写（onclick），react 小驼峰（onClick）
   - 事件函数处理不同：原生为字符串，react 为函数
   - react 不能用 `return false` 阻止浏览器默认行为，必须调用 `e.preventDefault()`来阻止
4. 合成事件三个阶段
- 事件注册：收集所有原生事件名称，并生成原生和合成事件的映射关系，同时生成所有合成事件对象构造函数
- 事件绑定：遍历所有原生事件名称，给容器 root 注册所有捕获和冒泡事件
- 事件触发：执行事件执行函数
```javascript
// react 16 合成事件
function dispatchEvent(event) {
  console.log(event, 'event')
  // 收集事件
  const allNativeEvents = []
  let current = event.target
  while (current) {
    allNativeEvents.push(current)
    current = current.parentNode
  }
  // 模拟捕获和冒泡，其实在这个时候，原生的捕获阶段已经结束了
  // 捕获阶段
  for (let i = allNativeEvents.length - 1; i >= 0; i--) {
    const handler = allNativeEvents[i].onClickCapture
    handler && handler()
  }
  // 冒泡阶段
  for (let i = 0; i < allNativeEvents.length; i++) {
    const handler = allNativeEvents[i].onClick
    handler && handler()
  }
}

// 注册React事件的事件委托
document.addEventListener('click', dispatchEvent)

// react 17 合成事件
function dispatchEvent(event, useCapture) {
  console.log(event, 'event')
  // 收集事件
  const allNativeEvents = []
  let current = event.target
  while (current) {
    allNativeEvents.push(current)
    current = current.parentNode
  }
  if(useCapture){
     // 捕获阶段
    for (let i = allNativeEvents.length - 1; i >= 0; i--) {
      const handler = allNativeEvents[i].onClickCapture
      handler && handler()
    }
  }else{
    // 冒泡阶段
    for (let i = 0; i < allNativeEvents.length; i++) {
      const handler = allNativeEvents[i].onClick
      handler && handler()
    }
  }
}
root.addEventListener('click', (event)=>dispatchEvent(event,true),true)
root.addEventListener('click', dispatchEvent)

```
react 17 以前执行顺序：document捕获、父元素原生捕获、子元素原生捕获、子元素原生冒泡、父元素原生冒泡、父元素React捕获、子元素react捕获、子元素react冒泡、父元素react冒泡、document冒泡
react 17 以后执行顺序：document捕获、父元素React捕获、子元素react捕获、父元素原生捕获、子元素原生捕获、子元素原生冒泡、父元素原生冒泡、子元素react冒泡、父元素react冒泡、document冒泡
## Hooks

1. hooks 可以使一个组件在同一时间拥有多个状态以达到并发的目的，class 组件无法实现
2. 由于 class 组价遵循 OOP 原则，逻辑和状态耦合在实例内部，无法同一时间拥有多个状态（多个 this.state）
3. hooks 组件是 FP(无状态)编程思想，更新状态保存在闭包中，所以可以同一时间保存不同闭包中的多个状态
## Mini React
参考：[手写 React Fiber 架构](https://cloud.tencent.com/developer/article/1717291)

1. 解析 JSX：通过 babel 将 JSX 语法转换成 React.createElement() 形式
   - React.createElement(type, config, children): type 为节点类型；config 节点属性；children 子元素
```javascript
// JSX
<div className="name" age="12">
  <div>1</div>
  <div>2</div>
  <div>3</div>
</div>;

// 经 babel 解析后的产物
React.createElement(
  div,
  { className: "name", age: "12" },
  React.createElement(div, {}, "1"),
  React.createElement(div, {}, "2"),
  React.createElement(div, {}, "3")
);
```

2. React.createElement() 将节点转换成 VDOM 形式
```javascript
// createElement 的作用就是重新包装数据格式，生成 vDom JS 数据格式
function createElement(type, props, ...children) {
  return {
    type,
    props: {
      ...props,
      children
    },
  };
}
```

3. 版本一：通过 React.render() 将 VDOM 渲染到页面上，这个过程是不断递归的，不能中断的
```javascript
// render 调用原生创建节点方法递归创建真实 DOM
function render(vDom, container) {
  let dom;
  if (typeof vDom !== "object") {
    dom = document.createTextNode(vDom);
  } else {
    dom = document.createElement(vDom.type);
  }

  if (vDom.props) {
    Object.keys(vDom.props)
      .filter((key) => key != "children")
      .forEach((item) => (dom[item] = vDom.props[item]));
  }

  if (vDom.props && vDom.props.children && vDom.props.children.length) {
    vDom.props.children.forEach((child) => render(child, dom));
  }
  
  container.appendChild(dom);
}
```

4. 版本二：使用 Fiber 使得 VDOM 到渲染成真实DOM 的过程是可中断的
   - 在 render() 中初始化第一个工作单元 fiber，当浏览器空闲时执行事件循环
   - 存在下一执行单元 fiber  且一帧还有剩余时间，不断执行剩下的单元
   - 还有未执行单元，但一帧时间不够了，就中断，等下一帧继续处理 fiber
   - fiber-reconcile：当层 dom 在内存中不存在旧 `workInProgressFiber.alternate`，则循环创建新 fiber，不需要 diff
   - fiber-reconcile-diff：不断循环链表比对：新旧节点类型一样，复用旧 fibre，标记为 更新；新旧节点类型不一样，但存在新节点，则创建新节点替换老节点；新旧节点不一样，没新节点，但有老节点，删除老节点；
   - 当所有单元执行完成后，统一渲染：递归 fiber 树，根据 fiber 节点的类型创建/更新/删除真实 DOM
## Api
### useState
```javascript
// 使用
function App(){
  const [name, setName] = useState('')
  return <div>{name}</div>
}
```

1. useState 接收一个初始值，返回一个数组，且 App() 作为函数组件，每次 render 都会重新运行，所以 state 不是一个局部变量，而且一个全局变量
```javascript
// 手写 useState：更新 state, 触发 render
// 将 hook.state 存在每个对应的 fiber 节点上
let wipFiber = null;
let hookIndex = null;
function useState(init) {
  const oldHook = wipFiber.alternate?.hooks?.[hookIndex];
  const hook = {
    state: oldHook ? oldHook.state : init,
  };
  wipFiber.hooks.push(hook);
  hookIndex++;
  const setState = (value) => {
    hook.state = value;
    workInProgressRoot = {
      dom: currentRoot.dom,
      props: currentRoot.props,
      alternate: currentRoot,
    };
    nextUnitOfWork = workInProgressRoot;
    deletions = [];
  };
  return [hook.state, setState];
}

function updateFunctionComponent(fiber) {
  // 支持useState，初始化变量
  wipFiber = fiber;
  hookIndex = 0;
  wipFiber.hooks = [];        // hooks用来存储具体的state序列 
  // ......下面代码省略......
}
```

2. useState()/setState() 多个一起执行，不会触发多次更新，内部会将多次优化为一次更新（批量更新）
3. 本质来说，useState 是预置了 reducer 的 useReducer
### useReducer
### useRef

1. useRef、createRef 
   - createRef 每次渲染都会返回一个新的引用，useRef 总是返回相同引用
   - useRef 不仅仅用来管理 DOM，它还相当于 this, 可以存放任何变量（给一个组件内全局的地方存放数据）
   - useRef 可以很好解决闭包带来的不方便
   - useRef 内容发生变化时不会通知，更新 .current 属性不会导致重新呈现，因为它一直是一个引用
```javascript
//   createRef 每次渲染都会返回一个新的引用，useRef 总是返回相同引用
function App() {
  const [count, setCount] = useState(1);
  const createInputRef = createRef();
  const useInputRef = useRef();
  const handleClick = () => {
    setCount(count + 1);
  };
  return (
    <div>
      {/* 每次 click, 此值都会 +1 */}
      <div ref={createInputRef}>createInputRef: {createInputRef.current}</div>
      {/* 每次 click, 此值都保持初始化值 1 */}
      <div ref={useInputRef}>useInputRef: {useInputRef.current}</div>
      <button onclick={handleClick}>click</button>
    </div>
  );
}
```
### useCallback

1. 功能：实现函数的缓存，该回调函数仅在某个依赖项改变时才会更新
```javascript
// 用法
const memoizedCallback = useCallback(
  ()=>{ console.log(state1, state2) },
  [state1, state2]
)
```
### memo

1. React.memo() 是一个高阶组件，接收组件 A 作为参数并返回组件 B，当 A 的 props 值没有改变，则 B 会组件 A 重渲染
```javascript
// memo.tsx
import { useRef, memo } from 'react';
const MemoTest = () => {
  // count 代表重渲染次数
  const count = useRef(0);
  return <div>render count: {count.current++}</div>;
};
// 使用 memo， Home 中 value 改变不会导致 MemoTest count 增加
// 不使用 memo， Home 中 value 改变会导致 MemoTest count 增加
export default memo(MemoTest);

// home.tsx 调用组件 MemoTest
const Home = () => {
  const [value, setValue] = useState("");
  return <>
    <Input value={value} onChange={(e)=>setValue(e.target.value)}/>
    <MemoTest />
    </>;
};
export default memo(MemoTest);
```
### useMemo

1. 功能：实现函数执行结果的缓存，如果 deps 没变，就直接拿之前的，否则才会执行函数拿到最新结果返回。
2. memo 和 useMemo 的区别
   - React.memo() 是一个高阶组件，可以用来包装不想重渲染的组件，除非其中的 props 发生变化
   - useMemo 是 React Hook，可以用来在组件中包装函数，确保该函数中的值在其依赖项之一发生变化时才重新计算
### useEffect

