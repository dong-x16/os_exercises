# lab2 SPOC思考题

NOTICE
- 有"w4l1"标记的题是助教要提交到学堂在线上的。
- 有"w4l1"和"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的git repo上。
- 有"hard"标记的题有一定难度，鼓励实现。
- 有"easy"标记的题很容易实现，鼓励实现。
- 有"midd"标记的题是一般水平，鼓励实现。

## 个人思考题
---

x86保护模式中权限管理无处不在，下面哪些时候要检查访问权限(A B C)  (w4l1)
- [x] 内存寻址过程中
- [x] 代码跳转过程中
- [x] 中断处理过程中
- [ ] ALU计算过程中
 
> 前三个需要。这里假定ALU完成计算所需数据都已经在CPU内部了。


请描述ucore OS建立页机制的准备工作包括哪些步骤？ (w4l1) 
```
  + 采分点：说明了ucore OS在让页机制正常工作的主要准备工作
  - 答案没有涉及如下3点；（0分）
  - 描述了对GDT的初始化,完成了段机制（1分）
  - 除第二点外进一步描述了对物理内存的探测和空闲物理内存的管理。（2分）
  - 除上述两点外，进一步描述了页表建立初始过程和设置CR0控寄存器某位来使能页（3分）

 ```
- [x]  

> 答：(1).初始化GDT，完成段机制：由于ucore OS是基于80386 CPU实现的，所以当CPU进入保护模式时，就直接使能了段机制；在lab1的介绍
中可知，在ucore中段式管理只起到了一个过渡作用，但为了建立页机制，首先要初始化GDT并建立段机制；
      (2).探测系统物理内存布局，管理空闲物理内存：ucore被启动后，需要知道还有多少内存可用。在ucore中，通过e820h中断获取内存信
息，又因e820h中断只能在实模式下使用，所以要在bootloader进入保护模式前调用该BIOS中断，并把e820映射结构保存在物理地址0x8000处；
这样可以通过BIOS将内存布局entry放入到一个保存地址范围描述符的缓冲区中；
      (3).建立一一映射关系的二级页表：ucore的页式管理通过一个二级的页表实现，二级页表结构中，页目录占4KB空间，页表的空间大小
取决于页表要管理的物理页数n，一个页表项可管理一个物理页；
      (4).使能分页机制：使能页表结构主要通过enable_paging函数实现，通过Icr3指令把页目录表的起始地址存入CR3寄存器中，通过Icr0
指令把cr0的CR0_PG标志位设置为1，执行完毕后，ucore就进入了分页模式。
      

---

## 小组思考题
---

（1）（spoc）请用lab1实验的基准代码（即没有修改的需要填空的源代码）来做如下实验： 执行`make qemu`，会得到一个输出结果，请给出合理的解释：为何qemu退出了？【提示】需要对qemu增加一些用于基于执行过的参数，重点是分析其执行的指令和产生的中断或异常。 

- [x]  

> 答：lab1中开启A20时需要屏幕中断，即中断还未使能，当出现时钟中断(100 ticks)时就会导致qemu退出

（2）(spoc)假定你已经完成了lab1的实验,接下来是对lab1的中断处理的回顾：请把你的学号对37(十进制)取模，得到一个数x（x的范围是-1<x<37），然后在你的答案的基础上，修init.c中的kern_init函数，在大约36行处，即

```
    intr_enable();              // enable irq interrupt
```
语句之后，加入如下语句(把x替换为你学号 mod 37得的值)：
```
    asm volatile ("int $x");
```    
然后，请回答加入这条语句后，执行`make qemu`的输出结果与你没有加入这条语句后执行`make qemu`的输出结果的差异，并解释为什么有差异或没差异？ 

- [x]  

> 答：还未完成lab1中断处理部分><，写完后会把该题补上T T

（3）对于lab2的输出信息，请说明数字的含义
```
e820map:
  memory: 0009fc00, [00000000, 0009fbff], type = 1.
  memory: 00000400, [0009fc00, 0009ffff], type = 2.
  memory: 00010000, [000f0000, 000fffff], type = 2.
  memory: 07ee0000, [00100000, 07fdffff], type = 1.
  memory: 00020000, [07fe0000, 07ffffff], type = 2.
  memory: 00040000, [fffc0000, ffffffff], type = 2.
```
修改lab2，让其显示` type="some string"` 让人能够读懂，而不是不好理解的数字1,2  (easy) 
- [x]  

> 答：阅读pmm.c的page_init函数，根据如下cprintf的信息可知，打印的分别是该memory的大小、起始地址、结束地址、类型

```
cprintf("  memory: %08llx, [%08llx, %08llx], type = %d.\n", memmap->map[i].size, begin, end - 1, memmap->map[i].type);
```

（4）(spoc)有一台只有页机制的简化80386的32bit计算机，有地址范围位0~256MB的物理内存空间（physical memory），可表示大小为256MB，范围为0xC0000000~0xD0000000的虚拟地址空间（virtual address space）,页大小（page size）为4KB，采用二级页表，一个页目录项（page directory entry ，PDE）大小为4B,一个页表项（page-table entries PTEs）大小为4B，1个页目录表大小为4KB，1个页表大小为4KB。
```
PTE格式（32 bit） :
  PFN19 ... PFN0|NOUSE9 ... NOUSE0|WRITABLE|VALID
PDE格式（32 bit） :
  PT19 ... PT0|NOUSE9 ... NOUSE0|WRITABLE|VALID
 
其中：
NOUSE9 ... NOUSE0为保留位，要求固定为0
WRITABLE：1表示可写，0表示只读
VLAID：1表示有效，0表示无效
```

假设ucore OS已经为此机器设置好了针对如下虚拟地址<-->物理地址映射的二级页表，设置了页目录基址寄存器（page directory base register，PDBR）保存了页目录表的物理地址（按页对齐），其值为0。已经建立好了从物理地址0x1000~0x41000的二级页表，且页目录表的index为0x300~0x363的页目录项的(PT19 ... PT0)的值=(index-0x300+1)。
请写出一个translation程序（可基于python, ruby, C, C++，LISP等），输入是一个虚拟地址和一个物理地址，能够自动计算出对应的页目录项的index值,页目录项内容的值，页表项的index值，页表项内容的值。即(pde_idx, pde_ctx, pte_idx, pte_cxt)

请用如下值来验证你写的程序的正确性：
```
va 0xc2265b1f, pa 0x0d8f1b1f
va 0xcc386bbc, pa 0x0414cbbc
va 0xc7ed4d57, pa 0x07311d57
va 0xca6cecc0, pa 0x0c9e9cc0
va 0xc18072e8, pa 0x007412e8
va 0xcd5f4b3a, pa 0x06ec9b3a
va 0xcc324c99, pa 0x0008ac99
va 0xc7204e52, pa 0x0b8b6e52
va 0xc3a90293, pa 0x0f1fd293
va 0xce6c3f32, pa 0x0f1fd293
```

参考的输出格式为：
```
va 0xcd82c07c, pa 0x0c20907c, pde_idx 0x00000336, pde_ctx  0x00037003, pte_idx 0x0000002c, pte_ctx  0x0000c20b
```

- [x]  

> 答：代码如下所示，输出结果如下：

```
C:\Windows\system32\cmd.exe /c vir_phy.exe
va 0xcd82c07c, pa 0x0c20907c, pde_idx 0x00000336, pde_ctx 0x00037003, pte_idx 0x0000002c, pte_ctx 0x3082400003
va 0xc2265b1f, pa 0x0d8f1b1f, pde_idx 0x00000308, pde_ctx 0x00009003, pte_idx 0x00000265, pte_ctx 0x363c400003
va 0xcc386bbc, pa 0x0414cbbc, pde_idx 0x00000330, pde_ctx 0x00031003, pte_idx 0x00000386, pte_ctx 0x1053000003
va 0xc7ed4d57, pa 0x07311d57, pde_idx 0x0000031f, pde_ctx 0x00020003, pte_idx 0x000002d4, pte_ctx 0x1cc4400003
va 0xca6cecc0, pa 0x0c9e9cc0, pde_idx 0x00000329, pde_ctx 0x0002a003, pte_idx 0x000002ce, pte_ctx 0x327a400003
va 0xc18072e8, pa 0x007412e8, pde_idx 0x00000306, pde_ctx 0x00007003, pte_idx 0x00000007, pte_ctx 0x1d0400003
va 0xcd5f4b3a, pa 0x06ec9b3a, pde_idx 0x00000335, pde_ctx 0x00036003, pte_idx 0x000001f4, pte_ctx 0x1bb2400003
va 0xcc324c99, pa 0x0008ac99, pde_idx 0x00000330, pde_ctx 0x00031003, pte_idx 0x00000324, pte_ctx 0x22800003
va 0xc7204e52, pa 0x0b8b6e52, pde_idx 0x0000031c, pde_ctx 0x0001d003, pte_idx 0x00000204, pte_ctx 0x2e2d800003
va 0xc3a90293, pa 0x0f1fd293, pde_idx 0x0000030e, pde_ctx 0x0000f003, pte_idx 0x00000290, pte_ctx 0x3c7f400003
va 0xce6c3f32, pa 0x0f1fd293, pde_idx 0x00000339, pde_ctx 0x0003a003, pte_idx 0x000002c3, pte_ctx 0x3c7f400003
Hit any key to close this window...

```

```
#include <iostream>
#include <cstdlib>
#include <cstdio>
#include <cstring>
#include<iomanip>

using namespace std; 

int main() {
    long long va[11] = {0xcd82c07c, 0xc2265b1f, 0xcc386bbc, 0xc7ed4d57, 0xca6cecc0, 0xc18072e8, 0xcd5f4b3a, 0xcc324c99, 0xc7204e52, 0xc3a90293, 0xce6c3f32};
    long long pa[11] = {0x0c20907c, 0x0d8f1b1f, 0x0414cbbc, 0x07311d57, 0x0c9e9cc0, 0x007412e8, 0x06ec9b3a, 0x0008ac99, 0x0b8b6e52, 0x0f1fd293, 0x0f1fd293};

    long long pde_idx = 0;
    long long pde_ctx = 0;
    long long pte_idx = 0;
    long long pte_ctx = 0;
    for (int i=0; i<11; i++) {
	pde_idx = (va[i] >> 22);
	pde_ctx = (((pde_idx - 0x300) + 1) << 12) + 3;
	pte_idx = (va[i] >> 12) & 0x3FF;
	pte_ctx = ((pa[i] >> 12) << 22) + 3;
	cout << hex << "va 0x" << setfill('0') << setw(8) << va[i] << ", pa 0x" << setfill('0') << setw(8) << pa[i] << ", pde_idx 0x" << setfill('0') << setw(8) << pde_idx << ", pde_ctx 0x" << setfill('0') << setw(8) << pde_ctx << ", pte_idx 0x" << setfill('0') << setw(8) << pte_idx << ", pte_ctx 0x" << setfill('0') << setw(8) << pte_ctx << endl;
    }
    return 0;
}
```

---

## 开放思考题

---

（1）请简要分析Intel的x64 64bit体系结构下的分页机制是如何实现的 
```
  + 采分点：说明Intel x64架构的分页机制的大致特点和页表执行过程
  - 答案没有涉及如下3点；（0分）
  - 正确描述了x64支持的物理内存大小限制（1分）
  - 正确描述了x64下的多级页表的级数和多级页表的结构（2分）
  - 除上述两点外，进一步描述了在多级页表下的虚拟地址-->物理地址的映射过程（3分）
 ```
- [x]  

>  

（2）Intel8086不支持页机制，但有hacker设计过包含未做任何改动的8086CPU的分页系统。猜想一下，hacker是如何做到这一点的？提示：想想MMU的逻辑位置

- [x]  

> 

---
