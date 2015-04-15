# lab4 spoc 思考题

- 有"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的ucore_code和os_exercises的git repo上。

## 个人思考题

### 总体介绍

(1) ucore的线程控制块数据结构是什么？
```
ucore中没有明确创建的TCB对线程进行控制。
但对进程PCB有明确的数据结构：proc_struct(proc.h中）
struct mm_struct *mm;   // Process's memory management field
uintptr_t cr3;          // CR3 register
int pid;                // Process ID
观察以上三个变量：
不同进程具有不同的pid值，同一进程的不同线程pid不同，所以采用cr3和mm这两个变量相同，进行不同线程属于同一进程的判断。

```
### 关键数据结构

(2) 如何知道ucore的两个线程同在一个进程？
```
同（1）
同一进程的两个线程cr3与mm相同。
```
(3) context和trapframe分别在什么时候用到？
```
context使用在switch_to函数中,该函数写在switch.S中。
作用是保存前一进程相应寄存器的值，将后一进程值压入寄存器中。
通过popl 0(%eax）实现。
trapframe使用在kernel_thread中，根据父进程的寄存器值创建子线程。
执行子线程创建函数do_fork()，并保存子线程执行的入口地址。
```
(4) 用户态或内核态下的中断处理有什么区别？在trapframe中有什么体现？
```
用户态转为内核态的时候需要将ss和esp压入栈中，即刷新内核栈。
在trapframe的体现为：
uintptr_t tf_esp;
uint16_t tf_ss;
kernel_thread赋值：
tf.tf_ss = KERNEL_DS;
proc->tf->tf_esp = esp;
（其中esp为0，分支为内核线程）
```
### 执行流程

(5) do_fork中的内核线程执行的第一条指令是什么？它是如何过渡到内核线程对应的函数的？
```
保持内核线程对应的函数指针：
 tf.tf_regs.reg_ebx = (uint32_t)fn;
 将子线程的执行指令指向内核线程的入口地址：
tf.tf_eip = (uint32_t) kernel_thread_entry;
调用/kern/process/entry.S的汇编代码：
kernel_thread_entry:        # void kernel_thread(void)

    pushl %edx              # push arg
    call *%ebx              # call fn

    pushl %eax              # save the return value of fn(arg)
    call do_exit            # call do_exit to terminate current thread
以上步骤完成指令的执行。
```

(6)内核线程的堆栈初始化在哪？
```
//tf和context中的esp
//kernel_thread中对tf进行初始化：
memset(&tf, 0, sizeof(struct trapframe));
tf.tf_cs = KERNEL_CS;
tf.tf_ds = tf.tf_es = tf.tf_ss = KERNEL_DS;
tf.tf_regs.reg_ebx = (uint32_t)fn;
tf.tf_regs.reg_edx = (uint32_t)arg;
tf.tf_eip = (uint32_t)kernel_thread_entry;
//copy_thread中对tf初始化：
proc->tf = (struct trapframe *)(proc->kstack + KSTACKSIZE) - 1;
*(proc->tf) = *tf;
proc->tf->tf_regs.reg_eax = 0;
proc->tf->tf_esp = esp;
proc->tf->tf_eflags |= FL_IF;
//对context初始化：
proc->context.eip = (uintptr_t)forkret;
proc->context.esp = (uintptr_t)(proc->tf);
//do_fork函数中：
setup_kstack(proc)；//为内核函数分配地址
```

(7)fork()父子进程的返回值是不同的。这在源代码中的体现中哪？
```
//子进程返回0。
proc->tf->tf_regs.reg_eax = 0;
//父进程
proc->pid = get_pid();
wakeup_proc(proc);
ret = proc->pid;
return ret;

kernel_thread(int (*fn)(void *), void *arg, uint32_t clone_flags):
return do_fork(clone_flags | CLONE_VM, 0, &tf);

syscall.c
sys_fork(uint32_t arg[])
return do_fork(0, stack, tf);
```
(8)内核线程initproc的第一次执行流程是什么样的？能跟踪出来吗？
```
check_slab() success
kmalloc_init() succeeded!
check_vma_struct() succeeded!
page fault at 0x00000100: K/W [no page found].
check_pgfault() succeeded!
check_vmm() succeeded.
cr3: 24c000
pid: 0
ide 0:      10000(sectors), 'QEMU HARDDISK'.
ide 1:     262144(sectors), 'QEMU HARDDISK'.
SWAP: manager = fifo swap manager
BEGIN check_swap: count 31950, total 31950
setup Page Table for vaddr 0X1000, so alloc a page
setup Page Table vaddr 0~4MB OVER!
set up init env for check_swap begin!
page fault at 0x00001000: K/W [no page found].
page fault at 0x00002000: K/W [no page found].
page fault at 0x00003000: K/W [no page found].
page fault at 0x00004000: K/W [no page found].
set up init env for check_swap over!
write Virt Page c in fifo_check_swap
write Virt Page a in fifo_check_swap
write Virt Page d in fifo_check_swap
write Virt Page b in fifo_check_swap
write Virt Page e in fifo_check_swap
page fault at 0x00005000: K/W [no page found].
swap_out: i 0, store page in vaddr 0x1000 to disk swap entry 2
write Virt Page b in fifo_check_swap
write Virt Page a in fifo_check_swap
page fault at 0x00001000: K/W [no page found].
swap_out: i 0, store page in vaddr 0x2000 to disk swap entry 3
swap_in: load disk swap entry 2 with swap_page in vadr 0x1000
write Virt Page b in fifo_check_swap
page fault at 0x00002000: K/W [no page found].
swap_out: i 0, store page in vaddr 0x3000 to disk swap entry 4
swap_in: load disk swap entry 3 with swap_page in vadr 0x2000
write Virt Page c in fifo_check_swap
page fault at 0x00003000: K/W [no page found].
swap_out: i 0, store page in vaddr 0x4000 to disk swap entry 5
swap_in: load disk swap entry 4 with swap_page in vadr 0x3000
write Virt Page d in fifo_check_swap
page fault at 0x00004000: K/W [no page found].
swap_out: i 0, store page in vaddr 0x5000 to disk swap entry 6
swap_in: load disk swap entry 5 with swap_page in vadr 0x4000
count is 5, total is 5
check_swap() succeeded!
++ setup timer interrupts
cr3: 24c000
this initproc, pid = 1, name = "init"
To U: "Hello world!!".
To U: "en.., Bye, Bye. :)"
kernel panic at kern/process/proc.c:355:
    process exit!!.

Welcome to the kernel debug monitor!!
Type 'help' for a list of commands.
```
## 小组练习与思考题

(1)(spoc) 理解内核线程的生命周期。

> 需写练习报告和简单编码，完成后放到git server 对应的git repo中

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


