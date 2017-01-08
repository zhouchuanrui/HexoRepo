title: Verification匠心
Status: Public
Tags: Makefile 
date: 2016-7-27 22:04:39
---

[TOC]

这篇笔记用于记录一些验证工作过程中的一些有益的小技巧和总结.

# Makefile中空变量有大用

做数字验证的时候, 仿真器的编译和运行需要很多的编译运行参数, 一般Makefile里面就会写了很多东西了. 不过, 大多数时候, 在验证平台的调试阶段, 你会需要添加很多额外的编译选项和运行选项比如编译时添加`-debug_all`编译为行调试模式, 运行时添加`+ntb_opt random_seed=<n>`指定随机种子运行. 另外, UVM也引入了很多的运行参数. 

<!--more-->

执行make的时候不能额外的追加参数, 比如`make vcs_compile -debug_all`的时候`-debug_all`是不能生效的. 这样一来, 你的Makefile会变的很复杂, target好多, 似乎用shell才是正解了. 

不过, make的时候, Makefile的变量是可以重写的, 这是个转机. 我在Makefile里会设置一个空变量:

```
ext = 
```

然后再各个target里都添加进这个变量:

```
vcs_compile:
    vcs ${VCS_CMP_OPT} ${ext} ...
vcs_run:
    ./simv ${VCS_RUN_OPT} ${ext} ...
ius_compile:
    irun ${IUS_CMP_OPT} ${ext} ...
ius_run:
    irun ${IUS_RUN_OPT} ${ext} ...
verdi_run:
    verdi ${VERDI_OPT} ${ext} ...
```

然后根据需要在make的时候重写`ext`变量, 比如`make vcs_compile ext=-debug_all`来编译为行调试模式, `make vcs_run ext=-ucli`进入`ucli`运行模式, 要添加多个选项的时候需要用引号(推荐用单引号, 双引号会转义, 这个跟shell的规则是一样的)比如`make vcs_run ext='+UVM_VERBOSITY=UVM_LOW +UVM_PHASE_TRACE'`添加多个UVM运行选项.

做个类比的话, Makefile里的其他变量相当于sequence item里面的一些通用约束块, 这个空变量相当于sequence中的inline constraint, 用起灵活了许多.

# 使用Package来组织源码

Mentor的UVM Cookbook的最后部分有一些语言规范, 里面推荐使用Package来组织源码, 其实这也是UVM源码的组织方式.

这个规范的内涵包括:

1. 使用`package endpackage`来`include`类定义的文件, 而且只在Package里进行`include`, 在类定义文件里不进行`include`;
1. Package文件以`_pkg`结尾并使用`.sv`后缀名, 表示一个编译单元, 类定义使用`.svh`后缀表示非编译单元, 这也是跟UVM一致的;
1. 为每个复用单元都建立Package, 在使用UVM的时候也就是从agent开始就要使用Package了;

举个例子吧, 有这样的一个目录:

```
~prj/
  ~tb/
    ~agent/
      -MyDriver.svh 
      -MyMonitor.svh 
      -MySequencer.svh 
      +agent_pkg.sv
    -MyEnv.svh
    -MyRefm.svh
    -MyScoreboard.svh
    +env_pkg.sv
```

就需要添加`agent_pkg.sv`和`env_pkg.sv`了, `agent_pkg.sv`里面大致是这样的:

```
package agent_pkg;
    import uvm_pgk::*;
    `include "uvm_macros.svh"
    ...
    `include "tb/agent/MyDriver.svh"
    `include "tb/agent/MyMonitor.svh"
    `include "tb/agent/MySequencer.svh"
    ...
endpackage
```

`env_pkg.sv`里面大致是这样的:

```
package agent_pkg;
    import uvm_pgk::*;
    `include "uvm_macros.svh"
    ...
    `include "tb/MyEnv.svh"
    `include "tb/MyRefm.svh"
    `include "tb/MyScoreboard.svh"
    ...
endpackage
```

然后在编译的filelist里需要把这些Package文件都放进去.

这么做的好处是什么呢? 归纳起来有这么几点:

1. 把include文件集中到一处显得比较清晰, 更容易复用;
1. 做引用的时候`import`比`include`要安全, 因为前者多次引用完全没问题, 后者的定义文件里如果没写`ifndef`编译预处理不小心出现多处`include`的话是会编译出错的;
1. Package创建了一个独立的编译单元, 仿真器在做增量编译的时候使用Package会获得一点点性能上的提升. 要注意`module`和`interface`是天然的独立编译单元, 是不可以include到Package里面的语法结构, 按照这个规范, top module和dut interface文件都应该以`.sv`结尾.

此外要注意的是, 虽然有`import`语法用来做显式引用(而且使用UVM的时候就是这么用的), 但是其实用`package_name::resource`的方式更安全. 比如说你的多个Package里面都定义了共用的枚举, 而这些枚举恰好有重名, 然后这些同名的枚举是不同的值, 这样你在用`import package_name::*;`的时候就可能会有枚举冲突, 而用`package_name::enum_item`这种格式就不会有问题. 

# 使用编译时外部宏和运行时命令行参数实现验证平台的"多态"

plusargs是Systemverilog继承自Verilog的语法, 分为`$test$plusargs`和`$value$plusargs`两类, 用于扩展运行时参数. UVM使用的命令行参数, `UVM_TESTNAME`, `UVM_VERBOSITY`和`UVM_OBJECTION_TRACE`等都是通过plusargs实现的. 

使用自定义plusargs, 可以灵活的实现很多很实用的功能. 为了实现运行不同的`uvm_test`时生成不同名字的fsdb波形文件, 我的验证平台top中有这样的代码:

```
`ifndef NO_DUMP
initial begin: dump_files
    string fn;
    if($value$plusargs("UVM_TESTNAME=%s", fn))
        fn = {fn, ".fsdb"};
    else
        fn = "test.fsdb";
    #0.1;
    $fsdbDumpfile(fn);
    ...
end
`endif
```

直接使用`UVM_TESTNAME`这个用于`run_test`的这个value plusargs再加上`.fsdb`后缀拼接成文件名, 然后再dump, 实现了fsdb文件和testcase的绑定. 当然这里也可以用单独的value pulsagrs来指定fsdb文件名. 中间的`#0.1`延时是考虑到运行的时候有可能打错case的名字, 这个时候`run_test`先运行报错退出, 就不会生成错误的fsdb文件了.

其他的诸如, 控制一个sequence的运行次数, 控制是否进行覆盖率采集, 控制scoreboard的信息输出到STDOUT还是文件等, 都可以用自定义的plusargs方便的实现. 这个比起定义不同的`uvm_test`使用`config_class`来生成不同的验证平台来, 要方便多了.

除了plusargs之外, 宏也是实现更改验证平台行为的一大利器. 注意到第一行的`ifndef`编译预处理命令了吗? 也就是说编译时添加宏`+define+NO_DUMP`可以去掉这个initial块. 其实要实现控制dump波形文件的开关的话, 直接用`$test$plusargs`才是更好的方案:

```
    ...
    #0.1;
    if (!$test$plusargs("NO_DUMP")) begin
        $fsdbDumpfile(fn);
        ...
    end
    ...
```

因为编译时的外部宏是编译时行为, 更改宏的话需要重新编译, 灵活性不如plusargs. 不过呢, 这里的代码已经显示出宏跟plusargs的区别了: plusargs只能出现在一个procedure block内, 而宏没有这个限制.

比如说, 为了方便, 你的验证平台的top里例化了多个可以独立运行的DUT, 在做一些单独的DUT验证用例然后仿真时间又比较长的时候, 你想要屏蔽不需要的DUT, 这个时候就只能用宏了比如:

```
`ifndef NO_DUT_HASH
hash m_hash(
    .clk   (clk),
    .rst_n (rst_n),
    ...
);
`endif 

`ifndef NO_DUT_AES
aes m_aes(
    .clk   (clk),
    .rst_n (rst_n),
    ...
);
`endif 
```

所以, 可以在procedure block中控制的行为使用plusargs, 其他地方用宏, 两者一结合, 就可以实现灵活强大的"多态"验证平台了.


