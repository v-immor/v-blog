---
title: 离开回调地狱，进入 Promise 时代
date: 2021-02-25 22:10
categories:
  - [javascripts, promise]
tags:
  - Promise
  - 原理
---

### 简介
在 JavaScript的世界中，所有代码都是单线程执行的。由于这个“缺陷”，导致JavaScript的所有网络操作，浏览器事件，都必须是异步执行，所以所期望的结果往往就会不一样，由于JS灵活性，便产生了callback回调函数，将函数作为参数，以获得期望结果，这在简单的异步操作中有很好的表现，但在复杂/多层异步中就会变得恐怖起来，代码将会看起来非常臃肿，且不易维护，这就是“回调地狱”。为了解决“回调地狱”，开发者就提出“Promise”对象，在ES6被统一规范之前，Promise就已经有各种开源实现了。并且在大多数现代浏览器下都是支持Promise对象，即使是少数不支持Promise的浏览器，在开源社区中也有丰富的解决方案。所以可以放心大胆地去使用Promise，规避“回调地狱”。

      Promise 必须为以下三种状态之一：等待态（Pending）、执行态（Fulfilled）和拒绝态（Rejected）。一旦 Promise 被 resolve 或 reject，不能再迁移至其他任何状态（即状态 immutable）。
**基本过程**：

1. 初始化 Promise 状态（pending）
2. 执行 then(..) 注册回调处理数组（then 方法可被同一个 promise 调用多次）
3. 立即执行 Promise 中传入的 fn 函数，将Promise 内部 resolve、reject 函数作为参数传递给 fn ，按事件机制时机处理
4. Promise里的关键是要保证，then方法传入的参数 onFulfilled 和 onRejected，必须在then方法被调用的那一轮事件循环之后的新执行栈中执行。
> **真正的链式Promise是指在当前promise达到fulfilled状态后，即开始进行下一个promise.**
> **本篇不涉及原理和实现，仅对Promise的API，和相应****常见****使用介绍。**

### API

| **API** | **描述** | **备注** |
| :---: | :---: | :---: |
| then | resolve的状态下的回调函数 |  |
| catch | rejected的状态回调 |  |
| all | 将多个 Promise 实例，包装成一个新的 Promise 实例，返回一个按参数数组顺序的数组 |  |
| allSettled | 返回一个promise，该promise在所有给定的promise已被解析或被拒绝后解析，并且每个对象都描述每个promise的结果。 | 提案中，预计加入ES2020 |
| race | 将多个Promise实例包装为一个实例,返回值取最先进入完成状态的实例 | 不支持IE |
| resolve | 实例状态变更为成功/接受,可调用实例的then方法获取传入值 |  |
| reject | 实例状态变更为失败/拒绝,可调用实例的catch方法获取传入值 |  |
| finally | 无论实例状态是接受还是拒绝，在链式调用最终会调用的函数，值取决于实例的状态 | ES9加入标准，兼容性较差 |


### 使用场景

1. then
```javascript
var promise1 = new Promise(function(resolve, reject) {
  setTimeout(function() {
    resolve('foo');
  }, 300);
});

promise1.then(function(value) {
  // expected output: "foo"
  console.log(value);
});
// expected output: [object Promise]
console.log(promise1);
```

2. catch
```javascript
var p1 = new Promise(function(resolve, reject) {
  resolve('Success');
});

p1.then(function(value) {
  console.log(value); // "Success!"
  throw 'oh, no!';
}).catch(function(e) {
  console.log(e); // "oh, no!"
}).then(function(){
  console.log('after a catch the chain is restored');
}, function () {
  console.log('Not fired due to the catch');
});

// 以下行为与上述相同
p1.then(function(value) {
  console.log(value); // "Success!"
  return Promise.reject('oh, no!');
}).catch(function(e) {
  console.log(e); // "oh, no!"
}).then(function(){
  console.log('after a catch the chain is restored');
}, function () {
  console.log('Not fired due to the catch');
});
```
> PS: 若实例状态没有调用过reject，则catch永远不会调用

```javascript
// 抛出一个错误，大多数时候将调用catch方法
var p1 = new Promise(function(resolve, reject) {
  throw 'Uh-oh!';
});

p1.catch(function(e) {
  console.log(e); // "Uh-oh!"
});

// 在异步函数中抛出的错误不会被catch捕获到
var p2 = new Promise(function(resolve, reject) {
  setTimeout(function() {
    throw 'Uncaught Exception!';
  }, 1000);
});

p2.catch(function(e) {
  console.log(e); // 不会执行
});

// 在resolve()后面抛出的错误会被忽略
var p3 = new Promise(function(resolve, reject) {
  resolve();
  throw 'Silenced Exception!';
});

p3.catch(function(e) {
   console.log(e); // 不会执行
});
```

3. all
```javascript
var promise1 = Promise.resolve(3);
var promise2 = 42;
var promise3 = new Promise(function(resolve, reject) {
  setTimeout(resolve, 100, 'foo');
});

Promise.all([promise1, promise2, promise3]).then(function(values) {
  console.log(values);
});
// expected output: Array [3, 42, "foo"]
```
> - 如果传入的参数是一个空的可迭代对象，则返回一个**已完成（already resolved）**状态的 [`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)。
> - 如果传入的参数不包含任何 `promise`，则返回一个**异步完成（asynchronously resolved）** [`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)。注意：Google Chrome 58 在这种情况下返回一个**已完成（already resolved）**状态的 [`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)。
> - 其它情况下返回一个**处理中（pending）**的[`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)。这个返回的 `promise` 之后会在所有的 `promise` 都完成或有一个 `promise` 失败时**异步**地变为完成或失败。 见下方关于“Promise.all 的异步或同步”示例。返回值将会按照参数内的 `promise` 顺序排列，而不是由调用 `promise` 的完成顺序决定。


4. allSettled
```javascript
const promise1 = Promise.resolve(3);
const promise2 = new Promise((resolve, reject) => setTimeout(reject, 100, 'foo'));
const promises = [promise1, promise2];

Promise.allSettled(promises).
  then((results) => results.forEach((result) => console.log(result.status)));

// expected output:
// "fulfilled"
// "rejected"
```
> 一个**悬而未决** [`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)将被**异步**完成一次承诺的指定集合在每一个承诺已经完成，无论是成功的达成或通过被拒绝。那时，返回的promise的处理程序作为输入传递一个数组，该数组包含原始promises集中每个promise的结果.




5. race
```javascript
var promise1 = new Promise(function(resolve, reject) {
    setTimeout(resolve, 500, 'one');
});

var promise2 = new Promise(function(resolve, reject) {
    setTimeout(resolve, 100, 'two');
});

Promise.race([promise1, promise2]).then(function(value) {
  console.log(value);
  // Both resolve, but promise2 is faster
});
// expected output: "two"
```
> 一个**待定的** [`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise) 只要给定的迭代中的一个promise解决或拒绝，就采用第一个promise的值作为它的值，从而**异步**地解析或拒绝（一旦堆栈为空）。




6. resolve
```javascript
var promise1 = Promise.resolve(123);

promise1.then(function(value) {
  console.log(value);
  // expected output: 123
});
```
> 返回一个解析过带着给定值的`Promise`对象，如果返回值是一个promise对象，则直接返回这个Promise对象。




7. reject
```javascript

Promise.reject("Testing static reject").then(function(reason) {
  // 未被调用
}, function(reason) {
  console.log(reason); // "Testing static reject"
});

Promise.reject(new Error("fail")).then(function(result) {
  // 未被调用
}, function(error) {
  console.log(error); // stacktrace
});
```
> 静态函数`Promise.reject`返回一个被拒绝的`Promise对象`。通过使用[`Error`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Error)的实例获取错误原因`reason`对调试和选择性错误捕捉很有帮助。




8. finally
```javascript
let doing = false;

var p1 = new Promise((resolve,reject)=>{
	// doing
  	if(doing){
    	resolve('success');
    }else{
    	reject('fail')
    }
})

p1.then((data)=>{
    // success
	console.log(data)
}).catch((err)=>{
    //fail
console.log(err)
}).finally((data)=>{
    // success OR fail
	console.log(data)
})
```
> 返回一个[`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)。在promise结束时，无论结果是fulfilled或者是rejected，都会执行指定的回调函数。这为在`Promise`是否成功完成后都需要执行的代码提供了一种方式。
> 这避免了同样的语句需要在[`then()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/then)和[`catch()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/catch)中各写一次的情况。


#### 实际应用
主要讲all、catch、finally在React实际场景中的使用

1. 场景一：当一个页面有多个请求数据需要同时加载时，可使用all来减少setState的次数来减少render，达到性能优化的目的。
```javascript
// Promise
const quest1,quest2,quest3,quest4;

const p = Promise.all([quest1,quest2,quest3,quest4]);

// 回调是一个数组参数，顺序为all数组参数顺序
// 若不使用all，单独处理数据则需要setState四次，发生render四次，浪费性能
p.then(([data1,data2,data3,data4])=>{
  // doing
	this.setState({
  	data1,data2,data3,data4
  })
});
```

2. 场景二：当一个请求需要有loading状态来处理时，可使用catch来处理请求失败后的处理方式，使用finally来处理必然执行函数(loading 状态，当请求完成时必然需要改变，无论请求失败还是成功都必须改变loading)
```javascript
// Promise
const quest1;

const p = Promise.all([quest1,quest2,quest3,quest4]);

// 回调是一个数组参数，顺序为all数组参数顺序
// 若不使用all，单独处理数据则需要setState四次，发生render四次，浪费性能
quest1.then(data=>{
  // success doing
  console.log('请求成功');
}).catch(err=>{
	//fail doing
	console.log('请求失败');
}).finally(res=>{
	this.setState({
  	loading:false
  })	
})
```

### 兼容性处理
大部分现代浏览器都支持ES6，少部分(IE，手机浏览器)对Promise不支持，可以通过第三方插件bluebird或者使用polyfill解决兼容性问题，具体使用方法可百度Get。下图为Promise的兼容性图表

![](https://images.weserv.nl/?url=https://cdn.nlark.com/yuque/0/2019/png/412560/1571845552462-0c39ab96-4150-466b-87fd-07b54348b170.png#align=left&display=inline&height=363&originHeight=363&originWidth=800&size=0&status=done&width=800)

> 原理及实现Promise将在下一节中进行介绍

