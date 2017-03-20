---
layout: post
title: 一个有关闭包的问题
date: 2017-03-17
tags: javascript, front-end
comments: true
---    

今天跟同学讨论一个问题，大致就是在for循环里批量对一些button绑定事件处理函数，一开始的写法并没有效果，后来发现是**闭包**的问题。本来以为自己之前看过一些闭包的知识，以为自己理解的还不错。今天实际碰到的时候，一下子懵逼了。先是不敢十分确定这是不是一个闭包的问题，然后是不知道该怎么去修改解决掉这个问题。好吧，为了让自己以后不至于继续出现这种尴尬的情况，我就记录一下今天的问题，以及尽量详细的分析一下问题的原因，以及如何解决这个问题。

不废话了，先看有问题的代码：    

<p data-height="265" data-theme-id="0" data-slug-hash="gmXpyr" data-default-tab="js,result" data-user="tank0317" data-embed-version="2" data-pen-title="for循环中绑定事件闭包的问题" class="codepen">See the Pen <a href="http://codepen.io/tank0317/pen/gmXpyr/">for循环中绑定事件闭包的问题</a> by tank0317 (<a href="http://codepen.io/tank0317">@tank0317</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

## 我们想要干什么？

我们给定了几个按钮，希望点击按钮的时候，能够改变一个**div块框**的长、宽、背景色或隐藏等。所以我们通过JS为DOM元素绑定相应的事件。我们把需要改变的css属性放在一个数组里，把相应的值value也放在一个数组里。然后，绑定的过程我们是在for循环里做的，通过索引值i，来分别对不同的按钮绑定不同的事件处理函数，对不同的属性设置不同的值。当然了，这是我们希望看到的过程，然后我们把它写成了JS代码。

但是。。。每个按钮点击都没有反应，跟我想要的不一样。你可以试试点击上面的例子。

所以问题出在哪里了？

## 问题出在哪里？

我们先说结论，问题是在for循环中我们给按钮绑定事件处理函数的时候，为每个按钮的onclick事件创建了一个匿名函数，也就是每个按钮的onclick都指向了一个函数体，其他的什么也没做（但是会给函数创建作用域链）。之后当用户点击某个按钮的时候，会调用相应的事件处理函数。此时，我们会为事件处理函数中的`changeStyle`函数传递参数，如`nDiv`, `nAtt[i]`， `nVal[i]`。但是此时匿名函数里并没有创建这些变量，所以我们会从父级作用域里去查找`nDiv`, `nAtt`,  `nVal`, `i` 的值。由于此时`i`的值在for循环结束以后已经变成了`nBut.length`，所以 `changeStyle` 函数无法获得正确参数。因此实际点击按钮时，没有任何反应。

以上是对问题的大概描述。

如果非要牵扯到一些
这其实是一个闭包的问题，也就是为每个按钮绑定的事件处理函数需要访问外部函数作用域中的变量。所以会出现上面的问题，当事件处理函数执行的时候，此时外部作用域中的变量已经不是我们想要的变量的值。此时其实有两个子问题

* 我们在绑定事件处理函数的时候为什么当时没有为`changeStyle` 函数传递参数（即，什么时候才会为函数传递参数）？
*  当事件处理函数执行的时候，`i` 既然不是函数内的变量，那`i` 该从哪里取值？ 

## 问题分析

### 函数的参数值什么时候确定？

我们先说答案，函数的参数值，只有在函数执行的是有才能确定。

再扯的话，就要扯到**执行环境**（*execution context*）了。当我们进入一个执行环境的时候，会做一些准备事情，比如对于全局执行环境，

 * 变量、函数表达式————变量声明、函数声明，默认赋值为`undefined` 
 * this 赋值
 * 函数声明 赋值

如果进入的执行环境是一个函数体，那么在函数执行之前还需要

* 参数——赋值
* arguments——赋值

从上面也可以看出，参数的确定是在函数即将执行时确定的。创建函数的时候是无法确定参数值的。

> 关于**执行环境**（执行上下文）可以参考[王福明的博客](http://www.cnblogs.com/wangfupeng1988/p/3986420.html)    

### `i` 该从哪里取值   

`i` 的取值这个要扯到作用域的问题，JS中的作用域是在函数创建的时候确定的。同时如果该函数是在一个外部函数内创建的，那么外部函数的作用域就会作为该函数的父级作用域。如果外层还有包裹函数，那么一样，都是作为父级作用域存在。也就是说，**函数创建的时候，会同时创建一个作用域链，作用域链包含了所有父级作用域的引用。需要强调的是<mark>这个作用域链是在函数创建的时候就确定了的</mark>。** 

可能又有人会说，作用域是个抽象的概念，实际中时怎么存在的呢，作用域链中的引用到底指向了什么？因为没有具体研究过，仅仅说下我的个人理解。**这些引用，都指向了父级函数或者全局作用域的变量对象（变量对象中包含了所有该函数或者全局作用域包含的变量）。** 关于变量对象的理解，可以参考[这里](https://www.zhihu.com/question/36393048)。

那么当时间处理函数执行的时候，同时会创建自身的作用域（变量对象），并把自身的作用域放到作用域链的顶端。然后，就遇到了自身作用域内不存在的`i` ，这时会沿着该函数的作用域链，向父级作用域查找，直到全局作用域为止。（全局作用域是没有父级作用域的。）

<hr>
*[Site: ](http://www.ecma-international.org/ecma-262/7.0/index.html#sec-lexical-environments) A global environment is a Lexical Environment which does not have an outer environment. The global environment's outer environment reference is null. *     
<hr>

> 关于作用域可以可以参考[王福明的博客](http://www.cnblogs.com/wangfupeng1988/p/3991151.html)       

好了，细节的东西基本弄清楚了，其实牵扯了好多的东西。。。接下来就是

## 解决问题 

这个时候，就不废话，直接上代码：
<p data-height="265" data-theme-id="0" data-slug-hash="QpOxvN" data-default-tab="js,result" data-user="tank0317" data-embed-version="2" data-pen-title="for循环中绑定事件闭包的问题解决办法" class="codepen">See the Pen <a href="http://codepen.io/tank0317/pen/QpOxvN/">for循环中绑定事件闭包的问题解决办法</a> by tank0317 (<a href="http://codepen.io/tank0317">@tank0317</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

解决办法就是在时间处理函数外部再套一个函数，这个函数会有一个参数，用来传递`i` 的值（实际是用来保存`i` 的值），并立即执行。这时，当事件处理函数执行过程中查找`j` 值的时候，回向父级函数查找`j` 的值，此时`j` 的值保存了我们想要的`i` 的值。然后问题就解决啦~  


## Reference 
[《JavaScript高级程序设计》第7章 函数表达式]()   
<http://www.cnblogs.com/wangfupeng1988/p/4001284.htmlhttp://www.cnblogs.com/wangfupeng1988/p/4001284.html>
<https://www.zhihu.com/question/36393048>
