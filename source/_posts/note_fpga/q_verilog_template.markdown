title: 无意中发现Quartus ii 里面的verilog template
Status: Public
Tags: verilog
date: 2013-4-3 17:27:42
---

今天才仔细的看了看quartus里面的template里面的内容，很有收获啊。。

```verilog
// Quartus II Verilog Template
// Signed multiply with input and output registers

module signed_multiply_with_input_and_output_registers
#(parameter WIDTH=8)
(
     input clk,
     input signed [WIDTH-1:0] dataa,
     input signed [WIDTH-1:0] datab,
     output reg signed [2*WIDTH-1:0] dataout
);

     // Declare input and output registers
     reg signed [WIDTH-1:0] dataa_reg;
     reg signed [WIDTH-1:0] datab_reg;
     wire signed [2*WIDTH-1:0] mult_out;

     // Store the result of the multiply
     assign mult_out = dataa_reg * datab_reg;

     // Update data
     always @ (posedge clk)
     begin
          dataa_reg <= dataa;
          datab_reg <= datab;
          dataout <= mult_out;
     end

endmodule
```

<!--more-->

首先是parameter的定义，原来还有#(parameter WIDTH=8)这样直接在module的列表前面这样定义的。这让我想起verilog-mode里面的自动例化的格式：

```verilog
	instModule #(/*autoinstparam*/)
	instName(/*autoinst*/);
```

原先还以为`#(/*autoinstparam*/)`这个是verilog－mode作者的独创，原来语法里面原先就有的。对于这类可配置参数的模块来讲，因为是全局的参数，这种定义方式的确是比较合适。之前也有看到用宏定义而不是parameter来定义状态机的状态常量的，我在想要不以后都用宏定义来定义状态机状态常量，这样便于统一使用autoinstparam对parameter进行重定义。

然后就是端口变量声明这里，竟然可以直接在module的io列表里面用input、output就声明了，下面就不用再定义了，要是早知道这个，写代码要省好多力气的啊。不过还好我后来有用Emacs里面的verilog-mode。

最后就是signed这个东西了，这个是很关键的一个东西啊。我原先一直不知道verilog里面有signed这个关键字，我还一直以为带符号数的运算都是需要自己重新编写的。。。不过想想FPGA毕竟是基于查找表实现的，实现带符号的整数运算也就修改下真值表就可以了。但是总的说来，这个东西简直是让我重新认识了verilog。

。。。刚刚去搜索了下，原来上面的3点全都是verilog2001相对与verilog1995的新增内容，孤陋寡闻了，我感觉好落伍。不过这些市面上的相关书籍也有责任，没有做到与时俱进，这都10多年过去了。要知道这些更新很重大，像signed这个简直是革命性的,书上标出这些东西的区别我感觉是非常有必要的。






