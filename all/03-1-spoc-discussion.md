# lec5 SPOC思考题


NOTICE
- 有"w3l1"标记的题是助教要提交到学堂在线上的。
- 有"w3l1"和"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的git repo上。
- 有"hard"标记的题有一定难度，鼓励实现。
- 有"easy"标记的题很容易实现，鼓励实现。
- 有"midd"标记的题是一般水平，鼓励实现。


## 个人思考题
---

请简要分析最优匹配，最差匹配，最先匹配，buddy system分配算法的优势和劣势，并尝试提出一种更有效的连续内存分配算法 (w3l1)
```
最优匹配：
		优点：大部分分配的尺寸很小时效果比较好；可以避免大块空间被切分
		缺点：产生外部和无用碎片，释放分区时缓慢。
  最差匹配：
		优点：中等大小分配较多时效果比较好，有效避免产生很多小碎片；
		缺点：释放分区比较慢，容易产生外部碎片，容易破坏比较大的分区空间，后续难以分配比较大的分区。
  最先匹配：
		优点：简单，在高地址有大块空闲分区；
		缺点：容易产生外部碎片，分配大块时缓慢
  buddy system分配算法：
		优点：快速合并，低外部碎片；
		缺点：产生内部碎片。合并有条件导致一些连续区域不可合并
	考虑将首次适应算法进行改进。
   在为进程分配内存空间时，从上次找到的空闲分区开始查找，直至找到一个能满足需求的空闲分区，并从中划出一块来分给作业。
	优点：该算法能使空闲中的内存分区分布得更加均匀。
   缺点：造成缺乏比较大的空闲分区。
```

## 小组思考题

请参考ucore lab2代码，采用`struct pmm_manager` 根据你的`学号 mod 4`的结果值，选择四种（0:最优匹配，1:最差匹配，2:最先匹配，3:buddy systemm）分配算法中的一种或多种，在应用程序层面(可以 用python,ruby,C++，C，LISP等高语言)来实现，给出你的设思路，并给出测试用例。 (spoc)

```
//学号为2012011270 mod 4 = 2 最先匹配
 
 >#include <iostream>
 using namespace std;

 struct page
{
    int pageSize;
    bool isfree;
    page *next;
};

struct pmm_manager
{
    page *pmm;
    void init()
    {
         this->pmm=new page;
         pmm->pageSize = 1024;
         pmm->isfree = 1;
         pmm->next=NULL;
    }
    page *alloc(int size){
         while(size % 4 != 0)
         {
             size++;
         }
         page *x=this->pmm;
         page *y;
         while(x)
         {
             if(x->isfree && x-> pageSize > size)
            {
                y=new page;
                y->pageSize=x->pageSize-size;
                y->isfree=true;
                y->next=x->next;
                x->pageSize=size;
                x->next=y;
                x->isfree=false;
                break;
        }
        else
        {
            x = x ->next;
        }
    }
    return x;
    }
    void free_pmm(page *x){
         x -> isfree = true;
    page *m = this -> pmm, *n;
    while(m -> next)
    {
        n = m -> next;
        if(m -> isfree && n -> isfree)
        {
            m -> pageSize += n -> pageSize;
            m -> next = n -> next;
            free(n);
        }
        m = m -> next;
    }}
};

int main()
{
    pmm_manager manage;
    page *n1,*n2,*n3,*n4,*n5;
    manage.init();
    n1=manage.alloc(90);
    n2=manage.alloc(30);
    n3=manage.alloc(60);
    n4=manage.alloc(80);
    n5=manage.alloc(100);

    manage.free_pmm(n1);
    manage.free_pmm(n2);
    manage.free_pmm(n3);
    manage.free_pmm(n4);
    manage.free_pmm(n5);
    system("pause");
    return 0;
}
```

--- 

## 扩展思考题

阅读[slab分配算法](http://en.wikipedia.org/wiki/Slab_allocation)，尝试在应用程序中实现slab分配算法，给出设计方案和测试用例。

## “连续内存分配”与视频相关的课堂练习

### 5.1 计算机体系结构和内存层次
MMU的工作机理？

- [x]  

>  http://en.wikipedia.org/wiki/Memory_management_unit

L1和L2高速缓存有什么区别？

- [x]  

>  http://superuser.com/questions/196143/where-exactly-l1-l2-and-l3-caches-located-in-computer
>  Where exactly L1, L2 and L3 Caches located in computer?

>  http://en.wikipedia.org/wiki/CPU_cache
>  CPU cache

### 5.2 地址空间和地址生成
编译、链接和加载的过程了解？

- [x]  

>  

动态链接如何使用？

- [x]  

>  


### 5.3 连续内存分配
什么是内碎片、外碎片？

- [x]  

>  

为什么最先匹配会越用越慢？

- [x]  

>  

为什么最差匹配会的外碎片少？

- [x]  

>  

在几种算法中分区释放后的合并处理如何做？

- [x]  

>  

### 5.4 碎片整理
一个处于等待状态的进程被对换到外存（对换等待状态）后，等待事件出现了。操作系统需要如何响应？

- [x]  

>  

### 5.5 伙伴系统
伙伴系统的空闲块如何组织？

- [x]  

>  

伙伴系统的内存分配流程？

- [x]  

>  

伙伴系统的内存回收流程？

- [x]  

>  

struct list_entry是如何把数据元素组织成链表的？

- [x]  

>  


