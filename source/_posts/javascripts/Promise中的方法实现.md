---
title: Promise 中的方法实现
date: '2022-12-09 06:20'
categories:
  - - JavaScript
    - Promise
tags:
  - Promise
  - 原理
abbrlink: c321570b
---

> JS 中的微任务
> - MutationObserver
> - Promise.then
> - queueMicroTask
>    - 兼容性不好
> - process.nextTick
>    - node 端

## 基础 Promise
#### Promise 解决函数实现
```typescript
function resolvePromise(promise, x, resolve, reject) {
  if (promise === x) return reject(new TypeError('x 不能和 promise 相等'))

  if (x && (typeof x === 'object' || typeof x === 'function')) {
    let executed = false
    try {
      const then = x.then
      if (typeof then === 'function') {
        then.call(
          x,
          (y) => {
            if (executed) return
            executed = true
            resolvePromise(promise, y, resolve, reject)
          },
          (e) => {
            if (executed) return
            executed = true
            reject(e)
          }
        )
      } else {
        resolve(x)
      }
    } catch (e) {
      if (executed) return
      executed = true
      reject(e)
    }
  } else {
    resolve(x)
  }
}
```
#### Promise 实现
```typescript
const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

// 微任务执行函数
function microTask(fn) {
  // setTimeout(fn, 0)
  queueMicrotask(fn)
}

class IPromise {
  constructor(exec) {
    this.state = PENDING
    this.value = null
    this.reason = null

    this.onFulfilledCallbacks = []
    this.onRejectedCallbacks = []

    const resolve = (value) => {
      if (value instanceof IPromise) {
        value.then(resolve, reject)
        return
      }
      if (this.state === PENDING) {
        this.state = FULFILLED
        this.value = value

        this.onFulfilledCallbacks.forEach((fnc) => fnc())
      }
    }

    const reject = (reason) => {
      if (this.state === PENDING) {
        this.state = REJECTED
        this.reason = reason

        this.onRejectedCallbacks.forEach((fnc) => fnc())
      }
    }

    try {
      exec(resolve, reject)
    } catch (e) {
      reject(e)
    }
  }

  then(onFulfilled, onRejected) {
    // prettier-ignore
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : (data) => data
    // prettier-ignore
    onRejected = typeof onRejected === 'function' ? onRejected : (err) => { throw err }

    const promise = new IPromise((res, rej) => {
      if (this.state === FULFILLED) {
        microTask(() => {
          try {
            const x = onFulfilled(this.value)
            resolvePromise(promise, x, res, rej)
          } catch (error) {
            rej(error)
          }
        })
      } else if (this.state === REJECTED) {
        microTask(() => {
          try {
            const x = onRejected(this.reason)
            resolvePromise(promise, x, res, rej)
          } catch (error) {
            rej(error)
          }
        })
      } else {
        this.onFulfilledCallbacks.push(() => {
          microTask(() => {
            try {
              const x = onFulfilled(this.value)
              resolvePromise(promise, x, res, rej)
            } catch (error) {
              rej(error)
            }
          })
        })

        this.onRejectedCallbacks.push(() => {
          microTask(() => {
            try {
              const x = onRejected(this.reason)
              resolvePromise(promise, x, res, rej)
            } catch (error) {
              rej(error)
            }
          })
        })
      }
    })

    return promise
  }

}
```
## catch 实现
```typescript
class IPromise {
  /// ...
  
  catch(onRejected) {
    return this.then(null, onRejected)
  }
  
  // ...
}
```
## finally 实现
```typescript
class IPromise {
  
  // ...
  
  finally(callback) {
    return this.then(
      (val) => IPromise.resolve(callback()).then(() => val),
      (reason) =>
      IPromise.resolve(callback()).then(() => {
        throw reason
      })
    )
  }  
  
  // ...
}
```
## Promise.all 实现
```typescript
class IPromise {
	
	// ...
	
	
	static all(promises) {
		return new IPromise((resolve, reject) => {
			const res = []
			let length = 0
			for (let promise of promises) {
				IPromise.resolve(promise).then((val) => {
					res[length] = val
					length++
					if (length === promises.length) resolve(res)
				}, reject)
			}
		})
	}
	
	// ...
}
```
## Promise.allSettled 实现
```typescript
class IPromise {
	
	// ...
	
	
	static allSettled(promises) {
		return new IPromise((resolve, reject) => {
			const res = []
			let length = 0
			if (promises.length === 0) return resolve([])
			
			for (let promise of promises) {
				IPromise.resolve(promise).then(
					(val) => {
						res[length] = {
							status: FULFILLED,
							value: val,
						}
						length++
						if (length === promises.length) resolve(res)
					},
					(reason) => {
						res[length] = {
							status: REJECTED,
							value: reason,
						}
						length++
						if (length === promises.length) resolve(res)
					}
				)
			}
		})
	}
	
	// ...
}
```
## Promise.race 实现
```typescript
class IPromise {
	
	// ...
	
	static race(promises) {
		return new IPromise((resolve, reject) => {
			for (let p of promises) IPromise.resolve(p).then(resolve).catch(reject)
		})
	}
	
	// ...
}
```
## Promise.resolve 实现
```typescript
class IPromise {
	
	// ...
	
  static resolve(value) {
    return new IPromise((res) => res(value))
  }
	
	// ...
}
```
## Promise.reject 实现
```typescript
class IPromise {
	
	// ...
	
  static reject(reason) {
    return new IPromise((res, rej) => rej(reason))
  }
	
	// ...
}
```
