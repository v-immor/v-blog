---
title: JS 中的函数
date: 2022-06-09 23:13
categories:
    - [javascripts]
tags:
  - JS 基础
---

## 函数是 JavaScript 世界的一等公民
所谓“一等公民”即可以享受世界规则内任何权益，并且在与其他等级公民享受同一权益时优先级更高。那么，如何在代码中去理解“一等公民”呢？作为 JSer 都清楚在 JavaScript 世界存在“变量提升”的规则。
```typescript
var a = 2

c()

var c = 2

b() // 这里执行将会报错：TypeError: b is not a function

var b = function() {}

function c() {
  console.log(b)
}

// 以上代码执行时的顺序，仅仅只是执行时的模拟顺序，JS 的词法分析并不会去改变的你的代码顺序！
function c() {}
var a = undefined
var b = undefined

a = 2

c()

c = 2

b()

b = function() {}

```
> 辟谣：
> 1. js 会将变量的声明提升到 js 顶部执行，对于 var a = 2 这种语句，会拆分开，var a 和 a = 2 分开执行。 -- 前半句是有歧义的，JS 是解释型语言，不会去改变你的代码顺序。
> 2. 当词法分析发现已经被定义的变量时，将不会执行“初始化”(var c = undefined)

## 函数的类型
#### 声明函数
**使用 function 定义的函数，JS 世界的一等公民，提前使用不会报错，存在函数提升。**
```javascript
function fn() {}
```
#### 变量函数（函数表达式）
**使用变量关键字(var/const/let) + function 定义的函数。**
**JS 世界的被贬的一等公民，提前使用将会报错，存在变量提升。**
```javascript
// JS 世界的被贬的一等公民，提前使用将会报错，存在变量提升
var fn = function() {}

// 等价于
var fn = undefined
fn = function() {}
// 故提前使用时， fn 为 undefined
```
#### 匿名函数
**除了使用 function 定义没有名字的声明函数，变量函数和箭头函数也属于匿名函数**
```javascript
// 没有名字的函数
function() {}

// 变量函数也属于匿名函数
var fn = function() {}

// 箭头函数自然也是匿名函数
var fn2 = () => {}
```
#### 箭头函数

- 没有自己的 this，只能从父作用域中继承 this。
- 在定义时就能确定 this 的指向，使用时不会改变 this 指向，call、apply、bind 都不会改变 this 的指向。
- 没有 arguments 对象，即使在箭头函数中使用 arguments 也只是从父作用域中取得的 arguments 对象。
- 箭头函数没有原型 prototype。
- 箭头函数中没有 yield 关键字，不能作为 generator 函数使用。
```javascript
// ES6 的快乐之一
var arrowFn = () => {}
```
#### 构造函数
**声明函数和使用 function 函数表达式定义的函数都可以作为构造函数，但箭头函数不可以作为构造函数。**
**鉴于开发习惯，通常把作为构造函数的名称定义时将首字母大写。**
**可被 new 关键字使用。**
> **构造函数主要通过 this 的应用来实现实例化的结果，而箭头函数没有自己的 this，同时箭头函数也没有 prototype 属性，无法将该属性赋给实例对象的 __proto__。**

```javascript
function A() {}
var a = new A()

var B = function (){}
var b = new B()

var D = () => {}
var d = new D() // Uncaught TypeError: D is not a constructor
```
#### 立即执行函数（IIFE）
**经常用于第三方库，用于模拟块级作用域，在一个函数表达式内部声明变量，然后立即调用这个函数，这样位于函数体作用域的变量就像是在块级作用域中一样。**
```javascript
// 一般写法
(function(){})()
// OR
(function(){}())
//OR
(() => {})()

// 如果函数结尾直接添加()执行，将会报错，因为不符合js的语法，想让其通过浏览器的语法检查，就必须添加符号，比如：()、+、!等
function test(){
  console.log("测试");
}(); // 报错 SyntaxError: Unexpected token ')'

+function test() {
  console.log("测试");
}(); // 正常执行

-function test() {
  console.log("测试");
}(); // 正常执行

!function test() {
  console.log("测试");
}();  // 正常执行

~function test() {
  console.log("测试");
}();  // 正常执行

void function test() {
  console.log("测试");
}();  // 正常执行

new function test() {
  console.log("测试");
}();  // 正常执行
```
## 函数奇怪的作用域
函数在浏览器环境和 node 环境下的作用域有着些许的不同。
```javascript
// 非严格模式的浏览器环境下
console.log(c in window, window.c, typeof c) // true undefined 'undefined'
{
  console.log(c in window, window.c, typeof c) // false undefined 'function'
  function c(){}
  console.log(c in window, window.c, typeof c) // false ƒ c(){} 'function'
}
console.log(c in window, window.c, typeof c) // false ƒ c(){} 'function'

// 在 node 环境或严格模式下
console.log(c in window, window.c, typeof c) // Uncaught ReferenceError: c is not defined
{
  console.log(c in window, window.c, typeof c) // false undefined 'function'
  function c(){}
  console.log(c in window, window.c, typeof c) // false undefined 'function'
}
console.log(c in window, window.c, typeof c) // Uncaught ReferenceError: c is not defined
```
这明确表明 JS 设计时是没有块级作用域的，JS 只有执行环境和作用域链。[JS 中的执行环境与作用域链](https://www.yuque.com/visionking/sxnyaw/cxqwa7)
