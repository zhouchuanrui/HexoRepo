title: 3倍数字分频器
date: 2013-9-4 15:10:49
---

[TOC]

# 分频器很简单

据说这种题目会出现在笔试题里, 我一听说3倍分频器, 我不加思索以为这个是PLL才做的出来的. 

我在网上搜了才知道这个是真的可以实现的, 而且是有直接的verilog代码. 要碰上这种题目的话我估计是马上要悲剧的. 

<!--more-->

```
clk_in:		_==__==__==__==__==__==__

clk_out:	_======______======______
```

直接画个时序图出来, 然后我就觉得这个当然是可以实现的. 就按上面的图来说, 三个`clk_in`为一个周期, 然后`clk_out`在第一个`clk_in`为高, 第二个时钟取`clk_in`的值然后第三个时钟取低, 这个就弄完了. 简直简单到我连verilog代码都不想看就想这直接自己写一个了. 

然后直接上代码吧:

```verilog
`define HIGH 1'b1
`define LOW 1'b0
`define ON 1'b1
`define OFF 1'b0
module triple (/*autoarg*/
   // Outputs
   clk_out,
   // Inputs
   clk_in
   );
	input clk_in;

	output clk_out;

	reg [2:0] shift_reg = 3'b001;
	always @(posedge clk_in)
	begin
		shift_reg <= {shift_reg[1:0], shift_reg[2]};
	end

	reg clk_out;
	always @(/*autosense*/clk_in or shift_reg)
	begin
		if (shift_reg == 3'b001)
		begin
			clk_out = `HIGH;
		end
		else if (shift_reg == 3'b010)
		begin
			clk_out = clk_in;
		end
		else if (shift_reg == 3'b100)
		begin
			clk_out = `LOW;
		end
	end
endmodule
```

# 其他方案和扩展

上面的代码输出的`clk_out`是跟`clk_in`同相50%占空比的三倍频时钟, 这个代码可以改一下:

```verilog
	reg [2:0] shift_reg = 3'b001;
	always @(posedge clk_in)
	begin
		shift_reg <= {shift_reg[1:0], shift_reg[2]};
	end

	wire clk_out;
	assign
		clk_out = shift_reg[0];
```

这个代码更少, 输出的`clk_out`是1/3占空比的三倍频同相时钟:

```
clk_in:		_==__==__==__==__==__==__

clk_out:	_====________====________
```

当时说到这个题目, 其实说的是要画一个三分频的逻辑电路出来, 用最上面的代码要画的话还是有点复杂的, 然后用这个代码那就简单了啊. 就是三个串联的触发器, 1号输出接2号输入, 2号输出接3号输出, 3号输出接1号输入, 三个触发器的上电值为`001`, 接同步时钟`clk_in`, 然后`clk_out`接到3号触发器的输出, 这样就完成了. 我感觉这才是做题的思路啊.

# 总结分析

按这个思路嘛, 当然占空比是1/6到5/6都是做的出来的, 然后要移相的话也是有5种方案, 所以这个题目应该是至少有25种可能的结果. 同理, 5倍分频7倍分频什么的也是这个道理.

怎么说呢, 这题目其实是脑筋急转弯性质的. 我一开始忽略了一个时钟里面其实是有高电平跟低电平这个性质, 就会想不出来, 然后想到之后嘛, 这个明显第二个代码才是最简的方案. 不过主要的一点是, 这种设计是没有实际意义的, 首先上面说的同相是肯定做不到的, 然后`clk_out`出来肯定是有电平切换毛刺的, 用来做时钟的话肯定还是有问题的.

这个题目就当作训练训练思维好了.

