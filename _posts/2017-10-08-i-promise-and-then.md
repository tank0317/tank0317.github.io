---
layout: post
title: promise细节
date: 2017-10-08
tags: javascript
comments: true
---

今天我们会根据promise A+规范[原文](https://promisesaplus.com/)[[中文译文](http://www.ituring.com.cn/article/66566)来对promise的几个细节做一点总结。如果没有阅读过promise规范的建议先看看中文译文，再看看原文。

## promise状态

我们都知道promise有三个状态，pending, fulfilled 和 rejected。而且当处于pending状态时可以切换到fulfilled或者rejected状态。但是有一点需要注意，当切换到fulfilled或者rejected状态的时候，就无法再切换到其他状态。

> When fulfilled, a promise: must not transition to any other state. When rejected, a promise: must not transition to any other state.

这决定以下三种情况下，只有第一次切换状态的操作会起作用。

```javascript
//得到一个fulfilled状态的promise
new Promise((resolve, reject) => {
	resolve(); //有效的操作
	reject();
});

//得到一个rejected状态的promise
new Promise((resolve, reject) => {
	reject(); //有效的操作
	resolve();
});

//得到一个rejected状态的promise
new Promise((resolve, reject) => {
	throw new Error("oh, error!"); //有效的操作
	resolve(); 
	reject();
});
```

## 关于then方法

promise另一项的重要内容就是提供了一个then方法。每个promie都会拥有一个then方法来获取promise的状态和终值（如果fulfilled）或拒因（如果rejected）。then方法接受两个参数。

> A promise’s then method accepts two arguments: promise.then(onFulfilled, onRejected)

### onFulfilled 和 onRejected
我们都知道一般onFulfilled和onRejected会是两个函数。当promise状态是fulfilled状态时会调用onFulfilled方法，否则调用onRejected方法。不过有一点需要注意的是，onFulfilled和onRejected方法的调用必须放到消息队列里去执行。

> onFulfilled or onRejected must not be called until the execution context stack contains only platform code.

> Here “platform code” means engine, environment, and promise implementation code. In practice, this requirement ensures that onFulfilled and onRejected execute asynchronously, after the event loop turn in which then is called, and with a fresh stack. This can be implemented with either a “macro-task” mechanism such as setTimeout or setImmediate, or with a “micro-task” mechanism such as MutationObserver or process.nextTick. Since the promise implementation is considered platform code, it may itself contain a task-scheduling queue or “trampoline” in which the handlers are called.

这里有提到macro-task和micro-task，如果你想知道更多关于这两个名词的知识，可以参考[这里](https://stackoverflow.com/questions/25915634/difference-between-microtask-and-macrotask-within-an-event-loop-context)和[这里](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)

比如下面的例子。
```javascript
Promise.resolve('tank0317').then((data) => {
	console.log(data); //后输出tank0317
});

console.log('hahaha');  //先输出hahaha
```

这就类似setTimeout(() => console.log('tank0317'), 0); 回调函数会被放在消息队列中，等待执行。

## catch方法

我们同样知道promise除了有一个then方法以为还有一个catch方法，只接受一个参数。但是实际上，catch方法也是then方法，只是对其进行了一次包装。

```javascript 
Promise.prototype.catch = function(onRejected) {
    this.then(null, onRejected);
}
```

或许你还知道我们可以级联使用then方法，并只提供onFulfilled方法，同时在最后使用catch方法来获取中间产生的错误。如

```javascript
Promise.resolve('tank0317').then(data => {
	throw new Error('oh, error.');
}).then((data) => {
	return data;
}).catch(error => {
	console.log(error.message);
});

//最终会输出，'oh, error.'
```

这是怎么而做到的？为什么可以这么写？当然，规范里有答案。

> then must return a promise. `promise2 = promise1.then(onFulfilled, onRejected);` If onRejected is not a function and promise1 is rejected, promise2 must be rejected with the same reason as promise1. 

我们先重写一下上面的代码：

```javascript
let promise1 = Promise.resolve('tank0317');

let promise2 = promise1.then(data => {
	throw new Error('oh, error.');
});

let promise3 = promise2.then((data) => {
	return data;
});

let promise4 = promise3.catch(error => {
	console.log(error.message);
});
```

promise2的状态是rejected同时拿到一个拒因error, 然后调用then方法得到promise3。由于promise2是rejected状态，同时then方法没有提供onRejected方法，因此promise3会切换到rejected状态，并得到和promise2相同的拒因error。如果这之后还有then方法并且只提供onFulfilled方法，那么所有中间得到的promise都会切换到rejected状态并得到相同的拒因error，使得这个拒因向下传递。因此我们可以再最后使用一个catch方法获取这个error。

### Promise.all

Promise.all()会接受一个数组作为参数，由多个promise组成的数组，同时返回一个promise。只有当数组中所有的promise的状态都切换为fulfilled状态是，这个promise才会切换为fulfilled状态，否则如果数组中人一个promise为rejected状态，则最终返回的promise都会切换到rejected状态，并得到相同的拒因。

如果此时我们希望，即使数组中有promise失败了我们还是希望能够得到其他所有promise的结果，该怎么做？

这里我还是想说then方法，promise2 = promise1.then(onFulfilled, onRejected);。

> If either onFulfilled or onRejected returns a value x, run the Promise Resolution Procedure [[Resolve]](promise2, x).
If onFulfilled is not a function and promise1 is fulfilled, promise2 must be fulfilled with the same value as promise1.

也就是说无论promise1是成功(fulfilled)还是失败(rejected)，只要then方法的onFulfilled或onRejected方法成功执行，最终返回的promise2都会是成功态(fulfilled)。我们可以利用这一点，来解决我们上面提出的问题。

```javascript
let promise1 = Promise.resolve('1'),
    promise2 = Promise.reject('error! 2'),
    promise3 = Promise.resolve('3');

//使用catch方法返回新的promise。我们这里没有提供onFulfilled方法
//因此当promise1是fulfilled状态时，promise2会得到相同的终值，并切换到fulfilled状态
promise1 = promise1.catch(error => {
	return error;
});
promise2 = promise2.catch(error => {
	return error;
});
promise3 = promise3.catch(error => {
	return error;
});

Promise.all([promise1, promise2, promise3]).then(data => {
	console.log(data);
});
```

今天就这样了
或许以后我还会补充一些我没有注意到的细节。。。