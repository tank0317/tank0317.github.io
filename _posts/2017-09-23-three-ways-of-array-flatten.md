---
layout: post
title: js数组扁平化的几种方法
date: 2017-09-23
tags: javascript lodash
comments: true
---

我了解的数组扁平化的方法有三种。

## 第一种，我们自己实现

```javascript
function flat(arr){
	var res = [];
	for(let el of arr){
		if(Array.isArray(el)){
			res = res.concat(flat(el));
		}else{
			res.push(el);
		}
	}
	return res;
}

let arr = [1,[2,[3,4]]];

flat(arr); // [1, 2, 3, 4]
```

## 第二种，利用generator

这是一种非常巧妙的方式，使用generator来创建一个迭代器对象生成函数，在这个generator中对于元素仍然是数组的元素，继续使用generator创建迭代器对象，同时使用yield*对其进行遍历，如此深入。这个方法在阮一峰老师[《ECMAScript6 入门》](http://es6.ruanyifeng.com/#docs/generator#yield--表达式)中有介绍。

```javascript
function* _flat(el){
	if(Array.isArray(el)){
		for(let item of el){
			yield* _flat(item);
		}
	}else{
		yield el;
	}
}

function flat(arr){
	return [..._flat(arr)];
}
```

## 第三种，lodash的baseFlatten

在lodash中和数组扁平化相关的方法有三个，`flatten()`, `flattenDepth()`和`flattenDeep()`。三个方法的区别只是扁平化的程度不同。实际实现上都是对`baseFlatten()`函数的包装。

其中`baseFlatten()`的核心代码如下：

```javascript
function baseFlatten(arr, depth, result){
	result || (result=[]);

	if(arr === null) return result;

	for(const val of arr){
		if(depth>0 && Array.isArray(val)){
			if(depth>1){
				baseFlatten(val, depth-1, result);
			}else{
				result.push(...val);
			}
		}else{
			result[result.length] = val;
		}
	}
	
	return result;
}
```

实际源码略有区别，可以参考[源码](https://github.com/lodash/lodash/blob/master/.internal/baseFlatten.js)。

