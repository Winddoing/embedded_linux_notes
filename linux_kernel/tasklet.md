# tasklet

## 解决的问题

* tasklet和工作队列是延期执行工作的机制，其实现基于软中断，但他们更易于使用，因而更适合与设备驱动程序...tasklet是“小进程”，执行一些迷你任务，对这些人物使用全功能进程可能比较浪费

* **软中断是将操作推迟到未来时刻执行的最有效的方法。但该延期机制处理起来非常复杂。因为多个处理器可以同时且独立的处理软中断，同一个软中断的处理程序可以在几个CPU上同时运行。对软中断的效率来说，这是一个关键，多处理器系统上的网络实现显然受惠于此。但处理程序的设计必须是完全可重入且线程安全的。另外，临界区必须用自旋锁保护（或其他IPC机制），而这需要大量审慎的考虑。**

## 数据结构

```
struct tasklet_head                                                                   
{                                                                                     
    struct tasklet_struct *head;                                                      
    struct tasklet_struct **tail;                                                     
};                                                                                    
                                                                                      
static DEFINE_PER_CPU(struct tasklet_head, tasklet_vec);                              
static DEFINE_PER_CPU(struct tasklet_head, tasklet_hi_vec);                           
```
>kernel/softirq.c

 tasklet通过软中断实现，软中断中有两种类型属于tasklet，分别是级别最高的HI_SOFTIRQ和TASKLET_SOFTIRQ。


 tasklet的核心结构体如下:
 
 ```
 struct tasklet_struct                        
{                                            
    struct tasklet_struct *next;             
    unsigned long state;                     
    atomic_t count;                          
    void (*func)(unsigned long);             
    unsigned long data;                      
};                                           
 ``` 
> include/linux/interrupt.h

>习惯上称之为tasklet描述符，func指针是具体的处理函数指针，data为可选参数，state表示该tasklet的状态，分别使用不同的bit表示两个状态：TASKLET_STATE_SCHED和TASKLET_STATE_RUN：

* TASKLET_STATE_SCHED置位表示已经被调度（挂起），也意味着tasklet描述符被插入到了tasklet_vec和tasklet_hi_vec数组的其中一个链表中，可以被执行。
* TASKLET_STATE_RUN置位表示该tasklet正在某个CPU上执行，单个处理器系统上并不校验该标志，因为没必要检查特定的tasklet是否正在运行。
* count为原子计数器，用于禁用已经调度的tasklet，如果该值不为0，则不予以执行。

## 操作接口

tasklet对驱动开放的常用操作包括：

1. 初始化，tasklet_init()，初始化一个tasklet描述符。
2. 调度，tasklet_schedule()和tasklet_hi_schedule()，将taslet置位TASKLET_STATE_SCHED，并尝试激活所在的软中断。
3. 禁用/启动，tasklet_disable_nosync()、tasklet_disable()、task_enable()，通过count计数器实现。
4. 执行，tasklet_action()和tasklet_hi_action()，具体的执行软中断。
5. 杀死，tasklet_kill()，。。。

## 参考

1. [Linux tasklet 分析笔记（转载）](http://blog.csdn.net/chengqianyun2002/article/details/1607005)
2. [tasklet原理](http://rock3.info/blog/2013/11/22/tasklet%E5%8E%9F%E7%90%86/)
