# lab4 spoc 思考题

- 有"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的ucore_code和os_exercises的git repo上。

## 个人思考题

### 总体介绍

(1) ucore的线程控制块数据结构是什么？
> 答：在ucore中，为了简化设计，并没有为线程专门增添新的线程控制块数据结构，而是与进程使用相同的进程控制块数据结构struct proc_struct，所以ucore的线程控制块函数也是struct proc_struct。

### 关键数据结构

(2) 如何知道ucore的两个线程同在一个进程？
> 答：可以根据mm保存的信息来判断，也可以根据cr3寄存器的值是否相同来判断页表是否相同，如果相同说明是在同一个线程中；如果不相同说明不是同一个进程的线程。

(3) context和trapframe分别在什么时候用到？
> 答：context是进程的上下文，用于进程切换，在ucore中，所有进程在内核中是相对独立的，使用context保存寄存器的目的是在内核中进行上下文的切换；
trapframe是中断帧，中断帧记录了进程在被中断前的状态，当内核需要跳回到用户空间时，需要调整中断帧以恢复让进程继续执行的各寄存器的值。

(4) 用户态或内核态下的中断处理有什么区别？在trapframe中有什么体现？
> 答：在用户态的中断需要保存比内核态更多的寄存器的值，如SS、ESP等堆栈寄存器；可以在trap.h中的struct trapframe中观察到：
```
    /* below here defined by x86 hardware */
    uint32_t tf_err;
    uintptr_t tf_eip;
    uint16_t tf_cs;
    uint16_t tf_padding4;
    uint32_t tf_eflags;
    /* below here only when crossing rings, such as from user to kernel */
    uintptr_t tf_esp;
    uint16_t tf_ss;
    uint16_t tf_padding5;
```

### 执行流程

(5) do_fork中的内核线程执行的第一条指令是什么？它是如何过渡到内核线程对应的函数的？
```
tf.tf_eip = (uint32_t) kernel_thread_entry;
/kern-ucore/arch/i386/init/entry.S
/kern/process/entry.S
```

(6)内核线程的堆栈初始化在哪？
```
tf和context中的esp
```

(7)fork()父子进程的返回值是不同的。这在源代码中的体现中哪？
> 答：fork()父进程的返回值为：do_fork()函数中的：

```
ret = proc->pid;
fork_out:
    return ret;
```
 
fork()子进程的返回值为：copy_tread()函数中的：
```
proc->tf->tf_regs.reg_eax = 0;
```

(8)内核线程initproc的第一次执行流程是什么样的？能跟踪出来吗？
> 答：可以跟踪出来，可以通过打印pid来知道当前运行的是哪个线程，同时可打印出线程的名字，可以通过打印cr3知道是否是同一个进程的线程。打印结果为：
```
... //删去了一些不相关的打印结果
this idleproc, pid = 0, name = "idle"
cr3: 24c000
ide 0: 10000(sectors), 'QEMU HARDDISK'.
ide 1: 262144(sectors), 'QEMU HARDDISK'.
SWAP: manager = fifo swap manager
... //删去了一些不相关的打印结果
++ setup timer interrupts
this initproc, pid = 1, name = "init"
cr3: 24c000
To U: "Hello world!!".
To U: "en.., Bye, Bye. :)"
kernel panic at kern/process/proc.c:354:
process exit!!.
Welcome to the kernel debug monitor!!
Type 'help' for a list of commands.
```

## 小组练习与思考题

(1)(spoc) 理解内核线程的生命周期。

> 需写练习报告和简单编码，完成后放到git server 对应的git repo中
> 答：在proc_init里创建第一个内核线程，之后会创建新的内核线程，并放入堆栈中，进行进程间的切换：
```
    if ((idleproc = alloc_proc()) == NULL) {
        panic("cannot alloc idleproc.\n");
    }
    idleproc->pid = 0;
    idleproc->state = PROC_RUNNABLE;
    idleproc->kstack = (uintptr_t)bootstack;
    idleproc->need_resched = 1;
    set_proc_name(idleproc, "idle");
    nr_process ++;
    current = idleproc;
    cprintf("this idleproc, pid = %d, name = \"%s\"\n", current->pid, get_proc_name(current));
    cprintf("cr3: %0x\n", current->cr3);
```

### 掌握知识点
1. 内核线程的启动、运行、就绪、等待、退出
2. 内核线程的管理与简单调度
3. 内核线程的切换过程

### 练习用的[lab4 spoc exercise project source code](https://github.com/chyyuu/ucore_lab/tree/master/related_info/lab4/lab4-spoc-discuss)

请完成如下练习，完成代码填写，并形成spoc练习报告

### 1. 分析并描述创建分配进程的过程

> 注意 state、pid、cr3，context，trapframe的含义

### 练习2：分析并描述新创建的内核线程是如何分配资源的

> 注意 理解对kstack, trapframe, context等的初始化


当前进程中唯一，操作系统的整个生命周期不唯一，在get_pid中会循环使用pid，耗尽会等待

### 练习3：阅读代码，在现有基础上再增加一个内核线程，并通过增加cprintf函数到ucore代码中
能够把进程的生命周期和调度动态执行过程完整地展现出来

### 练习4 （非必须，有空就做）：增加可以睡眠的内核线程，睡眠的条件和唤醒的条件可自行设计，并给出测试用例，并在spoc练习报告中给出设计实现说明

### 扩展练习1: 进一步裁剪本练习中的代码，比如去掉页表的管理，只保留段机制，中断，内核线程切换，print功能。看看代码规模会小到什么程度。


