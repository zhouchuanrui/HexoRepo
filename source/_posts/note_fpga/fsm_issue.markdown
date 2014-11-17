title: 状态机要注意异步输入信号
Status: Public
Tags: FSM
date: 2014-1-16 21:35:01
---

最近的项目里写了一个管脚信号滤波的模块，可以设置延迟参数，主要是想用来做一个自制协议的信号解码的前置滤波，里面的主体是这个状态机：

<!--more-->

```verilog
(* syn_encoding = "safe" *)reg [3:0] ns_sig, cs_sig;
//state parameters
localparam 
    sLOW    = 4'b0001,
    sTO_HIGH    = 4'b0010,
    sHIGH   = 4'b0100,
    sTO_LOW = 4'b1000;

always @(posedge clk or negedge rst_n)
    if (!rst_n)
        cs_sig <= sLOW;
    else
        cs_sig <= ns_sig;

always @(*)
begin
    ns_sig = cs_sig;
    case (cs_sig)
        sLOW:
            if (sig == `HIGH)
                ns_sig = sTO_HIGH;
        sTO_HIGH:
            if (valid_HIGH == `ON)
                ns_sig = sHIGH;
            else if (sig == `OFF)
                ns_sig = sLOW;
        sHIGH:
            if (sig == `OFF)
                ns_sig = sTO_LOW;
        sTO_LOW:
            if (valid_LOW == `ON)
                ns_sig = sLOW;
            else if (sig == `ON)
                ns_sig = sHIGH;
        default:
            ns_sig = sLOW;
    endcase
end
```

用的是标准学院派写法，还用了`safe`这个状态机编码`synthesis attribute`。想来必然是万无一失的。

但是这个运行的时候总是时不时的出错，像是我在一个工程里设置了高低电平滤波时长为200个`clk`，在板子上调试的时候出错了，在输出信号下降沿的时候输出直接也下降了，并没有延迟200个时钟，设计的功能也就失效了。

然后我就用SignalTap抓信号调试，后来发现是状态机这里会出错。SignalTap里可以直接显示状态机的参数名称，`sLOW, sTO_HIGH, sHIGH, sTO_LOW`，然后我在抓下降沿的时候看到了一个其他的数字编码，后面跟着的是sLOW这个状态。这显然是跳转到其他状态引起状态机复位了。

但是这个现象我不理解，因为这完全是正常的输入，抓耳挠腮想不出原因来。这个`safe`状态编码一点也不safe嘛。我就去[stackoverflow上提问](http://stackoverflow.com/questions/21085315/when-would-a-verilog-state-machine-go-wrong)。

在等答案的时候我灵光一现，试着把状态机编码改成2位的格雷码，然后把`syn_encoding`改成了`user`，结果一编译运行，竟然对了。

这个时候我还是很激动的，然后回来一看stackoverflow上的问题已经有了个答案了，答题是个分很高的牛逼老兄，跟我说的亚稳态的事情。我一想觉得很有道理啊，这个`sig`输入直接接到状态机的前级，只是没想到直接让状态机复位了。

这个时候我突然想起之前有一个板子上，用一个相似的状态机写了一个抓取写时序操作的模块，这个运行的时候直接是整个FPGA都复位了，当时的直觉是这块板子的硬件方面有问题，实际上这个板子的确有很多问题，但是这个FPGA复位恐怕是这个输出亚稳态引起的。

原来这个东西会有这么大的影响。

照这个情况来看，格雷码自然是有效的，不过还不是根本方法。照这个老兄的意思，我是应该在`sig`和状态机之间再插一个触发器才行。

然后我就做了这个测试，加了一个触发器一运行，果然是没问题的。最后我的终极版本是“状态机格雷码编码+前级同步触发器”：

```verilog
//--IMPORTANT to deal with the physical ports
//--add an FF between sig and state-machine
reg sig_r;
always @(posedge clk)
begin
	sig_r <= sig;
end

//state reg
(* syn_encoding = "user" *)reg [1:0] ns_sig, cs_sig;
//state parameters
localparam 
	sLOW	= 2'b00,
	sTO_HIGH	= 2'b01,
	sHIGH	= 2'b11,
	sTO_LOW	= 2'b10;
```

-----

这次经历获得的教训：

- 不要对状态机太自信，状态机的正确运行跟其他模块其实没差别。
- 状态机一旦运行错误，破坏性无法估量。
- 学院派状态机的前级是组合逻辑，要特别注意对异步输出信号的处理，这个是以前不曾想到的。

