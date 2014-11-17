title: 将Python设为cmd.exe下的命令
Status: Public
Tags: Python
date: 2014-8-22 7:57:59	
---

[TOC]

# 背景

我写了一个[批量文件命名的Python脚本](https://github.com/zhouchuanrui/PyReName)，可以对**脚本所在的路径下的文件重新进行增量式命名**。内容很简单，但是目前只能把脚本复制到需要进行文件重命名的文件夹下面运行，比较麻烦。

<!--more-->

改进方案这这么两种：

1. 修改代码，改成可以传入文件夹路径参数的方式。但是这种方案每次都要先复制一下路径，还是有点麻烦的。
2. 直接把这个脚本设成全局的命令，然后就可以在任意地方使用的了。

这里显然第二种方案是比较好的，可以最方便的使用这个脚本。

我现在用的还是WinXP系统，使用脚本的方式还是最原始的在cmd.exe下使用`Python script.py`命令。因此，为了实现第二个方案，还需要分成两步。

# 在任意路径下打开cmd.exe

这部分在网上找的[资料](http://bbs.csdn.net/topics/190002407),把下面的脚本:

```bat
Windows Registry Editor Version 5.00 

[HKEY_CLASSES_ROOT\folder\shell\cmd]
@="open CMD here"
[HKEY_CLASSES_ROOT\folder\shell\cmd\command]
@="cmd.exe /k cd %1"
```

保存为`cmd.reg`文件，然后双击运行，这样就可以右键一个文件夹的时候添加`open CMD here`的打开cmd.exe的菜单项了。

Win7下似乎是可以`Shift+Win+R`直接操作，这样可以省去这个步骤。

# 设置脚本为全局cmd命令

这个是在[stackoverflow上问的](http://stackoverflow.com/questions/25416382/how-can-i-make-my-python-script-a-system-command-and-run-it-easily-in-any-direct/25416640?noredirect=1#comment39671007_25416640),具体就是要使用`doskey`命令来设置，`doskey`的[文档资料可以参考这里](http://technet.microsoft.com/zh-cn/library/cc773208%28WS.10%29.aspx),一个完全陌生的命令。

然后具体的操作就是：

1. 先跳转到脚本所在的路径；
1. 在路径下打开cmd并输入：
`doskey rfs=Python rfs.py .`

这样就算设置成功了，之后要重命名文件，就可以在任意的路径下`Win+r`，然后输入`rfs`就启动脚本了。

