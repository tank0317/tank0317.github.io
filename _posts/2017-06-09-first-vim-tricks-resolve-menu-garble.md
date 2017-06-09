---
layout: post
title: vimС���ɣ�һ�������gvim�˵��������Լ����ز˵���
date: 2017-06-09
tags: vim
comments: true
---

�����¼򵥽����������gvim������˵��������������ͬʱ�������ϲ���˵��������������������ȣ�����������ء���Щ��vim�ж��ǿ���������õġ�

## �˵�����

Ҫ˵�˵����룬��һ����gvim����Ϊvim������û�в˵��ġ���������϶��Ǳ��뷽ʽ�����⣬����������Ҫ����ֻ���޸ı��뷽ʽ��

���Ͻ��
```
"--------Gvim���Ĳ˵�����������
" �����ļ������ʽ
set encoding=utf-8
set fileencodings=utf-8,chinese,latin-1,gbk,gb18030,gk2312
if has("win32")
 set fileencoding=chinese
else
 set fileencoding=utf-8
endif

"����˵����� ɾ���˵�����������Ӳ˵���vim�ᰴ��֮ǰ�趨�ı����ʽ�����˵���
source $VIMRUNTIME/delmenu.vim
source $VIMRUNTIME/menu.vim

"���consle��ʾ��Ϣ�������
language messages zh_CN.utf-8

"--------������Gvim���Ĳ˵�����������------------
```

���ϸ�����[��ƪ����](http://blog.csdn.net/gatieme/article/details/55047156)

�������Ǵ��½���������ļ���option�ĺ���

| option | ����|
|--- | ---|
|encoding | Sets the character encoding used inside Vim.  It applies to text in the buffers, registers, Strings in expressions, text stored in the viminfo file|
| fileencoding | Sets the character encoding for the file of this buffer. |
| fileencodings | This is a list of character encodings considered when starting to edit an existing file.  When a file is read, Vim tries to use the first mentioned character encoding.  If an error is detected, the next one in the list is tried.  When an encoding is found that works, 'fileencoding' is set to it.  If all fail, 'fileencoding' is set to an empty string, which means the value of 'encoding' is used.|

�´���������ɡ�����

## ���ز˵���

Ȼ�󣬻���һ�������ǣ�����Ҳ�ϲ��gVim�Ĳ˵�����������������������أ�

```
"����gvim���ز˵�������������������
:set guioptions-=m  "remove menu bar
:set guioptions-=T  "remove toolbar
:set guioptions-=r  "remove right-hand scroll bar
:set guioptions-=L  "remove left-hand scroll bar
```

�����й����ÿ��Բο�`:h guioptions`��

## Reference

<http://blog.csdn.net/gatieme/article/details/55047156>
<http://vimdoc.sourceforge.net/htmldoc/help.html>



