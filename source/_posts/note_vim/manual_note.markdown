title: Vim笔记【整理】
Status: Public
Tags: 
date: 2014-1-14 12:53:09
---

[TOC]

# 模式

Vim的模式跟Emacs的模式是完全不同的概念，指的是对每个打开的窗口都有4个模式：

* 普通：初始模式，可进行浏览，复制，删除这些操作；
* 编辑：用来做文本输入；
* 可视：用来进行快速的移动，选择文本；
* 命令：退出、保存等操作；

这个应该是Vim最独特的一点了，正因为有编辑模式之外的模式，所以字母键就可以用来作快捷键，快捷键很是灵活简洁啊。

<!--more-->

普通模式为主模式，目测Vim里面基本就是普通模式跟编辑模式来回切换，所以esc键会用的非常多，使用频率估计堪比Emacs的ctrl。

移动：

* hjkl分别为光标的左下上右移动；
* C-b，C-f分别为上下翻页；
* 上面的都太弱了，看看词的移动；

```
	   ge      b	  w				e
	   <-     <-	 --->			       --->
This is-a line, with special/separated/words (and some more). ~
   <----- <-----	 -------------------->	       ----->
	 gE      B			 W			 E
```

E和W表示连续字符的移动，中文这个也当连续字符了。

```
		  ^
	     <------------
	.....This is a line with example text ~
	<-----------------   --------------->
		0		   $
	To err is human.  To really foul up you need a computer. ~
	---------->--------------->
	    fh		 fy
	To err is human.  To really foul up you need a computer. ~
		  --------------------->
			   3fl
	To err is human.  To really foul up you need a computer. ~
		  <---------------------
			    Fh
```

f表示find，t表示to，大写的表示反向。这四个命令可以用';'和','重复以及反向重复。只能查找一个单词比较可惜，不过亮点是对中文有效，也就是可以查找输入法打出来的字，这个反而是在中文里更实用了。

命令前可加数字前缀，表示重复次数，这里就已经可以形成很多组合了，已经体现出智能来了。

注：光标移动方面的命令大全可以参考Vim手册的|Q\_lr|和|Q\_ud|。

# 移动和滚屏

光标的移动是最基本的操作，还是应该多记记，强化一下：

* H,M,L--head,middle,last分别可以移动到当前视图的头、中、尾行；
* xxxG为移动到xxx行，G是大写。另，G为跳到末行，gg为到首行；

下面是滚屏，滚屏是移动屏幕不是移动光标：

* C-E和C-Y为上滚和下滚单行——不知道为啥下滚滚不了，大写小写都不行；
* C-U和C-D为滚半屏，感觉比C-F和C-B要实用；
* zz为当前光标所在行滚到中间，这个应该是最常用的，相关的还有zt，zb。

注：滚屏方面的命令大全可以参考Vim手册的|Q\_sc|。

# 寄存器

一共有三种用途：

* 位置标记
	* 设置标记是"m[x]"，[x]可以是[a-zA-Z]，然后跳转到这个位置是" '[x] "，[x]的定义相同。值得注意的是，[a-z]只能是文件内部的跳转，[A-Z]可以实现文件间的跳转，这也太犀利了。。。
	* ":marks"可以查看有哪些位置标记
* 键盘宏
	* 记录宏的操作是"q[x]"-->"键盘操作"-->'q'，这样就把一组操作记在了寄存器[x]里面；
	* 运行宏的操作是"@[x]"，一次运行之后再按"@@"是运行上一个宏[x]，然后这个可以带数字前缀，批量修改文本的利器啊；
	* 需要注意光标位置的定位问题，要多用'/'查找定位，还有句首句尾定位；
	* 'C-a'和'C-x'可以分别对数值进行加1减1操作，这个好有用啊，相当与一个参数了；
	* 又一个大招，修改宏的内容：
	
```
Suppose you have recorded a few commands in register n.  When you execute
this with "@n" you notice you did something wrong.  You could try recording
again, but perhaps you will make another mistake.  Instead, use this trick:
	G			Go to the end of the file.
	o<Esc>			Create an empty line.
	"np			Put the text from the n register.  You now see
				the commands you typed as text in the file.
	{edits}			Change the commands that were wrong.  This is
				just like editing text.
	0			Go to the start of the line.
	"ny$			Yank the corrected commands into the n
				register.
	dd			Delete the scratch line.
Now you can execute the corrected commands with "@n".  (If your recorded
commands include line breaks, adjust the last two items in the example to
include all the lines.)
```

这里可以看出宏寄存器和剪切板寄存器是共享的，而标记寄存器是独立的。可能当时设计的时候本意是要都设计成独立的，但是这一条的这个功能刚好可以巧妙的实现了对键盘宏内容的修改，无心插柳啊。以上纯属猜测。

* 剪切板缓存
	* 剪切\复制操作为' "[R] '+"剪切\复制操作"，粘贴操作为' "[R] '+"粘贴操作"；
		* 大写字母和小写字母指的是同一个寄存器，但是连续使用小写字母表示复制，连续使用大写字母表示内容追加。
	* 要使用系统剪切板的话就用" "\* "+"剪切\复制\粘贴操作"，或者是" "+ "+"剪切\复制\粘贴操作"。'\*'是表示当前选择区，'+'表示“真”剪切板

Vim的操作咋看之下挺乱的，其实稍微深入了解一下的话就会发现其实还是挺有规律的。

# :behave

`:behave`原来是用来定制鼠标行为的，困扰了好久。。。

>Standards are wonderful.  In Microsoft Windows, you can use the mouse to
select text in a standard manner.  The X Window system also has a standard
system for using the mouse.  Unfortunately, these two standards are not the
same.
   Fortunately, you can customize Vim.  You can make the behavior of the mouse
work like an X Window system mouse or a Microsoft Windows mouse.  The following
command makes the mouse behave like an X Window mouse: >
	:behave xterm
The following command makes the mouse work like a Microsoft Windows mouse: >
	:behave mswin

# substitute和global

substitute命令可以实现连续的替换，其实就是替换功能，不过比较灵活比较智能。命令的格式为：

```
:[range]substitute/from/to/[flags]
```

这是一个命令模式的操作，substitute可以用's'缩写替换，from字符可以用正则表达式，间隔符'/'还可以用'+'或'='替代。

[range]表示范围，

[range]|解释
---|---
缺省|只作用与当前行
%|作用于所有行
n,m|作用与n至m行，n和m可以扩展哦：用'.'表示当前行，用'$'表示最末行，所以'%'相当于"1,$"；n和m都可以是正则表达式； 可以是标记'm1,'m2； 可以用+-指定行偏移以扩展或缩小相对范围
n|作于与第n行

[flags]表示模式：

[flags]|解释
---|---
缺省|对应一行的第一个匹配点
g|对应匹配行的所有匹配点
p|执行时打印最后一个被修改的行
c|确认模式，就是query-replace，每次匹配会有多个选项选择

global命令的格式为：

```
:[range]global(g)/(+|=){patten}/(+|=){command}
```

顾名思义，就是找到pattern的匹配位置，然后执行命令command

# 高级跳转

```
You are halfway editing a file and it's time to leave for holidays.  You exit
Vim and go enjoy yourselves, forgetting all about your work.  After a couple
of weeks you start Vim, and type:
>'0
And you are right back where you left Vim.  So you can get on with your work.
   Vim creates a mark each time you exit Vim.  The last one is '0.  The
position that '0 pointed to is made '1.  And '1 is made to '2, and so forth.
Mark '9 is lost.
   The |:marks| command is useful to find out where '0 to '9 will take you.
```

这个功能真是实用，新发现。

单引号跳转果然是高阶跳转，小小的总结一下：

符号|功能
---|---
'[ or ']|跳转到前一次修改处的第一或最后一个字符
'< or '>|跳转到前一次选择的可视区的第一行（字符）或最后一行（字符）
''|跳转到前一次跳转的位置
'"|跳转到离开该buffer时的光标位置
'^|跳转到前一次insert mode退出时的光标位置
'.|跳转到最后一次修改的位置
'( or ')|跳到当前句的起始或结束
'{ or '}|跳转到当前段落的其实或结束
'[0-9]|离线位置栈，0为栈头
'[a-z] or '[A-Z]|跳转到文件位置标记，其中[a-z]为文件内标记，[A-Z]为文件间标记


# 插入模式下的快捷键

插入模式下的快捷键没什出奇之处，比如光标移动的指令基本跟Windows默认的没区别：

```
<C-Home>	to start of the file
<PageUp>	a whole screenful up
<Home>		to start of line
<S-Left>	one word left
<C-Left>	one word left
<S-Right>	one word right
<C-Right>	one word right
<End>		to end of the line
<PageDown>	a whole screenful down
<C-End>		to end of the file
```

删除倒是有点不一样的地方，`<C-w>`是删除到词首，`<C-u>`是删除到句首。

补全比较犀利，`<C-n>`和`<C-p>`为向后及向前搜索补全。还可以补全特定的文本：

```
CTRL-X CTRL-D	complete defined identifiers
CTRL-X CTRL-F	complete file names
CTRL-X CTRL-I	complete identifiers
CTRL-X CTRL-K	complete identifiers from dictionary
CTRL-X CTRL-L	complete whole lines
CTRL-X CTRL-N	next completion
CTRL-X CTRL-O	omni completion
CTRL-X CTRL-P	previous completion
CTRL-X CTRL-S	spelling suggestions
CTRL-X CTRL-T	complete identifiers from thesaurus
CTRL-X CTRL-U	complete with 'completefunc'
CTRL-X CTRL-V	complete like in : command line
CTRL-X CTRL-]	complete tags
CTRL-X s	spelling suggestions
```

另外，`<C-x C-o>`是万能补全，用于补全源码。自带的补全就已经很厉害了。

`<C-k>`可以写入特殊按键名，如`<F4>``<BS>``<F21>``<F21>``<Insert>``<Del>``<Home>``<PageUp>``<PageDown>``<End>``<Up>``<Down>``<Left>``<Right>``<F5>`这些。。。

`<C-a>`为重复上一次插入模式下输入的内容。`<C-y>`为输入光标上方的字符，这个指令比较清新啊；`<C-e>`插入光标下方的字符。`<C-r>`[r]用于插入寄存器中的内容。

风骚的缩写替换，需要设置：

```
:iabbrev short-text long-text
```

然后插入模式打个short-text然后按空格就会被替换成long-text。举两个比较好的例子：

这个东西重启是不保存的，要常用的话要写到vimrc文件里面，或者专门写个.vim文件:so之。

# 键映射指导


```
If you are going to map something, you will need to choose which key(s) to use
for the {lhs}.  You will have to avoid keys that are used for Vim commands,
otherwise you would not be able to use those commands anymore.  Here are a few
suggestions:
- Function keys <F2>, <F3>, etc..  Also the shifted function keys <S-F1>,
  <S-F2>, etc.  Note that <F1> is already used for the help command.
- Meta-keys (with the ALT key pressed).  Depending on your keyboard accented
  characters may be used as well. |:map-alt-keys|
- Use the '_' or ',' character and then any other character.  The "_" and ","
  commands do exist in Vim (see |_| and |,|), but you probably never use them.
- Use a key that is a synonym for another command.  For example: CTRL-P and
  CTRL-N.  Use an extra character to allow more mappings.
- The key defined by <Leader> and one or more other keys.  This is especially
  useful in scripts. |mapleader|

See the file "index" for keys that are not used and thus can be mapped without
losing any builtin function.  You can also use ":help {key}^D" to find out if
a key is used for some command.  ({key} is the specific key you want to find
out about, ^D is CTRL-D).
``` 

键映射种类：

- :map普通、可视及操作符模式
- :vmap可视模式
- :nmap普通模式
- :omap操作符模式
- :map!插入和命令模式
- :imap插入模式
- :cmap命令行模式

注：map前面加"nore"表示无嵌套映射，即对映射内容中已定义另外的映射的命令不进行映射展开；map前加"un"表示取消映射
     

# `<c-a>`和`<c-x>`

 vim的普通模式下，`<c-a>`和`<c-x>`可以分别对数值进行加1和减1，而且可以带数字前缀的。

这个非常犀利，可以作为键盘宏的参数用来生成很多固定格式文本序列。像是：

	  data_reg0 <= ***;
	  data_reg1 <= ***;
	  data_reg2 <= ***;
	  data_reg3 <= ***;
	  data_reg4 <= ***;
	  data_reg5 <= ***;
	  data_reg6 <= ***;
	  data_reg7 <= ***;
	  data_reg8 <= ***;
	  data_reg9 <= ***;

这种代码序列简直就是so easy。

但是我在用的时候有出了一个问题，就是数值是00这个格式的时候，加到07之后再按`<c-a>`会变成10而不是08，是把这个格式当作是8进制的了。

查了下文档发现这个是nrformats这个参数的配置问题——nrformats这个参数决定了`<c-a>`和`<c-x>`操作是按那个数制来做加减的。这个参数里面默认带的是"otcal,hex"，就是8进制和16进制的，会把0开头的认为是8进制，0x开头的认为是16进制。其实16进制的还是蛮有用的，8进制的就基本用不到了。

nrformats这个参数里面还可以添加的一个是"alpha"，会对单个字母做加减操作，这个也非常有用啊。

所以用的时候可以这样配置：

	  set nrformats-=otcal
	  set nrformats+=alpha

# ctags

vim的ctags是默认支持verilog的，屌爆了。。。

taglist里面使用的ctags的路径不能有空格，不然会报错，然后我就把ctags.exe移到e盘根目录，然后设置taglist的命令目录：

	let Tlist_Ctags_Cmd='E:\ctags.exe'

这样taglist就能正常使用了。


# 查看vim选项参数

要查看vim的选项参数的值，其格式为：

	set <option>?

就是要在最后面加一个问号来参看这个选项的值，这个还是蛮有用的。

# 缓存寄存器

缓存寄存器其实还是比较复杂的，其实除了a-z之外都是自动生成的，然后作用又是各不相同。

文档里面直接把这些寄存器分成了9类：

* 字母寄存器    [a-zA-Z]
字母寄存器的用法最直接，是用"[a-zA-Z]{motion}这种格式来，是用来存取文本的。可以用a-z和A-Z，但是这里指的其实不是52个寄存器而是26个寄存器。用小写寄存器每次都是直接用新文本覆盖，而用大写寄存器是每次用新文本追加到寄存器里面，这个比较有用，可以用来收集不同地方的文本然后集合到一起。
之前都没注意的很重要的命令——“:registers/:display”，这个可以显示出非空寄存器里面的内容。非空其实是针对字母寄存器而言的，因为其他寄存器都是存了东西的。
用用这个命令其实就能发现小写寄存器和大写寄存器是同一个寄存器。而且，宏寄存器跟文本寄存器是同一个寄存器。
* 数字寄存器    [0-9]
就是0-9，其实这里0跟1-9还是有点不一样的。0寄存器是保存最后一次复制(yank)操作的内容，1-9寄存器则是删除或修改操作的缓存队列，1为入口，9为出口。
另外，在指定了字母寄存器的操作是不会改变数字寄存器的内容的，而且删除操作的范围必须是要一行以上。
vim里面的复制还有删除操作是非常频繁的，所以要想用好数字寄存器的话要经常使用查看寄存器的命令。
* 无名寄存器    "
双引号寄存器，也是默认寄存器。每次删除和复制操作都会把内容覆盖到这个寄存器里面，而且这个是不管你有没有用字母寄存器指定你的这些操作。
* 短删除寄存器    -
缓存一行以内删除操作的文本，而且是没用字母寄存器指定的删除操作。
* 只读寄存器    :.%#
[.]用于保存最后一次插入模式下插入的文本，但是不能在命令行里面用CTRL-R导出；[%]保存当前文件的文件名；[#]保存轮换(alternate)文件名；[:]保存最近用过的命令，可以用"@:"再次使用。
* 表达式寄存器    =
文档有说的很不清楚，不管了。
* 选择和拖放寄存器    \*+~
[\*]和[+]用于剪切板操作，[~]用于拖放操作。
* 黑洞寄存器    _
用于彻底销毁文本，只用于删除操作，类似于shift+del。
* 搜索寄存器 \\
用于存放最后一次搜索命令里面的文本。

