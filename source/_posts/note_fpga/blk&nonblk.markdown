title: 一个很好的解释阻塞赋值和非阻塞赋值的例子
Status: Public
Tags: verilog
date: 2013-5-4 19:51:39
---

```verilog
module bnbasm  (/*AUTOARG*/
	// Outputs
	q1, q2,
	// Inputs
	clk_osc
	) ;
	input clk_osc;
	output [7:0] q1,q2;

	reg [7:0] q1,q2;
	always @(posedge clk_osc)
	begin
		q1=q1+8'd1;
		q2=q1;
	end
endmodule //
```

<!--more-->

这段代码综合出的RTL模型为：

![Image Title](/article_pics/blk_rtl.png)

将过程块中的赋值语句改成非阻塞赋值：

```verilog
always @(posedge clk_osc)
begin
	q1<=q1+8'd1;
	q2<=q1;
end
```

则综合出来的RTL模型为：

![Image Title](/article_pics/nonblk_rtl.png)

阻塞语句和非阻塞语句的认识是教科书上的：阻塞赋值语句在所在的块中是按从上到下执行，而非阻塞赋值为并发执行。难理解的地方在于阻塞赋值的“从上到下执行”，这个说法很容易会被理解成时间上的先后顺序，会认为这些赋值语句之前会有延时存在。但是这个延时是指的什么延时，怎么安排的这些延时，会往这方面去想，觉得哪里都不合理，因此会产生疑惑。

仔细观察两个RTL模型，就感觉恍然大悟，原来阻塞赋值的“从上到下执行”是通过空间逻辑结构来实现的，延时什么的根本跟这个没有关系。

至于通常的说法“组合逻辑使用阻塞赋值，时序逻辑使用非阻塞幅值”并不是准确的。阻塞赋值跟非阻塞赋值的混用也有可能会是设计的需要，至于Quartus里面对两种赋值混用的报警，设计者应该检查下是不是无意的行为，因为这个可能是会导致意外的结果的。



-----
初学verilog的时候没有多关注这些小细节，没有多使用RTL viewer这些有用的工具，以后还是要多补补课啊。《Verilog HDL 应用程序设计实例精讲》是一本好书啊，感觉比夏宇闻的那本入门书好多了。







