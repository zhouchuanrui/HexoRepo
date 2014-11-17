title: NERDCommenter
Status:
Tags: plugin
date: 2013-8-28 11:17:09
---

[TOC]

# 介绍

这是一个非常简洁清新的插件，简洁到不需要在\_vimrc里面做任何的配置，非常的不错。

插件的功能是实现代码的快速注释和反注释，说实话是相当基础的一个功能。我之前用过很长时间的notepad++，里面注释是Ctrl+q，多行注释就是先选多行然后再Ctrl+q，而且Ctrl+q其实是翻转注释，用起来风骚的紧。Vim里面没有自带的智能注释操作，才让我越发的想弄一个来。

<!--more-->

# 使用

NERDCommenter默认的键绑定是\<leader\>c再加一个后缀，c指的应该就是comment，还是比较合理的，也没跟\_vimrc里面的有冲突。然后会在menubar里面增加选项，把所有的命令和快捷键都列上，这个其实很好，对快速的熟悉这些命令很有帮助。其实越来越觉得menubar是个好东西，对像我这样的新手而言，是很多设置的快速入口。

这些命令都是在普通模式和可视模式下使用，普通模式下要实现多行的话就是要带个计数前缀，也是蛮直观的。

# 命令描述

命令是只有10来条，直接列个表：

命令名|键绑定|介绍
---|---|---
NERDComComment|,cc|招牌，实现注释
NERDComNestedComment|,cn|层级注释，我推测是可以解决选中内容中已有注释的问题，但是实际上的效果跟,cc是一样的，目测是鸡肋
NERDComToggleComment|,c<space>|注释翻转，取第一行的注释状态作翻转来实现全部行的注释\反注释
NERDComMinimalComment|,cm|最小注释，用的时候就是感觉单行也是用的多行注释包起来，目测鸡肋
NERDComInvertComment|,ci|每行都是单独翻转注释状态，就是notepad++里面的Ctrl+q了
NERDComSexyComment|,cs|sexy版注释，鸡肋
NERDComYankComment|,cy|注释并且复制注释的内容，感觉在这个时候会想不起来用到复制功能
NERDComEOLComment|,c$|望文生义，注释到行尾
NERDComAppendComment|,cA|望文生义，在行尾插入注释并进入插入模式，对我来说这个很是实用
NERDComAltDelim|,ca|切换注释的默认样式，就是统一用单行注释的样式或者多行注释的样式，当然只选行内一部分的代码的时候还是会用多行注释样式的，是让人困惑的指令，不碰为妙
NERDComAlignedComment|,cl和,cb|对齐注释，l为左对齐，b为两端对齐，鸡肋
NERDComUncommentLine|,cu|反注释

# 总结

虽然用这个也没多久，但是还是很看好NERDCommenter的，小巧干净好用，简直是插件之典范。

