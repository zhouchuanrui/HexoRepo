title: Verilog的逻辑判断语句怎么写
Status: Public
Tags: verilog
date: 2013-4-12 15:57:52
---

verilog是可读性相当差的一门语言，其实这个不是语言的原因，而是天生的工种的原因。描述数字逻辑嘛，东西一多就显的乱得很，变量信号的耦合很严重，找一个信号的驱动源经常是需要跳转查找好几个文件。因此，提高verilog代码的可读性很有必要。

<!--more-->

我在这个方面的思路就是尽量借鉴C语言的写法。begin-end就是大括号｛｝，always下面必定要一个，每个if-else还有case的selector都带着。还有就是宏定义的运用，像是简单的1位的1和0都定义成了：

```verilog
`define ON 1'b1
`define OFF 1'b0
`define HIGH 1'b1
`define LOW 1'b0
```

然后1位的信号的操作都用上，像是赋值会写成类似
```verilog
data_flag<=`ON，
```
逻辑判断会成类似
```verilog
sys_reset_n==`LOW。
```

一般的示例中描述一个高电平使能触发器的代码：

```verilog
always @(posedge clk)
	if(en)     data_reg<=data_in;
```

我会写成：

```verilog
always @(posedge clk)
begin
	if(en==`HIGH)
	begin
		data_reg<=data_in;         
	end
end      
```

我感觉这样可读性会比较好，但我之前一直担心这两者会编译出不同的东西。begin-end有没有是完全没关系的，主要就是两个if语句if(en)跟if(en==\`HIGH)的区别。因为前者很明显是可以直接作为触发器的使能端输入，而后者是有一个显式的比较操作，不知道会不会在使能端之前多出逻辑判断的一个比较电路模块，这样会显得得不偿失。然后我就分别用下面的代码编译了一下，

```verilog
always @(posedge clk)			always @(posedge clk)
begin                           begin
	if(en)                          if(en==`HIGH)
	begin                           begin
		data_reg<=data_in;              data_reg<=data_in;    
 	end                             end
end                             end
```

结果出来的东西都是一样的，用RTL viewer看都是：

![Image Title](/article_pics/if_rtl.png)

这下我就放心了，然后我还用if(!en)和if(en==\`LOW)编译了一下，结果出来的东西也是一样的，是一个低电平使能的触发器：

![Image Title](/article_pics/if_rtl.png)

看来完全是可以用if(en==\`LOW)取代if(!en)的。

