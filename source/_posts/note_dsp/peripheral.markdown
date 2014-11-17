title: 28335的外设结构与寄存器空间映射
Status: Public
Tags: 
date: 2013-4-18 10:53:02
---

[TOC]

# 外设简述

28335不单单是个CPU，还有非常多的外设功能模块，像是ADC、SCI、PWM、CAN什么的。这些模块的的功能是有专门的硬件控制器来完成的，在运行时不会占用CPU资源，只是在配置和进行数据交互时才会用到CPU指令。就像你使用SCI只需设置好波特率和相关的中断，然后做好数据的收发，数据的串并/并串转换、fifo的控制、并行帧监测这都是由硬件模块完成。

<!--more-->

# 寄存器物理结构

CPU跟这些模块的接口就是这些模块的寄存器，模块的配置和访问操作都是通过读写相关寄存器来完成的。这些寄存器的物理存储空间是直接并入数据地址空间的，所以不需要另外的读写指令来操作这些寄存器。

![Image Title](/article_pics/peripheral.bmp)

28335里面又把这些模块的寄存器分为4组，分配在不同的地址空间下。上图可以看出4个寄存器组的地址分配情况。中间的Reserved阴影块应该是留给后续版本升级的地址空间，隔开了组号0跟1、2、3。组1、2、3其实是在连续的地址上的，这些组除了所包含的模块不同之外，其总线结构也是稍有不同的。

NAME|ADDRESS RANGE|SIZE (x16)
---|---|---
Device Emulation Registers|0x00 0880 – 0x00 09FF|384         
FLASH Registers(3)|0x00 0A80 – 0x00 0ADF|96         
Code Security Module Registers|0x00 0AE0 – 0x00 0AEF|16         
ADC registers (dual-mapped) |0x00 0B00 – 0x00 0B0F|16         
XINTF Registers|0x00 0B20 – 0x00 0B3F|32         
CPU–TIMER0/1/2 Registers|0x00 0C00 – 0x00 0C3F|64         
PIE Registers|0x00 0CE0 – 0x00 0CFF|32         
PIE Vector Table|0x00 0D00 – 0x00 0DFF|256         
DMA Registers|0x00 1000 – 0x00 11FF|512         

上表是Peripheral Frame 0的寄存器分配排列信息，各个不同模块的寄存器占用的空间各有不同，在地址空间上连续排列。

一个模块包含着多个不同功能的寄存器，寄存器的不同位代表着不同的信息。每个寄存器都分配了的物理地址。在CCS的C语言开发系统中，在代码源文件里面用结构体描述外设模块的寄存器结构，然后用cmd文件为其一一分配物理地址，这样就完成了寄存器的映射。

# 寄存器地址空间映射

下面就以GPIO模块寄存器为例来展示下这种映射的细节好了。

Name|Address
---|---
GPIO Control Registers|0x6F80 - 0x6FBF
GPIO Data Registers|0x6FC0 - 0x6FDF
GPIO Interrupt and LPM Select|0x6FE0 - 0x6FFF

以上为GPIO三个寄存器的硬件地址分配情况，而DSP2833x\_Headers\_nonBIOS.cmd这个文件里面有这样的地址空间定义：

```cmd
MEMORY
{
	PAGE 0:    /* Program Memory */
	PAGE 1:    /* Data Memory */

	GPIOCTRL : origin = 0x006F80, length = 0x000040     /* GPIO control registers */
	GPIODAT  : origin = 0x006FC0, length = 0x000020     /* GPIO data registers */
	GPIOINT  : origin = 0x006FE0, length = 0x000020     /* GPIO interrupt/LPM registers */
}
```

`origin`表示起始地址，`length`表示长度，再结合名字，很容易就可以推出这正好是GPIO的三个寄存器组的物理地址空间。

```c
//DSP2833x_Gpio.h
extern volatile struct GPIO_CTRL_REGS GpioCtrlRegs;
extern volatile struct GPIO_DATA_REGS GpioDataRegs;
extern volatile struct GPIO_INT_REGS GpioIntRegs;
```

↓↓↓

```c
//DSP2833x_GlobalVariableDefs.c
#ifdef __cplusplus
#pragma DATA_SECTION("GpioCtrlRegsFile")
#else
#pragma DATA_SECTION(GpioCtrlRegs,"GpioCtrlRegsFile");
#endif
volatile struct GPIO_CTRL_REGS GpioCtrlRegs;
#ifdef __cplusplus
#pragma DATA_SECTION("GpioDataRegsFile")
#else
#pragma DATA_SECTION(GpioDataRegs,"GpioDataRegsFile");
#endif
volatile struct GPIO_DATA_REGS GpioDataRegs;
#ifdef __cplusplus
#pragma DATA_SECTION("GpioIntRegsFile")
#else
#pragma DATA_SECTION(GpioIntRegs,"GpioIntRegsFile");
#endif
volatile struct GPIO_INT_REGS GpioIntRegs;
```

↓↓↓

```c
//DSP2833x_Gpio.h
extern volatile struct GPIO_CTRL_REGS GpioCtrlRegs;
extern volatile struct GPIO_DATA_REGS GpioDataRegs;
extern volatile struct GPIO_INT_REGS GpioIntRegs;
```

↓↓↓

```c
//DSP2833x_Headers_nonBIOS.cmd
SECTIONS
{
   GpioCtrlRegsFile  : > GPIOCTRL     PAGE = 1
   GpioDataRegsFile  : > GPIODAT      PAGE = 1
   GpioIntRegsFile   : > GPIOINT      PAGE = 1
}
```

↓↓↓

```cmd
MEMORY
{
	PAGE 0:    /* Program Memory */
	PAGE 1:    /* Data Memory */

	GPIOCTRL : origin = 0x006F80, length = 0x000040     /* GPIO control registers */
	GPIODAT  : origin = 0x006FC0, length = 0x000020     /* GPIO data registers */
	GPIOINT  : origin = 0x006FE0, length = 0x000020     /* GPIO interrupt/LPM registers */
}
```

而在源文件里面，则是GPIO\_CTRL\_REGS、GPIO\_DATA\_REGS、GPIO\_INT\_REGS分别表示这三个寄存器组，DSP2833x\_Gpio.h文件里面声明了这三个寄存器组全局结构，然后是DSP2833x\_GlobalVariableDefs.c为这三个结构体定义自定义数据段GpioCtrlRegsFile 、GpioDataRegsFile、GpioIntRegsFile，在DSP2833x\_Headers\_nonBIOS.cmd文件里面将这三个数据段映射到定义好的三个数据空间GPIOCTRL、GPIODAT、GPIOINT里面，就如上图所示。

物理地址的映射就是这些，那寄存器结构就是简单的在这些结构体的成员里面做文章了：

```c
//DSP2833x_Gpio.h
struct GPIO_CTRL_REGS {
   union  GPACTRL_REG  GPACTRL;   // GPIO A Control Register (GPIO0 to 31)
   union  GPA1_REG     GPAQSEL1;  // GPIO A Qualifier Select 1 Register (GPIO0 to 15)
   union  GPA2_REG     GPAQSEL2;  // GPIO A Qualifier Select 2 Register (GPIO16 to 31)
   union  GPA1_REG     GPAMUX1;   // GPIO A Mux 1 Register (GPIO0 to 15)
   union  GPA2_REG     GPAMUX2;   // GPIO A Mux 2 Register (GPIO16 to 31)
   union  GPADAT_REG   GPADIR;    // GPIO A Direction Register (GPIO0 to 31)
   union  GPADAT_REG   GPAPUD;    // GPIO A Pull Up Disable Register (GPIO0 to 31)
   Uint32              rsvd1;
   union  GPBCTRL_REG  GPBCTRL;   // GPIO B Control Register (GPIO32 to 63)
   union  GPB1_REG     GPBQSEL1;  // GPIO B Qualifier Select 1 Register (GPIO32 to 47)
   union  GPB2_REG     GPBQSEL2;  // GPIO B Qualifier Select 2 Register (GPIO48 to 63)
   union  GPB1_REG     GPBMUX1;   // GPIO B Mux 1 Register (GPIO32 to 47)
   union  GPB2_REG     GPBMUX2;   // GPIO B Mux 2 Register (GPIO48 to 63)
   union  GPBDAT_REG   GPBDIR;    // GPIO B Direction Register (GPIO32 to 63)
   union  GPBDAT_REG   GPBPUD;    // GPIO B Pull Up Disable Register (GPIO32 to 63)
   Uint16              rsvd2[8];
   union  GPC1_REG     GPCMUX1;   // GPIO C Mux 1 Register (GPIO64 to 79)
   union  GPC2_REG     GPCMUX2;   // GPIO C Mux 2 Register (GPIO80 to 95)
   union  GPCDAT_REG   GPCDIR;    // GPIO C Direction Register (GPIO64 to 95)
   union  GPCDAT_REG   GPCPUD;    // GPIO C Pull Up Disable Register (GPIO64 to 95)
};
```

↓↓↓

Name (1)|Address|Size (x16)   
---|---|---
GPACTRL| 0x6F80|2       
GPAQSEL1| 0x6F82|2       
GPAQSEL2| 0x6F84|2       
GPAMUX1| 0x6F86|2       
GPAMUX2| 0x6F88|2       
GPADIR| 0x6F8A|2       
GPAPUD| 0x6F8C|2       
GPBCTRL| 0x6F90|2       
GPBQSEL1| 0x6F92|2       
GPBQSEL2| 0x6F94|2       
GPBMUX1| 0x6F96|2       
GPBMUX2| 0x6F98|2       
GPBDIR| 0x6F9A|2       
GPBPUD| 0x6F9C|2       
GPCMUX1| 0x6FA6|2       
GPCMUX2| 0x6FA8|2       
GPCDIR| 0x6FAA|2       
GPCPUD| 0x6FAC|2       

这里可以根据变量名来一一对应这些寄存器，结构体里面的这些联合体类型都是2个16位长度的。**里面的两个rsvd变量是为保证寄存器地址完全对齐而设置的，这说明为寄存器分配的地址上面并不是每一位的空间都是有利用的，这点直接对着文档看地址分配很容易忽略，虽然这个没什么重要性。**另外体现的信息，不同的寄存器所用的结构体是不同的，这也是针对寄存器的具体物理结构所做的设置。

```c
// GPIO A control register bit definitions */                                    
union GPACTRL_REG {
   Uint32              all;
   struct GPACTRL_BITS bit;
};
struct GPACTRL_BITS {        // bits   description
   Uint16 QUALPRD0:8;        // 7:0    Qual period 
   Uint16 QUALPRD1:8;        // 15:8   Qual period 
   Uint16 QUALPRD2:8;        // 23:16  Qual period 
   Uint16 QUALPRD3:8;        // 31:24  Qual period  
};

=============↓↓↓===============

31			24|23			16
------------------------------
	QUALPRD3  |		QUALPRD2
------------------------------
	  R/W-0			  R/W-0

15			 8|7			 0
------------------------------
	QUALPRD1  |		QUALPRD0
------------------------------
	  R/W-0			  R/W-0
```

GPACTRL寄存器的数据结构类型是GPACTRL\_REG，**使用联合体就是既可以用32位的all也可以用结构体GPACTRL\_BITS bit里面的四个8位数据来访问这个寄存器。**跟GPACTRL的物理逻辑结构一对照就知道为什么要用这种数据结构来定义GPACTRL寄存器了。

# 小总结

28335的外设功能很多，整个寄存器体系结构跟映射关系还是很复杂的，但是找准一个模块慢慢研究，其他的寄存器模块也就触类旁通了。



