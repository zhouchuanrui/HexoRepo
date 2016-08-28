title: Verification匠心
Status: Public
Tags: Makefile 
date: 2015-7-27 22:04:39
---

[TOC]

这篇笔记用于记录一些验证工作过程中的一些有益的小技巧和总结.

# Makefile中空变量的妙用

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

# 运行时指定fsdb的文件名

# 使用Package来管理源码



