今天学了第六节——寄存器

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

![img](file:///C:\Users\76170\AppData\Local\Temp\ksohtml16160\wps1.jpg) 

\11. c语言里面*代表取地址。

\12. (unsigned int*)——强制类型转换为地址.

\13. STM32和51不同——没有<reg32.h>需要自己找寄存器对应地址.只能通过指针来操作.