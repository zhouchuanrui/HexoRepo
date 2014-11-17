title: FPGA怎么抑制地弹
Status: Public
Tags: 
date: 2014-1-17 11:02:33
---

[TOC]

# 什么是地弹

芯片的管脚与地之间有寄生电感，然后芯片的输出信号切换的时候，流到地端的电流产生突变，这样会在寄生电感上面产生压降。这个就是地弹。

地弹的大小由两个因素决定，一个就是寄生电感的大小，电感越大地弹就越大；另外一个就是信号切换的频率，信号切换的频率越大地弹的发生的频率也越大，特别是多个管脚信号（就比如数据总线）同时切换的时候这个地弹就很大了。

<!--more-->

地弹会影响芯片的正常工作，具体说来就是地弹严重的时候可以让芯片的输出信号直接变化到反逻辑去。PCB上面有模拟部分的时候，地弹也会对这些部分产生干扰。

抑制地弹的方案也是从这两个因素来入手:

1. 减小寄生电感
方案很多，有减小GND管脚的接地路径，即立即接地；使用多个接地管脚提供更多分流路径，即多点接地。
1. 减少信号切换频率
减小芯片工作频率，减小芯片信号同步切换的次数等。这些可能会有悖与整体功能设计，与设计的指标有冲突。

# FPGA抑制地弹的方法

[Altera的白皮书Minimizing Ground Bounce & VCCSag](http://www.altera.com.cn/literature/wp/wp_grndbnce.pdf)，里讲了很多FPGA设计中抑制地弹的方案，方案的内容和效果的简化表如下：

Design Method|GND Bounce % Improvement  
---|---
 Device package (Flip Chip vs. wire-bonded)|72               
 Slow slew rate|65
 Half the number of SSOs|11 to 35       
 Tri-state every other I/O pin (Z-I/O-Z-I/O)|13               
 Tri-state every third I/O pin (Z-I/O-I/O-Z-I/O-I/O)|7                
 Programmable GND or VCC on every other I/O pin. Programmable GNDs and VCC s are connected to board GND or VCC.|45               
 Programmable GND or VCC on every other I/O pin. Programmable GNDs and VCC s are not connected to board GND or VCC and have a 7.5-pF load.|24 
 Programmable GND or VCC on every 3rd I/O pin. Programmable GNDs and VCC s are connected to board GND or VCC.|41 
 Programmable GND or VCC on every third I/O pin. Programmable GNDs and VCC s are not connected to board GND or VCC and have a 7.5-pF load.|71 
 Programmable GND or VCC on every third I/O pin with a 10-Ω series resistor on each I/O pin. Programmable GNDs and VCC s are not connected to board GND or VCC and have a 7.5-pF load.|68               
 PLL clock drives every other I/O pin|12               
 Programmable output delay|12               
 10-Ω series termination resistor|46               
 Series-RC parallel termination|47               

第一个方案——反片封装的提升率很高。就是说用特制的反片BGA封装，比普通BGA封装的寄生电感要小很多，然后普通BGA封装比QPF封装的又要好很多。当时这个成本方面也是如是的排列。

然后第二个方案，降低速率，提升也相当高，不过这个还是取决于具体的设计指标。

后面的就值得注意了，是可编程管脚的配置。就是把管脚输出为VCC或GND，用这种方法来增加接地点，这个是低成本的。里面最有效的是每隔2个设一个GND或VCC，然后接一个7.5pF负载电容。

-----

项目组里面照着这个方法测试了下，就是把与DAC数据总线旁边的管脚设为GND，然后就观察到DAC输出上的毛刺有了比较显著的改善~要获得很大性能提升用**每隔2个设一个GND或VCC**只能是重新设计板子才行了。

