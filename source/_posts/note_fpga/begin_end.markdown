title: begin-end还是有用的
Status: Public
Tags: verilog
date: 2014-1-11 21:09:53
---

最近代码越写越长，感觉begin-end去掉倒也看着清爽许多。有一段这样的代码：

```verilog
always @(posedge clk or negedge rst_n)
begin
	if (!rst_n)
		wave <= `LOW;
	else if (en == `ON)
		if (cnt == pPOS_HIGH)
			wave <= `HIGH;
		else if (cnt == pos_low)
			wave <= `LOW;
	else
		wave <= `HIGH;
end
```

<!--more-->

一做仿真竟然会出错，我看了好久才看出来原来是最后一个else的问题，上面的代码其实相当于是： 

```verilog
always @(posedge clk or negedge rst_n)
begin
	if (!rst_n)
		wave <= `LOW;
	else if (en == `ON)
		if (cnt == pPOS_HIGH)
			wave <= `HIGH;
		else if (cnt == pos_low)
			wave <= `LOW;
		else
			wave <= `HIGH;
end
```

最后的else是算在`else if (en == `ON)`下了，不管怎么缩进是没用的。

还是老老实实的用`begin-end`好了：

```verilog
always @(posedge clk or negedge rst_n)
begin
	if (!rst_n)
		wave <= `LOW;
	else if (en == `ON)
	begin
		if (cnt == pPOS_HIGH)
			wave <= `HIGH;
		else if (cnt == pos_low)
			wave <= `LOW;
	end
	else
		wave <= `HIGH;
end
```

