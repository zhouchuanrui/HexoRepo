title: Learning Python the Hard Way笔记
Status: Public
Tags: 
date: 2013-4-15 20:56:41
---

[TOC]

# 2.X与3.X的不同

LPHD里面用的是2.x，我装的是3.4，目前发现的不同之处有：

- `print`：2.x里面直接使用`print <args>`格式，3.x里面要使用`print (<args>))`；
- `input`：2.x里面使用`raw_input()`，3.x里面使用`input()`

<!--more-->

# 格式控制符

好像都是`print`里面使用的：

```python
print ("...%m1...%m2..." %(v1, v2,...))
```
# 字典

其格式为:

```python
dict = {
	key1: value1,
	key2: value3,
	key3: value3,
	...: ...
}
```

要关注下`dict.items()`和`dict.get()`的用法。

添加字典成员的方法为：

	dict[new_key] = new_value

删除字典成员使用关键字`del`：

	del dict[key]

# 类

类定义格式为：

	class X(Y)	

表示X继承于Y，没有直接继承关系的类需要写成：

	class X(object)

表示继承至object

# list

**string也是tuple**,是一种不可变的list。

使用三重引号建立的字符串，里面的内容是不进行转义的。

list的slice格式为：

	list[n:m]

# 搭建自动测试环境

搭建自动测试环境需要两个环节：安装相应的包；建立测试路径。

## pip包管理

Python3.4以自带pip包管理工具（先见之明~），可以在cmd.exe下使用`pip install <package>`来安装包。

需要安装的包有：

```
pip install nose
pip install distribute 
pip install virtualenv
```

# id()

`id(var)`可以获取变量`var`的地址。

`type(var)`可以获取变量`var`的类型。

# raw string

	r"string" or R"string"

使用这种方式来指定raw string，正则表达式要使用这种方式来创建。

