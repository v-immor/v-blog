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

## NPM

1. `npm` 的安装流程 {.quiz .fill}

2. `npm`、`yarn`、`pnpm` 的区别 {.quiz .fill}

3. 为什么需要 `.lock` 文件，它的作用是什么 {.quiz .fill}

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

4. 描述 `Webpack` 的`热更新`原理 {.quiz .fill}

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