---
layout: post
title: 获取鼠标位置
date: 2017-09-23
tags: javascript
comments: true
---

## 获取鼠标位置信息

在开发过程中有时候我们需要获取鼠标的位置信息。如果去看[DOM3](https://www.w3.org/TR/DOM-Level-3-Events/#events-mouseevents)标准中对鼠标事件对象的定义，可以看到在鼠标事件对象中有两对相关的鼠标位置属性。

* clientX/Y 相对浏览器的visual viewport的左上角为原点，得到的坐标（不受页面滚动影响）
* screenX/Y 相对物理显示器的左上角为原点，得到的坐标。

完整的鼠标事件对象定义为
```
interface MouseEvent : UIEvent {
  readonly attribute long screenX;
  readonly attribute long screenY;
  readonly attribute long clientX;
  readonly attribute long clientY;

  readonly attribute boolean ctrlKey;
  readonly attribute boolean shiftKey;
  readonly attribute boolean altKey;
  readonly attribute boolean metaKey;

  readonly attribute short button;
  readonly attribute unsigned short buttons;

  readonly attribute EventTarget? relatedTarget;

  boolean getModifierState(DOMString keyArg);
};
```

其实除了以上两对鼠标相关的属性外，没有被加入DOM3标准的但是几乎所有浏览器都支持的两对属相为：

* pageX/pageY 相对浏览器的layout viewport的左上角为原点，得到的坐标（受页面滚动影响）
* offsetX/offsetY 相对事件目标元素padding box左上角为原点，得到的坐标

<figure>
<img src="http://om0jxp12h.bkt.clouddn.com/pagex_and_clientx.png" alt="pageX/Y和clientX/Y">
  <figcaption>Fig.1 - pageX/Y和clientX/Y</figcaption>
</figure>

以上属相在[CSSOM View Module](https://drafts.csswg.org/cssom-view/#extensions-to-the-mouseevent-interface)的草案文件中有定义。

## 使用`element.getBoundingClientRect()`获取鼠标相对目标元素的位置信息。

在开发过程中，有时候为了得到鼠标相对某一元素的位置信息我们还会使用`element.getBoundingClientRect()`函数。该函数会返回元素left, top, right, bottom，x, y, width和height信息，其中位置信息为元素boder box相对visual viewport的数据。**但是大多数浏览器实现中都去掉了x, y信息。**

<figure>
<img src="http://om0jxp12h.bkt.clouddn.com/getBoundingClientRect.PNG" alt="getBoundingClientRect()">
  <figcaption>Fig.2 - getBoundingClientRect()</figcaption>
</figure>

利用鼠标位置clientX/Y和element.getBoundingClientRect()得到的元素位置信息rect对象,我们可以使用

* x: clientX - rect.x
* y: clientY - rect.y

表示鼠标相对特定元素的位置信息。如我在[h5lock](https://tank0317.github.io/H5lock/)中获取鼠标位置信息的代码如下：

```javascript
/**
 * 获取鼠标相对于canvas画布的位置
 **/
function getPosition(e) {   
    var rect = e.currentTarget.getBoundingClientRect(), 
        touch,
        po;
    if(this.clientType === 'mobile'){    
        touch = e.touches[0];  
        //console.log(touch);
        po = {
          x: touch.clientX - rect.left,
          y: touch.clientY - rect.top
        };
    }else{
        po = {
          x: e.clientX - rect.left,
          y: e.clientY - rect.top
        }
    }
    
    return po;
}
```

这里我们也可以使用offsetX/Y来获取鼠标相对元素的位置信息，只不过需要注意一点的是，offsetX/Y得到的是相对元素padding box的相对位置，和通过clientX/Y-rect.left/top得到的是相对元素border box得到的相对位置。

## 其他 

和getBoundingClientRect()长得很像的还有getClientRects()函数，这两个函数对block元素得到的结果相同，只不过getClientRects()得到的是一个数组。对于inline元素来说，如果inline元素跨行后，每行都会形成一个DOMRect，因此getClientRects()会得到多个DOMRect的位置大小信息组成的数组。而getBoundingClientRect()会得到包围所有DOMRect的最小矩形的位置大小信息。

这里有一个测试页面<https://www.quirksmode.org/dom/tests/rectangles.html>

## Reference

<https://stackoverflow.com/questions/6073505/what-is-the-difference-between-screenx-y-clientx-y-and-pagex-y#>

<https://stackoverflow.com/questions/6333927/difference-between-visual-viewport-and-layout-viewport>

<https://stackoverflow.com/questions/33688549/getbbox-vs-getboundingclientrect-vs-getclientrects>

<https://www.w3.org/TR/cssom-view-1/#event-summary>

<https://www.quirksmode.org/dom/tests/rectangles.html>

<https://developer.mozilla.org/en-US/docs/Web/API/Element/getBoundingClientRect>
