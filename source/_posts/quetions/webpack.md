---
title: 工程化
date: '2023-02-24 15:00:51'
quiz: true
categories:
  - Quetions
tags:
  - 面试
abbrlink: 764f33bc
---

## 基础

1. 什么是`模块化`? 有哪些标准？都有什么区别 {.quiz .fill}

2. 什么是`工程化`？ {.quiz .fill}

3. `import` 和 `require` 的区别？ {.quiz .fill}

   > - `require` 是 `CommonJS` 的标准，引入模块是运行时同步执行的，可以在运行时使用，引入是模块的值拷贝，不会共享模块的变量；搭配的导出语法是`moudle.exports`
   > - `import` 是 `ES Module` 的标准，引入模块是编译时异步加载的，早期的版本不允许在运行时调用，引入的是模块的值引用，可以共享模块变量；导出语法是`export`

## NPM

1. `npm` 的安装流程 {.quiz .fill}

   > - `npm install` -> 查找 `package.json` -> 获取远程包信息 -> 构建依赖结构 -> 查找缓存 -> 下载远程包 -> 添加到缓存 -> 添加到 `node_modules` -> 生成/更新 `package-lock.json`;
   > - `npm install` -> 比对 `package.json` 和 `package-lock.json` -> 查找缓存 -> 添加到 `node_modules`

2. `npm` 如何解决依赖冲突? {.quiz .fill}

   > - 在依赖的包中新增 `node_modules`，项目中的依赖优先置于顶层包，其余相同依赖会在对应的包下；

3. `npm`、`yarn`、`pnpm` 的区别 {.quiz .fill}

4. 为什么需要 `.lock` 文件，它的作用是什么 {.quiz .fill}

## Webpack

0. `Webpack` 能做什么? {.quiz .fill}

   > 1. 模块打包。
   > 2. 编译兼容。 -> 通过 `webpack` 置入兼容代码可以将一些兼容代码隔离出项目，可以将浏览器无法理解的文件转换成可理解的文件。
   > 3. 能力扩展。 -> 通过 `webpack` 的插件机制，可以在实现代码兼容和模块化的基础上，实现`按需加载`、`代码压缩`、`移除冗余代码`等功能。

1. `Webpack` 打包原理 {.quiz .fill}

   > - 在 `webpack` 中会将所有文件都视为`模块`，`webpack` 的职责就是将这些`模块`经过 `loader` 的文件转换和 `plugin` 的处理打包输出成客户端可理解运行的静态资源。
   > - `webpack` 的运行逻辑是
   >
   > 1. 初始化配置参数；
   > 2. 通过配置参数，初始化 `Compiler` 对象，加载所有配置的插件，执行 `run()` ，开始编译；
   > 3. 确定入口模块，从入口模块开始，按顺序调用所有的 `loader` 对模块进行转换，并分析出模块的依赖关系，直到所有依赖的模块都经过这些处理。
   > 4. 经过上一步的处理可以得到每个模块的翻译的最终内容，以及模块的依赖关系；
   > 5. 根据模块的依赖关系，形成一个个 `Chunk` 内容，再把 `Chunk` 转化成文件，加入到输出列表。
   > 6. 将上一步的输出列表，结合配置参数，确定输出的路径和文件名，写入到文件系统中。

2. `plugin` 和 `loader` 的区别 {.quiz .fill}

   > 1. `plugin` 会在 `Complier` 对象初始化的时候加载，在 `webpack` 运行的特定时期触发，`loader` 是在进行模块分析时按顺序执行，用于翻译转化文件。
   > 2. `loader` 只用于文件转换，`plugin` 则基于`发布-订阅`模式，通过监听不同的事件，来执行不同的操作。
   > 3. `loader` 的执行是同步的，上一个 `loader` 的执行结果会作为下一个 `loader` 的入参；`plugin` 则有同步和异步两种执行机制。

3. 常用的 `loader` 和 `plugin` 有哪些？他们的作用分别是什么？{.quiz .fill}

   > - loader：`css-loader`、`style-loader`、`less-loader`、`sass-loader`、`jsx-loader`、`image-loader`、`thread-loader`
   > - plugin: `html-webpack-plugin`、`mini-css-extract-plugin`、`copy-webpack-plugin`、`happypack`、`commons-chunk-plugin`、`hot-module-replacement-plugin`

4. 描述 `Webpack` 的`热更新`原理 {.quiz .fill}

   > > webpack 的热更新是由 `hot-module-replacement-plugin` 提供能力，在 webpackV5 时内置为 module.hot 上；
   >
   > 1. `HMR` 提供了 `HMR Server` 和 `HMR runtime` 两个部分；
   > 2. `HMR Server` 部分是 `webpack-dev-middleware` 开发服务器的中间件。通过监听 `webpack` 的 `compiler done` 将编译后的模块新 hash 通过 `websocket` 发送给浏览器的 `HMR runtime` 部分进行 hash 比较后，若不一致则通过模块新 hash 请求 `manifest` 文件确认模块变更的范围列表，再请求对应的`[hash].hot-update.js`，再进行替换和执行；
   > 3. `HMR runtime` 会执行 `module.hot.accept` 的回调函数进行热替换，所以在需要热替换的 JS 需要手动添加 `module.hot.accept` 逻辑。通常只需要在应用入口进行添加就够用了。对于样式文件，在 loader 中已经添加了 `module.hot.accept` 逻辑，所以只需添加相应的 loader 也会开启热替换的功能；
   > 4. 如果在模块替换或者执行的时候出现了意外，就会用 reload 兜底，刷新整个页面

5. 什么是`联邦模块`？ {.quiz .fill}

## Vite

1. `Vite` 是什么？ {.quiz .fill}

2. `Vite` 的核心原理 {.quiz .fill}

3. `Vite` 对比 `Webpack` 的优点和缺点 {.quiz .fill}

4. `Vite` 能输出什么格式的代码 {.quiz .fill}

## Rollup

1. `Rollup` 是什么？ {.quiz .fill}

2. `Rollup` 的打包流程 {.quiz .fill}

3. `Rollup` 的插件机制 {.quiz .fill}

4. `Rollup` 常用的插件 {.quiz .fill}

## 微前端

1. `JS 沙箱` 的实现方式有几种，原理是？ {.quiz .fill}

   > 1. iframe `sandbox`, `contentWindow` 天然沙箱，iframe 必须同域才能够使用 contentWindow 属性
   > 2. with + new Function 代理沙箱
   > 3. 快照沙箱，在运行时将 window 对象进行一个快照拷贝，将 window 上所有的属性复制到快照对象上，当子应用卸载时，将当前 window 对象与快照对象进行 diff ，将变更的属性使用 map 保存起来，便于下一次挂载时再次进行还原。通过 `Proxy` 也能够监听到属性修改，用 `Proxy` 实现的也叫代理沙箱；

2. CSS `样式隔离` 的实现方式以及原理？ {.quiz .fill}

   > 1. 没有开启 `shadowDOM` 模式下，`micro-app` 默认会为每一个样式加上 `micro-app[name=xxx]` 的前缀作为样式隔离；
   > 2. 可以在 `microApp.start()` 中配置全局禁用，也可以在 `micro-app` 的 `disableScopecss` 属性进行单个应用的禁用。在样式文件中可以通过添加注释 `! scopecss-disable` 来禁用单个样式的隔离。
   > 3. 开启 `shadowDOM` 模式后样式将不进行隔离处理，因为 `shadowDOM` 天然隔离样式。

3. `micro-app` 元素隔离原理 {.quiz .fill}

   > 1. `micro-app` 是通过修改 `document` 原型链上的 `querySelector` 实现的元素隔离，当子应用调用时，会存在 `appName`，则会调用子应用的 `querySelector`，当主应用访问 `document` 的属性时，通过拦截将 `appName` 置空，从而访问到原始的 `querySelector`
   > 2. 基座应用是可以访问到子应用的元素的，基座应用在没有开启 `shadowDOM` 模式下可以访问到所有的元素；
   > 3. 在子应用卸载时，元素隔离的解除是异步的，可能会造成渲染异常，可以通过手动调用 `window.removeDomScope()` 来解除元素的绑定。

4. 微前端中应用间的通讯方式 {.quiz .fill}

   > 1. 通过基座应用的 `state` 变化，传递 `props` 来更新子应用；
   > 2. 通过`发布-订阅`模式，通过事件订阅来通讯；
   > 3. `MessageChannel`
