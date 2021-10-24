# 让人又爱又恨的变量提升？

## Guide

首先来看一个例子，来猜一下这两条语句会打印出什么内容。

```js
console.log(a);
var a = 1;
```

其实这个例子也是挺简单的，基本功好的人会立马知道，他的输出结果为 `undefined`。  

这时我们要考虑下。
他的输出结果为什么是 `undefined` 呢？
正常情况下，如果一个变量未声明，不是会抛出以下错误吗？
```js
Uncaught ReferenceError: a is not defined。
```

这就牵扯出js的一个特性：**变量提升**。

在 JavaScript 中，变量提升已经是老生常谈的一个话题。  
无论是google，百度都有很多关于它的内容探讨。

## 概念

变量提升就是在 **预编译阶段**，**变量和函数** 的声明会移动到代码的 **最前面**。  

JS引擎会在正式执行代码之前进行一次 **预编译** ，预编译简单理解就是在内存中开辟一些空间，存放一些变量和函数。  

但是有一点需要注意的是**函数的权重等级最高**。也就是函数会放到最前面，其次是变量。  

**但这么说并不完全准确，在物理上变量和函数声明在代码里的位置是不会动的，
而是JavaScript引擎在预编译阶段做了一些小动作。**

## 预编译阶段做了什么

### 第一个例子


```js
console.log(a); // 输出：undefined// 输出：undefined
var a = 1;
```
这段代码在预编译阶段，会被处理成以下样子：

```js
var a; // 变量提升
console.log(a); // 输出：undefined
a = 1;
```
变量 `a` 的声明被提升到最顶端。然后其他语句依次执行，只不过原来的 `var a = 1;` 变成了赋值形式的存在 `a = 1;`。

### 第二个例子


```js
catName(“Chloe”); // 输出我的猫名叫 Chloe
function catName(name) {
  console.log(“我的猫名叫 ” + name);
}
```
这段代码在预编译阶段，会被处理成以下样子：
```js
function catName(name) {
  console.log(“我的猫名叫 ” + name);
}
catName(“Chloe”); // 输出我的猫名叫 Chloe
```

名为 `catName` 的函数被原封不动的提升到最顶端。然后执行 `catName` 函数，所以不会抛出任何错误，输出正确内容，这种现象叫做 **函数提升** ，和 **变量提升** 一个道理。


### 第三个例子

```js
myFunc(); // Uncaught ReferenceError: myFunc is not defined
var myFunc = function() {
  console.log(‘myFunc’);
}
```
需要注意的是，这个是你变量赋值的形式生命的函数。  
这段代码在预编译阶段，会被处理成以下样子：
```js
var myFunc;
myFunc(); // Uncaught ReferenceError: myFunc is not defined
myFunc = function() {
  console.log(‘myFunc’);
}
```

由于 `myFunc` 变量提升了，所以值为 `undefined`，所以会抛出以下错误：
```
Uncaught ReferenceError: myFunc is not defined
```

### 第四个例子
```js
console.log(a); // Uncaught ReferenceError: a is not defined
function myFunc() {
  console.log(a); // 输出：undefined
  var a = 1;
  console.log(a); // 输出：1
}
myFunc();
```
需要注意的是，`var` 声明的变量是函数级作用域，所以 `myFunc` 内的 `a` 变量只会被提升到函数的最顶端。  

这段代码在预编译阶段，会被处理成以下样子：
```js
function myFunc() {
  // 函数执行上下文环境
  var a; // 变量提升
  console.log(a); // 输出：undefined
  a = 1;
  console.log(a); // 输出：1
}
console.log(a); // 终止执行 - Uncaught ReferenceError: a is not defined
myFunc();
```

其实这段代码在第一条 `console.log(a);` 语句执行后就终止执行了，因为它会抛出以下错误：

```
Uncaught ReferenceError: a is not defined
```
假设此条语句不存在，那么函数内的语句会怎么执行呢？  

`a` 这个变量会被提升到函数的最顶端，然后执行 `console.log(a);`， 输出 `undefined`，然后依次正常执行之后的语句，最后输出 `1`。

### 第五个例子

```js
a(); // Uncaught TypeError: a is not a function
var a = 1;
var a = function() {
  console.log(‘myFunc’);
}
```

根据之前的几个例子，这个例子对大家来说应该简单很多了。   
这段代码在预编译阶段，会被处理成以下样子：  
```js
var a； // 变量提升
a(); // Uncaught TypeError: a is not a function
a = 1;
a = function() {
  console.log(‘myFunc’);
}
```

变量 `a` 被提升到最顶端，值为 `undefined` ，所以 `a();` 这条语句会抛出以下错误：
```
Uncaught TypeError: a is not a function
```

### 第六个例子
```js
a(); // 输出 myFunc
var a = 1;
function a() {
  console.log('myFunc');
}
```

这个例子与上个例子稍微不同的是声明函数的方式不再是使用变量声明。

```js
function a() {
  console.log('myFunc');
}
a(); // 输出 myFunc
a = 1;
```
函数被提升到最顶端，其他语句依次执行。

## 总结

在解析阶段，JS会检查语法，并对函数进行 **预编译**。  

解析的时候会 **先创建一个全局执行上下文环境** ，先把代码中即将执行的变量、函数声明都拿出来， **变量先赋值为undefined，函数先声明好可使用** 。  

在一个函数执行之前，也会 **创建一个函数执行上下文环境** ，跟全局执行上下文类似，不过函数执行上下文会 **多出this、arguments和函数的参数** 。  
在执行阶段，就是按照代码的顺序依次执行。
### 优缺点（爱与恨）
#### 爱：
1. 提高性能：在JS代码执行之前，会进行语法检查和预编译，并且这一操作只进行一次。
2. 容错性更好：不会因为变量或函数定义的顺序抛出一些莫名其妙的错误。
#### 恨：
1. 可读性差
2. 可维护性差
3. 造成逻辑混乱


**所以，在实际开发中，无论是变量还是函数，都应先声明后使用。并且应尽量使用 let 和 const 来声明变量，从而约束变量提升。**