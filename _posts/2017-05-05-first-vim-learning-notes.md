---
layout: post
title: vim学习笔记（一）：配置文件vimrc
date: 2017-05-05
tags: vim
comments: true
---

要让vim按照我们的习惯设置启动，我们可以把这些设置写到一个叫vimrc的文件中。当vim启动的时候会执行这个文件中的命令。

如果你已经有了vimrc文件，你可以这样编辑它

```
:edit $MYVIMRC
```

如果没有vimrc文件，我们也可以自己新建一个。但是需要注意的是这个文件的存放位置。使用vim打开一个文件，然后通过`:version`命令可以得到你的vim查找的所有vimrc的位置。

对于Unix和Mac OS系统总是使用也推荐使用如下的文件位置：

~/.vimrc

对于MS-DOS和MS-Windows，可以使用下面的文件位置

$HOME/_vimrc

$VIM/_vimrc

vimrc文件可以包含任何冒号命令。最简单的就是设置选项(option)命令。所有的option命令列表和每个option的命令解释可以查找vim help文档（通过`:help`命令可以打开vim help文档）。

简单的vimrc的配置项如下：

```
"打开语法高亮
syntax enable

"在启动新行的时候使用与当前行一样的缩进
set autoindent
"Tab等于4个空格
set tabstop=4
"一次（自动）缩进等于4个空格
set shiftwidth=4

"在vim的右下角显示光标位置
set ruler

"在vim的右下角显示未完成的命令
set showcmd

"在输入部分查找模式时显示响应匹配点
set incsearch
```

[这里](https://github.com/tank0317/vimrc)是我的vimrc配置文件。

[电子版vim help文档](http://vimdoc.sourceforge.net/htmldoc/help.html)


