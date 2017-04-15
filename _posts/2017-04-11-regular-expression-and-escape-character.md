---
layout: post
title: C++/java的正则表达式里为什么那么多“\\\\”？
date: 2017-04-11
tags: canvas, front-end
comments: true;
---

## 问题来源

在javascript中我们新建一个正则表达式是这样的：

```javascript
var myRe = /d(b+)d/g;
```
但是在C++中我们会遇到这种情况：
```c++
// regular expression: \d -> matches a digit character
std::regex e1 ("\\d");  

// regular expression: \\ -> matches a single backslash (\) character
std::regex e2 ("\\\\"); 
```
C++里怎么有这么多“\\\\”？在Java新建正则表达式也是一样的，总是遇到好多“\\\\”。对于大神可能不屑这种问题，但是对于刚开始学习正则的小白来说，我们今天就好好来扒一扒这个问题。

## 分析问题

如果仔细看的话，会发现C++在创建正则表达式的时候是通过字符串来创建的。其实javascript也有通过字符串创建正则表达式的方式。如下：
```javascript
//匹配一个或多个数字或者英文字母同时后面跟一个空格
var re = new RegExp('\\w+\\s', 'g');
```
这个时候会发现，javascript中也会遇到好多“\\\\”？根据javascript的前后两种创建正则表达式的方式，我们会发现，真相是跟字符串有关。**如果通过字符串去创建正则表达式，会经常遇到多个“\\\\”出现的情况。**

所以现在的问题是，为什么使用字符串创建正则表达式会经常“\\\\”？这时候我们就要说一说**‘\’**以及**转义字符（Escape Character）**了。
> *Notice that, in C++, character and string literals also escape characters using the backslash character (\), and this affects the syntax for constructing regular expressions*

待续，编辑中。。。。

## Reference 

<https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions>
<http://www.cplusplus.com/reference/regex/ECMAScript/>
