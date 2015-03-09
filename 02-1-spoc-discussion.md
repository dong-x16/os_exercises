#lec 3 SPOC Discussion

## 第三讲 启动、中断、异常和系统调用-思考题

## 3.1 BIOS
 1. 比较UEFI和BIOS的区别。
 1. 描述PXE的大致启动流程。

## 3.2 系统启动流程
 1. 了解NTLDR的启动流程。
 1. 了解GRUB的启动流程。
 1. 比较NTLDR和GRUB的功能有差异。
 1. 了解u-boot的功能。

## 3.3 中断、异常和系统调用比较
 1. 举例说明Linux中有哪些中断，哪些异常？
 
 1. Linux的系统调用有哪些？大致的功能分类有哪些？  (w2l1)
 
答：Linux的系统调用函数2.4.4版内核系统调用函数狭义上看大约有220个，不同版本略有不同，通用的系统函数约300多个；
系统函数按功能大致可分为八大类功能，列举如下：
(1).进程控制：如fork、clone等
(2).文件系统控制：如open、close等
(3).系统控制：如time、reboot等
(4).内存管理：如brk、mmap等
(5).网络管理：如gethostid、sethostname等
(6).socket控制：如bind、connect等
(7).用户管理：如getuid、setuid等
(8).进程间通讯：如ipc、pipe等

```
  + 采分点：说明了Linux的大致数量（上百个），说明了Linux系统调用的主要分类（文件操作，进程管理，内存管理等）
  - 答案没有涉及上述两个要点；（0分）
  - 答案对上述两个要点中的某一个要点进行了正确阐述（1分）
  - 答案对上述两个要点进行了正确阐述（2分）
  - 答案除了对上述两个要点都进行了正确阐述外，还进行了扩展和更丰富的说明（3分）
 ```
 
 1. 以ucore lab8的answer为例，uCore的系统调用有哪些？大致的功能分类有哪些？(w2l1)
 
 答：在ucore lab8的answer中，系统调用函数一共有22个：分别为
    [SYS_exit]              sys_exit,
    [SYS_fork]              sys_fork,
    [SYS_wait]              sys_wait,
    [SYS_exec]              sys_exec,
    [SYS_yield]             sys_yield,
    [SYS_kill]              sys_kill,
    [SYS_getpid]            sys_getpid,
    [SYS_putc]              sys_putc,
    [SYS_pgdir]             sys_pgdir,
    [SYS_gettime]           sys_gettime,
    [SYS_lab6_set_priority] sys_lab6_set_priority,
    [SYS_sleep]             sys_sleep,
    [SYS_open]              sys_open,
    [SYS_close]             sys_close,
    [SYS_read]              sys_read,
    [SYS_write]             sys_write,
    [SYS_seek]              sys_seek,
    [SYS_fstat]             sys_fstat,
    [SYS_fsync]             sys_fsync,
    [SYS_getcwd]            sys_getcwd,
    [SYS_getdirentry]       sys_getdirentry,
    [SYS_dup]               sys_dup,

  上述系统调用函数按功能分为：
  (1).进程控制：sys_exit、sys_fork、sys_wait、sys_exec、sys_yield、sys_kill、sys_getpid、sys_sleep
  (2).文件系统控制：sys_pgdir、sys_gettime、sys_open、sys_close、sys_read、sys_write、sys_seek、sys_fstat、sys_getdirentry、sys_dup
  (3).内存管理：sys_putc、sys_lab6_set_priority、sys_fsync、sys_getcwd
  
 ```
  + 采分点：说明了ucore的大致数量（二十几个），说明了ucore系统调用的主要分类（文件操作，进程管理，内存管理等）
  - 答案没有涉及上述两个要点；（0分）
  - 答案对上述两个要点中的某一个要点进行了正确阐述（1分）
  - 答案对上述两个要点进行了正确阐述（2分）
  - 答案除了对上述两个要点都进行了正确阐述外，还进行了扩展和更丰富的说明（3分）
 ```
 
## 3.4 linux系统调用分析
 1. 通过分析[lab1_ex0](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex0.md)了解Linux应用的系统调用编写和含义。(w2l1)
 
答：objdump命令是Linux下的反汇编目标文件或者可执行文件的命令，同时它还可以显示文件头信息等作用，具体用法有：-f/-d/-D/-h/-x/-s
    nm命令在linux中，用来列出目标文件的符号清单，命令显示关于指定File中符号的信息，文件可以是对象文件、可执行文件或对象文件库
    file在Linux中是检测文件类型的命令
    系统调用的含义：linux内核中设置了一组用于实现系统功能的子程序，称为系统调用。系统调用和普通库函数调用非常相似，只是系统调用由操作系统核心提供，运行于核心态，而普通的函数调用由函数库或用户自己提供，运行于用户态。系统调用在用户空间进程和硬件设备之间添加了一个中间层，它有三个作用：
    (1).它为用户空间提供了一种统一的硬件的抽象接口
    (2).系统调用保证了系统的稳定和安全
    (3).每个进程都运行在虚拟系统中，而在用户空间和系统的其余部分提供这样一层公共接口，也是出于这种考虑；在Linux中，系统调用是用户空间访问内核的惟一手段；除异常和中断外，它们是内核惟一的合法入口

 ```
  + 采分点：说明了objdump，nm，file的大致用途，说明了系统调用的具体含义
  - 答案没有涉及上述两个要点；（0分）
  - 答案对上述两个要点中的某一个要点进行了正确阐述（1分）
  - 答案对上述两个要点进行了正确阐述（2分）
  - 答案除了对上述两个要点都进行了正确阐述外，还进行了扩展和更丰富的说明（3分）
 
 ```
 
 1. 通过调试[lab1_ex1](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex1.md)了解Linux应用的系统调用执行过程。(w2l1)
 答：strace命令正在Linux中常用来跟踪进程执行时的系统调用和所接收的信号，在Linux世界，进程不能直接访问硬件设备，当进程需要访问硬件设备(比如读取磁盘文件，接收网络数据等等)时，必须由用户态模式切换至内核态模式，通过系统调用访问硬件设备。strace可以跟踪到一个进程产生的系统调用,包括参数，返回值，执行消耗的时间。

    系统调用：在Linux中，系统调用是用户访问内核的唯一手段，它们是内核的唯一的合法入口。Linux系统调用过程如下所示：
    用户程序-->C库（即API）：INT 0x80 -->system_call-->系统调用服务例程-->内核程序
    系统调用就是通过软中断指令INT 0x80 实现的，而这条INT 0x80指令就被封装在C库的函数中，它的执行会让系统跳转到一个预设的 内核空间地址，它指向系统调用处理程序，即system_call函数，system_call函数通过系统调用号查找系统调用表sys_call_table，软中断指令INT 0x80执行时，系统调用号会被放入 eax寄存器中，system_call函数可以读取eax寄存器获取，然后将其乘以4，生成偏移地址，然后以 sys_call_table为基址，基址加上偏移地址，就可以得到具体的系统调用服务例程的地址，然后就到了系统调用服务例程了。
 ```
  + 采分点：说明了strace的大致用途，说明了系统调用的具体执行过程（包括应用，CPU硬件，操作系统的执行过程）
  - 答案没有涉及上述两个要点；（0分）
  - 答案对上述两个要点中的某一个要点进行了正确阐述（1分）
  - 答案对上述两个要点进行了正确阐述（2分）
  - 答案除了对上述两个要点都进行了正确阐述外，还进行了扩展和更丰富的说明（3分）
 ```
 
## 3.5 ucore系统调用分析
 1. ucore的系统调用中参数传递代码分析。
 1. ucore的系统调用中返回结果的传递代码分析。
 1. 以ucore lab8的answer为例，分析ucore 应用的系统调用编写和含义。
 1. 以ucore lab8的answer为例，尝试修改并运行ucore OS kernel代码，使其具有类似Linux应用工具`strace`的功能，即能够显示出应用程序发出的系统调用，从而可以分析ucore应用的系统调用执行过程。
 
## 3.6 请分析函数调用和系统调用的区别
 1. 请从代码编写和执行过程来说明。
   1. 说明`int`、`iret`、`call`和`ret`的指令准确功能
 
