---
title: 如何满足 React 组件开发的最佳实践？
date: '2022-03-07 14:32'
categories:
  - - React
    - 开发实践
abbrlink: 1750c17f
---

> 本篇主要内容
> - [x] 怎样的才能称为最佳实践
> - [x] 如何拆分组件

## 什么是 React？
> React 是一个用于构建用户界面的声明式的 JavaScript 库。
> 组件化: 构建管理自身状态的封装组件，然后对其组合以构成复杂的 UI。

## React 组件化的哲学
> [https://zh-hans.reactjs.org/docs/thinking-in-react.html](https://zh-hans.reactjs.org/docs/thinking-in-react.html)
> 满足 React 组件化哲学的实现就可以称之为“最佳实践”了。

从拿到设计稿开始，我们大致需要 5 步就可以构建出我们的应用了。

### **第一步：将设计好的 UI 划分为组件层级**

**将组件当作一种函数或者是对象来考虑，根据单一功能原则来判定组件的范围。**
**举个例子：**
![image.png](https://cdn.nlark.com/yuque/0/2022/png/412560/1649513238571-f6e7c569-2807-405f-b044-95bd075fe4d4.png#averageHue=%23cfd3a0&clientId=u4af3e3d1-f3b2-4&from=paste&height=915&id=ubc84a2f1&name=image.png&originHeight=1830&originWidth=870&originalType=binary&ratio=1&rotation=0&showTitle=false&size=240165&status=done&style=none&taskId=u85c38b9e-de84-4c20-8e06-5a04c0de072&title=&width=435)
**如何控制细粒度？**

- 是否存在交互？交互影响的范围？
- 是否存在共性？
> 1. 静态视图，大部分情况可以不抽成组件
> 2. 纯展示类可以抽离成纯组件，视图的变更交给 props
> 3. 存在复杂交互、或状态变更频繁的，可以抽离成组件。
> 4. 在以上三点考虑完之后，根据业务的复杂度可以考虑复合组件

按我个人的理解，按照用途可以将组件级别大致分为三类。以 antd 举例

1. 基础组件
   1. 提供基础的外观以及可以一定程度的自定义外观
   2. 拥有基础的事件
   3. 是可以接受完全控制的

例：Button、Input、Textarea、Radio、CheckBox、Message、Layout...

2. 复合组件
   1. 由多个基础组件复合而成
   2. 具有多个事件控制
   3. 能够接受一定程度自定义

例：Modal、TreeSelect、AutoComplete、Menu、Upload

3. 业务组件
   1. 与具体业务流程强耦合，或拥有独立的业务。
   2. 能够接受一定程度外观自定义。

例：用户选择器、企业选择器...

### **第二步：用 React 创建一个静态版本**

如果不是非常明确状态的流动、那么就把状态先丢掉，先实现组件的外观。然后再根据业务的需求来判断数据的流动轨迹。

### **第三步：确定 UI state 的最小（且完整）表示**

**如何确定组件的 state？先给自己一个致命三连。**

1. **该数据是否是由父组件通过 props 传递而来的？如果是，那它应该不是 state。**
2. **该数据是否随时间的推移而保持不变？如果是，那它应该也不是 state。**
3. **你能否根据其他 state 或 props 计算出该数据的值？如果是，那它也不是 state。**

### **第四步：确定 state 放置的位置**

我们已经确定了应用所需的 state 的最小集合。接下来，我们需要确定哪个组件能够改变这些 state，或者说拥有这些 state。

**注意：React 中的数据流是单向的，并顺着组件层级从上往下传递。哪个组件应该拥有某个 state 这件事，对初学者来说往往是最难理解的部分。尽管这可能在一开始不是那么清晰，但你可以尝试通过以下步骤来判断：**

**对于应用中的每一个 state：**

1. **找到根据这个 state 进行渲染的所有组件。**
2. **找到他们的共同所有者（common owner）组件（在组件层级上高于所有需要该 state 的组件）。**
3. **该共同所有者组件或者比它层级更高的组件应该拥有该 state。**
4. **如果你找不到一个合适的位置来存放该 state，就可以直接创建一个新的组件来存放该 state，并将这一新组件置于高于共同所有者组件层级的位置。**




### **第五步：添加反向数据流**
> React 通过一种比传统的双向绑定略微繁琐的方法来实现反向数据传递。

即如果组件接受 props 的控制，且需要在组件内部去控制 props ，则必须在 props 中传入控制的函数，而不能直接修改 props，或者通过内部 state 拦截 props 导致状态不一致。

这是 Vue 或者响应式框架诟病 React 的地方，相对于响应式的直接，这种反向数据流显得不是那么的优雅。尽管如此，这种需要显式声明的方法明前更有助于人们理解程序的运作方式。

满足 React 哲学原则的组件设计可以认为是“最佳实践”。同时也能满足最基本的“如何拆分组件”需求。

