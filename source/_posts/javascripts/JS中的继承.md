---
title: JS 中的继承
date: '2022-12-07 21:42'
categories:
  - - JavaScript
tags:
  - JS 基础
abbrlink: 5c1a1e7b
---

> 参考：
> - [https://github.com/mqyqingfeng/Blog/issues/16](https://github.com/mqyqingfeng/Blog/issues/16) 冴羽的 JavaScript 深入系列《JavaScript深入之继承的多种方式和优缺点》
> - 红宝书

#### 原型链继承
> 缺点：
> 1. 引用类型会被所有子类共享
> 2. 子类在实例化时不能够像父类传递参数

```javascript
function Person () {}

function Kid() {}

// 直接将 Person 的实例做为 Kid 的原型
Kid.prototype = new Person()

```
#### 借用构造函数继承
> 即借用了父类的构造函数为自己“创建”父类的属性
> 改进：
> 1. 避免了引用类型的属性被所有实例共享。
> 2. 可在子类在实例化时向父类传递参数。
> 
缺点：
> 1. 如需继承方法就必须在父类的构造函数中实现，否则将无法继承，也就是说方法不能够复用。

```javascript
function Person (name) {
	this.name = name
}

function Kid(name, age) {
	// 将 Person 的构造函数借过来绑定一下 Person 的属性
  Person.call(this, name)
  this.age = age
}

```
#### 组合继承
> 即将原型链继承和借用构造函数继承的组合起来使用，以达到可以继承属性也能继承方法，同时子类还能够拥有自己的属性和方法（引用类型不会在类之间共享）。
> 改进：
> 1. 可以继承父类的属性和方法。
> 2. 方法可以复用，也能拥有子类自己的方法。
> 3. 可以被 instanceof 和 isPrototypeof 识别
> 
缺点：
> 1. 重复调用了两次父类构造函数 (a. 子类构造函数内一次；b. 改变原型时实例化一次

```javascript
function Person(name) {
	this.name = name
}

function Kid(age) {
  Person.call(this, name)
  this.age = age
}

Kid.prototype = new Person()
// 重写被覆盖掉的构造函数
Kid.prototype.constructor = Kid


```
#### 原型式继承
> 基于原有的对象创建子类，将原型指向这个对象，创建者模式。ES5 中的 Object.create 就是原型式继承。
> 即基于一个对象，创建一个新的对象与其保持类似。
> 改进/优点：
> 1. 快速创建一个与旧对象相似的子类，同时可以避免生成构造函数占用内存？(猜测
> 
缺点：
> 1. 引用类型会被所有子类共享

```javascript
function creator(obj) {
	const Fun = function() {}
  Fun.prototype = obj
  return Fun
}
```
#### 寄生式继承
> 基于原型式继承的改进。工厂模式，创建一个用于封装继承的过程函数，在该函数内部增强对象。
> 改进：
> 1. 子类拥有自己的方法？
> 2. 只做增强，不会修改到原来的对象
> 
缺点：
> 1. 跟借用构造函数模式一样，每次创建对象都会创建一遍方法

```javascript
function creator(obj) {
	const Fun = function() {}
  Fun.prototype = obj
  return Fun
}

function creatorChild(orginal) {
  const obj = creator(original)
  // 增强对象
  obj.getAge = function(){
  	console.log('age')
  }
  return obj
}
```
#### 寄生组合式继承
> 即寄生式和组合式的再次组合。组合模式的缺点是会重复调用两次父类的构造函数。
> 改进思路就是当子类的原型不指向父类的实例，而直接指向父类的原型。
> 也是 ES6 实现的继承的方式

```javascript
function creator(obj) {
	const Fun = function() {}
  Fun.prototype = obj
  return Fun
}

// 这只是一个处理过程，所有不需要返回
function prototype(child, parent) {
    // 以父类的原型创建新的对象
    const prototype = creator(parent.prototype);
    // 将构造函数指向于子类
    prototype.constructor = child;
    // 改变子类的原型
    child.prototype = prototype;
}
```

