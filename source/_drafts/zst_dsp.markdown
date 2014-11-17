title: 中试通DSP代码
Status: Draft
date: 2013-8-28 11:14:36
---


# 中试通DSP代码笔记

## 宏命令
    
<!--more-->

- iTask命令
```c
#define WRITE_WAVE_STRUCT		'h'
#define STOP					'j'
#define WRITE_HARMONIC_STRUCT	'k'
#define HARMONIC_GENERATOR		'l'
#define WRITE_ZERO_OFFSET		'm'
#define SWITCH_WAVE_STRUCT		'n'
#define ADD_STEP				'o'
#define SUB_STEP				'p'
#define DSP_EXT_INSTR_DECODE	'q'
#define CLEAR_HARMONIC_TABLE	'r'
//\#define WRITE_WAVE_PARAM		's'
#define UPDATE_WAVE0_WITH_CACHE		'v'
#define WAVE0_ADD_STEP_TO_CACHE		'w'
#define WAVE0_SUB_STEP_TO_CACHE		'x'
```

<!--more-->

- iSpiRecv命令
是比iTask多了4条命令,其实是iSpiRecv向iTask赋值的命令。
```c
#define RESET_BUFFER_INDEX		'g'
#define DSP_PC_MODE				't'
#define DSP_NORMAL_MODE			'u'
#define RUN						'i'
```
- TABLE_PAGE 一个带参数的宏，看文档的话PBDATDIR的高8位表示方向，低8位表示数据，再根据0-15的取值可以推测是取一个8位的输出。可能是根据这个来译码然后控制一个外设存储器。

		#define TABLE_PAGE(page) PBDATDIR=0x0f00|page //page 0-15 16k/page 2E14=16384 

- TABLE_PAGE的参数
```assembly
PAGE_UA		.set	0F00h
PAGE_UB		.set	0F01h
PAGE_UC		.set	0F02h
PAGE_UD		.set	0F03h
PAGE_IA		.set	0F04h
PAGE_IB		.set	0F05h
PAGE_IC		.set	0F06h
PAGE_ID		.set	0F07h
PAGE_UA_HAR	.set	0F08h
PAGE_UB_HAR	.set	0F09h
PAGE_UC_HAR	.set	0F0Ah
PAGE_IA_HAR	.set	0F0Bh
PAGE_IB_HAR	.set	0F0Ch
PAGE_IC_HAR	.set	0F0Dh
PAGE_BUFFER	.set	0F0Eh
PAGE_SINE	.set	0F0Fh
```
- `#define TABLE_SIZE  16384`
- DAC管脚指令
```c++
#define DAC_LOAD   PFDATDIR=0x1818;PFDATDIR=0x1810//PF3 rising edge triggered,low when idle
#define DAC_RESET  PFDATDIR=0x1800;PFDATDIR=0x1810//PF4 rising edge triggered,hige when idle
```
- DAC数据端口
```c
#define DAC_U0	port0000 //U0  0xffff=10v,0x0000=-10v,0x8000=0v,
#define DAC_U1	port0001 //U1
#define DAC_U2	port0002 //U2
#define DAC_U3	port0003 //U3 
#define DAC_I0	port0004 //I0
#define DAC_I1	port0005 //I1
#define DAC_I2	port0006 //I2
#define DAC_I3	port0007 //I3    
```
- 定时器操作指令
```c
#define TIMER_ON  T1CON=0x5040;  //continous_up count mode,timer enable
#define TIMER_OFF T1CON=0x5000;
```
## 变量与数据结构

- iTask --> uint iTask=0;
spi中断函数中会对iTask进行功能置位，main函数的switch中会对其复位。
- iPcMode --> unit iPcMode=0;
默认为0，spi中断函数中DSP_NORMAL_MODE和DSP_PC_MODE会对其进行置位复位,1表示pc mode，0表示normal mode。
- `WAVE stWave[64];`
```c 
typedef	struct	
{
//long	lTableIndex;     //Q16.14,current out point,high 2bit masked
	uint	iTablePage; 	 //0~15		
	long	lFrequency;   	 //Q16.16,   				 				  				  					
	long    lFrequencyStep;
	int	    iPhase;          //0~16383,0~2PI,negative phrase converts to positive
	int     iPhaseStep;      //0~16383
	uint 	iAmplitude;      //0~32767,do &0x7fff when intializing 	
	long 	lAmplitude;      // 0~0x7fffff  
	long    lAmplitudeStep;  //0~+/-0x7fffff
	int     iDcOffset;       //0~+/-32767(0x7fff)
	long    lDcOffset;       //0~+/-0x7fffff
	long    lDcOffsetStep;	 //0~+/-0x7fffff		
}	WAVE;  
```
- `HARMONIC stHarmonic[21];//0 dc,1 base,2-20multiple harmonic`
```c
typedef	struct	
{  				  				
	int	    iAmplitude;      //0~32768,positive	
	int    iPhase;          //0~16383
}
```
- iZeroOffset，零漂数据。
		`uint iZeroOffset[8] = {0x8000,0x8000,0x8000,0x8000,0x8000,0x8000,0x8000,0x8000};`
- iWaveTable
		`extern int iWaveTable[TABLE_SIZE];`
一个数组，大小为2^14w，用于存放波形数据。
- lTableIndex
		`ulong lTableIndex[8];`
只有清零的操作，有的地方还注释了，估计是被弃用了。
- iSpiRecv
串口数据缓存，取用SPIRXBUF的低8位数据，用于传送控制命令。
- iBank 
stWave[]的指针，iBank以8为单位增减，估计是以8路信号为一组。
- szRxBuffer
		`extern char szRxBuffer[256];`

## 函数

- sine_table_init，将一个周期的sine数据存放到iWaveTable数组中。
		`void sine_table_init(void)`
- clear_harmonic_table，将谐波数据清除，具体就是将TABLE_PAGE的8-14页数据清零。
		`void clear_harmonic_table(void)`
- clear_all_wave_struct，复位stWave结构体数组的0-16
		`void clear_all_wave_struct(void)`
- clear_all_harmonic_struct，复位谐波结构体数组。
		`void clear_all_harmonic_struct()`
- clear_channel，将DAC数据端口的值都复位为零漂值。
		`void clear_channel(void)`
- write_wave_struct，将szRxBuffer的内容写入stWave的一个成员,就是一次只修改一个成员。
		`void write_wave_struct(void)`
- write_harmonic_struct，将szRxBuffer的内容写入stHarmonic的一个成员,就是一次只修改一个成员。
		`void write_harmonic_struct(void)`
- write_zero_offset，将szRxBuffer的内容写入iZeroOffset的所有成员。
		`void write_zero_offset(void)`




