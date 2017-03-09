---
layout: post
title: 关于BFC（Block Formatting Context）的那些东西
date: 2017-03-09
---

@(前端学习笔记)[学习笔记]

# 触发BFC
我们先来看看，什么情况下会触发BFC（Block Formatting Context）。
根据[CSS2.1标准](https://www.w3.org/TR/CSS21/visuren.html#block-formatting)，以下情况下会触发BFC。
* float
* absolutely positioned elements
* inline-blocks, table-cells, and table-captions
* block boxes with 'overflow' other than 'visible'
* elements styled with "display:flex" or "inline-flex" (flex boxes) （CSS3中添加）

***
*Floats, absolutely positioned elements, block containers (such as inline-blocks, table-cells, and table-captions) that are not block boxes, and block boxes with 'overflow' other than 'visible' (except when that value has been propagated to the viewport) establish new block formatting contexts for their contents.*
***

这里需要注意，`display: table`并不会创建BFC，实际上当我们使用`display: table`的时候会创建一些匿名box（相关内容看[这里](https://www.w3.org/TR/CSS21/tables.html#anonymous-boxes)）这些匿名box中有`display: table-cell`的匿名box，创建了BFC。也就是说创建BFC的不是`display: table`，而是其中的匿名box。

>在我的另一篇介绍[清除浮动的文章]()中有使用`display: table`修饰伪元素从而避免margin collapse的原因也是这个。

# BFC的特性
然后我们再来看看BFC有哪些特性？   相比其他block box，
* **BFC会阻止margin collapsing。**   
关于[margin collapse]()有一点是除非他们处于同一个BFC，否则相邻block box的竖直方向上的margin会发生重叠（collapse）。也就是说，如果相邻的两个block box没有在同一个BFC，那么他们不会发生margin collapse。
<br>
比如：
 <div style="background-color: lightblue"><p style="margin: 20px"> this paragraph inside a DIV with a blue background, styled with `margin: 20px;` </p></div><div style="background-color: lightblue"><p style="margin: 20px"> this paragraph inside a DIV with a blue background, styled with `margin: 20px;` </p></div><div style="background-color: lightblue; overflow: hidden"><p style="margin: 20px"> This is a paragraph inside a DIV with a blue background, it is styled with `margin:20px`, The parent DIV is styled with `overflow:hidden`.</p></div>

对于前两个蓝色的box，段落的top和bottom margin发生了collapse（间隙是20px，而不是40px）。但是最后一个box，创建了一个新的BFC，因此他们的margins没有发生collapse。
<br>
* **BFC能包含浮动元素**
<br>
比如：
 <div style="background:lightblue"><p style="float:left; margin:20px"> This paragraph is a float inside a DIV with a blue background, it is styled with `margin:20px`</p></div><div style="background-color: lightblue; overflow: hidden; clear: left"><p style="float: left; margin: 20px">This paragraph is a float inside a DIV with a blue background, it is styled with `margin:20px`. The parent DIV is styled with `overflow:hidden`.</p></div>

第一个段落因为是浮动元素，所以它脱离了文档流，使得容器元素（父级box）发生了collapse，高度变为了0；因此没有显示蓝色背景。   
第二个背景同样是一个浮动元素，但是包含它的容器元素创建了一个新的BFC，使得它会包含浮动元素。

 **Note**：只有处于同一个BFC时，clear才会起作用。
<br>
* **BFC不会与浮动元素发生重叠**
<br>
根据[CSS标准](https://www.w3.org/TR/CSS21/visuren.html#bfc-next-to-float)BFC的border-box不会与浮动元素的margin-box发生重叠。

***
 *The border box of a table, a block-level replaced element, or an element in the normal flow that establishes a new block formatting context (such as an element with 'overflow' other than 'visible') must not overlap the margin box of any floats in the same block formatting context as the element itself.*
 ***   
 
 比如：   
 <br>
 <div style="background:skyBlue;float:left;width:180px"><pre>
 .sideBar { 
background: skyBlue; 
float: left; 
width: 180px; 
}</pre></div><div style="background:yellow;float:right;width:180px"><pre>.sideBar { 
background: yellow; 
float: right; 
width: 180px; 
}</pre></div><div class="gainLayout" style="/* margin: -20px; */background:pink;overflow:hidden;border:5px solid teal;"><pre>#main { 
background: pink; 
overflow: hidden; 
zoom: 1; 
border: 5px solid teal; 
} </pre></div>   
如果想要在pink box和兄弟元素之间产生空隙（比如20px），那么可以：
  * 给float元素设置20px的margin；
  * 为BFC设置margin值大于float元素的宽度（比如`margin: 0 220px`）；
注意第二条，BFC需要设置`margin: 0 220px`才可以，原因就是，这里不能与float元素发生重叠的是BFC的border box而不是margin box。并且当上述例子中，两个float元素之间的宽度不足以放下BFC的border box时，那么BFC将会下移，重起一行放置，防止与float元素发生重叠。

# Reference
<https://yuiblog.com/blog/2010/05/19/css-101-block-formatting-contexts/>   
<https://www.w3.org/TR/2011/REC-CSS2-20110607/visuren.html#q9.0>
 