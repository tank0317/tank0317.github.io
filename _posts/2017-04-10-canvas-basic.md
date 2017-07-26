---
layout: post
title: canvas基础（一）
date: 2017-04-10
tags: canvas front-end
comments: true;
---

# canvas 基础（一）

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

<figure>
<img src="http://om0jxp12h.bkt.clouddn.com/Canvas_default_grid.png" alt="canvas坐标">
  <figcaption>Fig.1 - canvas坐标</figcaption>
</figure>


然后我们可以通过以下的canvas API进行简单的作图：

```javascript
fillRect(x, y, width, height)  // 画填充矩形
strokeRect(x, y, width, height) //画轮廓矩形
clearRect(x, y, width, height) //清除矩形
```
以上三个函数使用相同的参数，x, y表示作图的起始坐标，width, height表示矩形的宽和高。

当我们使用fillRect()和strokeRect()画矩形的时候（或者draw text 以及 draw image），这些图形都是在**immediate mode**下画的。就如字面的意思，我们调用这两个函数，他们会马上画出一个矩形。

然而还有一种模式是**path mode**或者**buffered mode**，我们下面会介绍。这种模式在我们画直线、画曲线、画弧线以及画矩形的时候更加高效。不过，矩形是唯一一种既有函数能够在immediate mode实现画图，也有函数能够在**buffer mode**下画图的图形。

### Drawing Path

在这里path（路径）可以是一系列的不同宽度、不同颜色的直线、曲线或者其他图形。路径的绘制是在**path mode/buffered mode**下工作的。通常使用路径去画图形会包含以下步骤：

1. 使用`beginPath()`命令创建路径
2. 使用[绘图命令](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D#Paths)绘制路径
3. 使用`closePath()`关闭路径（可选）
4. 使用`stroke()`或`fill()`命令渲染图形。

#### 渲染路径（render path）

渲染路径有两种方式，绘制路径轮廓和填充路径区域。分别对应`stroke()`或者`fill()`两个命令。

在**buffered mode**下任何的绘图命令都不会立即在屏幕上绘制出图形，而是暂存在内存里，当我们调用`stroke()`或者`fill()`命令的时候才会将所有之前添加的在内存里的路径一次性绘制到屏幕上。另外，需要注意的是，执行`stroke()`或者`fill`命令，内存中的路径并不会清空，如果想要清空之前的路径，需要调用`beginPath()`命令。

这里我们给出一个例子来说明`beginPath()`、`stroke()`以及`fill`的正确使用方式：

<figure>
<img src="http://om0jxp12h.bkt.clouddn.com/twicedrawnpathcorrectnew_copie.jpg" alt="beginPath()示例图片">
<figcaption>Fig.2 - beginPath()示例图片</figcaption>
</figure>

```javascript
var canvas=document.getElementById('myCanvas');
var ctx=canvas.getContext('2d');
// first part of the path
ctx.moveTo(20,20);
ctx.lineTo(100, 100);
ctx.lineTo(100,0);
// indicate stroke color + draw first part of the path
ctx.strokeStyle = "#0000FF";
ctx.stroke();
// start a new path, empty the current buffer
ctx.beginPath();
// second part of the path
ctx.moveTo(120,20);
ctx.lineTo(200, 100);
ctx.lineTo(200,0);
// indicate stroke color + draw the path
ctx.fillStyle = "pink";
ctx.fill();
```

#### 常用绘图命令

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
  <img src="http://om0jxp12h.bkt.clouddn.com/circle1.jpg" alt="arc命令的使用">
  <figcaption>Fig.3 - arc命令的使用</figcaption>
</figure>

atcTo一般的使用方式如下：

```javascript
ctx.moveTo(x0, y0);

ctx.arcTo(x1, y1, x2, y2, radius);
```
<figure>
  <img src="http://om0jxp12h.bkt.clouddn.com/arcTo.jpg" alt="arcTo命令的使用">
  <figcaption>Fig.4 - arcTo命令的使用</figcaption>
</figure>

#### 关闭路径 

我们使用closePath()关闭路劲。那关闭路径（closing path）是什么意思呢？我们先看一个例子：

```javascript
var canvas=document.getElementById('myCanvas');
var ctx=canvas.getContext('2d');
// Path made of three points (defines two lines)
ctx.moveTo(20,20);
ctx.lineTo(100, 100);
ctx.lineTo(100,0);
// Close the path, try commenting this line
ctx.closePath();
// indicate stroke color + draw first part of the path
ctx.strokeStyle = "blue";
ctx.stroke();
```

代码中我们只画了两条线，然后我们是否使用`colsePath()`会造成下图所示的区别：
<figure>
<img src="http://om0jxp12h.bkt.clouddn.com/closePath.jpg" alt="有无closePath的区别">
<figcaption>Fig.5 - 有无closePath的区别</figcaption>
</figure>

上图中当我们使用`closePath()`的时候，这个时候调用`stroke()`会把路径（path）的终点和起始点连接起来，形成一个封闭的路径轮廓，如果不调用的时候，终点和起始点是不会自动连接在一起的。但是如果最后调用的是`fill()`命令，那么即使没有调用`closePath()`也会绘制出路径填充图如之前的Fig.3所示。