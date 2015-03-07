# lab0 SPOC思考题

## 个人思考题

---

能否读懂ucore中的AT&T格式的X86-32汇编语言？请列出你不理解的汇编语言。
- [x]  

>  http://www.imada.sdu.dk/Courses/DM18/Litteratur/IntelnATT.htm
>  由于很久没有使用汇编语言了，所以对寄存器的功能记不清楚，查询了各个寄存器的功能。

虽然学过计算机原理和x86汇编（根据THU-CS的课程设置），但对ucore中涉及的哪些硬件设计或功能细节不够了解？
- [x]  

> 学习了计算机组成原理对硬件各个部分的功能有了基本的了解。中断和寄存器等不是十分清楚。


哪些困难（请分优先级）会阻碍你自主完成lab实验？
- [x]  

>   操作系统的具体细节的实现>硬件与汇编和操作系统之间的配合实现>对linux不那么熟悉

如何把一个在gdb中或执行过程中出现的物理/线性地址与你写的代码源码位置对应起来？
- [x]  

> 1. GDB之所以能够知道对应的源代码，是因为调试版的可执行程序中记录了源代码的位置；因为源代码的位置在编译之后可能会移动到其它地方，所以GDB还会在当前目录中查找源代码，另外GDB也允许明确指定源代码的搜索位置。默认情况下，GDB在编译时目录中搜索，如果失败则在当前目录中搜索，即$cdir:$cwd
set listsize count：设置list命令显示的源代码数量最多为count行，0表示不限制行数。
show listsize：显示listsize的值。
search regexp：从当前行的下一行开始向前搜索。
rev regexp ：从当前行的上一行开始向后搜索。
有的时候，你会发现search命令总是提示“Expression not found”，这是因为当前行可能已经是最后一行了，特别是文件很短的时候。这里需要注意的是，任何list命令都会影响当前行的位置，并且由于每次都是多行输出，所以对当前行的影响并非简单地向前一行或者向后一行。
break加行号即为物理地址，list加*物理地址即为行号。
> 2. 用nm, objdump工具可以看到

了解函数调用栈对lab实验有何帮助？
- [x]  

> 了解了函数在内存中的存储情况，esp与ebp的位置移动决定了当前变量在函数栈中的位置，栈帧的跳转保证了递归等调用的正确性。也就是说在汇编级别上对lab实验有了整体的把握。
> 对于函数的调用过程和程序的运行过程有更好的理解。


你希望从lab中学到什么知识？
- [x]  

>   掌握操作系统的运行过程，深入理解操作系统的作用，机制等。

---

## 小组讨论题

---

搭建好实验环境，请描述碰到的困难和解决的过程。
- [x]  

> 配置虚拟机不会导入mooc_os虚拟硬盘。上网百度查询。
可能由于电脑的配置不行，带动虚拟机的时候上不了网，运行make mequ指令的时候系统直接卡死，没有运行结果。所以重新装了linux双系统。
双系统装好之后配置各种软件超级麻烦。。。不能使用vdi配好的硬盘。

熟悉基本的git命令行操作命令，从github上
的 http://www.github.com/chyyuu/ucore_lab 下载
ucore lab实验
- [x]  
-
> gitclone http://www.github.com/chyyuu/ucore_lab

尝试用qemu+gdb（or ECLIPSE-CDT）调试lab1
- [x]   

> 清除文件夹：make clean 
> 编译lab1：make 
> 调出debug命令行：make debug

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

> 数字为位域大小

对于如下的代码段，
```
#define SETGATE(gate, istrap, sel, off, dpl) {            \
    (gate).gd_off_15_0 = (uint32_t)(off) & 0xffff;        \
    (gate).gd_ss = (sel);                                \
    (gate).gd_args = 0;                                    \
    (gate).gd_rsv1 = 0;                                    \
    (gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;    \
    (gate).gd_s = 0;                                    \
    (gate).gd_dpl = (dpl);                                \
    (gate).gd_p = 1;                                    \
    (gate).gd_off_31_16 = (uint32_t)(off) >> 16;        \
}
```
如果在其他代码段中有如下语句，
```
unsigned intr;
intr=8;
SETGATE(intr, 0,1,2,3);
```
请问执行上述指令后， intr的值是多少？

- [x]  0x10002

> https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab0/lab0_ex3.c

请分析 [list.h](https://github.com/chyyuu/ucore_lab/blob/master/labcodes/lab2/libs/list.h)内容中大致的含义，并能include这个文件，利用其结构和功能编写一个数据结构链表操作的小C程序
- [x]  

>   int main(int argc, char *argv[])
  {
      list_entry_t a, b, c, d;
      list_init(&a);
      a.prev = NULL;
      a.next = &b;
      b.prev = &a;
      b.next = &c;
      c.prev = &b;
      c.next = NULL;
      list_add(&b, &c);
      list_add(&a, &b);
      printf("%d %d %d\n", a.prev, a.next, &a);
      printf("%d %d %d\n", b.prev, b.next, &b);
      printf("%d %d %d\n", c.prev, c.next, &c);
      printf("%d %d %d\n", d.prev, d.next, &d);
      list_del(d);
      list_del(c);
      printf("%d %d %d\n", a.prev, a.next, &a);
      printf("%d %d %d\n", b.prev, b.next, &b);
      printf("%d %d %d\n", c.prev, c.next, &c);
      printf("%d %d %d\n", d.prev, d.next, &d);
      return 0;
  }

---

## 开放思考题

---

是否愿意挑战大实验（大实验内容来源于你的想法或老师列好的题目，需要与老师协商确定，需完成基本lab，但可不参加闭卷考试），如果有，可直接给老师email或课后面谈。
- [x]  

>  

---
