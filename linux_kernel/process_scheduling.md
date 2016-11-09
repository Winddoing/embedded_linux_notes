# 进程调度

主要进程调度是为了并发执行

## 进程

http://wenku.baidu.com/link?url=hpHxRSz8ix9nvqJqn2nWzlFF_BF3N7pZUQMnoFQY967rB-t0fWd3gl2NzPIV008mHEtl0uaqHZlNbRQIkMgcjqix_WkOH5KtRwbyaRGjmIO

## 抢占

## 进程的状态

## 进程切换

时间片

栈的切换 ——改变栈指针

## 进程的创建





## 内核抢占

一个在内核态运行的进程，可能在执行内核函数期间被另一个进程取代。

在2.6版的内核中，内核引入了抢占能力；现在，只要重新调度是安全的，那么内核就可以在任何时间抢占正在执行的任务。

那么，什么时候重新调度才是安全的呢?只要没有持有锁，内核就可以进行抢占。锁是非抢占区域的标志。由于内核是支持SMP的，所以，如果没有持有锁，那么正在执行的代码就是可重新导入的，也就是可以抢占的。

为了支持内核抢占所作的第一处变动就是每个进程的thread_info引入了 preempt_count(thread_info.preempt_count)计数器。该计数器初始值为0，每当使用锁的时候数值加1，释放锁的时候数值减1。当数值为0的时候，内核就可执行抢占。从中断返回内核空间的时候，内核会检查flag和preempt_count的值。如果flag中TIF_NEED_RESCHED被设置，并且preempt_count为0的话，这说明有一个更为重要的任务需要执行并且可以安全地抢占，此时，调度程序就会调度(抢占当前进程)。如果preempt_count不为0，说明当前任务持有锁，所以抢占是不安全的。这时，就会像通常那样直接从中断返回当前执行进程。 如果当前进程所持有的所有的锁都被释放了,那么preemptcount就会重新为0。此时，释放锁的代码会检查need_resched是否被设置。如果是的话，就会调用调度程序。有些内核代码需要允许或禁止内核抢占。

如果内核中的进程被阻塞了，或它显式地调用了schedule()，内核抢占也会显式地发生。这种形式的内核代码从来都是受支持的，因为根本无需额外的逻辑来保证内核可以安全地发生被抢占。如果代码显式的调用了schedule()，那么它应该清楚自己是可以安全地被抢占的。


### 内核抢占发生在: 

1. 当"从中断处理程序"正在执行，且返回内核空间之前 
2. 内核代码再一次具有可抢占性的时候 
3. 如果内核中的任务显式的调用schedule() 
4. 如果内核中的任务阻塞(这同样也会导致调用schedule()) 

注:
	current->threadinfo.flags中TIF_NEED_RESCHED为1，表示当前进程需要执行schedule()释放CPU控制权 
	current->threadinfo.preemptcount的值不为0，表示当前进程持有锁不能释放CPU控制权(不能被抢占)

### preempt_count()

#define preempt_count() (current_thread_info()->preempt_count)

linux中mips架构使用寄存器28来指向当前进程的thread_info.相关代码:
		
/* How to get the thread information struct from C.  */
static inline struct thread_info *current_thread_info(void)                        
{
    register struct thread_info *__current_thread_info __asm__("$28");

    return __current_thread_info;
}
arch/mips/include/asm/thread_info.h 

## 进程内核栈

http://www.360doc.com/content/12/0614/01/4672432_218018481.shtml

### 内核栈和进程结构体的关联

1. 进程结构体struct task_struct
在内核中代表一个进程,其中记录进程的所有信息.
其中,void *stack;指向内核栈结构体的"栈底"

2. 内核栈结构体union thread_union

union thread_union {                                              
    struct thread_info thread_info;
    unsigned long stack[THREAD_SIZE/sizeof(long)];
};

#define THREAD_SIZE (PAGE_SIZE << THREAD_SIZE_ORDER)  //MIPS 32bit  4K << 1 = 8K

二者是相互关联



### 内核栈的产生

do_fork()->copy_prcess()->dup_task_struct()




## 进程创建和调度

通过什么而创建,因为什么而调度

何时睡眠,何时唤醒

### 什么时候创建

### 什么时候调度

### 什么时候唤醒



## 队列

### 运行队列

### 工作队列

### 等待队列




module_init  是否创建新的进程

	#define module_init(initfn)                 \                   
   	 static inline initcall_t __inittest(void)       \
   	 { return initfn; }                  \
   	 int init_module(void) __attribute__((alias(#initfn)));

