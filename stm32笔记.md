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

