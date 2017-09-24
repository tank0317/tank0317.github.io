---
layout: post
title: 利用canvas实现网页版手势解锁
date: 2017-03-27
tags: javascript html5 front-end
comments: true
---

2017.03.23   
这是360的2017年暑期实习前端星计划的作业题。要求：实现一个移动网页，允许用户设置手势密码和验证手势密码。已设置的密码记录在本地 localStorage 中。

界面大致如下图：

![h5lock页面展示](http://om0jxp12h.bkt.clouddn.com/h5lock.png)

2017.03.26   
今天在PC端实现了作业要求手势解锁的所有功能，同时希望能把这几天的实现思路整理一下，写一个大概的实现过程，算是对自己这几天学习的一个梳理。

整个过程，大致分为三个阶段：

* 初始化面板 
* 随着touch或鼠标移动，更新touch或鼠标移动路径
* touch或者鼠标操作结束后，判断密码状态（由两个radio按钮决定）   
	* 如果此时为设置密码状态，则存储密码
	* 如果此时为验证密码状态，则验证是否和存储密码相同

以上的部分主要是参考了<http://www.alloyteam.com/2015/07/html5-shi-xian-ping-mu-shou-shi-jie-suo/>的实现思路。在此之上，优化了部分代码，同时添加了部分代码实现了PC端的支持，此时无论使用PC还是mobile都能看到正确的效果。

[H5手势解锁最终效果](https://tank0317.github.io/H5lock)

[Github源码](https://github.com/tank0317/H5lock)

### 初始化面板    

这里主要是通过canvas画出面板上的所有（9个）圆点，实现过程为，我们用arr[]数组存放所有圆点的坐标及索引（唯一标识圆点），同时初始化`lastPoint[]`和`restPoint[]`数组，其中`lastPoint`数组存放已经记录在手势路径中的圆点，`restPoint`存放还没有被记录在手势路径中的圆点。代码如下：
```javascript
/**
 * 存储每个圆点的坐标及索引index
 * 存放在arr[]数组中
 **/
function savePointPosition(){    
    this.r = ctx.canvas.width / (2 + 4 * 3);
    this.arr = [];
    var i = 0,
        j = 0,
        x = 0,
        y = 0,
        r = this.r,
        count = 0,        
        obj;
    for(i=0; i<3; i++){
        for(j=0; j<3; j++){
            obj = {
                x: j*4*r + 3*r,
                y: i*4*r + 3*r,
                index: count++
            };
	        this.arr.push(obj);
        }
    }
}

function drawCircle(x, y) {
    this.ctx.save();   //保存画图状态，

    //设置画图边框颜色，线宽，填充颜色属性
    this.ctx.strokeStyle = '#bbbbbb';  
    this.ctx.lineWidth = 2;
    this.ctx.fillStyle = 'white';

    this.ctx.beginPath();
    this.ctx.arc(x, y, this.r, 0, Math.PI*2, true);
    this.ctx.closePath();
    this.ctx.stroke();
    this.ctx.fill();

    this.ctx.restore();  //恢复原本画图状态
}

function drawPanel(){
    this.lastPoint = []; //存放已经记录在手势路径中的圆点
    this.restPoint = this.arr.slice();  //存放还没有被记录在手势路径中的圆点

    //画出所有圆点
    this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height); //清理canvas区域
    for(i=0; i<this.arr.length; i++){
        drawCircle(this.arr[i].x, this.arr[i].y);
    }
}
```

### 更新移动路径  

在这个部分，首先我们要为canvas元素绑定两个touch事件处理函数，分别是touchstart事件的moveStart处理函数，touchmove事件的move处理函数。在moveStart函数中判断用户的手势是否是一次有效的手势开始（只有从任一圆点处开始触摸滑动才认为是一次有效的手势），此时将`touchFlag = true`，之后触摸滑动的过程中，主要是通过不断触发touchmove事件，然后调用`update()`函数更新手势路径。这部分代码如下：
```javascript
/**
 * touchmove事件处理函数
 * 如果touchFlag为true，表明是一次有效的触摸手势，
 * 不断更新手势路径
 */
function move(e){    
    if(self.touchFlag){ 
        console.log('move fired');
        self.update(self.getPosition(e)); //更新手势路径
    }
}

/**
 * update主要为更新并画出手势路径
 **/
function update(po){
    var i = 0;
    this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
    
    for(i=0; i<this.arr.length; i++){ //先画面板
        this.drawCircle(this.arr[i].x, this.arr[i].y);
    }
    
    this.drawPoint(this.lastPoint);  //标记所有已经记录在路径中的圆点
    this.drawLine(po, this.lastPoint); //画出已经记录的手势路径
    
    //console.log(this.restPoint.length);
    for(i=0; i<this.restPoint.length; i++){
        //判断触摸点是否在圆点内，
        if(Math.abs(po.x - self.restPoint[i].x) < self.r && Math.abs(po.y - self.restPoint[i].y) < self.r){
            this.lastPoint.push(this.restPoint[i]); 
            this.restPoint.splice(i, 1);
            break;
        }
    }
}
``` 

### 处理手势密码

在每次手势操作结束后，如果我们选择的是设置密码，那会将这次手势作为密码保存；如果选择的是验证密码，我们会将这次手势和保存的手势密码对比，判断是否解锁成功。所以我们为canvas绑定了一个touchend事件，当一次手势结束后，我们会调用moveEnd处理函数。
```javascript
function moveEnd(e){
    var i = 0;
    if(self.touchFlag){
        console.log('moveEnd fired.');
        self.touchFlag = false;        
        self.storePass(self.lastPoint); //处理手势，存储密码或验证密码是否正确
        
        self.drawPanel(self.arr[i].x, self.arr[i].y); //重新画面板
    }
}

/**
 * 存储或判断密码是否正确
 * pswObj.step = 1 表示设置密码且已经输入过一次密码，则将手势与第一次输入的手势对比
 *             = 2 表示已经设置过密码，然后判断输入的手势是否正确；
 *             = undefined 表示还没有设置过密码，则将手势保存；
 * pswObj.fpassword 中存放第一次设置密码的手势路径
 **/
function storePass(psw) {
    if(this.pswObj.step == 1){
        if(this.checkPass(this.pswObj.fpassword, psw)){            
            //将密码保存在本地
            this.pswObj.password = psw; 
            window.localStorage.setItem('passwordH5lock', JSON.stringify(this.pswObj.password)); 
            //更新密码状态，已经保存密码
            this.pswObj.step = 2;                     
            document.getElementById('title').innerHTML = '密码设置成功';
            document.forms[0].identify.checked = true;
        }else{
            document.getElementById('title').innerHTML = '两次输入不一致，请重新输入';
            delete this.pswObj.step;
        }
      
    }else if(this.pswObj.step == 2){
        if(this.checkPass(this.pswObj.password, psw)){
            document.getElementById('title').innerHTML = '密码正确';
        }else{
            document.getElementById('title').innerHTML = '输入密码不正确';
        }
      
    }else {
        this.pswObj.step = 1;
        this.pswObj.fpassword = psw; //暂存第一次设置的密码
        document.getElementById('title').innerHTML = '再次输入手势密码';
    }
}
```

以上三个过程是这次手势解锁的三个主要过程。

在实现过程中，还有

* 判断客户端类型，PC or mobile
* 画出手势路径，包括画圆点，画直线
* 为radio绑定事件，切换设置密码和验证密码两种状态
* 如何存储密码，判断两次手势是否相同
* 判断密码是否不小于4个点

等等一些细节问题，可以直接看h5lock.js中的源码，我为每个函数都添加了尽可能详细的注释。

