---
layout: post
title: vim小技巧（一）：解决gvim菜单栏乱码以及隐藏菜单栏
date: 2017-06-09
tags: vim
comments: true
---

该文章简单介绍如何配置gvim，避免菜单乱码的现象发生。同时，如果不喜欢菜单栏、工具栏、滚动条等，如何设置隐藏。这些在vim中都是可以灵活配置的。

## 菜单乱码

要说菜单乱码，那一定是gvim，因为vim本身是没有菜单的。另外乱码肯定是编码方式的问题，所以我们需要做的只是修改编码方式。

先上结果
```
"--------Gvim中文菜单乱码解决方案
" 设置文件编码格式
set encoding=utf-8
set fileencodings=utf-8,chinese,latin-1,gbk,gb18030,gk2312
if has("win32")
 set fileencoding=chinese
else
 set fileencoding=utf-8
endif

"解决菜单乱码 删除菜单，再重新添加菜单，vim会按照之前设定的编码格式创建菜单栏
source $VIMRUNTIME/delmenu.vim
source $VIMRUNTIME/menu.vim

"解决consle提示信息输出乱码
language messages zh_CN.utf-8

"--------以上是Gvim中文菜单乱码解决方案------------
```

以上复制自[这篇博客](http://blog.csdn.net/gatieme/article/details/55047156)

现在我们大致解释下上面的几个option的含义

| option | 含义|
|--- | ---|
|encoding | Sets the character encoding used inside Vim.  It applies to text in the buffers, registers, Strings in expressions, text stored in the viminfo file|
| fileencoding | Sets the character encoding for the file of this buffer. |
| fileencodings | This is a list of character encodings considered when starting to edit an existing file.  When a file is read, Vim tries to use the first mentioned character encoding.  If an error is detected, the next one in the list is tried.  When an encoding is found that works, 'fileencoding' is set to it.  If all fail, 'fileencoding' is set to an empty string, which means the value of 'encoding' is used.|

下次再来翻译吧。。。

## 隐藏菜单栏

然后，还有一个问题是，如果我不喜欢gVim的菜单栏，工具栏，该如何隐藏呢？

```
"设置gvim隐藏菜单栏，工具栏，滚动条
:set guioptions-=m  "remove menu bar
:set guioptions-=T  "remove toolbar
:set guioptions-=r  "remove right-hand scroll bar
:set guioptions-=L  "remove left-hand scroll bar
```

其他有关设置可以参考`:h guioptions`。

## Reference

<http://blog.csdn.net/gatieme/article/details/55047156>
<http://vimdoc.sourceforge.net/htmldoc/help.html>



