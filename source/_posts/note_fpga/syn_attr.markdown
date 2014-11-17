title: 真正的Synthesis Attributes用法
Status: Public
Tags: verilog SignalTap
date: 2013-11-2 20:44:37
---

[TOC]

# 起因

最早的时候我在用SignalTap的时候，就发现Node Finder里面有Register这个选项，但是有些（其实可以算大部分）信号是没有的。后来网上找到的解释是这些没有输出到output的信号是会被综合工具优化掉的。

然后找到的解决方案是，把这些信号设置为输出，并在Assignment Editor里面设为virtual pin（因为没有这么多物理管脚的），这样就可以在Node Finder的Pins里面用了。

这个方法是有效的，主要我当时用的是Emacs。随着时间的推进，virtual pin自然是越来越多，今天翻看之前的一个工程，发现virtual pin就有2000多，很是骇人。

<!--more-->

这个方法虽然有效，但是缺点也很是明显。首先就是麻烦，代码部分即使是有Emacs，还是容易有疏漏，层次越深的信号越是如此。然后就是这个显然是会对时序约束有影响。

之后我就又发现了另外的方案，就是在代码里设置Synthesis Attributes。Synthesis Attributes的作用不止是这样。

# 注释格式的Synthesis Attributes是不对的

在网上找到的就是这种Synthesis Attributes，出处可以搜《Altera的几个常用的Synthesis attributes》，格式为：

```
/* synthesis, <any_company_specific_attribute = value_or_optional_value */
```

还讲了例子：

```verilog
//Noprune
//A Verilog HDL synthesis attribute that prevents the Quartus II software from removing a register that does not directly or indirectly feed a top-level output or bidir pin.
//For example:
reg reg1 /* synthesis noprune */;
 
//keep
//A Verilog HDL synthesis attribute that directs Analysis & Synthesis to not minimize or remove a particular net when optimizing combinational logic.
//For example:
wire keep_wire /* synthesis keep */;
 
//preserve
//A Verilog HDL synthesis attribute that directs Analysis & Synthesis to not minimize or remove a particular register when eliminating redundant registers or registers with constant drivers.
//For example:
reg reg1 /* synthesis preserve */;
```

但我用的时候就是不对，在Node Finder里面，该找不到的还是找不到。

# 真正的Synthesis Attributes用法

我对这个其实也不是太执着，一方面是到后来项目的改动已经是很小了，该改的信号都改的差不多了，另一方面就是万一这个成功了，要把原来的都改回来那也不是个轻松的事情呵。

不过今天在Quartus ii里无意识的看verilog template的时候，意外的发现了竟然是有Synthesis Attributes这栏。激动的点开，一个个看了，之后就更激动了，我感觉我找到了真正的Synthesis Attributes的用法了。这个感觉非常强烈，搞的我都感觉不需要去确认了。

先来看看keep的：

```verilog
// Prevents Quartus II from minimizing or removing a particular
// signal net during combinational logic optimization.	Apply
// the attribute to a net or variable declaration.

(* keep *) wire <net_name>;
(* keep *) reg <variable_name>;
```

写法是完全不一样的。Quartus里面都这么写了，总不至于诓我把。刚好这几天要写一个新的工程，新的工程里就用这个方案试试了。

-----

把template里面的全都贴出来好了：

full_case，用来应对case的default不知道怎么写的情况。

```verilog
// Indicates that Quartus II should consider a case statement
// to be full, even if the case items do not cover all possible
// values of the case expression.

(* full_case *) case(...)	
```

parallel_case，指示Quartus不生成优先逻辑，不过貌似case本来就不会生成优先逻辑的啊。

```verilog
// Indicates that Quartus II should consider the case items
// in a case statement to be mutually exclusive, even if they
// are not.  Without this attribute, the Quartus II software
// may add priority logic when elaborating your case statement.
// The Quartus II software will only add this logic if one or 
// more case items overlap or if one or more case items are
// constant expressions.

(* parallel_case *) case(...)	
```

上面贴的keep，对wire和reg都是可以用的，这应该就是实现SignalTap观测内部信号的方法，要注意了。

```verilog
// Prevents Quartus II from minimizing or removing a particular
// signal net during combinational logic optimization.	Apply
// the attribute to a net or variable declaration.

(* keep *) wire <net_name>;
(* keep *) reg <variable_name>;
```

maxfan，指定最大扇出，这个应该是个综合约束。

```verilog
// Sets the maximum number of fanouts for a register or combinational
// cell.  The Quartus II software will replicate the cell and split
// the fanouts among the duplicates until the fanout of each cell
// is below the maximum.

// Register q should have no more than 8 fanouts
(* maxfan = 8 *) reg q;
```

preserve，针对寄存器的去优化策略。

```verilog
// Prevents Quartus II from optimizing away a register.	 Apply
// the attribute to the variable declaration for an object that infers
// a register.

(* preserve *) <variable_declaration>;
(* preserve *) module <module_name>(...);
```

noprune，防止无扇出的寄存器被优化。

```verilog
// Prevents Quartus II from removing or optimizing a fanout free register.
// Apply the attribute to the variable declaration for an object that infers
// a register.

(* noprune *)  <variable_declaration>;
```

```verilog
// Prevents Quartus II from merging a register with a duplicate
// register

(* dont_merge *) <variable_declaration>;
(* dont_merge *) module <module_name>(...);
```

```verilog
// Prevents Quartus II from replicating a register.

(* dont_replicate *) <variable_declaration>;
(* dont_replicate *) module <module_name>(...);
```

```verilog
// Prevents Quartus II from retiming a register

(* dont_retime *) <variable_declaration>;
(* dont_retime *) module <module_name>(...);
```

```verilog
// Identifies the logic cone that should be used as the clock enable
// for a register.  Sometimes a register has a complex clock enable
// condition, which may or may not contain the critical path in your
// design.  With this attribute, you can force Quartus II to route
// the critical portion directly to the clock enable port of a register
// and implement the remaining clock enable condition using regular 
// logic.

(* direct_enable *) <variable_or_net_declaration>;

// Example
(* direct_enable *) variable e1;
reg e2;
reg q, data;

always@(posedge clk) 
begin
	if(e1 | e2) 
	begin
		q <= data;
	end
end
```

useioff，为I/O管脚添加寄存器（FF？），可以起到优化时序的作用。

```verilog
// Controls the packing input, output, and output enable registers into
// I/O cells.  Using a register in an I/O cell can improve performance
// by minimizing setup, clock-to-output, and clock-to-output-enable times.

// Apply the attribute to a port declaration
(* useioff *) output reg [7:0] result;        // enable packing
(* useioff = 0 *) output reg [7:0] result;    // disable packing
```

ramstyle，指定使用的ram的类型，这个其实跟器件也有关系。

```verilog
// Controls the implemententation of an inferred memory.  Apply the
// attribute to a variable declaration that infers a RAM or ROM.  

// Legal values = "M512", "M4K", "M-RAM", "M9K", "M144K", "MLAB", "no_rw_check"

(* ramstyle = "M512" *) reg [<msb>:<lsb>] <variable_name>[<msb>:<lsb>];

// The "no_rw_check" value indicates that your design does not depend
// on the behavior of the inferred RAM when there are simultaneous reads
// and writes to the same address.  Thus, the Quartus II software may ignore
// the read-during-write behavior of your HDL source and choose a behavior
// that matches the behavior of the RAM blocks in the target device.

// You may combine "no_rw_check" with a block type by separating the values
// with a comma:  "M512, no_rw_check" or "no_rw_check, M512"  
```

multstyle，指定乘法用逻辑资源实现还是DSP实现。

```verilog
// Controls the implementation of multiplication operators in your HDL 
// source.  Using this attribute, you can control whether the Quartus II 
// software should preferentially implement a multiplication operation in 
// general logic or dedicated hardware, if available in the target device.  

// Legal values = "dsp" or "logic"

// Examples (in increasing order of priority)

// Control the implementation of all multiplications in a module
(* multstyle = "dsp" *) module foo(...);

// Control the implementation of all multiplications whose result is
// directly assigned to a variable
(* multstyle = "logic" *) wire signed [31:0] result;
assign result = a * b; // implement this multiplication in logic

// Control the implementation of a specific multiplication
wire signed [31:0] result;
assign result = a * (* multstyle = "dsp") b;
```

```verilog
// Controls the encoding of the states in an inferred state machine.

// Legal values = "user" or "safe" or "user, safe"

// The value "user" instructs the Quartus II software to encode each state 
// with its corresponding value from the Verilog source. By changing the 
// values of your state constants, you can change the encoding of your state 
// machine

// The value "safe" instructs the Quartus II software to add extra logic 
// to detect illegal states (unreachable states) and force the state machine 
// into the reset state. You cannot implement a safe state machine by 
// specifying manual recovery logic in your design; the Quartus II software 
// eliminates this logic while optimizing your design.

// Examples

// Implement state as a safe state machine
(* syn_encoding = "safe" *) reg [7:0] state;
```

```verilog
// Assigns pin location to ports on a module.

(* chip_pin = "<comma-separated list of locations>" *) <io_declaration>;

// Example
(* chip_pin = "B3, A3, A4" *) input [2:0] i;
```

```verilog
// Associates arbitrary Quartus II assignments with objects in your HDL
// source.  Each assignment uses the QSF format, and you can associate
// multiple assignments by separating them with ";".

// Preserve all registers in this hierarchy
(* altera_attribute = "-name PRESERVE_REGISTER on" *) module <name>(...);

// Cut timing paths from register q1 to register q2
(* altera_attribute = "-name CUT on -from q1" *) reg q2;
```



