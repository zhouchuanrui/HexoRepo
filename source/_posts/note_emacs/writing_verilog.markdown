title: 我的Verilog HDL代码快速编辑环境
Status: Public
Tags: 
Date: 2015-1-9 23:18:52
---

[TOC]

# 概述

由于工种的特殊性，Verilog可以说是一门朴实到简陋的设计语言，由于语法功能都非常的底层，因此在做规模比较大的设计的时候，Verilog代码必定是密密麻麻、相当繁琐的。

于是从开始做FPGA开始，我就在不断的探索verilog的编辑方案。使用的编辑器完成了从“IDE自带编辑器--》Notepad++--》Emacs--》Vim”的变迁。

其实我这么做的还有一个很大的推力，就是IDE自带的编辑器实在太差。我用的是Quartus II，里面的代码编辑器连换行缩进都会出问题，而且语法高亮也太过简陋，还比不上Notepad++这种通用编辑器，简直不像是一个Verilog的IDE，跟VS一比更是不忍直视。不过也要感谢Quartus，让接触学习了Emacs和Vim，并因此顺带学到了很多相关的技巧和思维。

目前，我使用的这套Verilog编辑环境是以gVim作为主编辑器的，此外还用到了Emacs的verilog-mode和snipmate插件。

<!--more-->

# verilog-mode

虽然现在使用的主编辑器是gVim，但是可以说这套编辑环境里面的功能核心还是verilog-mode。verilog-mode是Emacs下的一个major-mode，作为一个verilog代码插件，能够被Emacs默认收录，可见verilog-mode的功能强大。

verilog-mode的精华是AUTO-COMMAND宏，在代码里面写下`/*auto<command>*/`格式的AUTO-COMMAND“种子”，之后按AUTO-COMMAND展开快捷键`C-c C-a`，verilog-mode就会为你展开相应的自动化宏。这些宏一般是用来完成信号的声明，比如用于自动完成端口列表的`/*AUTOARG*/`：

```verilog
module <name_of_module>
(/*AUTOARG*/);
```

按这个格式写，执行`C-c C-a`之后，verilog-mode会读取模块之中的用`input`和`output`定义的信号，然后添加到模块端口列表的括号里。还有用于自动产生例化模块信号的/*AUTOINST*/：

```verilog
<name_of_module> <inst_of_module>
(/*AUTOINST*/):
```

执行`C-c C-a`之后，verilog-mode会根据例化模块的端口列表，自动添加`.<sig> (<sig>)`格式的同名例化信号列表，然后你只需要修改括号里面的信号名就可以了，非常的方便。如果你的模块是带全局参数的，可以使用`/*AUTOINSTPARAM*/`，然后参数也自动展开了。

我最常用的基本就是这三个宏，此外还有很多功能不同的宏啦：`/*AUTOSENSE*/`可以把组合逻辑的`always`块里的敏感信号都添加到敏感列表里；`/*AUTORESET*/`可以自动展开信号的复位（也就是置零），可以放在带复位信号的`always`块里配合使用；`/*AUTOWIRE*/`和`/*AUTOREG*/`可以用来检查例化后没有定义的信号。

除了这些之外，verilog-mode还带有格式化文件注释头、交互式状态机和语法元素snippet等功能，可以说是非常的完备。总之，最初接触使用的时候简直是让我叹为观止——一个编辑器竟然可以智能到这种程度。Vim下有仿verilog-mode实现AUTO-COMMAND宏的插件，甚至有一个韩国人写的插件是直接调用系统里的Emacs然后执行verilog-mode下的auto-complete。说起来Emacs的社区活跃程度跟Vim比起来差距应该是越来越大了，但是Emacs的插件质量还是有点让Vim望尘莫及的意思啊。这些verilog-mode插件用起来我估摸着就跟Emacs的viper-mode是一个意思，于是我就还是继续用Emacs来完成`C-c C-a`,还是留着Emacs。

PS：在Windows下，把Emacs的安装路径（比如"/E/Emacs24/bin"）添加在环境变量path里面，就可以在gVim里面用`:!emacs`快速调用Emacs了。一个更快的操作是：

	:!emacs --batch <filename.v> -f verilog-batch-auto

# IEEE 1364-2001 

IEEE 1364-2001就是所谓的verilog-2001了，其实verilog的官方标准不只是有1995跟2001两个而已，但是2001提出了很多语法特性上的改进，因此厂家就取出了这两个最有代表性的标准来作为verilog-1995跟verilog-2001。

为什么要把这个语法标准提出来呢？也正是因为2001标准提出的语法改进对整个verilog的面貌有了很大的提升，严格的verilog-2001代码实在是比verilog-1995简洁许多，这也是Verilog代码快速编辑的一个因素。比如，在verilog-1995里面完整的端口信号定义是很复杂的：

```verilog
module mod_emp
(clk, rst, d, q, ...);
	input clk, rst；
	input d;
	output q;
	...
	wire clk, rst;
	wire d;
	reg q;

	...
endmodule
```

这些，到了verilog-2001里面就大变样了：

```verilog
module mod_emp
(	
	input wire clk, rst,
	input wire d,
	output reg q,
	...
);
	...
endmodule
```

信号定义直接合并在module端口列表里了，回头再看看verilog-1995的这种设定是不是觉得有点二百五呢？（这么一改进甚至连verilog-mode里的`/*AUTOARG*/`宏都可以省了，不过用verilog-2001写出来的module在verilog-mode里用`/*AUTOINST*/`宏自动例化的时候是完全没问题的，看来verilog-mode开发人员果然是与时俱进的，这点国内的那些Verilog教材作者就要好好反省了。）再比如，verilog-2001里面写组合逻辑`always`块的时候，可以直接写成：

```verilog
always @(*)
	...
```

这样也替代了verilog-mode里的`/*AUTOSENSE*/`宏。

其他的内容这里也就不再赘述了，其实我也就对这两点比较熟悉，但是就这两点而言就已经能让你的verilog代码变的清爽很多了，因此verilog-2001还是肯定是要好好研读的。

# Snippet

Snippet代码块,是指一些代码格式化结构，可以快速的写出if-else、while、函数定义等结构，是写代码——不只是verilog——的利器。这个其实Emacs里面的verilog-mode已经带了不少了。我在gVim里面使用的是snipmate这个插件，具体的介绍看[这一篇](http://zhouchuanrui.github.io/2013/08/28/note_vim/snipmate/)吧。

snipmate的一个好处是可以很方便的按照语法规则自定义Snippet，每个人都会有自己的编码习惯，因此自己写出来的Snippet自然才是最适合自己的。我写的[跟verilog-mode匹配的snippet](https://github.com/zhouchuanrui/mySnippet/blob/master/verilog.snippets)，里面自然就是带了AUTO-COMMAND的，然后还有文件头、格式化状态机等从verilog-mode里借鉴的一些结构。

平心而论，Snippet用的顺手了，可以让你写代码的速度大大提升而且降低低级错误出现的概率，这样就能把注意力更多的放在思路的实现上了。

# 其他

到这里为止，其实已经可以算是完结了。但是还是有点奇技淫巧要分享下，说不定能带来额外的启发。

如果你电脑里面有Python，拿Windows下的Python 3.x来说好了，Vim里面有一个很强大的隐藏功能。在你的Vim里面输入下面的Python代码：

```python
for i in range(10):
	print ("reg" + str(i) + " = 8'hff")
```

然后用可视行模式（大V）选中这两行，然后进入命令模式，输入:

	'<,'>!python.exe 

然后上面的代码就执行了，输出是这样的：

```verilog
reg0 = 8'hff
reg1 = 8'hff
reg2 = 8'hff
reg3 = 8'hff
reg4 = 8'hff
reg5 = 8'hff
reg6 = 8'hff
reg7 = 8'hff
reg8 = 8'hff
reg9 = 8'hff
```

注意这个代码不是.py文件而是.v文件，实际上在任何文本文件里面都是可以执行的。其实Perl、Ruby、Lua这些**可以在shell/CMD里面执行的解释器**还有shell/CMD命令也可以。比如文件里面的一行写上`dir`这个CMD下的命令，然后可视模式选中，然后用

	'<,'>!cmd

执行，就输出了：

```
Microsoft Windows XP [版本 5.1.2600]
(C) 版权所有 1985-2001 Microsoft Corp.

F:\hexo\source\_posts\note_emacs>dir
 驱动器 F 中的卷是 Files
 卷的序列号是 5E22-CA2E

 F:\hexo\source\_posts\note_emacs 的目录

2015-01-09  23:18    <DIR>          .
2015-01-09  23:18    <DIR>          ..
2014-10-12  18:02             3,312 keyswitch.markdown
2014-10-12  18:02             4,340 many_languages.markdown
2014-10-12  18:03             4,559 regex.markdown
2015-01-25  23:24             7,577 writing_verilog.markdown
               4 个文件         19,788 字节
               2 个目录 31,343,042,560 可用字节

F:\hexo\source\_posts\note_emacs>
```

这个功能真是屌，在这里抛砖引玉一下，希望能激发到大家的灵感。

