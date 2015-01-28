title: $readmem是可综合的
Status: Public
Tags: 
date: 2014-10-27 21:46:05
---

[TOC]

之前在做了一个[verilog版的呼吸灯](https://github.com/zhouchuanrui/brl)，主要思路是把正弦信号调制到PWM里面循环输出。这个工程目前还处于还处刚刚完成仿真（用的ModelSim）的阶段，其中里面的正弦波数据是这样调用的：

```verilog
	reg [19:0] mem[199:0];
	initial 
	begin
		$readmemh ("sine.dat", mem);
	end
```

之前需要用到正弦波数据的时候，都是老老实实的用`ROM IP+Matlab+mif`文件来实现的。这还是我第一次用`$readmem`，之前只知道这个语句是不可综合的。当时手头没有板子可以用，心想还是快点弄个出来先仿真，于是就用了这个了，数据文件也是直接用Python弄出来的（Python真是好）。

近来开始换用Vivado了，从A家换到X家，从新开始，很老实的看各种官方User Guide，然后就在机缘巧合之下，突然看到一篇文档里面直接提到说`$readmem`是可综合的。这个发现让我很震惊，这个带`$`的语句竟然是可综合的？！然后我又回头翻Quartus的文档，发现原来`$readmem`在Quartus里面也是可以综合的。思维定势蒙蔽了我的双眼，这个语句要是可综合的那用起来可就方便多了。

<!--more-->

不过，还是有些注意事项，要在这里记录一下：

- `$readmem`分为`$readmemb`和`$readmemh`，一个读二进制数据，一个读16进制数据；
- 输入的文本文件，可以是任意后缀；
- 文件中只写数据即可，数据顶格写，每行存放一个数据，无需带`0x`或`0b`这种前缀。

最后再把Python脚本贴一下：

```python
import math
import os

list = []
for i in range(200):
	rad = math.pi*i/100
	list.append(rad)
	#print (rad, end = ', ')

res = map(math.sin, list)

fd = open("../rtl/sine.dat", 'w')
for i in res:
	#dat = hex(int(((i+1)*100)))
	dat = hex(int(((i+1)*499999)))
	print (dat)
	fd.write(dat+'\n')

fd.close()
```
