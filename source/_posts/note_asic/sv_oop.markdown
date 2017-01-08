title: Systemverilog的OOP相关语法特性来自于哪里
Status: Public
Tags: 
date: 2016-2-9 21:18:52
---

[TOC]

Systemverilog的语法内容非常丰富. 一般认为, Systemverilog的语法除了一些基础语法元素之外, 可以明显的分成四个部分: 与Verilog兼容的RTL部分, 断言(SVA)部分, 功能覆盖率(FC)部分和面向对象(OOP)部分.

其中OOP部分作为各种验证方法学的基础, 是Systemverilog中最重要的内容了. Verilog跟Systemverilog的关系, 很容易让人联想到C跟C++的关系, 再加上verilog的基础语法部分基本都是C语言里的(把`begin-end`换成大括号), 那是不是Systemverilog的OOP语法特性是来自于C++的呢? 

<!--more-->

其实说起来, Systemverilog的OOP相关语法中最显眼的部分是来自于Java的:

- 单继承和继承的形式`extends`, 类接口`interface`这些跟Java是完全一致的. 
- 然后是泛型(也就是参数化类), 只是语言格式上有一点点不一样. C++也是有泛型的, 不过C++里面叫模版, 而且还支持函数的泛型(模版函数), 所以这个部分来讲还是可以认为是Java的语法.
- 最后是自动垃圾回收, 这个其实不能说是来自于Java, 现在的常用语言里还需要程序员手动管理内存释放的恐怕只剩C/C++了, 由于这个特性是C++所不具备的, 那就算到Java头上吧:)

上面几点来看, Systemverilog的OOP似乎跟C++已经完全没有关系了, 不过也不是这样的, Systemverilog中来自于C++的特性也不少:

- 基于virtual的动态绑定, 也就是说使用virtual关键字的方法才是动态绑定方法, 不像Java里面默认的方法都是动态绑定方法. 这个是个比较重要的多态行为.
- 动态类型转换, 也就是`$cast`操作.
- 其他的一些小语法元素, 比如类作用域操作符`::`, 方法的`extern`实现, 静态成员和方法的访问形式等.

所以总的说来, Systemverilog的OOP语法特性可以看作是Java和C++的结合体, 对Systemverilog的OOP语法有疑问的话可能需要从Java和C++中分别去找答案.


