---
layout: post
title: JavaScript中的this值
date: 2017-03-10
tags: javascript front-end
comments: true
---
本片文章主要翻译自[MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this)。   

this是JavaScript中的一个关键字。通常情况下，this值主要和function如何调用有关。另外this无法在运行过程中赋值，并且可能随着使用场合的不同，this值会发生变化。ES5中引进了bind()方法来为function绑定this值而不需要考虑它是如何调用的。ES6中引入了arrow function，它的this则由所在上下文中的this值决定(具体可参考[这里](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)。   

<!--more-->

## Global Context（全局作用域）   

在全局作用域下，this值指向global object。无论是否在严格模式。

```javascript
console.log(this.document === document); //true

// In web browsers, the window object is also the global object:
console.log(this === window); //true

this.a = 37;
console.log(window.a) //37
```   

## Function Context(函数作用域）

函数中的this值和函数的调用有关。

### 简单的调用   

在非严格模式下，如果没有使用call方法绑定this值，那么this指向global object。

```javascript
function f() {
    return this;
}

//In a browser,
f() == window; // the window is the global object in browsers

//In Node,
f() == global;
```   
在严格模式下，this由进入执行环境时的设定值决定。   
```javascript
function f2() {
    "use strict"
    return this;
}

f2() === undefined;//
```    
如上所示，如果没有设置this值，则默认为undefined。   

### call or apply   

如果一个函数中使用了this关键字，那么可以使用[call](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call)或者[apply](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)方法来修改这个this值，使其指向一个特定的对象。所有的函数都能从Function.prototype继承call和apply方法。   

```javascript
function add(c, d) {
    return this.a + this.b + c + d;
}

var o = { a: 1, b: 3};

add.call(o, 5, 7); // 1+3+5+7 = 16

add.apply(o, [10, 20]); // 1+3+10+20
```   

需要注意的是，当向call或者apply中传递this的时候，如果传递的参数不是一个对象，那么将会尝试使用toObject操作将这个参数转化为一个对象。所以，如果我们使用的是基本类型的参数如 7 或者 "foo"，那么将会使用相应的构造器转化为对象。比如基本类型的数字7将会转化为好像使用new Number(7)生成的对象，而字符串"foo"转化为new String("foo"）生成的对象。   

```javascript
function bar() {
    return Object.prototype.toString.call(this);
}

bar.call(7); // [object Number]
```   

### The bind method

ES6中引入了Function.prototype.bind方法。使用f.bind(someObject)将会生成一个新的函数，这个函数与f拥有相同的函数体与作用域，但是原本出现在原始函数中的this值在新函数中将会永久的绑定到bind方法的第一个参数上，不论这个函数式怎么调用。   

```javascript
function f() {  
   return this.a;
}  

var g = f.bind({a: "tank0317"});
console.log(g()); //tank0317 

var h = g.bind({a: "tank"}); //bind只能用一次
console.log(h()); //tank0317

var o = { a: 37, f: f, g: g, h: h });
console.log(o.f(), o.g(), o.h()); //37， tankk0317, tank0317
```   

### Arrow Function   

在箭头函数中，this值由内附执行上下文（enclosure execution context）决定。如果在全局作用域中，则this值为global。   

```javascript
var globalObject = this;
var foo = (() => this);
console.log(foo === globalObject); // true 
```   
这里无论foo方法怎么调用this值都不会改变。无论是作为一个对象的方法来调用，还是使用call、apply或者bind函数绑定this值。   

```javascript
//作为一个对象的方法调用
var obj = { foo : foo};
console.log(obj.foo() === globalObject); //true  

//使用call方法设置this值
console.log(foo.call(obj) === globalObject); //true 

//使用bind方法绑定this值
var foo = foo.bind(obj);
console.log(foo() === globalObject); //true 
```
由上面可以看出，foo的this始终被设置为被创建时的所在上下文的this值。**同样的，在函数内部创建箭头函数时，箭头函数的this被永久的设置为箭头函数被创建时外部函数的this值</mark>**   

```javascript
var obj = {
	bar : function () {
    var x = ( () => this );
		return x;
    }
}
// bar作为obj的方法调用，这时内部箭头函数的this值被设置为obj
var fn = obj.bar();
console.log(fn() === obj); //true 

// 当把bar函数引用赋给fn2时，调用fn2()()会发生两个过程，
// 第一过程执行fn2()，此时会创建一个箭头函数并返回，这时箭头函数的this值被设置为外部函数的this值，即window
// 第二个过程执行fn2()返回的箭头函数，此时this === window。
var fn2 = obj.bar;
console.log(fn2()() === window); //true
```   
### As an object method   

当一个函数作为一个对象的方法被调用是，此时this值指向这个对象。

```javascript
var o = {
    prop : 37,
    f : function () {
	   return this.prop;
    }
}

console.log(o.f()); // logs 37
```
需要注意的是，函数在哪里定义声明的并不会影响结果。比如上面的例子是在定义对象的同时，把定义了一个函数作为该对象的属性成员。但是，并不一定非要这么做，我们还可以先定义一个函数，然后再赋值给o.f。   

```javascript
var o = {prop : 37};
function independent () {
	return this.prop;
}

o.f = independent;

console.log(o.f()); //logs 37
```
另外，这种方式还遵循就近原则。比如下面的例子

```javascript
o.b = { g: independent, prop: 32};
console.log(o.b.g()); //logs 32

o.c = { h: independent }
console.log(o.c.h()); //logs undefined
```
在上面的例子中函数g和h是分别作为对象o.b和o.c的方法来调用的，因此此时函数中的this值指向对象o.b和o.c。

#### this on the object's prototype chain

类似的机制，当函数被定义在对象的原型链中时，函数中的this值指向调用该函数的对象。

```javascript
var o = { f : function() { return this.a + this.b; }};
var p = Object.create(o);
p.a = 10;
p.b = 20;
console.log(p.f()); //logs 30
```
在上面的例子中，虽然对象p中没有定义方法f，但是p的原型对象中有f。这时当使用p.f()时，f中的this值会指向调用它的对象p。这是javascript中有关原型继承的非常有趣的一点。

#### this with getter or setter

类似的还有，当函数被当做getter和setter函数调用时，此时函数中的this指向get或者set该属性的对象。

```javascript
function sum () {
    return this.a + this.b + this.c;
}
 
var o = {
    a: 1,
    b: 2,
    c: 3,
    get average() {
    	return (this.a + this.b + this.c)/3;
    }
}
 
Object.defineProperty(o, "sum", {
	get: sum, enumerable: true, configurable: true
});
console.log(o.average, o.sum); // logs 2, 6
```

### As a constructor 

当函数被用做构造器，和new关键字一起使用时，函数中的this值指向新创建的对象。

```javascript
function C() {
	this.a = 37;
}

var o = new C(); 
console.log(o.a); // logs 37

function C2() {
    this.a = 37;
    return {a: 38};
}
var o = new C2();
console.log(o.a); // logs 38
```
在上面的例子中， C2最后返回的是一个对象，那么这个对象将作为这个new表达式的返回值。

### As a DOM event handler

如果一个函数作为一个事件处理函数，那么函数中的this值指向绑定该事件的元素。   

```javascript
function bluify(e) {
    console.log(this == e.currentTarget);
    console.log(this === e.target);
    if(this == e.target) {
        this.style.backgroundColor = "lightgreen";
    }
}

var elements = document.getElementsByTagName('*');

for(var i=0; i<elements.length; i++) {
    elements[i].addEventListener('click', bluify, false);
}
```






















