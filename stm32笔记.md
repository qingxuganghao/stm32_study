# 2021/8/20 第十五节RCC—使用HSE/HSI配置时钟
* **RCC ：reset clock control 复位和时钟控制器。本章我们主要讲解时钟部分，特别是要着重理解时钟树，理解了时钟树，STM32 的一切时钟的来龙去脉都会了如指掌.**
## 一.RCC 主要作用—时钟部分

## 二.**RCC** 框图剖析—时钟部分







*****

# 2021/8/20 第十四节 启动文件详解

## 一.启动文件简介

* 启动文件由汇编编写，是系统上电复位后第一个执行的程序。主要做了以下工作：

1. 初始化堆栈指针SP=_initial_sp.

2. 初始化 PC 指针 =Reset_Handler.

3. 初始化中断向量表

4. 配置系统时钟. 

5. 调用 C 库函数 _main 初始化用户堆栈，从而最终调用 main 函数去到 C 的世界

## 二.查找ARM汇编指令
* 在讲解启动代码的时候，会涉及到 ARM 的汇编指令和 Cortex 内核的指令，有关 Cortex 内核的指
令我们可以参考《CM3 权威指南 CnR2》第四章：指令集。剩下的 ARM 的汇编指令我们可以在
MDK->Help->Uvision Help 中搜索到.
## 三.启动文件代码讲解
1. Stack—栈
* 开辟栈的大小为 0X00000400（1KB），名字为 STACK，NOINIT 即不初始化，可读可8（2^3）字节对齐。栈的作用是用于局部变量，函数调用，函数形参等的开销，栈的大小不能超过内部SRAM 的大小。如果编写的程序比较大，定义的局部变量很多，那么就需要修改栈的大小。如果某一天你写的程序出现了莫名奇怪的错误，并进入了硬 fault 的时候，这时你就要考虑下是不是栈不够大，溢出了。

* EQU：宏定义的伪指令，相当于等于，类似与 C 中的 define。

* AREA：告诉汇编器汇编一个新的代码段或者数据段。STACK 表示段名，这个可以任意命名；NOINIT 表示不初始化；READWRITE 表示可读可写，ALIGN=3，表示按照 2^3 对齐，即 8 字节
  对齐.

* SPACE：用于分配一定大小的内存空间，单位为字节。这里指定大小等于 Stack_Size。
  标号 __initial_sp 紧挨着 SPACE 语句放置，表示栈的结束地址，即栈顶地址，栈是由高向低生长
  的。

2. Heap 堆
* 开辟堆的大小为 0X00000200（512 字节），名字为 HEAP，NOINIT 即不初始化，可读可写，8（2^3）字节对齐。__heap_base 表示对的起始地址，__heap_limit 表示堆的结束地址。堆是由低向高生长的，跟栈的生长方向相反。堆主要用来动态内存的分配，像 malloc() 函数申请的内存就在堆上面。这个在 STM32 里面用的比较少.
* PRESERVE8：指定当前文件的堆栈按照 8 字节对齐。
* THUMB：表示后面指令兼容 THUMB 指令。THUBM 是 ARM 以前的指令集，16bit，现在 Cortex-M系列的都使用 THUMB-2 指令集，THUMB-2 是 32 位的，兼容 16 位和 32 位的指令，是 THUMB的超集.
3. 向量表
* 定义一个数据段，名字为 RESET，可读。并声明 __Vectors、__Vectors_End 和 __Vectors_Size 这三个标号具有全局属性，可供外部的文件调用。
* EXPORT：声明一个标号可被外部的文件使用，使标号具有全局属性。如果是 IAR 编译器，则使
用的是 GLOBAL 这个指令。
* DCD：分配一个或者多个以字为单位的内存，以四字节对齐，并要求初始化这些内存。在向量表
中，DCD 分配了一堆内存，并且以 ESR 的入口地址初始化它们。
4. 复位程序
* 复位子程序是系统上电后第一个执行的程序，调用 SystemInit 函数初始化系统时钟，然后调用C库函数 _mian，最终调用 main 函数去到 C 的世界。
5. 中断服务程序
* B：跳转到一个标号。这里跳转到一个‘.’，即表示无线循环。
6. 用户堆栈初始化
* ALIGN：对指令或者数据存放的地址进行对齐，后面会跟一个立即数。缺省表示 4 字节对齐。

*****

# 2021/8/14 第十三节 位带操作——GPIO输出和输入

## 一.位带简介 

* 位操作就是可以单独的对一个比特位读和写,在51单片机中用sbit来实现位操作；
* 在stm32中通过位带操作来实现，在 STM32 中，有两个地方实现了位带，一个是 **SRAM 区的最低 1MB 空间**，令一个是**外设区最低 1MB 空间**。这两个 1MB 的空间除了可以像正常的 RAM 一样操作外，他们还有自己的位带别名区，位带别名区把这 **1MB 的空间的每一个位膨胀成一个 32 位的字**，当访问位带别名区的这些字时，就可以达到访问位带区某个比特位的目的。
### 1.1外设位带区

* 外 设 外 带 区 的 地 址 为：0X40000000~0X40100000， 大 小 为 1MB， 这 1MB 的 大 小 在

  103 系 列 大/中/小 容 量 型 号 的 单 片 机 中 包 含 了 片 上 外 设 的 全 部 寄 存 器.

### 1.2SRAM 位带区
* SRAM 的位带区的地址为：0X2000 0000~X2010 0000，大小为 1MB，经过膨胀后的位带别名区
地址为：0X2200 0000~0X23FF FFFF，大小为 32MB。操作 SRAM 的比特位这个用得很少。
### 1.3位带区和位带别名区地址转换
* 可以通过指针的形式访问位带别名区地址从而达到操作位带区比特位的效果。这两个地
址直接转换。
* 为了方便操作，我们可以把这两个公式合并成一个公式，把“位带地址 + 位序号”转换成别名区
地址统一成一个宏。
```c
// 把“位带地址 + 位序号”转换成别名地址的宏
#define BITBAND(addr, bitnum) ((addr & 0xF0000000)+0x02000000+((addr &␣ ,→0x00FFFFFF)<<5)+(bitnum<<2))
```
```c
// 把一个地址转换成一个指针
#define MEM_ADDR(addr) *((volatile unsigned long *)(addr))
// 把位带别名区地址转换成指针
#define BIT_ADDR(addr, bitnum) MEM_ADDR(BITBAND(addr, bitnum))
```
### 2.1寄存器映射
```c
// GPIO ODR 和 IDR 寄存器地址映射
#define GPIOA_ODR_Addr (GPIOA_BASE+12) //0x4001080C
#define GPIOB_ODR_Addr (GPIOB_BASE+12) //0x40010C0C
#define GPIOC_ODR_Addr (GPIOC_BASE+12) //0x4001100C
#define GPIOD_ODR_Addr (GPIOD_BASE+12) //0x4001140C
#define GPIOE_ODR_Addr (GPIOE_BASE+12) //0x4001180C
#define GPIOF_ODR_Addr (GPIOF_BASE+12) //0x40011A0C
#define GPIOG_ODR_Addr (GPIOG_BASE+12) //0x40011E0C
#define GPIOA_IDR_Addr (GPIOA_BASE+8) //0x40010808
#define GPIOB_IDR_Addr (GPIOB_BASE+8) //0x40010C08
#define GPIOC_IDR_Addr (GPIOC_BASE+8) //0x40011008
#define GPIOD_IDR_Addr (GPIOD_BASE+8) //0x40011408
#define GPIOE_IDR_Addr (GPIOE_BASE+8) //0x40011808
#define GPIOF_IDR_Addr (GPIOF_BASE+8) //0x40011A08
#define GPIOG_IDR_Addr (GPIOG_BASE+8) //0x40011E08
```
### 2.2位操作
```c
// 单独操作 GPIO 的某一个 IO 口，n(0,1,2...16),n 表示具体是哪一个 IO 口 
#define PAout(n) BIT_ADDR(GPIOA_ODR_Addr,n) //输出
#define PAin(n) BIT_ADDR(GPIOA_IDR_Addr,n) //输入
#define PBout(n) BIT_ADDR(GPIOB_ODR_Addr,n) //输出
#define PBin(n) BIT_ADDR(GPIOB_IDR_Addr,n) //输入
#define PCout(n) BIT_ADDR(GPIOC_ODR_Addr,n) //输出
#define PCin(n) BIT_ADDR(GPIOC_IDR_Addr,n) //输入
#define PDout(n) BIT_ADDR(GPIOD_ODR_Addr,n) //输出
#define PDin(n) BIT_ADDR(GPIOD_IDR_Addr,n) //输入
#define PEout(n) BIT_ADDR(GPIOE_ODR_Addr,n) //输出
#define PEin(n) BIT_ADDR(GPIOE_IDR_Addr,n) //输入
#define PFout(n) BIT_ADDR(GPIOF_ODR_Addr,n) //输出
#define PFin(n) BIT_ADDR(GPIOF_IDR_Addr,n) //输入
#define PGout(n) BIT_ADDR(GPIOG_ODR_Addr,n) //输出
#define PGin(n) BIT_ADDR(GPIOG_IDR_Addr,n) //输入
```
### 2.3主函数
* 该工程我们直接从 LED-库函数操作移植过来，有关 LED GPIO 初始化和软件延时等函数我们直
接用，修改的是控制 GPIO 输出的部分改成了位操作。该实验我们让 IO 口输出高低电平来控制
LED 的亮灭，负逻辑点亮。具体使用哪一个 IO 和点亮方式由硬件平台决定。
```c
int main(void) 
{ // 程序来到 main 函数之前，启动文件：statup_stm32f10x_hd.s 已经调用
// SystemInit() 函数把系统时钟初始化成 72MHZ
// SystemInit() 在 system_stm32f10x.c 中定义
// 如果用户想修改系统时钟，可自行编写程序修改
LED_GPIO_Config();
while ( 1 ) {
		// PB0 = 0, 点亮 LED
		PBout(0)= 0;
		SOFT_Delay(0x0FFFFF);
		// PB1 = 1, 熄灭 LED
		PBout(0)= 1;
		SOFT_Delay(0x0FFFFF);
	}
}
```


*****

# # 2021/8/10 第十二节 GPIO输入——按键检测

* 按键已经带有电容（硬件去抖），不用再软件去抖；

* 按键按下为高电平，上升沿输入；

## 1.1添加新建文件

1. 在工程目录user里添加bsp_key.h和bsp_key.c两个新文件.

2. 在keil5里面添加bsp_key.c.
## 1.2在bsp_key.h里面定义新的宏
1. 添加条件编译的宏
```c
#ifndef __KEY_H
#define	__KEY_H
#endif /* __KEY_H */
```
2. 定义封装硬件相关的代码
```c
//  引脚定义
#define    KEY1_GPIO_CLK     RCC_APB2Periph_GPIOA
#define    KEY1_GPIO_PORT    GPIOA			   
#define    KEY1_GPIO_PIN		 GPIO_Pin_0

#define    KEY2_GPIO_CLK     RCC_APB2Periph_GPIOC
#define    KEY2_GPIO_PORT    GPIOC		   
#define    KEY2_GPIO_PIN		  GPIO_Pin_13
```
3. 声明bsp_key.c里面的函数，使得其可以被其他函数调用。
```c
void Key_GPIO_Config(void);
uint8_t Key_Scan(GPIO_TypeDef* GPIOx,uint16_t GPIO_Pin);
```
## 1.3在bsp_key.c里面写函数
1. 初始化按键
```c
void Key_GPIO_Config(void)
{
	GPIO_InitTypeDef GPIO_InitStructure;
	
	/*开启按键端口的时钟*/
	RCC_APB2PeriphClockCmd(KEY1_GPIO_CLK|KEY2_GPIO_CLK,ENABLE);
	
	//选择按键的引脚
	GPIO_InitStructure.GPIO_Pin = KEY1_GPIO_PIN; 
	// 设置按键的引脚为浮空输入
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IN_FLOATING; 
	//使用结构体初始化按键
	GPIO_Init(KEY1_GPIO_PORT, &GPIO_InitStructure);
	
	//选择按键的引脚
	GPIO_InitStructure.GPIO_Pin = KEY2_GPIO_PIN; 
	//设置按键的引脚为浮空输入
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IN_FLOATING; 
	//使用结构体初始化按键
	GPIO_Init(KEY2_GPIO_PORT, &GPIO_InitStructure);	
}
```
2. 按键读取函数
```c
uint8_t Key_Scan(GPIO_TypeDef* GPIOx,uint16_t GPIO_Pin)
{			
	/*检测是否有按键按下 */
	if(GPIO_ReadInputDataBit(GPIOx,GPIO_Pin) == KEY_ON )  
	{	 
		/*等待按键释放 */
		while(GPIO_ReadInputDataBit(GPIOx,GPIO_Pin) == KEY_ON);   
		return 	KEY_ON;	 
	}
	else
		return KEY_OFF;
}
```
## 1.4在main函数里初始化和编写驱动程序
1. 	初始化端口
```c
int main(void)
{
/* LED端口初始化 */
	LED_GPIO_Config();

	/* 按键端口初始化 */
	Key_GPIO_Config();
}
```
2. 编写按键函数
```c
	/* 轮询按键状态，若按键按下则反转LED */
	while(1)                            
	{	   
		if( Key_Scan(KEY1_GPIO_PORT,KEY1_GPIO_PIN) == KEY_ON  )
		{
			/*LED1反转*/
			LED1_TOGGLE;
		} 

		if( Key_Scan(KEY2_GPIO_PORT,KEY2_GPIO_PIN) == KEY_ON  )
		{
			/*LED2反转*/
			LED2_TOGGLE;
		}		
```
## 1.5 在bsp_led.h 添加反转宏定义
```c
/* 直接操作寄存器的方法控制IO */
#define	digitalHi(p,i)		 {p->BSRR=i;}	 //输出为高电平		
#define digitalLo(p,i)		 {p->BRR=i;}	 //输出低电平
#define digitalToggle(p,i) {p->ODR ^=i;} //输出反转状态


/* 定义控制IO的宏 */
#define LED1_TOGGLE		 digitalToggle(LED1_GPIO_PORT,LED1_GPIO_PIN)
#define LED1_OFF		   digitalHi(LED1_GPIO_PORT,LED1_GPIO_PIN)
#define LED1_ON			   digitalLo(LED1_GPIO_PORT,LED1_GPIO_PIN)

#define LED2_TOGGLE		 digitalToggle(LED2_GPIO_PORT,LED2_GPIO_PIN)
#define LED2_OFF		   digitalHi(LED2_GPIO_PORT,LED2_GPIO_PIN)
#define LED2_ON			   digitalLo(LED2_GPIO_PORT,LED2_GPIO_PIN)

#define LED3_TOGGLE		 digitalToggle(LED3_GPIO_PORT,LED3_GPIO_PIN)
#define LED3_OFF		   digitalHi(LED3_GPIO_PORT,LED3_GPIO_PIN)
#define LED3_ON			   digitalLo(LED3_GPIO_PORT,LED3_GPIO_PIN)

```






*****

# 2021/8/9 更新第十二节笔记

# 2021/8/6 第十二节 使用固件库点亮LED

* 要点：

1. 使能 GPIO 端口时钟；```RCC_APB2PeriphClockCmd( LED_G_GPIO_CLK , ENABLE);```

2. 初始化 GPIO 目标引脚为推挽输出模式；

3. 编写简单测试程序，控制 GPIO 引脚输出高、低电平。

* 在“工程模板”之上新建“bsp_led.c”及“bsp_led.h”文件，其中的“bsp”即 Board Support Packet 的缩写 (板级支持包).

  并在 C/C++ 选项卡中添加“bsp_led.h”的路径.

* 函数执行流程如下：

  (1) 使用 GPIO_InitTypeDef 定义 GPIO 初始化结构体变量，以便下面用于存储 GPIO 配置。

  (2) 调用库函数 RCC_APB2PeriphClockCmd 来使能 LED 灯的 GPIO 端口时钟，在前面的章节中

  我们是直接向 RCC 寄存器赋值来使能时钟的，不如这样直观。该函数有两个输入参数，第

  一个参数用于指示要配置的时钟，如本例中的“RCC_APB2Periph_GPIOB”，应用时我们使用“|”操作同时配置 3 个 LED 灯的时钟；函数的第二个参数用于设置状态，可输入“Disable”关闭或“Enable”使能时钟。

  (3) 向 GPIO 初始化结构体赋值，把引脚初始化成推挽输出模式，其中的 GPIO_Pin 使用宏

  “LEDx_GPIO_PIN”来赋值，使函数的实现方便移植。

  (4) 使用以上初始化结构体的配置，调用 GPIO_Init 函数向寄存器写入参数，完成 GPIO 的初始化，这里的 GPIO 端口使用“LEDx_GPIO_PORT”宏来赋值，也是为了程序移植方便。

  (5) 使用同样的初始化结构体，只修改控制的引脚和端口，初始化其它 LED 灯使用的 GPIO 引脚。

  (6) 使用宏控制 RGB 灯默认关闭。

* 注意：初始化时的顺序也很重要否则会影响灯一开始的变化
```c
/*定义一个GPIO_InitTypeDef类型的结构体*/
		GPIO_InitTypeDef GPIO_InitStructure;

		/*开启LED相关的GPIO外设时钟*/
		RCC_APB2PeriphClockCmd( LED1_GPIO_CLK | LED2_GPIO_CLK | LED3_GPIO_CLK, ENABLE);
		/*选择要控制的GPIO引脚*/
		GPIO_InitStructure.GPIO_Pin = LED1_GPIO_PIN;	

		/*设置引脚模式为通用推挽输出*/
		GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;   

		/*设置引脚速率为50MHz */   
		GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz; 

		/*调用库函数，初始化GPIO*/
		GPIO_Init(LED1_GPIO_PORT, &GPIO_InitStructure);	
		
		/*选择要控制的GPIO引脚*/
		GPIO_InitStructure.GPIO_Pin = LED2_GPIO_PIN;
```


* 延时函数
```c
void Delay( uint32_ count )
{
    for(;count!=0;count--);
}
int main(void)
{
    while(1)
    {
        GPIO_SetBits(LED_G_GPIO_PORT,LED_G_GPIO_PIN);
        Delay(0xFFFFF);
        GPIO_ResetBits(LED_G_GPIO_PORT,LED_G_GPIO_PIN);
        Delay(0xFFFFF);
    }
}
```
* 使能不同的led灯时，模式和速度不用改变，只需要选择不同的GPIO引脚将每个led的GPIO初始化即可

  ``````c
  		/*选择要控制的GPIO引脚*/
  		GPIO_InitStructure.GPIO_Pin = LED2_GPIO_PIN;
  
  		/*调用库函数，初始化GPIO*/
  		GPIO_Init(LED2_GPIO_PORT, &GPIO_InitStructure);
  		
  		/*选择要控制的GPIO引脚*/
  		GPIO_InitStructure.GPIO_Pin = LED3_GPIO_PIN;
  
  		/*调用库函数，初始化GPIOF*/
  		GPIO_Init(LED3_GPIO_PORT, &GPIO_InitStructure);

\：续行，后面不能有任何符号

*****

# 2021/8/4 第十一节 新建工程——库函数版

## 新建工程

### 1.1新建本地工程文件

1. 新建一个工程模板的文件夹，在它之下新建六个文件夹

   * Doc ：用来存放程序说明的文件

   * Libraries ：用来存库文件

   * Listing ：用来存放编译器编译时产生的c列表清单（自动生成）

   * Output：用来存放调试信息，hex文件，封装库等（自动生成）

   * Project：用来存放工程文件

   * User：用户编写的驱动文件
2. 新建好后将库文件添加到相应文件夹下：
   * Doc：readme.txt
   
   * Libraries：1.CMSIS：里面存放CM3内核有关的库文件；               2.STM32F10x_Stdperiph_Driver:STM32外设库文件。
   
   * User：1.stm32f10x_conf.h: 用来配置库的头文件
   
     2.stm32f1.x_it.h    stm32f10x_it.c :中断相关的函数在这个文件李编写
   
     3.main.c：main函数文件

### 1.2新建工程

1. 新建工程

2. 选择CPU型号

3. **添加组文件夹**

   * 在新建的工程中添加 5 个组文件夹，用来存放各种不同的文件：

     1.STARTUP:startup_2f10x_hd.s

     2.CMSIS:core_cm3.c、system_stm32f10x.c

     3.FWLB:STM32F10X_StdPeriph_Driver\src下的全部c文件，即固件库

     4.USER:main.c、stm32f10x_it.c

     5.DOC:README.TXT

4. Target 中选中微库Use MicroLib，为的是在日后编写串口驱动的时候可以使用 printf 函数。

5. 在 Output 选项卡中将Create HEX File 选项勾上。
6. 在 Listing 选项卡中把输出文件夹定位到我们工程目录下的“Listing”文件夹
7. 在 C/C++ 选项卡中添加处理宏及编译器编译的时候查找的头文件路径
8. Debug 中选择 CMSIS-DAP Debugger
9. Utilities 选择 Use Debug Driver


*****


# 2021/8/2 第十节 初识stm32固件库

1-汇编编写的启动文件
startup_stm32f10x_hd.s:设置堆栈指针、设置PC指针、初始化中断向量表、配置系统时钟、对用C库函数_main最终去到C的世界

2-时钟配置文件
system_stm32f10x.c：把外部时钟HSE=8M，经过PLL倍频为72M。

3-外设相关的
stm32f10x.h：实现了内核之外的外设的寄存器映射
xxx：GPIO、USRAT、I2C、SPI、FSMC
stm32f10x_xx.c：外设的驱动函数库文件
stm32f10x_xx.h：存放外设的初始化结构体，外设初始化结构体成员的参数列表，外设固件库函数的声明

4-内核相关的
CMSIS - Cortex 微控制器软件接口标准
core_cm3.h：实现了内核里面外设的寄存器映射
core_cm3.c：内核外设的驱动固件库

NVIC(嵌套向量中断控制器)、SysTick(系统滴答定时器)
misc.h
misc.c

5-头文件的配置文件
stm32f10x_conf.h：头文件的头文件
//stm32f10x_usart.h
//stm32f10x_i2c.h
//stm32f10x_spi.h
//stm32f10x_adc.h
//stm32f10x_fsmc.h
......

6-专门存放中断服务函数的C文件
stm32f10x_it.c
stm32f10x_it.h

中断服务函数你可以随意放在其他的地方，并不是一定要放在stm32f10x_it.c

#include "stm32f10x.h"   // 相当于51单片机中的  #include <reg51.h>

int main(void)
{
	// 来到这里的时候，系统的时钟已经被配置成72M。
}









*****

# 2021/8/2 第九节 总结及如何提高程序的可移植性

## 原理：通过宏定义改变端口，从而只需要修改宏定义的值来改变控制的不同的灯

```c
#define   LED_G_GPIO_PORT                   GPIOB
#define   LED_G_GPIO_CLK_ENABLE            (RCC->APB2ENR  |=  ( (1) << 3 ))
#define   LED_G_GPIO_PIN                    GPIO_Pin_0
```

作用：可以通过修改`GPIO_Pin_0`->`GPIO_Pin_5`来使得控制绿灯转变为控制红灯，通过宏定义不需要修改下面的代码，只需要修改宏定义，使得程序的可移植性更高

*****

# 2021/8/1 第九节 编写GPIO初始化结构体和初始化函数

## 目的：提高函数的可读性和可移植性

1. 使用结构体定义寄存器的三个模式相关属性

   ```c
   typedef struct
   {
   	uint16_t GPIO_Pin;
   	uint16_t GPIO_Speed;
   	uint16_t GPIO_Mode;
   }GPIO_InitTypeDef;
```

2. 使用枚举来定义各个属性的模式(**枚举中使用逗号，结构体使用分号**)

   ```c
   typedef enum
   { 
     GPIO_Speed_10MHz = 1,         // 10MHZ        (01)b
     GPIO_Speed_2MHz,              // 2MHZ         (10)b
     GPIO_Speed_50MHz              // 50MHZ        (11)b
   }GPIOSpeed_TypeDef;
   
   typedef enum
   { GPIO_Mode_AIN = 0x0,           // 模拟输入     (0000 0000)b
     GPIO_Mode_IN_FLOATING = 0x04,  // 浮空输入     (0000 0100)b
     GPIO_Mode_IPD = 0x28,          // 下拉输入     (0010 1000)b
     GPIO_Mode_IPU = 0x48,          // 上拉输入     (0100 1000)b
     
     GPIO_Mode_Out_OD = 0x14,       // 开漏输出     (0001 0100)b
     GPIO_Mode_Out_PP = 0x10,       // 推挽输出     (0001 0000)b
     GPIO_Mode_AF_OD = 0x1C,        // 复用开漏输出 (0001 1100)b
     GPIO_Mode_AF_PP = 0x18         // 复用推挽输出 (0001 1000)b
   }GPIOMode_TypeDef;
```

3. 声明调用的官方函数

   ```c
   void GPIO_Init(GPIO_TypeDef* GPIOx, GPIO_InitTypeDef* GPIO_InitStruct);
   ```

4. 在main函数中使用该初始化函数时，先进行声明

   ```c
   GPIO_InitTypeDef  GPIO_InitStructure;
   ```

5. 然后便可以直接使用该函数进行赋值

   ```c
   	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0;
   	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
   	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
   	GPIO_Init(GPIOB, &GPIO_InitStructure);	
   ```



*****
# 2021/7/31 第九节 编写端口复位置位函数

* **在有多个函数调用时，在头文件里面添加**
```c
#ifndef _STM32F10X_H
#define _STM32F10X_H


#endif/*_STM32F10X_H*/
```

**用来防止同一个函数被调用多次而报错.**

* ## 编写端口复位置位函数

1. 在工程文件中添加*stm32f10x.gpio.c* 和*stm32f10x.gpio.h*文件.

2. 添加进Q5工程中.

3. 在*stm32f10x.gpio.c*中编写置位复位函数

   * ***即用宏定义端口复位置位来代替指针，使得操作更加简便***.

   * *stm32f10x.gpio.c*
     
     ```c
     #include "stm32f10x_gpio.h"
     
     void GPIO_SetBits(GPIO_TypeDef *GPIOx,uint16_t GPIO_Pin)
     {
     	GPIOx->BSRR |= GPIO_Pin;
     }
     
     void GPIO_ResetBits( GPIO_TypeDef *GPIOx,uint16_t GPIO_Pin )
     {
     	GPIOx->BRR |= GPIO_Pin;
     }
     ```
     
   * *stm32f10x.gpio.h*
    ```c
    #ifndef __STM32F10X_GPIO_H
    #define __STM32F10X_GPIO_H
    
    #include "stm32f10x.h"
    
    #define GPIO_Pin_0    ((uint16_t)0x0001)  /*!< 选择Pin0 */    //(00000000 00000001)b
    #define GPIO_Pin_1    ((uint16_t)0x0002)  /*!< 选择Pin1 */    //(00000000 00000010)b
    #define GPIO_Pin_2    ((uint16_t)0x0004)  /*!< 选择Pin2 */    //(00000000 00000100)b
    #define GPIO_Pin_3    ((uint16_t)0x0008)  /*!< 选择Pin3 */    //(00000000 00001000)b
    #define GPIO_Pin_4    ((uint16_t)0x0010)  /*!< 选择Pin4 */    //(00000000 00010000)b
    #define GPIO_Pin_5    ((uint16_t)0x0020)  /*!< 选择Pin5 */    //(00000000 00100000)b
    #define GPIO_Pin_6    ((uint16_t)0x0040)  /*!< 选择Pin6 */    //(00000000 01000000)b
    #define GPIO_Pin_7    ((uint16_t)0x0080)  /*!< 选择Pin7 */    //(00000000 10000000)b
    
    #define GPIO_Pin_8    ((uint16_t)0x0100)  /*!< 选择Pin8 */    //(00000001 00000000)b
    #define GPIO_Pin_9    ((uint16_t)0x0200)  /*!< 选择Pin9 */    //(00000010 00000000)b
    #define GPIO_Pin_10   ((uint16_t)0x0400)  /*!< 选择Pin10 */   //(00000100 00000000)b
    #define GPIO_Pin_11   ((uint16_t)0x0800)  /*!< 选择Pin11 */   //(00001000 00000000)b
    #define GPIO_Pin_12   ((uint16_t)0x1000)  /*!< 选择Pin12 */   //(00010000 00000000)b
    #define GPIO_Pin_13   ((uint16_t)0x2000)  /*!< 选择Pin13 */   //(00100000 00000000)b
    #define GPIO_Pin_14   ((uint16_t)0x4000)  /*!< 选择Pin14 */   //(01000000 00000000)b
    #define GPIO_Pin_15   ((uint16_t)0x8000)  /*!< 选择Pin15 */   //(10000000 00000000)b
    #define GPIO_Pin_All  ((uint16_t)0xFFFF)  /*!< 选择全部引脚*/ //(11111111 11111111)b
    
    void GPIO_SetBits(GPIO_TypeDef *GPIOx,uint16_t GPIO_Pin);
    void GPIO_ResetBits( GPIO_TypeDef *GPIOx,uint16_t GPIO_Pin );
    
    
    #endif /* __STM32F10X_GPIO_H */
    

* ### 于是在main函数中可以用

```c
GPIO_SetBits(GPIOB,GPIO_Pin_0);
GPIO_ResetBits( GPIOB,GPIO_Pin_0 );
```

**来代替指针位移置位复位**



*****

# 2021/7/29第九节 自己写库——创建库函数雏形

## 知识点：通过结构体定义实现寄存器映射

```c
typedef unsigned int           uint32_t;


typedef unsigned short       uint16_t;

typedef struct

{

uint32_t CRL;

uint32_t CRH;

uint32_t IDR;

uint32_t ODR;

uint32_t BSRR;

uint32_t BRR;

uint32_t LCKR;

}GPIO_TypeDef;

#define GPIOB ((GPIO_TypeDef*)GPIOB_BASE)
```


引用寄存器：

```c
GPIOB->CRL
```

*****

# 2021/7/28第八节寄存器映射代码讲解

* __将寄存器映射写在stm32f10x.h里面__

* __用#define 来映射__

 一、总线基地址

```c
#define PERIPH_BASE   ((unsigned int)0x40000000)

#define APB1PERIPH_BASE  PERIPH_BASE

#define APB2PERIPH_BASE  (PERIPH_BASE+0x10000)

#define AHBPERIPH_BASE  (PERIPH_BASE+0x20000)
```

二、寄存器基地址

```c
#define RCC_BASE   (AHBPERIPH_BASE+ 0x1000)

#define GPIOB_BASE (APB2PERIPH_BASE+ 0x0c00)
```


 三、具体寄存器绝对地址
```c
#define RCC_APB2ENR  *(unsigned int *)(RCC_BASE+ 0x18)

#define GPIOB_CRL  *(unsigned int *)(GPIOB_BASE+ 0x00)

#define GPIOB_CRH  *(unsigned int *)(GPIOB_BASE+ 0x04)

#define GPIOB_ODR  *(unsigned int *)(GPIOB_BASE+ 0x0C)
```
*****

# 2021/7/23 第七节点亮led—— 跟51单片机一样写代码

一、` #if 0                #endif`   ——可以使某部分代码不编译

二、端口对应地址算法：

如用GPIOB的基地址加上端口输出寄存器的地址偏移；（I/O口是挂载到APB2高速总线上的，所以GPIOB在APB2的基地址里面找）

三、端口对应地址操作时，需要强制类型转换      添加*( unsigned int *)   

四、程序步骤

1.打开GPIOB端口的时钟` *(unsigned int *)0x40021018 |= ((1)<<3);`
    
2.配置IO口为输出 `*(unsigned int  * )0x40010C00 |= ( (1)<<(4*0) ) ;`
    
3.控制ODR寄存器` *(unsigned int  *)0x40010C0C &=~(1<<0);`

(每个寄存器或者端口的地址都可以在用户手册中找到，用其基地址加上对应寄存器的偏移)

#2021/7/22 第七节新建工程模板

一.Q5的使用 

1.使用Q5，先选择对应的单片机型号.

2.添加启动文件，以hd.s结尾的文件（在固件库将该文件复制到工程目录下）

3.新建main.c文件

二.代码的编译

1.`#include"stm32f10x.h"`（如果没有去工程目录下创建）（用于实现寄存器映射）

2.`int main（void）`

3.`void SystemInit(void)`（引用固件库）

三、烧录

1.在output勾上生成hex文件（默认生成在object）

2.开发板上电.

3.选择dap，勾上bebug.

4.烧录

*****

# 2021/7/20 第六节下 ——寄存器映射

一、1.三条总线及其基地址：APB1-基地址0x4000 0000   APB2-0x4001 0000   AHB-0x4001 8000（根据速度划分）

 2.先找到总线的基地址，由基地址加上某个寄存器外设的偏移地址，可以找到某个寄存器外设的基地址。（总线基地址+寄存器外设偏移地址=外设基地址）

 3.由寄存器相对于外设的基地址，可以算出外设绝对的地址.然后就可以通过c语言指针访问某个寄存器.

二、c语言对寄存器的封装

1.先对总线和外设基地址宏定义

（1）先定义外设基地址（外设的地址就等于APB1的地址，从APB1开始）（2）定义总线基地址（外设基地址加其偏移）（3）定义GPIO外设基地址（APB2+ABCDEFG的对应的偏移地址）（4）具体某个外设寄存器绝对地址（  GPIO基地址加上具体寄存器对应偏移）

2.用指针操作`* (unsigned int*)`

3.`GPIOB_ODR &= ~(1<<0)//让PB0输出低电平；（取反相与实现清零）`

`GPIO_ODR |= (1<<0)//让PB0输出高电平；（置1相加实现高电平）`

（1<<0左移0位——则为PB0；“|”相或 ； "~"取反；"&"相与）

三、使用结构体封装寄存器列表或使用结构体指针访问寄存器

*****

# 2021/7/18 第六节——寄存器

1.看丝印，辨别芯片正正方向

2.stm32系统框图——内部驱动单元cpu

四个被动单元：内部SRAM用来存取数据； flash用来存程序；FSMC特殊外设用来驱动外存；AHB——APB系统总线。

3总线： ICode——程序执行总线；DCode——数据总线；DMA——也可以读取数据；System——用来读取外设寄存器.

4.可以用const将数据常量放到内部flash中。

5.变量都放在内部SRAM中；

6.总线矩阵——当DMA和Dcode读取发生冲突时，仲裁.

7.DMA主要用来搬数据，可以让SRAM里面的数据不通过cpu直接搬到外设里面，这样可以加快速率不占cpu。

8.AHB上挂载有SDIO和RCC（复位和时钟控制）

9.APB分为APB1（高速总线）（有GPIO）和APB2（低速总线）（有定时器和串口）.

10.ARM为32位——4G内存，分为了8块，每一块512M。block0用来放flash；block1放SRAM；block2放外设寄存器；block3.4放FSMC；block5时FSMC寄存器，block6没有用；block7放内核的寄存器外设。

11. c语言里面*代表取地址。
12. (unsigned int*)——强制类型转换为地址.
13. STM32和51不同——没有<reg32.h>需要自己找寄存器对应地址.只能通过指针来操作.

 
