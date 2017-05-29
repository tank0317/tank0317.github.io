---
layout: post
title: vim学习笔记（四）：插件
date: 2017-05-28
tags: vim, vim-plugin
comments: true
---

我们可以通过给vim安装不同的插件来丰富vim的功能。比如常用的插件有**代码补全插件**[YouCompleteMe](https://github.com/Valloric/YouCompleteMe)、**文件浏览插件**[NERDTree](https://github.com/scrooloose/nerdtree)、**主题插件**[Molokai](https://github.com/tomasr/molokai)以及**管理插件的插件**[Vundle](https://github.com/VundleVim/Vundle.vim/wiki/Vundle-for-Windows)。

今天我就详细的介绍下常用插件安装的一个详细过程。因为现在的电脑是公司的，最后难免要换电脑，到时候还要重新配置一遍，所以今天先把这个过程记录一下。又公司只给配了Lenovo，所以这里主要介绍windows下vim插件的安装过程。linux和mac下的大致过程是一样，也可以参考。

首先明确下windows下vim配置文件的存放位置。通过`:version`命令可以看到

<figure>
<img src="http://om0jxp12h.bkt.clouddn.com/vimversion.PNG" alt="vim veriosn">
  <figcaption>Fig.1 - windows下vimrc存放位置</figcaption>
</figure>

windows下vimrc存放位置为**`$HOME\_vimrc`**，其他插件等配置文件可以放在**`$HOME\vimfiles`**文件夹下。

## vundle

 首先安装一个管理插件的插件vundle，有了它之后，我们安装插件的过程会非常方便。（其实因为我是一开始就学着用vundle安装插件，所以不用vundle怎么安装插件我并不是很清楚。）

 <https://github.com/VundleVim/Vundle.vim/wiki/Vundle-for-Windows>

上面是vundle给的windows下的安装教程。首先我们需要在windows下先安装**git**（[我是链接](https://git-for-windows.github.io/)），因为vim的插件几乎都是通过git下载。*然后上面的教程里说还需要单独安装curl，不过我在安装好的gitbash里通过`curl --version`验证发现我的gitbash已经有了curl所以我跳过了curl的安装。*

然后在gitbash里通过git下载vundle到vimfiles文件夹，

```shell
git clone https://github.com/VundleVim/Vundle.vim.git ~/vimfiles/bundle/Vundle.vim
```

之后需要按照[quick start](https://github.com/VundleVim/Vundle.vim#quick-start)中的配置信息保存到vimrc文件。需要注意的是，windows环境下还需要做如下修改：

<figure>
<img src="http://om0jxp12h.bkt.clouddn.com/vundle%E6%9B%B4%E6%94%B9vimrc.PNG" alt="vundle配置信息修改">
  <figcaption>Fig.2 - vundle配置信息修改</figcaption>
</figure>

至此vundle的安装就完成了。关于vundle如何使用详细信息可以通过`:help vundle`查看。通常我们会用到的命令有：

<figure>
<img src="http://om0jxp12h.bkt.clouddn.com/vundle-helpinfo.PNG" alt="vundle常用命令">
  <figcaption>Fig.3 - vundle常用命令</figcaption>
</figure>

## YouCompleteMe

接下来我们在说一说，补全插件的安装。有了它，可以让vim媲美IDE的补全功能。另外由于我主要用作前端开发，因此这里主要介绍javascripe的补全功能。

<https://github.com/Valloric/YouCompleteMe#windows>

上面的链接是YouCompleteMe在windows下的安装教程。首先需要确定一点的是，YCM（YouCompleteMe）需要python的支持，因此需要你的vim的版本在至少是 7.4.1578并且支持python。所以我们要先确认自己的vim是否支持python。可以通过`:version`命令查看vim特性中是否支持python。

<figure>
<img src="http://om0jxp12h.bkt.clouddn.com/vimPython.PNG" alt="vim features">
  <figcaption>Fig.3 - 查看vim是否支持python</figcaption>
</figure>

上图中标黄的部分`python/dyn`和`python3/dyn`前面的加号**+**表示支持python，如果是减号**-**则表示不支持。如果不支持，可以在[这里](https://bintray.com/micbou/generic/vim)下载vim。

然后我们需要先安装如下软件：

* [**Python 2 or Python 3**](https://www.python.org/downloads/windows/). Be sure to pick the version corresponding to your Vim architecture. It is Windows x86 for a 32-bit Vim and Windows x86-64 for a 64-bit Vim. We recommend installing Python 3.
* [**CMake**](https://cmake.org/download/). Add CMake executable to the PATH environment variable.
* [**Visual Studio**](https://www.visualstudio.com/zh-hans/downloads/?rr=https%3A%2F%2Fgithub.com%2FValloric%2FYouCompleteMe). Download the community edition. During setup, select Desktop development with C++ in Workloads.
* [**7-zip**](http://www.7-zip.org/download.html). Required to build YCM with semantic support for C-family languages.

然后利用vundle安装YCM，即通过在vimrc文件的vundle配置信息中添加`Plugin 'Valloric/YouCompleteMe'`，然后`:PluginInstall`安装。此时你的`vimfiles\bundle`文件夹下会看到下载完成的`YouCompleteMe`文件夹。此时通过DOS命令行，执行

```shell
cd %USERPROFILE%/vimfiles/bundle/YouCompleteMe
python install.py --tern-completer
```
对YCM进行编译，其中上面的--tern-competer表示javascript的补全引擎。
如果你想让vim支持其他更过语言的补全功能，可以查看[这里](https://github.com/Valloric/YouCompleteMe#windows)的详细安装教程。

### [.tern-project](https://github.com/Valloric/YouCompleteMe#javascript-semantic-completion)
最后如果想要尝试vim的javascript补全功能，还需要在一个.tern-project文件。这是因为javascript的补全是基于tern，这个补全引擎启动的时候需要一个名为.tern-project存在当前工作目录或者当前工作目录的祖先文件夹。YCM在第一次编辑js文件的时候回启动tern server，所以这时vim的工作目录必须是.tern-project目录的子目录。

关于.tern-project的设置可以看[这里](http://ternjs.net/doc/manual.html#configuration)，也可以看[这里的简单介绍](http://ecomfe.github.io/blog/vim-javascript-completion/#tern_for_vim)

另外，YCM并不支持多个tern server，因此当切换到另一个javascript工程的时候，需要做如下事情：

* start a new instance of Vim from the new project's directory
* change Vim's working directory (:cd /path/to/new/project) and restart the ycmd server (:YcmRestartServer)
* change Vim's working directory (:cd /path/to/new/project), open a JavaScript file (or set filetype to JavaScript) and restart the Tern server using YCM completer subcommand :YcmCompleter RestartServer.

最后来一张效果图，其中左边是.tern-project文件，我只添加了jquery库的补全：

<figure>
<img src="http://om0jxp12h.bkt.clouddn.com/showcompletion.PNG" alt="补全效果图">
  <figcaption>Fig.4 - 补全效果图</figcaption>
</figure>



## 其他

其他插件如NERDTree和Molokai的安装相对简单，可以直接在_vimrc配置文件中Vundle的配置信息中添加`Plugin 'scrooloose/nerdtree'`以及 `Plugin 'tomasr/molokai'`然后使用`：PluginInstall`命令即可。其他信息可以查看相应插件的github主页，会有详细介绍。

## reference

<http://ecomfe.github.io/blog/vim-javascript-completion/>

<https://github.com/VundleVim/Vundle.vim>

<https://github.com/Valloric/YouCompleteMe#javascript-semantic-completion>

<https://github.com/scrooloose/nerdtree>

<https://github.com/tomasr/molokai>
