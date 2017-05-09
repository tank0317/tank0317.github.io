----
layout: post
title: vim学习笔记（三）：多窗口编辑
date: 2017-05-08
tags: vim
comments: true
---

上一篇文章中，介绍了如何在不退出vim的情况下对多个文件进行操作，以及如何在多个文件之间来回跳转。**这次我们来学习在使用vim编辑文件过程中，如何进行多窗口编辑**

## 分屏

编辑文件时，使用如下命令：
```
:split
```
可以将屏幕切分成两个窗口，大致如下：

<pre>
	+----------------------------------+
	|/* file one.c */                  |
	|~                                 |
	|~                                 |
	|one.c=============================|
	|/* file one.c */                  |
	|~                                 |
	|one.c=============================|
	|                                  |
	+----------------------------------+
</pre>

此时，两个窗口打开的是同一个文件，你可以在两个窗口中浏览文件的不同部分。当需要在两个窗口之间来回跳转的时候，可以使用`CTRL-W w`快捷键。

然后你可以使用`:close`命令关闭窗口。如果你打开了多个窗口并且只想保留一个，那么可以使用`:only`命令。

## 分屏打开其他文件

上面通过分屏打开了两个窗口，但都是打开的同一个文件。其实我们更多的还会遇到希望多个窗口打开不同的文件。可以使用：
```
:split two.c
```
此时界面大致如下

<pre>
	+----------------------------------+
	|/* file one.c */                  |
	|~                                 |
	|~                                 |
	|one.c=============================|
	|/* file two.c */                  |
	|~                                 |
	|two.c=============================|
	|                                  |
	+----------------------------------+
</pre>

另外如果你只是想打开一个新文件的话，可以使用`:new`命令。

## 调整窗口高度

**增加窗口高度**可以使用`CTRL-W +`命令，由于"+"输入的时候需要同时按`SHIFT`键，所以我更推荐使用`:resize +N`来增加窗口高度，其中N为任意整数。

**减小窗口高度**可以使用`CTRL-W -`，类似的更推荐使用`:resize -N`来调整窗口高度。

## 垂直分屏

前面我们都是对屏幕进行水平的切分。其实我们还可以使用：
```
:vsplit two.c
```
命令对屏幕进行垂直切分。效果大致如下：

<pre>
	+--------------------------------------+
	|/* file two.c */   |/* file one.c */  ||||
	|~                  |~                 ||||
	|~                  |~                 ||||
	|~                  |~                 ||||
	|two.c===============one.c=============|
	|				                       |
	+--------------------------------------+
</pre>

类似的，你可以使用`:vnew`命令垂直分割窗口打开一个新的文件。

## 窗口间跳转

现在我们可以水平的分割窗口，也可以垂直的分割窗口，而且可以重复的做上述的操作，因此我们可以创建任意布局的水平垂直分布的窗口。此时，我们需要一些命令让我们能够在窗口之间任意跳转。这些命令有：

```
	CTRL-W h	move to the window on the left
	CTRL-W j	move to the window below
	CTRL-W k	move to the window above
	CTRL-W l	move to the window on the right

	CTRL-W t	move to the TOP window
	CTRL-W b	move to the BOTTOM window
```
## 移动窗口

前面我们能够完成任意布局的窗口分布，同时能够完成任意窗口之间的跳转，我们还可以做窗口的移动。

假如此时你的窗口分布大致如下：

<pre>
	+----------------------------------+
	|/* file two.c */                  |
	|~                                 |
	|~                                 |
	|two.c=============================|
	|/* file three.c */                |
	|~                                 |
	|~                                 |
	|three.c===========================|
	|/* file one.c */                  |
	|~                                 |
	|one.c=============================|
	|                                  |
	+----------------------------------+
</pre>

假如你现在想把最后一个窗口移动到最上方。那么可以将光标移动到这个窗口，然后使用命令（注意下面是大写的K）:

```vim
CRTL+W K
```

然后，会发现窗口`one.c`会移动到最上方。

如果你的窗口布局是这样的：

<pre>
	+-------------------------------------------+
	|/* two.c */  |/* three.c */  |/* one.c */  |
	|~            |~              |~            |
	|~            |~              |~            |
	|~            |~              |~            |
	|~            |~              |~            |
	|~            |~              |~            |
	|two.c=========three.c=========one.c========|
	|                                           |
	+-------------------------------------------+
</pre>

那么在`one.c`窗口使用`CTRL-W K`命令后的布局如下：

<pre>
	+-------------------------------------------+
	|/* three.c */                              |
	|~                                          |
	|~                                          |
	|three.c====================================|
	|/* two.c */	       |/* one.c */         ||||
	|~                     |~                   ||||
	|two.c==================one.c===============|
	|                                           |
	+-------------------------------------------+
</pre>

同样，不只是可以将窗口移动到上方，类似的还有：

```vim
	CTRL-W H	move window to the far left
	CTRL-W J	move window to the bottom
	CTRL-W L	move window to the far right
```

## 针对所有窗口的命令

当我们打开了很多个窗口，然后想要退出的时候，我们可以对每个窗口使用`:close`命令。但是，当窗口很多的时候，这种方式效率太低 了。我们还可以使用更高效的命令如：

```vim
:qall quit all windows
```
但是，如果有窗口没有保存更改的话，vim是不允许我们就这样随意退出的。所以我们就有了如下命令：

```vim
:wall     write all windows
:wqall    write and quit all windows
:qall!    quit all windows discard all changes
```
更多有关window操作的命令可以参考[这里](http://vimdoc.sourceforge.net/htmldoc/windows.html)

## 标签页

虽然我们可以很方便的通过分屏编辑多个文件，但是屏幕空间总会有不够用的时候。这时我们可以通过打开多个标签页来编辑文件。

```vim
:tabe[dit] anotherfile.c
```

此时vim会为`anotherfile.c`打开一个新的标签页。

如果我们想要让新的标签页打开同一个文件，可以使用

```
:tab split
```

关闭标签页可以使用：

```
:tabc[lose]              Close a Tab page
:tabc[lose]!             Close a Tab page discard changes
:tabc[lose] {count}   Close Tab page {count}. The first tab page has number one.
:tabc[lose]! {count}
```
在标签页之间跳转的命令有：

```vim
gt 				 Go to the next Tab page
{count}gt	     Go to tab page {count}.  The first tab page has number one.
gT               Go to the previous Tab page
:tabn[next]      Go to the next Tab page
:tabp[revious]   Go to the previous Tab page
:tabN[ext]       Go to the previous Tab page
:tabfir[st]      Go to the first Tab page
:tabl[ast]       Go to the last Tab page
```

更过有关Tab page的命令，参考[这里](http://vimdoc.sourceforge.net/htmldoc/tabpage.html#tab-page)

## Reference
<http://vimdoc.sourceforge.net/htmldoc/usr_08.html>

