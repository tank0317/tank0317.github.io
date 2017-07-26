---
layout: post
title: 深入理解setTimeout 和 setInterval
date: 2017-03-03
tags: javascript front-end
comments: true
---

Javascript中关于定时器，常用的有**setTimeout**和**setInterval**。比较新的还有一个setImmediate， 但大多数浏览器并不支持，所以这里不做研究。

<!--more-->

### setTimeout
**setTimeout(fn, x)**通常接收两个参数：要执行的代码和以毫秒表示的时间（即在执行代码前需要等待多少毫秒）。其中第一个参数也可以是一个包含Javascript代码的字符串（就和eval( )函数中使用的字符串一样），也可以是一个函数。例如下面对setTimeout( )的两次调用都会在一秒钟后显示一个警告框。

```javascript
//不建议使用字符串
setTimeout("alert('Hello world!')", 1000);

//推荐的调用方式
setTimeout(function() {
	alert("Hello world!");
}, 1000);
```
> 由于传递字符串可能导致性能损失，因此不建议使用字符串作为第一个参数。

**使用的时候，延迟的时间严格来说总是大于x毫秒的，这和JS的执行机制有关。**HTML5规范规定最小延迟时间不能小于4ms，即x如果小于4，会被当做4来处理。 不过不同浏览器的实现不一样，比如，Chrome可以设置1ms，IE11/Edge是4ms。

setTimeout注册的函数fn会交给浏览器的定时器模块来管理，延迟时间到了就将fn加入主进程执行队列，如果队列前面还有没有执行完的代码，则又需要花一点时间等待才能执行到fn，所以实际的延迟时间会比设置的长。如在fn之前正好有一个超级大循环，那延迟时间就不是一丁点了。
```javascript
(function testSetTimeout() {
    const label = 'setTimeout';
    console.time(label);
    setTimeout(() => {
        console.timeEnd(label);
    }, 10);
    for(let i = 0; i < 100000000; i++) {}
})();
//结果是：setTimeout: 330.462ms，远远不止10ms。
```
> 多个定时器如不及时清除（使用cleartimeout清除），会存在干扰，是延迟时间更加琢磨不透。所以及时清除已经不需要的定时器是个好习惯。

### setInterval

setInterval的实现机制跟setTimeout类似，只不过setInterval是重复执行的。

对于setInterval(fn, 100)容易产生一个误区：并不是上一次fn执行完了之后再过100ms才开始执行下一次fn。 事实上，setInterval并不管上一次fn的执行结果，而是每隔100ms就将fn放入主线程队列，而两次fn之间具体间隔多久就不一定了，跟setTimeout实际延迟时间类似，和JS执行情况有关。

```javascript
(function testSetInterval() {
    let i = 0;
    const start = Date.now();
    const timer = setInterval(() => {
        i += 1;
        i === 5 && clearInterval(timer);
        console.log(`第${i}次开始`, Date.now() - start);
        for(let i = 0; i < 100000000; i++) {}
        console.log(`第${i}次结束`, Date.now() - start);
    }, 100);
})();
```
输出
```
第1次开始 112
第1次结束 1025
第2次开始 1025
第2次结束 1926
第3次开始 1926
第3次结束 2828
第4次开始 2828
第4次结束 3730
第5次开始 3731
第5次结束 4609
```
由结果可以看出，下一次并不是等上一次执行完了之后才开始执行，实际情况是，早已经等在队列里了。另外可以看出，当setInterval的回调函数执行时间超过了延迟时间，已经完全看不出有时间间隔了。而setTimeout总是可以保证时间间隔总是不小于设置的时间。

> 如果setTimeout和setInterval都在延迟100ms之后执行，那么谁先注册谁就先执行回调函数。

同样我们可以使用clearInterval()方法并传入相应的ID来取消定时器。取消setInterval远远要比取消setTimeout更加重要，因为在不加干涉的情况下，间歇调用将会一直执行到页面卸载。

因此一般认为，使用setTimeout来模拟setInterval是一种最佳模式。如
```javascript
var num = 0;
var max = 10;
function incrementNumber() {
	num++;
	if(num < max){
		setTimeout(incrementNumber, 500);
	}else{
		alert("done");
		clearTimeout(id);
	}
}
var id = setTimeout(incrementNumber, 500);
```
> 在开发环境下，很少使用setInterval， 原因是setInterval后一次调用可能会在前一次调用结束前启动，而像前面事例中那样使用setTimeout，则完全可以避免这一点。所以最好不要使用setInterval。

### 关于“this”值的问题

实际调用setTimeout回调函数的执行环境和调用setTimeout的执行环境是不相同的。所以，如果没有没有使用call() 或者 bind()绑定this值时，在非严格模式下this默认值为golbal (或者就是window)，严格模式下为undefined。下面的例子可以说明：
```javascript
myArray = ['zero', 'one', 'two'];
myArray.myMethod = function (sProperty) {
    alert(arguments.length > 0 ? this[sProperty] : this);
};

myArray.myMethod(); // prints "zero,one,two"
myArray.myMethod(1); // prints "one"

setTimeout(myArray.myMethod, 1000); // prints "[object Window]" after 1 second
setTimeout(myArray.myMethod, 1500, '1'); // prints "undefined" after 1.5 seconds

setTimeout.call(myArray, myArray.myMethod, 2000); // error: "Uncaught TypeError: Illegal invocation at <anonymous>:12:12"
setTimeout.call(myArray, myArray.myMethod, 2500, 2); // same error
```
上面的代码， 当myArray.myMethod作为setTimeout的回调函数时，函数实际执行时myMethod中的this值指代[object  Window]。因为setTimeout的回调函数实际是在全局作用域中调用执行的。

同时当我们尝试使用call()方法改变this值时，会出现错误。为什么setTimeout不能使用call()方法，还没有搞懂具体原因，以后再研究。

现在的问题是，我们如何使this值正确的指代我们想要的对象？可能的几种方法是：
使用一个包装函数或者箭头函数
```javascript
setTimeout(function(){myArray.myMethod()}, 2000); // prints "zero,one,two" after 2 seconds
setTimeout(function(){myArray.myMethod('1')}, 2500); // prints "one" after 2.5 seconds

setTimeout(() => {myArray.myMethod()}, 2000); // prints "zero,one,two" after 2 seconds
setTimeout(() => {myArray.myMethod('1')}, 2500); // prints "one" after 2.5 seconds
```
> 关于this值的相关内容可以查看[Javascript中的this值]()

我们还可以通过重写全局函数中的setTimeout和setInterval函数，让setTimeout能够接收this值，并对回调函数使用Function.prototype.apply将this值赋给回调函数。
```javascript
// Enable setting 'this' in JavaScript timers
 
var __nativeST__ = window.setTimeout, 
    __nativeSI__ = window.setInterval;
 
window.setTimeout = function (vCallback, nDelay /*, argumentToPass1, argumentToPass2, etc. */) {
  var oThis = this, 
      aArgs = Array.prototype.slice.call(arguments, 2);
  return __nativeST__(vCallback instanceof Function ? function () {
    vCallback.apply(oThis, aArgs);
  } : vCallback, nDelay);
};
 
window.setInterval = function (vCallback, nDelay /*, argumentToPass1, argumentToPass2, etc. */) {
  var oThis = this,
      aArgs = Array.prototype.slice.call(arguments, 2);
  return __nativeSI__(vCallback instanceof Function ? function () {
    vCallback.apply(oThis, aArgs);
  } : vCallback, nDelay);
};
```
或者使用Function.prototype.bind来为回调函数绑定this值
```javascript
myArray = ['zero', 'one', 'two'];
myBoundMethod = (function (sProperty) {
    console.log(arguments.length > 0 ? this[sProperty] : this);
}).bind(myArray);

myBoundMethod(); // prints "zero,one,two" because 'this' is bound to myArray in the function
myBoundMethod(1); // prints "one"
setTimeout(myBoundMethod, 1000); // still prints "zero,one,two" after 1 second because of the binding
setTimeout(myBoundMethod, 1500, "1"); // prints "one" after 1.5 seconds
```

> Function.prototype.bind()函数式Javascript 1.8.5中引入的。IE 9以下是不支持的。



### 执行机制

先看一段代码
```javascript
function foo() {
  console.log('foo has been called');
}
setTimeout(foo, 0);
console.log('After setTimeout');
```
控制台输出结果为：
```
After setTimeout
foo has been called
```
从上面可以看出，即使定时器setTimeout的延迟时间设置为0，也不会马上执行setTimeout的回调函数。这是因为setTimeout的回调函数是被放在消息队列（或者任务队列）中的。而任务队列中的函数必须要等到主线程中正在执行的代码执行完毕才会去执行消息队列中的函数。这部分具体相关内容可以查看[并发模型与EventLoop]()

### Reference

[https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/setTimeout](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/setTimeout)  
[https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/setInterval](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/setInterval)   
[http://www.alloyteam.com/2016/05/javascript-timer/](http://www.alloyteam.com/2016/05/javascript-timer/)     
[http://www.cnblogs.com/Uncle-Keith/p/6443115.html](http://www.cnblogs.com/Uncle-Keith/p/6443115.html)   