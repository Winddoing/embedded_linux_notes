# 内核相关

## Linux内核里面，内存申请有哪几个函数，各自的区别？

> kmalloc, kzalloc, vmalloc, __get_get_page, __get_free_pages, dma_alloc_coherent, dma_pool_alloc

### kmalloc

```
static __always_inline void *kmalloc(size_t size, gfp_t flags)
```
> include/linux/slab.h

* 内存分配和malloc相似，除非被阻塞否则他执行的速度非常快，而且不对获得空间清零(memset手动清零)

参数:

size : 申请的内存大小
flags: 申请内存的属性(最常用的GFP_KERNEL)

> 内存分配（最终总是调用get_free_pages来实现实际的分配；这就是GFP前缀的由来）是代表运行在内核空间的进程执行的。使用GFP_KERNEL容许kmalloc在分配空闲内存时候如果内存不足容许把当前进程睡眠以等待。因此这时分配函数必须是可重入的。如果在进程上下文之外如：中断处理程序、tasklet以及内核定时器中这种情况下current进程不该睡眠，驱动程序该使用GFP_ATOMIC

kmalloc 能够处理的最小分配是 32 或者 64 字节, 依赖系统的体系所使用的页大小. kmalloc 能够分配的内存块的大小有一个上限. 这个限制随着体系和内核配置选项而变化. 如果你的代码是要完全可移植, 它不能指望可以分配任何大于 128 KB. 如果你需要多于几个 KB, 但是, 有个比 kmalloc 更好的方法来获得内存。在设备驱动程序或者内核模块中动态开辟内存，不是用malloc，而是kmalloc ,vmalloc，或者用get_free_pages直接申请页。释放内存用的是kfree,vfree，或free_pages. kmalloc函数返回的是虚拟地址(线性地址). kmalloc特殊之处在于它分配的内存是物理上连续的,这对于要进行DMA的设备十分重要. 而用vmalloc分配的内存只是线性地址连续,物理地址不一定连续,不能直接用于DMA.
　　注意kmalloc最大只能开辟128k-16，16个字节是被页描述符结构占用了。kmalloc用法参见khg.kmalloc最多只能开辟大小为32XPAGE_SIZE的内存,一般的PAGE_SIZE=4kB,也就是128kB的大小的内存　　

### vmalloc


### kmalloc和vmalloc的区别

* vmalloc()与 kmalloc()都可用于分配内存
* kmalloc()分配的内存处于3GB~high_memory之 间,这段内核空间与物理内存的映射一一对应
* vmalloc()分配的内存在 VMALLOC_START~4GB之间,这段非连续内 存区映射到物理内存也可能是非连续的
* 在内核空间中调用kmalloc()分配连续物理空间,而调用vmalloc()分配非物理连续空间。
* 把kmalloc()所分配内核空间中的地址称为内核逻辑地址
* 把vmalloc()分配的内核空间中的地址称 为内核虚拟地址
* vmalloc()在分配过程中须更新内核页表

### kzalloc

```
static inline void *kzalloc(size_t size, gfp_t flags)
{
    return kmalloc(size, flags | __GFP_ZERO);
}
```





## IRQ和FIQ有什么区别，在CPU里面是是怎么做的？

> 非MIPS架构，ARM架构的相关名词和对中断的实现方式

## 中断的上半部分和下半部分的问题：讲下分成上半部分和下半部分的原因，为何要分？讲下如何实现？

* 中断处理程序正在执行时，会屏蔽同条中断线上的中断请求
* 下半部可以通过多种机制来完成: tasklet，工作队列，软中断
* 中断上下两部分最大的不同是上半部分不可中断，而下半部分可中断

**划分：**
 Ⅰ．如果该任务对时间比较敏感，将其放在上半部中执行。
 Ⅱ．如果该任务和硬件相关，一般放在上半部中执行。
 Ⅲ．如果该任务要保证不被其他中断打断，放在上半部中执行（因为这是系统关中断）。
 Ⅳ．其他不太紧急的任务，般考虑在下半部执行。

 > 下半部分并不需要指明一个确切时间，只要把这些任务推迟一点，让它们在系统不太忙并且中断恢复后执行就可以了。通常下半部分在中断处理程序一返回就会马上运行，下半部分执行的关键在于当它们运行的时候，允许响应所有的中断。下半部分是一种推后执行任务，它将某些不那么紧迫的任务推迟到系统更方便的时刻运行。 内核中实现下半部的手段不断演化，目前已经从最原始的BH（bottom half）衍生出BH（在2.5中去除）、软中断（softirq在2.3引 入）、tasklet（在2.3引入）、工作队列（work queue在2.5引入）。这里主要分析讨论后三种方式。

## 在软件上，有中断号，中断向量表，中断函数，3者的关系是什么？

## 内核函数mmap的实现原理，机制？

## 驱动里面为什么要有并发、互斥的控制？如何实现？讲个例子？

## spinlock自旋锁是如何实现的？

## 任务调度的机制？

## 嵌入式设备，为加快启动速度，可以做哪些方面的优化？

## 时钟变慢的影响？

## 发生中断，芯片会做什么？

## 芯片如何降低功耗




## 参考

1. [kmalloc/kfree,vmalloc/vfree函数用法和区别](http://blog.csdn.net/tigerjibo/article/details/6412881)
2. [中断的上半部分和下半部分](http://blog.csdn.net/zhanghuaichao/article/details/48601587)
