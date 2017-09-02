---
layout: post
title: clientHeight, scrollHeight, scrollTop以及滚动无限加载的实现
date: 2017-09-02
tags: layLoad front-end
comments: true;
---
## clientHeight, scrollHeight, scrollTop以及滚动无限加载的实现

我们浏览网页的时候经常会遇到这样的情况，当我们把鼠标拖动要底部的时候，会出现一个转动的菊花图，然后菊花图消失出现了更多的内容供我们浏览。当我们再次滚动到底部的时候，又会出现上面的情况，菊花图出现又消失，我们又有新的资源可以浏览。那么这种无限滚动渐进的获取内容的方式是怎么实现的呢？

其实想一想，思路就是，当我们判断用户滚动到页面底部的时候，我们会异步获取更多的数据，这个过程中会有一个菊花图出现的页面中，当数据成功返回的时候，我们用把新的数据添加到页面中，同时移除菊花图。

显示一个菊花图，移除一个菊花图，异步获取数据，把新数据包装成DOM节点添加到页面中这都不是问题，所以关键在于怎么判断用户已经滚动到页面底部了。嗯，废话那么多，都是凑字数的，下面才是重点。

### clietHeight, scrollHeight, scrollTop

这里我们需要先介绍下这三个属性的作用。

* clientHeight 只读属性，返回元素内部的高度(单位像素)，包含内边距，但不包括水平滚动条、边框和外边距。(content + padding)
* scrolltHeight 只读属性，返回计量元素内容高度，包括overflow样式属性导致的视图中不可见内容。(content + hidden content + padding)
* scrollTop 属性可以设置或者获取一个元素被卷起的像素距离。

更形象的描述可以看下面的图片：

<figure>
<img src="http://om0jxp12h.bkt.clouddn.com/scrollHeightVSoffsetHeight.png" alt="clientHeight和offsetHeight">
  <figcaption>Fig.1 - clientHeight和offsetHeight</figcaption>
</figure>

<figure>
<img src="http://om0jxp12h.bkt.clouddn.com/scrollHeight.png" alt="scrollHeight">
  <figcaption>Fig.2 - scrollHeight</figcaption>
</figure>

<figure>
<img src="http://om0jxp12h.bkt.clouddn.com/scrollTop.png" alt="scrollTop">
  <figcaption>Fig.3 - scrollTop</figcaption>
</figure>

### 判断元素是否滚动到底

如果元素滚动到底，下面等式返回true，没有则返回false.

```javascript
element.scrollHeight - element.scrollTop === element.clientHeight
```

核心就是我们上面提到的三个属性的组合使用。我们通过监听页面的`scroll`事件，每次滚动都使用上面的公式来判断页面是否已经滚动到底部，从而加载更多的数据。

### 实例

<https://github.com/jiangchangzzz/demo/tree/master/src/scrollInfinite>

上面的链接是百度前端技术学院中[jiangchangzzz](https://github.com/jiangchangzzz)给出的demo源码。

### 其他

上面我们看到clientHeight, scrollHeight, scrollTop。其他和他们经常一起出现的还有clientHeight/clientTop, scrollHeight/scrollTop, offsetHeight/offsetTop。类似的还有clientWidth/clientLeft, scrollWidth/scrollLeft, offsetWidth/offsetLeft。有兴趣的可以去MDN或者别人的博客上搜一搜相应的属性的作用。

这里放一张古老的，有点不是很清晰的图片（我自己都懒得画，所以还是要感谢图片的原作者）。

<figure>
<img src="http://om0jxp12h.bkt.clouddn.com/HeightandTop.gif" alt="所有表征元素宽高位置的属性集合">
  <figcaption>Fig.4 - 全家福</figcaption>
</figure>

### Reference

<http://www.imooc.com/article/7945>

<https://stackoverflow.com/questions/22675126/what-is-offsetheight-clientheight-scrollheight>

<https://developer.mozilla.org/zh-CN/docs/Web/API/Element/scrollTop>

<https://developer.mozilla.org/zh-CN/docs/Web/API/Element/scrollHeight>

<https://developer.mozilla.org/en-US/docs/Web/API/Element/clientHeight>



