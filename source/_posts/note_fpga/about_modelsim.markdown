title: ModelSim使用笔记
Status: Public
Tags: ModelSim
date: 2014-1-4 15:18:32
---

[TOC]

# 一些让人惊讶的地方

可以设断点, 做单步运行调试!!!

modelsim是跨平台的, 可以在Linux和Unix下用!!!

<!--more-->

# 帮助文档

在ModelSim里面点击[help]->[Documentation-PDF bookcase], 里面是文档目录, 有详细的User's Manual和Reference Manual, 还有一篇100多页的Tutorial, Tutorial写的很不错.

这篇笔记的内容基本来源于Tutorial.

# 软件版本以及安装破解

ModelSim版本众多, 有SE(System Edition), PE(Personal Edition). 还有厂商OEM版本, 如Xilinx的XE和Altera的AE等.

这些版本里面最牛逼的是SE版, 这里用的是SE6.5版本.

最新版好像是到10.多了, 要下最新版可以去官网注册然后下载, 破解方法好像是通用的.

>运行“MentorKG.exe”来更新“LICENSE.TXT”文件。安装时用此“LICENSE.TXT”来安装。
本人破解步骤：
生成的LICENSE.TXT内容另存为LICENSE.dat
LICENSE.dat拷贝到modeltech\_6.5g\\win32下
系统变量增加
变量名  LM\_LICENSE\_FILE
变量值  你的安装路径\modeltech\_6.5g\win32\LICENSE.dat

# 简单的仿真过程(不带modelsim工程)

步骤总结如下:

1. 在工作目录(对我来说就是Quartus目录)下编辑完源文件与test bench文件. 
1. 打开ModelSim, cd(change directory)到工作目录下.
1. 新建library, 默认为work即可, 当然要改别的名字肯定也是可以的.
1. 编译源文件, 点击[compile]->[compile], 然后选中需要的文件, 编译完之后, work目录下会显示出编译的对象.
1. **优化Design**, 使用vopt命令:`vopt +acc <testbench_name> -o <output_obj>``+acc`指定信号可视化, `-o`后面为优化的输出对象指定名称.
1. 载入Design, 其实就是开始仿真, 可以在shell里用`vsim <output_obj>`或者直接在work下双击`<output_obj>`.
1. 后面就是简单的选择信号, 然后运行仿真这些了.

# 编译第三方库

像是我在工程中用了Quartus的IP核, PLL和FIFO, 这样在仿真的时候就需要编译然后添加这些库(因为用的版本是SE的, 如果用的Altera的OEM版本AE, 就不用编译了), 这里Quartus变成第三方了.

Quartus的仿真库目录在`<install_dir>\quartus\eda\sim_lib`下, 常用的有220model, altera\_mf和altera\_primitives.

ModelSim的通用库目录就在安装目录下, 这里贴一个推荐的库编译步骤:

1. 在ModelSim的目录下新建路径`Altera\src\`, 把Quartus仿真库目录里的文件全都拷到这个路径下.
1. 打开ModelSim, cd到`Altera\`目录.
1. 新建库, 库名为220model.
1. 在这个库中编译`\src`目录中的源文件, 只用verilog的编译.v文件, 只用vhdl的编译.vhd文件, 用混合仿真的两个文件都选了. 
1. 按照这个方法重复上面两个步骤编译出库altera\_mf和altera\_primitives.
1. 把编译的库设置为全局库:
	1. 找到ModelSim安装目录下的modelsim.ini文件, 取消只读属性.
	1. 在[library]下加入新建的库, 格式参照`<lib_name> = <lib_dir>`, 跟其他的项保持一致即可.
	1. 编辑完之后退出, 把modelsim.ini重新设为只读.

这样新建的库就可以用了.

网上也有把220model, altera\_mf和altera\_primitives一起编译到一个库的, 这个步骤还少一点. 但是这里讲的步骤可以让库更加清晰简洁, 使用起来可以用更轻量的库, 还是更推荐这种做法.

# 建立ModelSim工程
	
上面讲的仿真过程其实是通用的, 建立ModelSim工程只是为了更好的管理源文件和仿真算例.

ModelSim工程就是一个.mpf文件, 里面可以存一些库信息和仿真算例信息. 新建工程跟新建文件一样, [File]->[New]->[Project], 然后可以配置源文件, 所用的库路径, 还有优化选项. 可以配置多个仿真算例. 配置完后仿真算例就保存了, 后面可以直接双击仿真算例对象来进行仿真, 这样就比较方便了.

# 自动生成的文件

- `<work>\`库目录, 默认名是work, 这种算是数据库文件.
- `.wlf`文件, 就是仿真出来的波形文件, 运行的越久, 文件越大, 要果断加到.gitignore里面.
- `.cr.mti`文件, 不知道怎么产生的, 是个文本文件, 里面包含了用到的module的信息, 也不大, 存着好了.
	
# 一点使用感受

ModelSim的界面跟Quartus风格比较相近, 都是糙的很. 编译器的高亮比Quartus的好像要稍好一点, 但是分段缩进要跟混乱一点. 

其实最大的感受就是卡, 一点仿真界面就漂的飞起, 看来仿真软件不是一般机器可以轻松驾驭的. 然后就是要做长时间的仿真的话还是会很慢, 像是做个IRIG-B这种秒级的仿真, 用真实时间标度根本做不下来.

不过不管怎么样这个工具还是要好好的学习掌握的.

