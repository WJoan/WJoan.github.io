---
layout:     post
title:      "JS中的深拷贝与浅拷贝"
date:       2017-04-13 16:07:45
author:     "Joan"
tags:		["JS"]
---


深拷贝与浅拷贝主要是针对Object和Array这样的引用类型。当我们把一个引用赋值给一个变量的时候，这个新的变量和原引用指向的内存空间是相同的，当我们通过其中一个引用修改变量时，其他的引用的结果也会发生响应变化，这样的操作就称之为浅拷贝。

---

## 如果要拷贝一个**数组**

```
var oldArr = [1,2,3];
var newArr = oldArr;            // 浅拷贝
var newArr1 = oldArr.slice(0);  // 深拷贝
var newArr2 = oldArr.contact(); // 深拷贝
```
数组的截取和连接操作，都是在不改变原有数组的情况下返回一个新的数组，因此实现了深拷贝。其本质因该是：新建一个空数组，然后对原数组进行`遍历`操作，然后一一赋值。

---

## 如果要拷贝一个简单的**对象**

```
// 将JS对象序列化为JSON后，再解析JSON成JS对象
var newObj = JSON.parse(JSON.stringify(Obj));
```
对于JSON安全的对象来说，这是一种很巧妙的复制方法，但是这个方法有一定的局限性。首先对于`正则表达式类型`、`函数类型`等无法进行深拷贝，会直接丢失相应的值。还有一点不好的地方是它会抛弃对象的`constructor`。也就是深拷贝之后，不管这个对象原来的构造函数是什么，在深拷贝之后都会变成Object。

```
function anotherFunction() {/*..*/}

var anotherObject = {
	c: true
};

var anotherArray = [];

var myObject = {
	a: 2,
	b: anotherObject,	// 引用，不是副本
	c: anotherArray,	// 另一个引用
	d: anotherFunction
};

// 使用ES6定义的 Object.assign(..) 方法来实现浅复制
var newObj = Object.assign( {}, myObject );
newObj.a; // 2
newObj.b === anotherObject;		// true
newObj.c === anotherArray;		// true
newObj.d === anotherFunction;	// true

```

ES6定义的 [Object.assign(..)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) 方法来实现浅复制。该方法的第一个参数是`目标对象`，之后还跟着一个或者多个`源对象`。它会遍历源对象的所有`自身的`可枚举（enumerable）的属性，并把它们复制（使用=操作符赋值）到目标对象，最后返回目标对象。

---

## 如果要深拷贝一个包含对象类型的**对象**

```
var deepCopy= function(obj) { 
    var result={};
    for (var key in obj) {
        result[key] = typeof obj[key]==='object'?deepCoyp(obj[key]): obj[key];
     } 
   return result; 
}
```

上述函数对于对象的每一个属性进行遍历并赋给新的对象，如果对象的某一属性是 Object 类型，那么通过`递归`完成深拷贝。

---

## 如果使用 **Jquery**

> jQuery.extend( [deep ], target, object1 [, objectN ] )
**参数描述**
- [deep] 可选/Boolean类型指示是否深度合并对象，默认为false。如果该值为true，且多个对象的某个同名属性也都是对象，则该”属性对象”的属性也将进行合并。 
- [target] Object类型目标对象，其他对象的成员属性将被复制到该对象上。
- [object1] 可选/Object类型第一个被合并的对象。 
- [objectN] 可选/Object类型第N个被合并的对象。

extend 函数：合并多个对象到 target 对象，注意 **target** 对象会改变。

```
var Chinese = {
  nation:'中国'
};
function object(o) {
  function F() {}
  F.prototype = o;
  return new F();
}
var Doctor = object(Chinese); // Doctor对象继承了Chinese对象
Doctor.career = '医生';        //此时Doctor为 F {career: "医生"}
var Doctor2 = $.extend({},Doctor); // 深拷贝之后 Doctor2 为 Object {career: "医生", nation: "中国"}
```
Jquery中的`$.extend()`函数本质其实就是通过`递归`完成的深拷贝，它会将要拷贝的对象中的所有属性都拷贝过来，包括继承对象的属性。