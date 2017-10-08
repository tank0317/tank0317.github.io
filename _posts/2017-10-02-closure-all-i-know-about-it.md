---
layout: post
title: 闭包————我所知道的一切//
date: 2017-10-02
tags: javascript
comments: true
---

在javascript学习过程中以及和其他javascript开发者交流中，你会经常遇到“闭包”这个词语。“闭包”似乎被神化了,你不知道闭包就好像你根本不懂javascript一样。然而当我真正弄懂“闭包”的时候（我想我现在懂了），我认为即使你不知道闭包这个名词，不知道闭包的概念，对你正常使用javascript开发来说并无影响，只要你知道在javascript中函数内的自由变量是根据词法作用域来确定其值的，这就足够了。

接下来我会逐渐深入的介绍我知道的闭包的所有东西，我希望你能够和我有相同的感受。

## 一个小例子

我们先来看一个例子，这是我们在学习闭包中都会看到的一个经典的代码片段：当循环遇到闭包

```
for(var i=1; i<=5; i++){
	setTimeout(function timer() {
		console.log(i);
	}, i*1000);
}
```

这段代码的结果是，每隔1秒就会输出一个6，一共输出了5次6。

**第一个问题来了，为什么会是这样的结果？**我想大部分知道闭包概念的人都知道为什么。答案是：在javascript中作用域是词法作用域。作用域是什么？作用域是用来管理引擎如何根据变量名称来进行变量查找的。词法作用域又是什么？词法作用域是定义在词法阶段（编译的第一个阶段）的作用域，即词法作用域是由你在写代码时将变量和块作用域写在哪里来决定的。那么针对上面的代码来说，定义了5个定时器，每当一秒（是不是严格一秒这里不讨论）时间到的时候，一个定时器的回调函数会被拿出来执行，此时当引擎遇到变量i的时候，会根据词法作用域来确定它的值，所以引擎先查找当前函数作用域，发现没有i的定义，然后向父级作用域寻找i，发现有i的定义，直接拿来使用，此时i=6（因为for循环已经结束了），于是输出6。（这里还需要说一下变量的生命周期，for循环结束后i变量并没有被销毁，具体可查看javascript内存管理相关的知识）

**第二个问题来了，闭包在哪里？谁是闭包？** 我想很多人会卡在这里。很多资料里都会告诉我们上面代码运行结果之所以那样是因为闭包的存在。但是并没有说清楚谁是闭包。

## 闭包的概念

根据[MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures)上的解释：

> A closure is the combination of a function and the lexical environment within which that function was declared.

这里的定义给出了，闭包是函数和函数所在词法作用域的结合体。嗯？好像哪里不对。所有的函数定义都会有相应的词法作用域。如果这么定义的话，所有的函数和与之对应的词法作用域都可以认为是闭包了。

我们再看看[wikipedia](https://en.wikipedia.org/wiki/Closure_(computer_programming))上的解释：

> In programming languages, closures (also lexical closures or function closures) are techniques for implementing lexically scoped name binding in languages with first-class functions. Operationally, a closure is a record storing a function[a] together with an environment:[1] a mapping associating each free variable of the function (variables that are used locally, but defined in an enclosing scope) with the value or reference to which the name was bound when the closure was created.

你可以先忽略掉第一句话，这个我们后面再说。放在这里是因为，这里提到了闭包还叫词法闭包或者函数闭包，可见词法作用域和函数确实是闭包的两个关键点。第二句给了闭包（closure)的定义：函数和一个确定了函数内自由变量取值的环境的结合体。（这里的环境可以认为是词法作用域）

相比MDN的定义这里提到了**自由变量**。自由变量即没有在该函数内定义，而是在函数的包含体（父级作用域）内定义的变量。实际上有的地方还会把闭包定义为，包含自由变量的函数。可见自由变量的存在也是闭包的关键。而我认为wiki在这里给的定义要更合理一些。我为什么说这里的定义更合理？这个我们后面说。总之我们现在有了闭包的定义，我们可以回答，嗯...，准确的回答什么是闭包了。

**第三个问题来了，闭包有什么作用？**

## 闭包有什么作用

## 闭包常见的使用场景
## 闭包为什么叫闭包，名字怎么来的？
## 闭包和匿名函数的关系

。。。未完待续
