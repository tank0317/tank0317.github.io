---
layout: post
title: IE事件模型
date: 2017-10-09
tags: javascript 
comments: true
---

我们通常使用的事件模型是W3C制定的DOM2级事件模型，这也是大多数现代浏览器支持的事件模型。然而在DOM2并没有并广泛支持之前，IE自己也有一套事件机制。有很多地方和DOM2级时间模型类似，也有一些地方不同。所以我们今天对比DOM2级事件模型，来学习一下IE的事件模型。

## 冒泡机制

在DOM2级事件模型中，事件的传递过程分为三个阶段，捕获阶段，目标阶段和冒泡阶段（浏览器实现中通常分为捕获阶段和冒泡阶段，其中冒泡阶段包含了目标阶段）。**IE事件模型的第一个不同就是，事件只有冒泡过程。**

<figure>
<img src="http://om0jxp12h.bkt.clouddn.com/eventflow.png" alt="eventFlow.png">
  <figcaption>Fig.1 - Event Flow</figcaption>
</figure>

还有一点需要注意的是，DOM2级事件传递过程中，捕获阶段是从window对象开始，冒泡阶段也是到window对象结束，但是在IE中，事件冒泡的最顶层的元素是document，不包含window。

## 事件对象

在DOM2级事件模型中，事件回调函数在执行时都会接受一个事件对象作为第一个参数，这个事件对象包含了刚刚发生事件的一些数据信息。**IE事件模型中，该事件对象是作为window的一个属性event存在的。**即我们在回调函数中如果想要获取该次时间的信息不是从函数参数获取而是从window.event获取。因此如果想要兼容两种事件模型，应该如下获取事件对象。

```javascript
function eventHandler(e) {
    if (!e) e = window.event;  // 兼容IE获取事件对象

    // 以下是事件处理函数的内容
}
```

另外IE的事件对象的属性设置与DOM2级事件模型也不同，其中具体属性设置可以查看[这里](https://msdn.microsoft.com/en-us/library/ms535863(v=vs.85).aspx);

其中我们经常使用过程中，不同之处在

* DOM2 event model中event对象的target属性，在IE中为srcElement;
* DOM2 event model中阻止事件进一步捕获或冒泡使用`event.stopPropagation()`方法，在IE中阻止冒泡使用`event.cancelBubble = true`;
* DOM2 event model中阻止时间的默认行为使用`event.preventDefault()`方法，在IE中使用`event.returnValue = false`;

因此如果想要兼容两种事件模型，应该如下处理：

```javascript
function eventHandler(e) {
    if (!e) e = window.event;  // 兼容IE获取事件对象

    var target = e.target || e.srcElement; //获取目标元素

    //阻止事件进一步传递
    if(e.stopPropagation){ 
        e.stopPropagation()
    }else {
        event.cancelBubble = true;
    }

    //阻止事件默认行为
    if(e.preventDefault){
        e.preventDefault();
    }else {
        e.returnValue = false;
    }
    // 以下是事件处理函数的内容
}
```

## 事件处理函数的注册和移除

在DOM2中提供了两个函数用作事件处理函数的注册和移除：addEventHandler()和removeEventHandler()。**IE事件模型中同样也提供了两个函数，attachEvent()和detachEvent()。**相同之处在于，函数的前两个参数都分别表示事件类型和要注册的事件处理函数。

不同之处有两点，

* 在IE中时间类型都以"on"开头，比如`attachEvent('onClick', function(){})`;
* DOME2级事件机制中事件注册和移除函数接受三个参数，第三个参数控制注册在时间捕获阶段还是冒泡阶段。IE中函数只接受两个参数，因为IE事件模型中并不支持事件捕获。

因此如果想要兼容两种事件模型，应该如下注册事件处理函数。

```javascript
var addEvent = function(elem, type, handler) {
	if(window.addEventListener){
        return elem.addEventListener(type, handler, false);
    }else if(window.attachEvent){
        return elem.attachEvent('on'+type, handler);
    }
}

//或者你可以使用下面的方式，可以对比下哪种更好
var addEvent = function(elem, type, handler) {
    if(window.addEventListener){
        addEvent = function(elem, type, handler){
            return elem.addEventListener(type, handler, false);
        }
    }else if(window.attachEvent){
        addEvent = function(elem, type, handler){
            return elem.attachEvent('on'+type, handler);
        }
    }

    addEvent(elem, type, handler);
}
```

另外有一点需要注意的是，在DOM2 event model中事件处理函数的this指向事件被绑定的元素，**在IE中，this指向window对象。**

我知道的就这么多了。

## Reference

<https://docstore.mik.ua/orelly/webprog/jscript/ch19_03.htm>

<https://testdrive-archive.azurewebsites.net/HTML5/ComparingEventModels/>

<https://msdn.microsoft.com/en-us/library/ms533023(v=vs.85).aspx>

<https://www.w3.org/TR/DOM-Level-3-Events/>

<https://developer.mozilla.org/zh-CN/docs/Web/API/Event>