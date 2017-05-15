---
layout:     post
title:      "JS初学常见问题"
date:       2017-05-08 20:19:40
author:     "Joan"
tags:		["JS"]
---

## js中有几种数据类型

JavaScript 的数据类型，共有六种。（ES6 又新增了第七种 Symbol 类型的值，本教程不涉及。）
> - number 数字
- string 字符串
- boolean 布尔值
- undefined 表示未定义或者不存在
- null 表示无值
- object 对象

## typeof 返回值有哪几个
typeof返回字符串，其中包括string\number\boolean\undefined\object\function

## 如果判断一个对象是不是数组？
1. Object.prototype.toString.call(obj) === ‘[object Array]’;
2. Array.isArray(obj);
3. obj instanceof Array;

## caller和callee有什么区别？
`caller`是该正在执行的函数的调用函数的引用，如果为(顶层)对象，则返回null。
`callee`是arguments对象的一个属性，它是正在执行的函数的引用。（常用来递归匿名函数本身 但是在严格模式下不可行）

## 简述js垃圾回收机制，请谈谈你在开发过程中遇到的内存泄露的情况，是如何解决的。

### 垃圾回收
JS具有自动垃圾回收机制（GC），会周期性的找到那些不再使用的变量，然后释放其内存空间。例如函数执行完成后，会释放其中局部变量的内存空间。垃圾回收器会跟踪内存中的变量，判断其有没有用。

- **标记清除（mark and sweep）** 

这是JavaScript **最常见的**垃圾回收方式，当变量进入执行环境的时候，比如函数中声明一个变量，垃圾回收器将其标记为“进入环境”，当变量离开环境的时候（函数执行结束）将其标记为“离开环境”。
垃圾回收器会在运行的时候给存储在内存中的所有变量加上标记，然后去掉环境中的变量以及被环境中变量所引用的变量（闭包），在这些完成之后仍存在标记的就是要删除的变量了，因为环境中的变量已经无法访问到这些变量了，然后垃圾回收器相会这些带有标记的变量机器所占空间。

- **引用计数(reference counting)** 

在低版本IE中经常会出现内存泄露，很多时候就是因为其采用引用计数方式进行垃圾回收。引用计数的策略是跟踪记录每个值被使用的次数，当声明了一个变量并将一个引用类型赋值给该变量的时候这个值的引用次数就加1，如果该变量的值变成了另外一个，则这个值得引用次数减1，当这个值的引用次数变为0的时候，说明没有变量在使用，这个值没法被访问了，因此可以将其占用的空间回收，这样垃圾回收器会在运行的时候清理掉引用次数为0的值占用的空间。这个方法最严重的问题就是**无法解决循环引用。**

### 解决方案
JS内存垃圾自动回收的机制下，内存泄露产生的原因往往和不需要的引用有关

- 无意的全局变量
下面代码就无意中声明了一个全局变量，会得到 window 的引用，bar 实际上是 window.bar，它的作用域在 window 上，所以 foo 函数执行结束后，bar 也不会被内存收回。

```
function foo(arg) {
    bar = "";
}
foo();
/** 另外一种无意的全局变量的情况是：**/

function foo() {
    this.bar = "";  // 在 foo 函数中，this 指的是 window
}
```

- DOM 片段
每单击一次按钮，就创建一个`<div>`，它没有引用任何对象，但是回调结束之后，这个空的 `<div>` 是不会被回收的。
在不使用DOM节点后将其引用手动覆为`null`。

```
var btn = document.getElementById('btn');
btn.onclick = function() {  var fragment = document.createElement('div');
}
```

- DOM的监听事件
虽然 `<button>`从 DOM 中移除了，由于它的监听器还在，所以无法被 GC 回收。
要避免这种情况就是通过 `removeEventListener` 将回调函数去掉。

```
var content = document.getElementById('content');
content.innerHTML = '<button id="button">click</button>';
var button = document.getElementById('button');
button.addEventListener('click', function() {});
content.innerHTML = '';
```


- 被遗忘的计时器和回调函数
下面的例子中，我们每隔一秒就将得到的数据放入到文档节点中去。但在 `setInterval` 没有结束前，回调函数里的变量以及回调函数本身都无法被回收。那什么才叫结束呢？就是调用了 `clearInterval`。如果回调函数内没有做什么事情，并且也没有被 clear 掉的话，就会造成内存泄漏。不仅如此，如果回调函数没有被回收，那么回调函数内依赖的变量也没法被回收。上面的例子中，`someResource` 就没法被回收。
同样的，setTiemout 也会有同样的问题。所以，当不需要 interval 或者 timeout 时，最好调用 `clearInterval` 或者 `clearTimeout`。

```
let someResource = getData();
setInterval(() => {
    const node = document.getElementById('Node');
    if(node) {
        node.innerHTML = JSON.stringify(someResource));
    }
}, 1000);
```

- 子元素存在引用引起的内存泄漏
子元素 refB 由于 `parentNode` 的间接引用，只要它不被删除，它所有的父元素（图中红色部分）都不会被删除。
![picture](http://staticc.qiniudn.com/detached-nodes.gif?_=6013490)
## jquery的事件委托方法on、live、delegate之间有什么区别？
live和delegate在底层均调用on方法，它们的区别在于live方法将this.context（document）作为事件委托的对象，而delegate则可以供用户选择事件委托的对象。on方法是底层方法，bind方法也在底层调用on方法，on与其它时间委托方法的区别在于type和selector换了位置，如果没有指定selector的话，就将事件绑定到元素本身上。

## 请说说对MVC，MVP和MVVM的理解

### MVC

`MVC`即model\view\controller，是最常见的软件架构模式。view是指用户界面，controller则处理业务逻辑，而model则用来存储数据。view传送指令到controller，controller完成业务逻辑后要求model改变状态，model将新数据发送给view，使用户得到反馈。各部分的通信都是单向的。
![MVC](http://image.beekka.com/blog/2015/bg2015020105.png)

### MVP

`MVP`将controller改为presenter，在这种模式中，各个部分之间的通信都是双向的。view与model不发送直接联系，都通过presenter传递。view非常薄，不部署任何业务逻辑，而presenter非常厚。
![MVP](http://image.beekka.com/blog/2015/bg2015020109.png)

### MVVM

`MVVM`将presenter改为viewModel，和MVP模式非常相似，唯一的区别在于它是双向绑定，view改变，自动反应在viewModel上，反之亦然。
![MVVM](http://image.beekka.com/blog/2015/bg2015020110.png)

## 何为跨域？跨域请求资源有几哪种方式？
由于浏览器同源策略，凡是发送请求url的协议、域名、端口三者之间任意一与当前页面地址不同即为跨域。
跨域请求资源的方式主要有：
### JSONP 动态创建 script 标签
JSONP的原理就是利用`<script>`标签没有跨域限制的“漏洞”来达到与第三方通讯的目的。（凡是拥有"src"这个属性的标签都拥有跨域的能力，比如`<script>`、`<img>`、`<iframe>`）。简单的说就是通过script标签访问服务器并将回调函数作为参数提供给服务器，服务器收到请求后，将要数据作为回调函数的参数一起返回给前端。由于`<script>`元素请求的脚本，直接作为代码运行。这时，只要浏览器定义了回调函数，该函数就会立即调用。
但缺点是只支持get请求，并且很难判断请求是否失败（一般通过判断请求是否超时）。
### Proxy代理
这种方式首先将请求发送给后台服务器，通过服务器来发送请求，然后将请求的结果传递给前端。
### CORS跨域
是现代浏览器提供的一种跨域请求资源的方法，需要客户端和服务器端的同时支持。整个CORS通信过程，都是浏览器自动完成，不需要用户参与。对于开发者来说，CORS通信与同源的AJAX通信没有差别，代码完全一样。浏览器一旦发现AJAX请求跨源，就会自动添加一些附加的头信息，有时还会多出一次附加的请求，但用户不会有感觉。因此，实现CORS通信的关键是服务器。只要服务器实现了CORS接口，就可以跨源通信。

## JS中错误类型有哪些？请简述一下
（1）syntaxError 语法错误 解析代码时发生的错误
（2）referenceError 引用类型错误 当访问一个不存在的变量时发生的错误
（3）RangeError是当一个值超出有效范围时发生的错误。比如把数组长度为负数，number超出范围（1.7976931348623157e+308），以及函数堆栈超过最大值。
（4）TypeError是变量或参数不是预期类型时发生的错误。比如，对字符串、布尔值、数值等原始类型的值使用new命令，就会抛出这种错误。
（5）EvalError eval函数没有被正确执行时，会抛出EvalError错误。该错误类型已经不再在ES5中出现了，只是为了保证与以前代码兼容，才继续保留。
（6）URIError是URI相关函数的参数不正确时抛出的错误，比如encodeUri

## JS的原型链与闭包

未完待续...

> **原文出处如下，本文内容有所更改** 
作者： 圣经的旋律 
链接：http://www.imooc.com/article/17009
来源：慕课网
本文原创发布于慕课网 ，转载请注明出处，谢谢合作！




