---
title: JS 中的数据类型
date: 2022-04-20 18:15
categories:
    - [javascripts]
tags:
  - JS 基础
---

## 基本数据类型
- string
- number
- boolean
- undefined
- null
> **除了 undefined 和 null 没有原型，其余基本类型的原型都是其包装引用类型(String、Number、Boolean)**
> **NaN 也属于 number，同时 NaN 进行任何数学运算结果都是 NaN**

## ES6 后增加的基本数据类型

- BigInt
- Symbol
## 引用类型
> 除基本数据类型外都是引用类型。

- Object
- Array
- Function
- Date
- RegExp
## 基本数据类型和引用数据类型的比较

- 基本数据类型在内存中占有固定的大小，会保存在栈内存中。
- 引用数据类型的大小不确定的，同时为了明确引用数据类型的使用，所以会将值保存在堆内存中，同时将其指针保存在栈内存中。使用时一般通过指针来查找到堆内存中的值。

由上两点可以知道，基本数据类型都是直接操作值的，引用数据类型只能通过栈内存中的指针来访问值，而 JS 是在执行栈中运行的，所以基本数据类型复制的是值，引用数据类型复制的是指针。指针一样则指向的堆内存地址一致。
## 数据类型的检测
通过 typeof 即可检测基本数据类型。由于null的历史遗留问题(前三位为000)，所以 typeof null === 'object' 。typeof 无法检测除 function 外的引用类型，其余的引用类型检测结果都是 object。
通过 Object.prototype.toString.call 可以检测基本数据类型和引用数据类型，在 Object.prototype.toString 方法没有被覆写下可返回 '[object Type]'，取 Type 即是当前数据类型。
```typescript
let _toString = Object.prototype.toString;
function toRawType (value: any): string {
  // 获取 从第九个到倒数第二个 字符
  // 比如 [object String]  获取 String
  return _toString.call(value).slice(8, -1)
}
```
#### typeof 和 instanceof
##### typeof

- typeof 检测的是类型在内存中二进制低位的表现。 javascript 的最初版本中，使用的 32 位系统，为了性能考虑使用低位存储了变量的类型信息：
   - 000：对象
   - 010：浮点数
   - 100：字符串
   - 110：布尔
   - 111：整数
- 但是对于 undefined 和 null 来说，这两个值的信息存储是有点特殊的。null 对应机器码的 NULL 指针，一般是全 0（所以低位也是 000 ）。undefined：用 −2^30 整数来表示。所以，typeof 在判断 null 的时候就出现问题了，由于 null 的所有机器码均为0，因此直接被当做了对象来看待。
##### instanceof

- instanceof 运算符用于检测构造函数的 prototype 是否出现在某个实例对象的原型链上。
- variable instanceof constructor
```javascript
// 定义构造函数
function C(){} 
function D(){} 

var o = new C();

o instanceof C; // true，因为 Object.getPrototypeOf(o) === C.prototype

o instanceof D; // false，因为 D.prototype 不在 o 的原型链上

o instanceof Object; // true，因为 Object.prototype.isPrototypeOf(o) 返回 true
C.prototype instanceof Object // true，同上

C.prototype = {};
var o2 = new C();

o2 instanceof C; // true

o instanceof C; // false，C.prototype 指向了一个空对象,这个空对象不在 o 的原型链上.

D.prototype = new C(); // 继承
var o3 = new D();
o3 instanceof D; // true
o3 instanceof C; // true 因为 C.prototype 现在在 o3 的原型链上
```
## 数据类型的转换
> [http://es5.github.io/#x9](http://es5.github.io/#x9)

#### 显式转换
显式转换就是通过“一些方法”主动去进行类型转换。
一些方法：

- Boolean()

![image.png](https://cdn.nlark.com/yuque/0/2022/png/412560/1650382009436-f999cc2a-03be-41fc-8895-5c00d4fdea37.png#clientId=u067bca08-7c24-4&from=paste&height=329&id=u54f7deb9&name=image.png&originHeight=658&originWidth=1558&originalType=binary&ratio=1&rotation=0&showTitle=false&size=96000&status=done&style=none&taskId=ue914dee8-e709-41d6-8469-2e49f32a58d&title=&width=779)

- Number()

![image.png](https://cdn.nlark.com/yuque/0/2022/png/412560/1650382029025-2d63da15-6eff-4bcd-ae91-bb04fe15df81.png#clientId=u067bca08-7c24-4&from=paste&height=412&id=u6ed163aa&name=image.png&originHeight=824&originWidth=1446&originalType=binary&ratio=1&rotation=0&showTitle=false&size=106534&status=done&style=none&taskId=u2d7a27ca-344d-40fc-b76e-03b338c479c&title=&width=723)

- String()

![image.png](https://cdn.nlark.com/yuque/0/2022/png/412560/1650382047559-8951b295-6118-4156-ab78-528de56eaf20.png#clientId=u067bca08-7c24-4&from=paste&height=352&id=u7de25f84&name=image.png&originHeight=704&originWidth=1428&originalType=binary&ratio=1&rotation=0&showTitle=false&size=102731&status=done&style=none&taskId=uad9d8e53-fdba-428b-982c-bfe25dc8d40&title=&width=714)

- toString()
   - 一个表示该对象的字符串
   - null 和 undefined 没有 toString，也不能进行转换，String() 可以转换 null 和 undefined
```javascript
console.log(({}).toString()) // [object Object]
console.log([].toString()) // ""
console.log([0].toString()) // 0
console.log([1, 2, 3].toString()) // 1,2,3
console.log((function(){var a = 1;}).toString()) // function (){var a = 1;}
console.log((/\d+/g).toString()) // /\d+/g
console.log((new Date(2010, 0, 1)).toString()) // Fri Jan 01 2010 00:00:00 GMT+0800 (CST)
```

- valueOf()
   - 返回值为对象的原始值
   - 日期是一个例外，它会返回它的一个内容表示: 1970 年 1 月 1 日以来的毫秒数。
```javascript
console.log(({}).valueOf()) // {}
console.log([].valueOf()) //  []
console.log([0].valueOf()) // [0]
console.log([1, 2, 3].valueOf()) // [1,2,3]
console.log((function(){var a = 1;}).valueOf()) // function (){var a = 1;}
console.log((/\d+/g).valueOf()) //  /\d+/g
console.log(new Date(2022, 4, 19).valueOf()) // 1652889600000
```

- toPrimitive(val, PreferredType)
```typescript
/**
* 第一个参数是 val，表示要处理的值。
* 第二个参数是 PreferredType，非必填，表示希望转换成的类型，有两个值可以选，Number 或者 String。
* 当不传入 PreferredType 时，如果 input 是日期类型，相当于传入 String，否则，都相当于传入 Number。
* 如果传入的 input 是 Undefined、Null、Boolean、Number、String 类型，直接返回该值。
*
* 如果是 ToPrimitive(obj, Number)，处理步骤如下：
* 如果 obj 为 基本类型，直接返回
* 否则，调用 valueOf 方法，如果返回一个原始值，则 JavaScript 将其返回。
* 否则，调用 toString 方法，如果返回一个原始值，则 JavaScript 将其返回。
* 否则，JavaScript 抛出一个类型错误异常。
*
* 如果是 ToPrimitive(obj, String)，处理步骤如下：
* 如果 obj为 基本类型，直接返回
* 否则，调用 toString 方法，如果返回一个原始值，则 JavaScript 将其返回。
* 否则，调用 valueOf 方法，如果返回一个原始值，则 JavaScript 将其返回。
* 否则，JavaScript 抛出一个类型错误异常。
*/
```
#### 隐式转换
> [https://github.com/mqyqingfeng/Blog/issues/164](https://github.com/mqyqingfeng/Blog/issues/164)

**触发隐式转换的条件：**

- **二元运算符(+、-、*、\)**
- **三元运算符(?:)**
- **关系运算符(>、<、>=、<=)**
- **逻辑运算符(!、&&、||)**
- **if、else if**
- **==**

**隐式转换的规则一般都按照 toNumber、toString、 toPrimitive 的规则进行转换。**
**{}、+ 在不同的写法下有着不同的表现。**

- **{} 在开头会作为代码块的标识。**
- **+ 在字符串的运算中作为字符连接符使用、而在数值计算时作为二元运算符的加号使用。**

**undefined、null 在进行转换时也有不同的表现。**
**二元运算符的操作数只能是数值，如果不是数值，则会进行隐式转换为数值。**
**逻辑运算符的操作数只能是布尔值，如果不是布尔值，则会进行隐式转换为布尔值。**
```typescript
// 如果 {} 是在开头，JS 解释器会将其作为一个代码块
{}.valueOf() // Uncaught SyntaxError: Unexpected token '.'

// 只要 {} 不在开头就行
({}).valueOf() // {}
console.log({}.valueOf) // {}
```
##### +:

- 如果两个操作数都是数值，则作为二元运算符的加号使用。
- 若一个操作数是数值，另一个操作数为 undefined 或 null，会把 undefined 和 null 用 Number 进行转换，然后作为二元运算符的加号使用。
- 若有一个操作数为字符串时，会将另一个操作数转换成字符串，作为字符连接符使用。
- 若有一个操作数为 object 时，会将这个操作数调用 toString() 进行转换，然后作为字符连接符使用。
- 若一个操作数为字符串，另一个操作数为 undefined 或 null 时，会将这个操作数用 String() 进行转换，然后作为字符连接符使用。
##### ==:

- 字符串和数字之间的相等比较，将字符串转换为数字之后再进行比较。
- 其他类型和布尔类型之间的相等比较，先将布尔值转换为数字后，再应用其他规则进行比较。
- null 和 undefined 之间的相等比较，结果为真。其他值和它们进行比较都返回假值。
- 对象和非对象之间的相等比较，对象先调用 ToPrimitive 抽象操作后，再进行比较。
- 如果一个操作值为 NaN ，则相等比较返回 false（ NaN 本身也不等于 NaN ）。
- 如果两个操作值都是对象，则比较它们栈中的引用是否相等。如果引用相等，则相等操作符返回 true，否则返回 false。
##### if/else if:

- if(x)/else if(x)使用 Boolean(x) 进行转换。
#### 关系运算符(>、<、>=、<=):

- 如果两个操作值都是数值，则进行数值比较。
- 如果两个操作值都是字符串，则比较字符串对应的字符编码值。
- 如果只有一个操作值是数值，则将另一个操作值转换为数值，进行数值比较。
- 如果一个操作数是对象，则调用 valueOf（如果对象没有 valueOf 则调用 toString），得到的结果按照前面的规则执行比较。
- 如果一个操作值是布尔值，则将其转换为数值，再进行比较。
> 注：NaN 是非常特殊的值，它不和任何类型的值相等，包括它自己，同时它与任何类型的值比较大小时都返回 false。

```typescript
NaN == NaN  //false
NaN == undefined //false
NaN == false //false
NaN == null //false
NaN == []  //false
NaN == ''  //false
NaN == {}  //false

false == false  //true
false == undefined  //false
false == null  //false
false == []  //true
false == {}  //false
false == ''  //true

undefined == undefined //true
undefined == null  //true
undefined == false //false
undefined == [] //false
undefined == {}  //false
undefined == '' //false

null == null   //true
null == NaN  //false  
null == []  //false
null == {}  //false
null == undefined  //true

0 == false    //true   
0 == []  //true
0 == {}  //false
0 == null  //false
0 == undefined //false
0 == '' //true
0 == NaN //false

false == []  //true
false == {}  //false
false == null  //false
false == undefined  //false
false == ''  //true
false == NaN  //false

[] == {} //false
```
## 有趣的比较
```javascript
console.log([] == 0) // true
console.log(![] == 0) // true 

console.log([] == ![])  // true 看这里！
console.log([] == [])  //false

console.log({} == !{}) // false  
console.log({} == {})  // false

console.log([] == 0) // true
/*
* 原理：其他类型和布尔类型之间的相等比较，先将布尔值转换为数字后，再应用其他规则进行比较。
* 1.[].valueOf().toString() 得到字符串""
* 2.将 "" 转为数字 Number("") 得到数字 0
* 所以 [] == 0 成立
*/

console.log(![] == 0) // true 
/*
* 其他类型和布尔类型之间的相等比较，先将布尔值转换为数字后，再应用其他规则进行比较。
* 与上面类似，只是逻辑运算符优先级高于关系运算符，所以先执行 ![] 得到 false
* false == 0 等价于 0 == 0
*/

console.log([] == ![])  // true  
/* 
* 其他类型和布尔类型之间的相等比较，先将布尔值转换为数字后，再应用其他规则进行比较。
* ![] 优先级高 先转换成 boolean 等价于 false
* [] == false 等价于 Number(([]).valueOf().toString()) == 0 即 0 == 0
*/

console.log([] == [])  //false
/*
* 引用数据类型数据存在堆中，栈中存储的是它们的地址，两个[]地址肯定不一样，所以是false
*/

console.log({} == !{}) //false
/*
* 其他类型和布尔类型之间的相等比较，先将布尔值转换为数字后，再应用其他规则进行比较。
* !{} 优先级高 先转换成 boolean 等价于 false
* {} == false 等价于 Number(([]).valueOf().toString()) == 0 即 NaN == 0
*/

console.log({} == {})  // false
/*
* 引用数据类型数据存在堆中，栈中存储的是它们的地址，所以肯定不一样
*/
```
