# lab0 SPOC思考题

## 个人思考题

---

能否读懂ucore中的AT&T格式的X86-32汇编语言？请列出你不理解的汇编语言。
- [x]  

> 基本可以读懂，不过部分命令源寄存器与目的寄存器记不清楚，需要查找一下指令集；部分命令不熟悉具体含义，需要根据指令集对应理解。

虽然学过计算机原理和x86汇编（根据THU-CS的课程设置），但对ucore中涉及的哪些硬件设计或功能细节不够了解？
- [x]  

> 部分计算机原理的知识对具体工作的细节掌握的不牢固，比如对中断的工作原理、页表结构、物理内存与虚拟内存的联系
等，但是不太清楚计算机内的具体实现方式，可能会对代码的理解有所影响。


哪些困难（请分优先级）会阻碍你自主完成lab实验？
- [x]  

> 1.对Linux系统不太熟悉，试验中列出的基本工具大多数之前未使用过，在开始可能会操作比较困难
> 2.对计算机的硬件知识不熟悉，可能会影响代码理解
> 3.使用汇编代码或C语言去实现可能会有难度，想到的与实现的不同等

如何把一个在gdb中或执行过程中出现的物理/线性地址与你写的代码源码位置对应起来？
- [x]  

> break加行号可以得到物理地址；list加*物理地址可以得到行号
> 也可以使用一些工具来获取：nm、objdump等

了解函数调用栈对lab实验有何帮助？
- [x]  

> 通过函数调用栈可以比较清楚的了解函数之间的调用关系，这样能更好的去了解程序是如何运行的，同时方便自己添加新的功能点，如果出错了也能较快的进行调试，有助于lab实验的进行。

你希望从lab中学到什么知识？
- [x]  

> 我希望能通过8个lab实验能加深对计算机工作原理的理解，对计算机组成原理里提到的理论知识能在实验过程中明白它们
在计算机中的实际应用，通过逐步完成8个lab实验，希望能够掌握操作系统的运行原理，为编写操作系统打好基础。

---

## 小组讨论题

---

搭建好实验环境，请描述碰到的困难和解决的过程。
- [x]  

> 实验环境现在利用VirtualBox+虚拟硬盘的方式来搭建，由于之前使用过VMware虚拟机，所以在搭建实验环境方面基本没有
遇到问题。不过在跟着2.7实验视频操作的时候，遇到过一些问题，比如终止qemu时卡死等，不过重启虚拟机，在shell里重新
尝试就可以了。

熟悉基本的git命令行操作命令，从github上的 http://www.github.com/chyyuu/ucore_lab 下载ucore lab实验
- [x]  

> 结合视频进行了操作

尝试用qemu+gdb（or ECLIPSE-CDT）调试lab1
- [x]   

> 使用make clean命令清除文件夹，接着使用make命令编译，使用make qemu运行，也可使用make debug调出debug命令行

对于如下的代码段，请说明”：“后面的数字是什么含义
```
 /* Gate descriptors for interrupts and traps */
 struct gatedesc {
    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment
    unsigned gd_ss : 16;            // segment selector
    unsigned gd_args : 5;            // # args, 0 for interrupt/trap gates
    unsigned gd_rsv1 : 3;            // reserved(should be zero I guess)
    unsigned gd_type : 4;            // type(STS_{TG,IG32,TG32})
    unsigned gd_s : 1;                // must be 0 (system)
    unsigned gd_dpl : 2;            // descriptor(meaning new) privilege level
    unsigned gd_p : 1;                // Present
    unsigned gd_off_31_16 : 16;        // high bits of offset in segment
 };
 ```

- [x]  

> 数字代表前面变量的位数

对于如下的代码段，
```
#define SETGATE(gate, istrap, sel, off, dpl) {             \
    (gate).gd_off_15_0 = (uint32_t)(off) & 0xffff;         \
    (gate).gd_ss = (sel);                                  \
    (gate).gd_args = 0;                                    \
    (gate).gd_rsv1 = 0;                                    \
    (gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;       \
    (gate).gd_s = 0;                                       \
    (gate).gd_dpl = (dpl);                                 \
    (gate).gd_p = 1;                                       \
    (gate).gd_off_31_16 = (uint32_t)(off) >> 16;           \
}
```
如果在其他代码段中有如下语句，
```
unsigned intr;
intr=8;
SETGATE(intr, 0,1,2,3);
```
请问执行上述指令后， intr的值是多少？

- [x] 

> intr的值是65538

请分析 [list.h](https://github.com/chyyuu/ucore_lab/blob/master/labcodes/lab2/libs/list.h)内容中大致的含义，并能include这个文件，利用其结构和功能编写一个数据结构链表操作的小C程序
- [x]  

> list_entry_t 是一个双向链表的结构体，结构体中包括它之前的是谁prev和它之后的是谁next；它的功能包括初始化、添加（包括before和after）、删除、返回list的前一个是谁、后一个是谁等基本功能。

> C程序如下所示：
```
#include <stdio.h>
#include <iostream.h>
using namespace std;
int main() {
    list_entry_t point0;
    list_init(&point0); //初始化
    cout << point0.prev << " " << &point0 << " " << point0.next << endl;
    list_entry_t point1, point2;
    point0.prev = NULL;
    point0.next = &point1;
    point1.prev = &point0;
    point1.next = &point2;
    point2.prev = &point1;
    point2.next = NULL;
    cout << list_prev(&point1) << endl;
    cout << list_next(&point1) << endl;
    return 0;
}
```

---

## 开放思考题

---

是否愿意挑战大实验（大实验内容来源于你的想法或老师列好的题目，需要与老师协商确定，需完成基本lab，但可不参加闭卷考试），如果有，可直接给老师email或课后面谈。
- [x]  

> 由于之前基础薄弱，参加大实验可能会比较困难。

---
