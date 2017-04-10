---
layout: post
title: canvas基础
date: 2017-04-10
tags: canvas, front-end
comments: true;
---

# canvas 基础

canvas>是HTML5中添加的HTML元素。有了canvas元素我们可以通过javascript实现绘制图形，比如说画图，照片的组合以及实现一些简单甚至复杂的动画效果。

canvas元素的简单使用并不困难。这里我总结了canvas使用中的一些基本知识。

* 拿到画布（获取rendering context）
* 简单的画图操作（画方框，画直线）
* 两种画图方式
* 设置画笔样式
* transform
* 动画制作

## 获取rendering context 

要想使用canvas作图，需要有两步准备工作。

**首先我们需要有一个`<canvas>` 元素**

```javascript
<canvas id="mycanvas" width="150" height="150">
Your browser does not support the canvas tag.
</canvas>
```
上面我们插入了一个宽150 pixel高150 pixel的canvas元素。如果不指定宽高的话，默认是宽300 pixel 高 150 pixel。如果浏览器不支持canvas，这时会显示`<canvas>` 和`</canvas>`之间的信息。

**然后我们需要拿到rendering context**

```javascript
var canvas = document.getElementById('tutorial');
var ctx = canvas.getContext('2d');
```

我们可以把`<canvas>`元素看做是一个画板，如果我们想要画图的话，我们不是直接在画板上作画，我们会准备一张画布，然后在画布上作画。在这里，我们可以画2D的图形，也能画3D的图形。这两种画图需要不同的画布，这里我们只学习2D的作图。拿到画布的方式如上面的代码所示。

这里`getcontext('2d')`会返回一个`CanvasRenderingContext2D`的对象。我们之后所有的作图操作都是通过这个对象来完成。

**另外在javascript中可以使用下面的方式判断浏览器是否支持canvas**，

```javascript
var canvas = document.getElementById('tutorial');

if (canvas.getContext) {
  var ctx = canvas.getContext('2d');
  // drawing code here
} else {
  // canvas-unsupported code here
}
```  

## 简单的画图操作

### Drawing Rectangles

画图的所有操作都是基于坐标来实现的，而canvas中默认的坐标原点是在画布中的左上角，如下图所示

![canvas坐标](http://om0jxp12h.bkt.clouddn.com/Canvas_default_grid.png)

然后我们可以通过以下的canvas API进行简单的作图：

```javascript
fillRect(x, y, width, height)  // 画填充矩形
strokeRect(x, y, width, height) //画轮廓矩形
clearRect(x, y, width, height) //清除矩形
```
以上三个函数使用相同的参数，x, y表示作图的起始坐标，width, height表示矩形的宽和高。

当我们使用fillRect()和strokeRect()画矩形的时候（或者draw text 以及 draw image），这些图形都是在**immediate mode**下画的。就如字面的意思，我们调用这两个函数，他们会马上画出一个矩形。

然而还有一种模式是**path mode**或者**buffered mode**，我们下面会介绍。这种模式在我们画直线、画曲线、画弧线以及画矩形的时候更加高效。不过，矩形是唯一一种既有函数能够在immediate mode实现画图，也有函数能够在buffer mode下画图的图形。

### Drawing Path

在这里path（路径）可以是一系列的不同宽度、不同颜色的直线、曲线或者其他图形。通常使用路径去画图形会包含以下步骤：

1. 使用`beginPath()`命令创建路径
2. 使用[绘图命令](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D#Paths)绘制路径
3. 使用`closePath()`关闭路径（可选）
4. 使用`stroke()`或`fill()`命令渲染图形。

其中常用的绘图命令包括：
```javascript
//Moves the starting point of a new sub-path to the (x, y) coordinates.
moveTo(x, y);

//Connects the last point in the subpath to the x, y coordinates with a straight line.
lineTo(x, y);

//Adds an arc to the path which is centered at (x, y) position with radius r starting at startAngle and ending at endAngle going in the given direction by anticlockwise (defaulting to clockwise).
arc(x, y, radius, startAngle, endAngle, anticlockwise); 

//Adds an arc to the path with the given control points and radius, connected to the previous point by a straight line.
arcTo(x1, y1, x2, y2, radius);

//Creates a path for a rectangle at position (x, y) with a size that is determined by width and height.
rect(x, y, width, height);

//Adds an ellipse to the path which is centered at (x, y) position with the radii radiusX and radiusY starting at startAngle and ending at endAngle going in the given direction by anticlockwise (defaulting to clockwise).
ellipse(x, y, radiusX, radiusY, rotation, startAngle, endAngle, anticlockwise);
```

另外还有[绘制cubic Bézier曲线](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/bezierCurveTo)以及[绘制quadratic Bézier曲线](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/quadraticCurveTo)的命令，由于使用起来有些复杂，我们这里不做介绍，有兴趣的可以深入学习。

这里放两张图片辅助理解`arc(x, y, radius, startAngle, endAngle, anticlockwise)`命令以及`arcTo(x1, y1, x2, y2, radius)`的使用。
<figure>
  <img src="http://om0jxp12h.bkt.clouddn.com/circle1.jpg"  style="width: 400px; margin: auto" alt="arc命令的使用">
  <figcaption style="text-align: center;">Fig.1 - arc命令的使用</figcaption>
</figure>
<br>
![arcTo命令的使用](http://om0jxp12h.bkt.clouddn.com/arcTo.jpg)


