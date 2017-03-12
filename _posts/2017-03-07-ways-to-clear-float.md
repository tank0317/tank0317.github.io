---
layout: post
title: 清除浮动的常用方式和优缺点
date: 2017-03-03
tags: css, front-end
---

首先我们要搞清楚一件事情，就是为什么要清除浮动？

我们这里谈论清除浮动的时，通常是指，子元素全部为float元素，而父级容器没有设置高度。此时，如果不清楚浮动，那么由于子元素为float元素，脱离了文档流，又因为，父级元素没有设置高度，因此父级元素会表现为，没有包含任何元素。结果就是，父元素的高度为0，无法显示。所以我们就要清除浮动。

<!--more-->

首先我们给定一个例子。如下
```html
<div class="clearfix" style="border: 1px solid red">
    <div style="float:left; width:100px; height:100px; background-color:lightblue"></div>
    <div style="float:left; width:100px; height:100px; background-color:lightgray"></div>
    <div style="float:left; width:100px; height:100px; background-color:lightgreen"></div>
</div>
```

## 方法一：使用伪类`:after`

```css
.clearfix:after {
	//content属性为空格用于兼容老版本Oprea出现的问题（具体什么问题，没有做研究）
	content: " ";
	display: block;
	clear: both;
}
```

关于使用伪元素实现清楚浮动，有很多种版本，主要是考虑了跨浏览器（兼容IE 6/7）以及margin collapse问题。**以上使用伪元素的方式是最简洁的版本，其中没有考虑兼容IE 6/7,(现在似乎没有这个需要)，而且没有考虑margin collapse问题。**具体为什么使用现在这个最简洁的版本，可以参考[关于使用伪元素清除浮动不同版本的讨论]()。

这种方式是最推荐的方式。    
优点：当现在不需要考虑兼容IE 6/7的时候，这种方式简洁有效，而且没有任何副作用。  
缺点：代码稍多一点。  


## 方法二：使用`overflow: hidden`

对父元素使用`overflow: hidden`, 这种方式的的原理即创建一个BFC(Block Formatting Context)。BFC有一个性质，即能够包含浮动元素。如果你还不知道什么是BFC，而且想知道什么是BFC，那么，可以参考我的另一篇总结[全面了解BFC]()。

优点：使用方式非常简单。这也是我之前经常使用的清除浮动的方式。（但是以后我决定使用第一种方式，比较高大上。）
缺点：`overflow: hidden`决定了无法显示需要溢出的元素，比如某些情况下和position配合使用。（其实我个人使用中还没有遇到这种情况，我现在还是一个小白 [笑脸]。）

由优缺点来判断，当我们不需要显示溢出元素的时候，这种方式也是一种非常好的清除浮动的方式。

## 方法三：结尾处添加空`<div>`标签使用`clear: both`

针对我们前面给定的例子，使用方式如下
```html
<div class="clearfix" style="border: 1px solid red">
    <div style="float:left; width:100px; height:100px; background-color:lightblue"></div>
    <div style="float:left; width:100px; height:100px; background-color:lightgray"></div>
    <div style="float:left; width:100px; height:100px; background-color:lightgreen"></div>
	<div style="clear:both"></div>
</div>
```
这种方式的原理为，我们在结尾处添加一个空白标签，同时设置样式`clear: both`，使得这个空白标签border-box只能出现在浮动元素的margin-box下面（如果这句话你不了解，看[这里]()）。空白标签是处于普通流中的，因此父元素为了能够包含这个空白标签，会使自身的高度增大到能够包含这个空白标签，这样就完成的包含浮动元素的目的。

> 如果你不知道什么是border-box和margin-box可以参考[盒模型](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Box_Model/Introduction_to_the_CSS_box_model)
> 


优点：浏览器支持好，不容易出现问题（这句话我是抄别人的）。
缺点：我们需要添加一个没有用的标签。而且如果多次需要使用清除浮动的时候，需要添加多个没有用的标签。这样做多么不环保！一点都不低碳！[微笑脸]

**结尾：第一种方式是网上最推崇的一种方式。但是在我看来如果能清楚这几种方式的原理，明白每种方式在使用中可能会出现什么其他问题，使用哪种方式无所谓。无论哪种方式，只要能够解决问题就是一种好的方式。**