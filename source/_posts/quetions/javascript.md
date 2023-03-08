---
title: JavaScript
date: '2023-02-24 23:04:24'
quiz: true
categories:
  - Quetions
tags:
  - 面试
abbrlink: cc1b9611
---

## JS 基础

1.  JS 的数据类型有哪些？ {.quiz .fill}

    > 1. 基本数据类型 (`string`、`number`、`boolean`、`null`、`undefined`、`symbol`、`bigInt`)
    > 2. 引用数据类型（`Object`、`Function`、`Date`、`Array`）
    > 3. 基本数据类型的存放在栈内存中，引用数据类型的引用指针存放于栈内存，内容存放于堆内存中。

2.  JS 的继承方式，各种继承方式的优缺点及实现方式 {.quiz .fill}

    > - 1. `原型链继承`；2. `借用构造函数继承`；3. `组合继承`；4. `原型式继承`；5. `寄生式继承`；6. `寄生组合式继承`；

3.  JS 的执行环境和作用域、作用域链 {.quiz .fill}

4.  `new` 的原理和实现 {.quiz .fill}

    > - `new` 主要是将一个新对象的原型指向构造函数的原型，并在这个对象上注入属性后将其返回。
    > - 实现 `new` 的步骤有
    >
    >   1. 创建一个新对象。
    >   2. 把这个对象的原型指向构造函数的原型
    >   3. 通过构造函数将属性初始化
    >   4. 返回这个对象。

    > ```js
    > function myNew(constructor, ...args) {
    >   const obj = {}
    >   obj.__proto__ = constructor.prototype
    >   constructor.apply(obj, args)
    >   return obj
    > }
    > ```

5.  `bind`、`call`、`apply` 的原理和实现 {.quiz .fill}

    > - 三者的本质都是通过改变 this 的指向来借用函数，不同的是 `bind` 是返回一个待调用的函数，`call` 和 `apply` 返回的是执行结果
    > - `bind`、`call`、`apply` 的第一个参数都是`借用方`，也就是函数执行时 this 的指向；
    > - `bind` 除了第一个参数外，剩余的参数会依次作为被借函数的入参，返回的函数的入参的顺序会在 `bind` 参数的后续依次入参；
    > - `call` 除了第一个参数外，剩余的参数会依次作为被借函数的入参；
    > - `apply` 的第二个参数是一个类数组，参数会按序作为被借用函数的入参；

    > ```js
    > function bind(context) {
    >   const ctx = context || {}
    >
    >   const fn = Symbol()
    >   ctx[fn] = this
    >   const args = arguments
    >
    >   return function () {
    >     return ctx[fn](...args, ...arguments)
    >   }
    > }
    >
    > function call(context) {
    >   const ctx = context || window
    >   const fn = Symbol()
    >   ctx[fn] = this
    >   const res = ctx[fn](...arguments)
    >
    >   delete ctx[fn]
    >   return res
    > }
    >
    > function apply(context, args) {
    >   const ctx = context || window
    >   const symbol = Symbol()
    >   ctx[symbol] = this
    >
    >   const res = ctx[symbol](...args)
    >
    >   delete ctx[symbol]
    >   return res
    > }
    > ```

6.  什么是原型，什么是原型链？ {.quiz .fill}

    > - 原型就是对象。JS 的对象系统是基于原型模式设计的，在原型模式里有一个原则是“要得到一个对象，不是通过实例化类，而是找到一个对象作为原型并克隆这个对象。”
    > - 原型链就是对象之间的关联，在 JS 中实例对象的 `__proto__` 会指向其构造函数的原型 `prototype`，构造函数的原型对象的 `__protot__` 也会指向这个原型对象构造函数的 `prototype`，一直到 `Object.prototype.__proto__` 为止，这种链式向上关联就是`原型链 `

7.  什么是闭包？有什么应用场景? {.quiz .fill}

    > - 闭包是指在函数内使用了非函数内的变量，导致这个变量无法在其作用域销毁时销毁的现象。
    > - 即使是创建这个变量上下文已经销毁，但这个变量它依旧存在内存中，这种现象就是`闭包`。
    > - 在代码中引用了自由变量也会造成闭包现象。
    > - 在闭包现象中，上下文的销毁时，它的`活动变量(AO)`不一定会从作用域链中移除，所以闭包函数可以通过作用域链访问到这个变量；

8.  `typeof` 和 `instanceOf` 的原理和区别 {.quiz .fill}

    > - `typeof` 判断的是对象在内存上的存储类型，`instanceOf` 判断的是原型链上的原型。
    > - `typeof` 只能判断除 `null` 外的基本数据类型和 `object`、`function`，无法判断实例的原型，`instanceOf` 不能判断 `number`、`string`、`boolean` 类型，可以判断实例的原型，可以区分 `Array` 和 `Object`。

9.  什么是变量提升 {.quiz .fill}

    > - 在变量未定义时就能够使用的现象就是`变量提升`

10. `this` 是什么？ {.quiz .fill}

    > 1. `this` 是函数执行的主体，如果将最外层当成一个 `main` 函数来看，那 `main` 的主体就是 `window` 或者是 `global`。
    > 2. 总的来说就是谁调用的函数 `this` 就会指向谁，箭头函数除外。

11. 描述 JS 的事件循环？{.quiz .fill}

12. 哪些任务是宏任务？哪些事件是微任务？宏任务和微任务的区别 {.quiz .fill}

    > - `setTimeout`、`setInterval`、`setImmediate`、`requestAnimationFrame`、`requestIdleCallback`、`MessageChannel` 属于宏任务，`Promise.then`、`MutationObserver`、`process.nextTick`
    > - 宏任务是在事件循环的起始执行的，微任务是在循环的末尾执行；
    > - `setTimeout`/`setInterval` 创建的宏任务，有一个最短 4ms 的延迟，也就是两个循环之间最短时间是 4ms；
    > - `requestAnimationFrame` 是浏览器按帧执行的，约为 16.6ms，若页签处于后台时，不更新帧则不会执行；
    > - `requestIdleCallback` 仅在浏览器渲染执行完还有剩余时间时会执行，没有时间则不执行；

13. 为什么 **for** 循环的性能比 **forEach** 高？ {.quiz .fill}

14. 描述一下 V8 引擎的`垃圾回收机制`，如何定位内存泄露？ {.quiz .fill}

## ES6

0. ES6 有哪些新特性？ {.quiz .fill}

   > ES6 主要的新特性有
   >
   > 1. 拥有 `块级作用域` 的 `const` 和 `let`
   > 2. `箭头函数`
   > 3. 便捷操作对象、数组的`扩展运算符`
   > 4. 用于处理异步任务的 `Promise`
   > 5. 定义类的 `Class` 语法糖
   > 6. 比 `definePrototype` 更加安全友好的 `Proxy` 代理对象
   > 7. 新的数据结构 `Set`、`WeakSet`、`Map`、`WeakMap`
   > 8. 新的基础数据类型 `Symbol`
   > 9. 一些数组方法...

1. `var`、`const`、`let` 有什么区别？如何用 var 模拟 const 和 let？ {.quiz .fill}

   > 1. `var` 定义的变量，可以在定义前使用，存在变量提升，`const` 和 `let` 定义的变量不能够在定义前使用，存在`暂时性死区`的概念。
   > 2. `const` 定义的变量不能修改， `let` 定义的变量可以修改。`const` 的不能修改指的是栈中引用不能够修改，对内部的属性依旧是可以修改的。
   > 3. `const` 和 `let` 声明的变量在同一作用域内不能重复声明，`var` 可以重复声明

   > ```js
   > var _scope = {}
   > var _variables = []
   >
   > for (let i = 0; i < _variables.length; i++) {
   >   var property = _variables[i]
   >   if (_scope.hasOwnPropertype(property)) {
   >     throw new SyntaxError('存在重复声明`' + key + '`变量')
   >   } else {
   >     Object.defineProperty(_scope, property, {
   >       configurable: true,
   >       enumerable: false,
   >       get() {
   >         throw new ReferenceError('变量`' + property + '`未定义')
   >       },
   >     })
   >   }
   > }
   >
   > function _checkScope(property) {
   >   if (_scope.hasOwnPropertype(property)) {
   >     throw new SyntaxError('不可重复声明' + key + '变量')
   >   }
   > }
   >
   > function _let(name, value) {
   >   _checkScope(name)
   >
   >   Object.defineProperty(_scope, name, {
   >     configurable: false,
   >     enumerable: false,
   >     get() {
   >       return value
   >     },
   >     set(val) {
   >       value = val
   >     },
   >   })
   > }
   >
   > function _const(name, value) {
   >   _checkScope(name)
   >
   >   Object.defineProperty(_scope, name, {
   >     configurable: false,
   >     enumerable: false,
   >     get() {
   >       return value
   >     },
   >     set() {
   >       throw new TypeError('不可修改变量')
   >     },
   >   })
   > }
   > ```

2. `箭头函数`和普通函数有什么区别？ {.quiz .fill}

   > - 箭头函数没有 `this` 指向的烦恼
   > - 箭头函数不能够作为构造函数使用，也就是不能够和 `new` 搭配使用
   > - 箭头函数默认是一种匿名函数

3. 什么是 `Promise`? {.quiz .fill}

   > - `Promise` 是 `ECMAScript` 提出解决 JS 中`回调地狱`问题的异步编程方案。
   > - `Promise` 认为任务执行后必然会存在一个结果，`成功`/`失败`。将任务的状态定义为 `pending`、`fulfilled`、`rejected` 三种状态，当任务从 `pending` 开始执行时，必然会到达 `fulfilled` 或者 `rejected` 的其中一个状态。
   > - 通过 `Promise` 定义的异步任务，在 `EventLoop` 中属于`微任务`，会在当前循环中完成执行。

4. 实现一个 `Promise` {.quiz .fill}

   > ```js
   > const FULFILLED = 'fulfilled'
   > const REJECTED = 'rejected'
   > const PENDING = 'pending'
   >
   > function scheduleCallBack(task) {
   >   setTimeout(task, 0)
   > }
   >
   > function resolvePromise(promise, x, resolve, reject) {
   >   if (x === promise) {
   >     return reject(new TypeError('Chaining cycle detected for promise'))
   >   }
   >
   >   if (x && (typeof x === 'object' || typeof x === 'function')) {
   >     let executed = false
   >     try {
   >       const then = x.then
   >       if (typeof then === 'function') {
   >         then.call(
   >           x,
   >           (y) => {
   >             if (executed) return
   >             executed = true
   >
   >             resolvePromise(promise, y, resolve, reject)
   >           },
   >           (e) => {
   >             if (executed) return
   >             executed = true
   >             reject(e)
   >           }
   >         )
   >       } else {
   >         resolve(x)
   >       }
   >     } catch (error) {
   >       if (executed) return
   >       executed = true
   >       reject(error)
   >     }
   >   } else {
   >     resolve(x)
   >   }
   > }
   >
   > class IPromise {
   >   status = PENDING
   >   value = null
   >   reason = null
   >   fulfilledCallbacks = []
   >   rejectedCallbacks = []
   >
   >   constructor(exec) {
   >     this.status = PENDING
   >     this.value = null
   >     this.reason = null
   >
   >     this.fulfilledCallbacks = []
   >     this.rejectedCallbacks = []
   >
   >     const that = this
   >
   >     function resolve(value) {
   >       if (value instanceof IPromise) {
   >         value.then(resolve, reject)
   >         return
   >       }
   >       if (that.status === PENDING) {
   >         that.value = value
   >         that.status = FULFILLED
   >         that.fulfilledCallbacks.forEach((cb) => cb())
   >       }
   >     }
   >
   >     function reject(reason) {
   >       if (that.status === PENDING) {
   >         that.reason = reason
   >         that.status = REJECTED
   >         that.rejectedCallbacks.forEach((cb) => cb())
   >       }
   >     }
   >
   >     try {
   >       exec(resolve, reject)
   >     } catch (error) {
   >       reject(error)
   >     }
   >   }
   >
   >   then(onFulfilled, onRejected) {
   >     // prettier-ignore
   >     onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : (value) => value
   >     // prettier-ignore
   >     onRejected = typeof onRejected === 'function' ? onRejected : (e) => { throw e }
   >
   >     const promise = new IPromise((resolve, reject) => {
   >       if (this.status === FULFILLED) {
   >         scheduleCallBack(() => {
   >           try {
   >             const x = onFulfilled(this.value)
   >             resolvePromise(promise, x, resolve, reject)
   >           } catch (error) {
   >             reject(error)
   >           }
   >         })
   >       } else if (this.status === REJECTED) {
   >         scheduleCallBack(() => {
   >           try {
   >             const x = onRejected(this.reason)
   >             resolvePromise(promise, x, resolve, reject)
   >           } catch (error) {
   >             reject(error)
   >           }
   >         })
   >       } else {
   >         this.fulfilledCallbacks.push(() => {
   >           scheduleCallBack(() => {
   >             try {
   >               const x = onFulfilled(this.value)
   >               resolvePromise(promise, x, resolve, reject)
   >             } catch (error) {
   >               reject(error)
   >             }
   >           })
   >         })
   >
   >         this.rejectedCallbacks.push(() => {
   >           scheduleCallBack(() => {
   >             try {
   >               const x = onRejected(this.reason)
   >               resolvePromise(promise, x, resolve, reject)
   >             } catch (error) {
   >               reject(error)
   >             }
   >           })
   >         })
   >       }
   >     })
   >
   >     return promise
   >   }
   > }
   > ```

5. 实现一下常见的 `Promise` 方法 {.quiz .fill}

   > ```js
   >  function race(promises) {
   >    return new IPromise((resolve, reject) => {
   >      for (let i = 0; i < promises.length; i++) {
   >        IPromise.resolve(promises[i]).then(resolve).catch(reject)
   >      }
   >    })
   >  }
   >
   >  function all(promises) {
   >    return new IPromise((resolve, reject) => {
   >      const res = []
   >      let len = 0
   >      for (let i = 0; i < promises.length; i++) {
   >        IPromise.resolve(promises[i]).then((val) => {
   >          res[i] = val
   >          if (++len === promises.length) resolve(res)
   >        }, reject)
   >      }
   >    })
   >  }
   >
   >  function resolve(value) {
   >    return new IPromise((resolve) => resolve(value))
   >  }
   >
   >  function reject(reason) {
   >    return new IPromise((res, rej) => rej(reason))
   >  }
   >
   >  function allSettled(promises) {
   >    return new IPromise((resolve, reject) => {
   >      const res = []
   >      let len = 0
   >      const plen = promises.length
   >
   >      for (let i = 0; i < plen; i++) {
   >        IPromise.resolve(promises[i]).then(
   >          (val) => {
   >            res[i] = {
   >              status: FULFILLED,
   >              value: val,
   >            }
   >            len++
   >            if (len === plen) resolve(res)
   >          },
   >          (reason) => {
   >            res[i] = {
   >              status: REJECTED,
   >              reason: reason,
   >            }
   >            len++
   >            if (len === plen) resolve(res)
   >          }
   >        )
   >      }
   >    })
   >  }
   >
   >  function catch(onRejected) {
   >    return this.then(null, onRejected)
   >  }
   >
   >  function finally(cb) {
   >    return this.then(
   >      (val) => IPromise.resolve(cb()).then(() => val),
   >      (reason) =>
   >        IPromise.reject(cb()).then(() => {
   >          throw reason
   >        })
   >    )
   >  }
   > ```

6. 什么是 `Proxy`？ {.quiz .fill}

   > - `Proxy` 是代理对象，通过在被代理对象上套一层拦截机制，让代理对象进行操作时可以做更多的事情；
   > - `Proxy` 是一个构造函数，一个入参是被代理对象，第二个是 `handler` 对象，通过拦截对象属性可以提供 `13` 种操作的拦截；
   > - `get`、`set`、`has`、`apply`、`construct`、`ownKeys`、`defineProperty`、`deleteProperty`、`getPrototypeOf`、`setPrototypeOf`、`getOwnPropertyDescriptor`、`preventExtensions`、`isExtensible`

7. 什么是 `Reflect`？ {.quiz .fill}

   > - `Reflect` 是反射对象，是 JS 的内置对象，提供 JS 对象操作的拦截方法，所有的方法都是静态方法，不能够进行实例化。
   > - `Reflect` 通常搭配 `Proxy` 一起使用，其内部方法也和 `Proxy` 的 hanlder 一一对应，主要是作用域 `Proxy` 中的 `Receiver` 让 `Proxy` 可以正确地处理 `Receiver` 的引用，保证代理对象在劫持时可以获取到正确的 `this` 指向。

8. `Proxy` 和 `Object.defineProperty` 的区别 {.quiz .fill}

   > 1. `Proxy` 是 ES6 提供对象代理方法，`Proxy` 只是将对象进行代理，后续操作都是对代理对象操作进行拦截。对原对象没有变动；
   > 2. `Object.defineProperty` 则是对对象的属性进行拦截，但对象新增的属性是监听不到的，需要对新属性也进行 `observer` 处理，比如 array 的 push、unshift 等操作时监听不到新值，或者监听异常的，这是由于 length 的变化导致新的索引没有被监听处理
   >
   > - `Proxy` 代理的是整个对象，可以通过 `get` / `set` 等方法劫持所有的一级属性，`Object.defineProperty` 监听的是对象本身的属性，需要逐个劫持每个属性；
   > - `Proxy` 和 `Object.defineProperty` 都只能够劫持对象自身的属性，而不能够深层劫持内部对象的属性，都需要进行递归处理深层对象，才能够完整劫持到全部属性；

9. `Set`、`WeakSet`、`Map`、`WeakMap` 的区别 {.quiz .fill}

10. `async`/`await` 和 `Promise` 的区别 {.quiz .fill}

## 应用

1.  防抖和节流的区别，如何实现？ {.quiz .fill}

    > - 防抖是指短时间内连续执行一个函数，只会产生一个有效执行。
    > - 节流是指短时间内连续执行一个函数时，会以固定的频率产生有效执行。
    > - 防抖常用于即时搜索，点击按钮中。
    > - 节流常用于滚动事件、监听事件。

    > ```js
    > function debounce(task, ms) {
    >   let timer = null
    >   return function () {
    >     const that = this
    >     const agrs = arguments
    >     if (timer) clearTimeout(timer)
    >     timer = setTimeout(() => task.call(that, ...agrs), ms)
    >   }
    > }
    >
    > function throttle(task, ms) {
    >   let last = null
    >   return function () {
    >     const now = +new Date()
    >     if (!last || now - last >= ms) {
    >       last = now
    >       task.call(this, ...arguments)
    >     }
    >   }
    > }
    > ```

2.  实现一个 `sleep` 函数 {.quiz .fill}

    > ```js
    > // 这种实现方式阻塞的是同步代码，阻塞了执行栈，是最接近 ms 的实现，还可以通过 Promise 实现
    > function sleep(ms) {
    >   const now = +new Date()
    >   while (now >= +new Date()) {}
    > }
    > ```

3.  实现一个 `add` 函数，满足 `add(1, 2, 3)` 和 `add(1)(2)(3)` 值相等（`柯里化`） {.quiz .fill}

    > ```js
    > function add() {
    >   const nums = [...arguments]
    >
    >   const fn = (...rest) => {
    >     nums.push(...rest)
    >     return fn
    >   }
    >
    >   fn.toString = () => nums.reduce((pre, cur) => pre + cur, 0)
    >
    >   return fn
    > }
    > ```

4.  实现一个 `compose` 函数 {.quiz .fill}

    > ```js
    > function compose() {
    >   const tasks = [...arguments]
    >
    >   if (tasks.length === 0) return (v) => v
    >   if (tasks.length === 1) return tasks[0]
    >
    >   const first = tasks.shift()
    >   return tasks.reduce(
    >     (pre, cur) =>
    >       (...args) =>
    >         cur(pre(...args)),
    >     first
    >   )
    > }
    > ```

5.  实现一个 `flatten` 函数 {.quiz .fill}

    > ```js
    > // 递归法
    > function flatten(arr) {
    >   const res = []
    >
    >   for (const item of arr) {
    >     if (Array.isArray(item)) {
    >       res.push(...flatten(item))
    >     } else {
    >       res.push(item)
    >     }
    >   }
    >
    >   return res
    > }
    >
    > // 迭代法
    > function flatten(arr) {
    >   const queue = [...arr]
    >   const res = []
    >
    >   while (queue.length) {
    >     const f = queue.shift()
    >     if (Array.isArray(f)) queue.unshift(...f)
    >     else res.push(f)
    >   }
    >
    >   return res
    > }
    > ```

6.  JS 异步解决方案的发展历程以及优缺点 {.quiz .fill}

    > - `回调函数` -> `Promise` -> `Generator` -> `async/await`
    > - `回调地狱` -> `链式地狱` -> `语义古怪` -> 同步代码解决异步任务

7.  `Promise` 构造函数是同步执行还是异步执行，那么 `then` 方法呢? {.quiz .fill}

    > 构造函数是同步执行的，`then` 是 `resolve` 调用后产生的微任务，属于异步执行
    >
    > ```js
    > new Promise((resolve) => {
    >   console.log('同步执行')
    >   resolve()
    > }).then(() => {
    >   console.log('异步执行')
    > })
    > ```

8.  实现一个异步任务并发调度函数 {.quiz .fill}

    > ```js
    > function asyncScheudle(tasks, limit = 3) {
    >   const queue = []
    >   let count = 0
    >
    >   const result = new Array(tasks.length)
    >   let idx = 0
    >
    >   return new Promise((resolve) => {
    >     function run(task, runIdx) {
    >       const asyncTask = async () => {
    >         count += 1
    >         try {
    >           const res = await task()
    >           result[runIdx] = res
    >           idx += 1
    >           if (idx === tasks.length) resolve(result)
    >         } catch (error) {
    >           throw error
    >         } finally {
    >           count -= 1
    >           if (queue.length) queue.shift()()
    >         }
    >       }
    >
    >       if (count < limit) {
    >         return asyncTask()
    >       }
    >
    >       queue.push(asyncTask)
    >     }
    >
    >     tasks.forEach((tk, idx) => run(tk, idx))
    >   })
    > }
    > ```

9.  实现`数组转树`和`树转数组`的方法 {.quiz .fill}

    > ```js
    > // prettier-ignore
    > const tree = {name:'string',id:'1',children:[{id:'1-1',name:'string-1',children:[{id:'1-1-1',name:'string-1-1',},{id:'1-1-2',name:'string-1-2',},],},{id:'1-2',name:'string-2',},],}
    >
    > function treeToArray(tree) {
    >   const res = []
    >
    >   const bfs = (node, parent = null) => {
    >     const item = { ...node, parent, parentId: parent ? parent.id : null }
    >     res.push(item)
    >
    >     if (node.children && node.children.length) {
    >       node.children.forEach((item) => bfs(item, node))
    >     }
    >   }
    >
    >   bfs(tree)
    >
    >   return res
    > }
    >
    > // prettier-ignore
    > const arr = [{name:'string',id:'1',parentId:null,},{id:'1-1',name:'string-1',parentId:'1',},{id:'1-1-1',name:'string-1-1',parentId:'1-1',},{id:'1-1-2',name:'string-1-2',parentId:'1-1',},{id:'1-2',name:'string-2',parentId:'1',},]
    >
    > function arrayToTree(arr) {
    >   const buildTree = (nodes, parentId = null) => {
    >     return nodes
    >       .filter((node) => node.parentId === parentId)
    >       .map((node) => ({
    >         ...node,
    >         parentId,
    >         children: buildTree(nodes, node.id),
    >       }))
    >   }
    >
    >   return buildTree(arr)
    > }
    > ```
