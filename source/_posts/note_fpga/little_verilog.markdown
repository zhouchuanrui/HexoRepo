title: Verilog小细节
Status: Public
Tags: verilog
date: 2013-5-4 19:35:02
---

初学verilog的刚知道还有可综合不可综合的时候，觉得可综合的verilog真是太简单了，用到的语法只有一点点，现在看看实在是孤陋寡闻了。今天了解到的新的东西总结一下：

<!--more-->

- verilog-2001的RTL可综合标准可以参考文档 IEEE P1364.1 / D1.6 Draft Standard for Verilog(R) Register Transfer Level Synthesis，这个文档规定了综合工具对语法的支持；
- 线网类型除了`wire`之外还有`tri,tri0,tri1,wand,wor,triand,trior,suppy0,supply1`，其中：
	- `tri,tri0,tri1`表示三态的线网（其实`wire`的z状态也可以表示高阻，当时对这个不在意可能是这个原因）`tri0`跟`tri1`分别是带下拉跟上拉电阻。其实上下拉电阻只在输入断开的时候起作用，这个设计中基本用不到，这个可能是对这个不在意的另一个原因。另，1364.1文档中说综合是不支持`tri0`跟`tri1`的；
	- `wand,wor,triand,trior`，后半部分可以看出这个是带逻辑结果线型变量，也就是说是多输入的；
	- `supply0`和`supply1`分别表示逻辑的0和1，是无输入的；
- 综合支持的编译指令有\`default\_nettype,\`define,\`undef,\`include,\`ifdef, \`else, \`elsif, \`endif, \`ifndef,其中：
	- \`defalut\_nettype后可跟`tri,tri0,tri1,wand,wor,triand,trior,none`，指示隐含的线网(模块的IO等)的默认类型，如果没有这条命令则默认为`wire`，如果选择none则有隐含的线网类型时会报错——这个其实蛮好的，可以做代码规范性检查；
	- \`define是支持参数的，跟C的带参数宏是一样的；
- interger其实是带符号的，不过verilog-2001中的`wire`和`reg`都已经是支持signed定义关键字了

总的说来，又一次让我认识到verilog的内容并不是这么简单的，细节还是相当多的。体会是还是应该相信官方的文档已经实践，不要道听途说。

