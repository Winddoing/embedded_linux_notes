# 进程调度

主要进程调度是为了并发执行

## 进程

http://wenku.baidu.com/link?url=hpHxRSz8ix9nvqJqn2nWzlFF_BF3N7pZUQMnoFQY967rB-t0fWd3gl2NzPIV008mHEtl0uaqHZlNbRQIkMgcjqix_WkOH5KtRwbyaRGjmIO


## 相关

在用户空间，或者应用编程领域 ，Linux提供了一些API或者系统调用来影响Linux的内核调度器，或者是获取内核调度器的信息。比如可以获取或者设置进程的调度策略、优先级，获取CPU时间片大小的信息。


## 调度

### 介绍

从2.4的非抢占内核发展到今天的可抢占内核，普通进程的调度算法也从O(1)到CFS，一个好的调度算法应当考虑以下几个方面：

* 公平：保证每个进程得到合理的CPU时间。
* 高效：使CPU保持忙碌状态，即总是有进程在CPU上运行。
* 响应时间：使交互用户的响应时间尽可能短。
* 周转时间：使批处理用户等待输出的时间尽可能短。
* 吞吐量：使单位时间内处理的进程数量尽可能多。
* 负载均衡：在多核多处理器系统中提供更高的性能

而在实际的内核中实时进程和普通进程为并存,因此调度算法至少存在两类.


### 调度策略

```
/*
 * Scheduling policies
 */
#define SCHED_NORMAL        0                                                              
#define SCHED_FIFO      1
#define SCHED_RR        2
#define SCHED_BATCH     3
/* SCHED_ISO: reserved but not implemented yet */
#define SCHED_IDLE      5
/* Can be ORed in to make sure the process is reverted back to SCHED_NORMAL on fork */
#define SCHED_RESET_ON_FORK     0x40000000
```

在linux中调度策略主要有五种:

1. SCHED_FIFO

实时进程使用的调度策略，此调度策略的进程一旦使用CPU则一直运行，直到有比其更高优先级的实时进程进入队列，或者其自动放弃CPU，适用于时间性要求比较高，但每次运行时间比较短的进程。

2. SCHED_RR

实时进程使用的时间片轮转法策略，实时进程的时间片用完后，调度器将其放到队列末尾，这样每个实时进程都可以执行一段时间。适用于每次运行时间比较长的实时进程。

3. SCHED_NORMAL

普通进程使用的调度策略，Linux默认的调度策略，其值为0 非实时进程的调度策略,也就是分时调度策略。分时进程则通过nice和counter值决定权值，nice越小，counter越大，被调度的概率越大，也就是曾经使用了cpu最少的进程将会得到优先调

4. SCHED_BATCH

除了不能抢占外与常规任务一样，允许任务运行更长时间，更好地使用高速缓存，适合于成批处理的工作。

5. SCHED_IDLE

它甚至比nice 19还有弱，为了避免优先级反转使用,这是由CFS导入的新等级。CPU空闲时，即SCHED_IDLE等级以外处于可执行状态的进程消失时，将被赋予执行权。也就是它将成为优先级最低的进程。


### 调度器类

| 名称  | 作用 |
| ---- | ---- |
| idle_sched_class | pid=0, 调度类属于：idel_sched_class，所以在ps里面是看不到的。一般运行在开机过程和cpu异常的时候做dump。|
| stop_sched_class |优先级最高的线程，会中断所有其他线程，且不会被其他任务打断。作用：1.发生在cpu_stop_cpu_callback 进行cpu之间任务migration；2.HOTPLUG_CPU的情况下关闭任务。|
| rt_sched_class   | RT，作用：实时线程 |
| fair_sched_class | CFS（公平），作用：一般常规线程 |

不明白?

### 调度

首先，我们需要清楚，什么样的进程会进入调度器进行选择，就是处于TASK_RUNNING状态的进程，而其他状态下的进程都不会进入调度器进行调度。系统发生调度的时机如下

* 调用cond_resched()时(mmc中mmc_delay())
* 显式调用schedule()时
* 从系统调用或者异常中断返回用户空间时
* 从中断上下文返回用户空间时

当开启内核抢占(默认开启)时，会多出几个调度时机，如下

* 在系统调用或者异常中断上下文中调用preempt_enable()时(多次调用preempt_enable()时，系统只会在最后一次调用时会调度)
* 在中断上下文中，从中断处理函数返回到可抢占的上下文时(这里是中断下半部，中断上半部实际上会关中断，而新的中断只会被登记，由于上半部处理很快，上半部处理完成后才会执行新的中断信号，这样就形成了中断可重入)

而在系统启动调度器初始化时会初始化一个调度定时器，调度定时器每隔一定时间执行一个中断，在中断会对当前运行进程运行时间进行更新，如果进程需要被调度，在调度定时器中断中会设置一个调度标志位，之后从定时器中断返回，*因为上面已经提到从中断上下文返回时是有调度时机的，在内核源码的汇编代码中所有中断返回处理都必须去判断调度标志位是否设置，如设置则执行schedule()进行调度*。而我们知道实时进程和普通进程是共存的，调度器是怎么协调它们之间的调度的呢，其实很简单，每次调度时，会先在实时进程运行队列中查看是否有可运行的实时进程，如果没有，再去普通进程运行队列找下一个可运行的普通进程，如果也没有，则调度器会使用idle进程进行运行。之后的章节会放上代码进行详细说明。

系统并不是每时每刻都允许调度的发生，当处于硬中断期间的时候，调度是被系统禁止的，之后硬中断过后才重新允许调度。而对于异常，系统并不会禁止调度，也就是在异常上下文中，系统是有可能发生调度的。


### 初始化

sched_init ---- FAIR_GROUP_SCHED

1. 初始化一个task_groups链表

在start_kernel中对调度器进行初始化的函数就是sched_init，其主要工作为

1. 对相关数据结构分配内存
2. 初始化root_task_group
3. 初始化每个CPU的rq队列(包括其中的cfs队列和实时进程队列)
4. 将init_task进程转变为idle进程
　　需要说明的是init_task在这里会被转变为idle进程，但是它还会继续执行初始化工作，相当于这里只是给init_task挂个idle进程的名号，它其实还是init_task进程，只有到最后init_task进程开启了kernel_init和kthreadd进程之后，才转变为真正意义上的idle进程。


### 相关流程

#### 数据结构

每个CPU对应包含一个运行队列结构(struct rq)，而每个运行队列又包含有其自己的实时进程运行队列(struct rt_rq)、普通进程运行队列(struct cfs_rq)，也就是说每个CPU都有他们自己的实时进程运行队列及普通进程运行队列

1. 运行队列(struct rq)
一个CPU一个运行队列

定义: DEFINE_PER_CPU_SHARED_ALIGNED(struct rq, runqueues);

相关操作:

``` C
#define cpu_rq(cpu)     (&per_cpu(runqueues, (cpu))) 	//返回指定CPU的运行队列指针.
#define this_rq()       (&__get_cpu_var(runqueues))	//返回当前CPU的运行队列指针.	
#define task_rq(p)      cpu_rq(task_cpu(p))		//返回给定任务的运行队列.
#define cpu_curr(cpu)       (cpu_rq(cpu)->curr)		//返回指定CPU的当前进程
#define raw_rq()        (&__raw_get_cpu_var(runqueues))
```


2. 调度组(struct task_group)

``` C
/* 进程组，用于实现组调度 */
struct task_group {
    /* 用于进程找到其所属进程组结构 */
    struct cgroup_subsys_state css;

#ifdef CONFIG_FAIR_GROUP_SCHED
    /* CFS调度器的进程组变量，在 alloc_fair_sched_group() 中进程初始化及分配内存 */
    /* 该进程组在每个CPU上都有对应的一个调度实体，因为有可能此进程组同时在两个CPU上运行(它的A进程在CPU0上运行，B进程在CPU1上运行) */
    struct sched_entity **se;
    /* 进程组在每个CPU上都有一个CFS运行队列(为什么需要，稍后解释) */
    struct cfs_rq **cfs_rq;
    /* 用于保存优先级默认为NICE 0的优先级 */
    unsigned long shares;

#ifdef    CONFIG_SMP
    atomic_long_t load_avg;
    atomic_t runnable_avg;
#endif
#endif

#ifdef CONFIG_RT_GROUP_SCHED
    /* 实时进程调度器的进程组变量，同 CFS */
    struct sched_rt_entity **rt_se;
    struct rt_rq **rt_rq;

    struct rt_bandwidth rt_bandwidth;
#endif

    struct rcu_head rcu;
    /* 用于建立进程链表(属于此调度组的进程链表) */
    struct list_head list;

    /* 指向其上层的进程组，每一层的进程组都是它上一层进程组的运行队列的一个调度实体，在同一层中，进程组和进程被同等对待 */
    struct task_group *parent;
    /* 进程组的兄弟结点链表 */
    struct list_head siblings;
    /* 进程组的儿子结点链表 */
    struct list_head children;

#ifdef CONFIG_SCHED_AUTOGROUP
    struct autogroup *autogroup;
#endif

    struct cfs_bandwidth cfs_bandwidth;
};
```


3. 调度实体(struct sched_entity)

``` C
/* 一个调度实体(红黑树的一个结点)，其包含一组或一个指定的进程，包含一个自己的运行队列，一个父亲指针，一个指向需要调度的运行队列指针 */
struct sched_entity {
    /* 权重，在数组prio_to_weight[]包含优先级转权重的数值 */
    struct load_weight    load;        /* for load-balancing */
    /* 实体在红黑树对应的结点信息 */
    struct rb_node        run_node;    
    /* 实体所在的进程组 */
    struct list_head    group_node;
    /* 实体是否处于红黑树运行队列中 */
    unsigned int        on_rq;

    /* 开始运行时间 */
    u64            exec_start;
    /* 总运行时间 */
    u64            sum_exec_runtime;
    /* 虚拟运行时间，在时间中断或者任务状态发生改变时会更新
     * 其会不停增长，增长速度与load权重成反比，load越高，增长速度越慢，就越可能处于红黑树最左边被调度
     * 每次时钟中断都会修改其值
     * 具体见calc_delta_fair()函数
     */
    u64            vruntime;
    /* 进程在切换进CPU时的sum_exec_runtime值 */
    u64            prev_sum_exec_runtime;

    /* 此调度实体中进程移到其他CPU组的数量 */
    u64            nr_migrations;

#ifdef CONFIG_SCHEDSTATS
    /* 用于统计一些数据 */
    struct sched_statistics statistics;
#endif

#ifdef CONFIG_FAIR_GROUP_SCHED
    /* 代表此进程组的深度，每个进程组都比其parent调度组深度大1 */
    int            depth;
    /* 父亲调度实体指针，如果是进程则指向其运行队列的调度实体，如果是进程组则指向其上一个进程组的调度实体
     * 在 set_task_rq 函数中设置
     */
    struct sched_entity    *parent;
    /* 实体所处红黑树运行队列 */
    struct cfs_rq        *cfs_rq;        
    /* 实体的红黑树运行队列，如果为NULL表明其是一个进程，若非NULL表明其是调度组 */
    struct cfs_rq        *my_q;
#endif

#ifdef CONFIG_SMP
    /* Per-entity load-tracking */
    struct sched_avg    avg;
#endif
};
```

4. task_struct中调度相关信息

``` C
struct task_struct {
    ........
    /* 表示是否在运行队列 */
    int on_rq;

    /* 进程优先级 
     * prio: 动态优先级，范围为100~139，与静态优先级和补偿(bonus)有关
     * static_prio: 静态优先级，static_prio = 100 + nice + 20 (nice值为-20~19,所以static_prio值为100~139)
     * normal_prio: 没有受优先级继承影响的常规优先级，具体见normal_prio函数，跟属于什么类型的进程有关
     */
    int prio, static_prio, normal_prio;
    /* 实时进程优先级 */
    unsigned int rt_priority;
    /* 调度类，调度处理函数类 */
    const struct sched_class *sched_class;
    /* 调度实体(红黑树的一个结点) */
    struct sched_entity se;
    /* 调度实体(实时调度使用) */
    struct sched_rt_entity rt;
#ifdef CONFIG_CGROUP_SCHED
    /* 指向其所在进程组 */
    struct task_group *sched_task_group;
#endif
    ........
}
```

5. 调度类(struct sched_class)

``` C
struct sched_class {
    /* 下一优先级的调度类
     * 调度类优先级顺序: stop_sched_class -> dl_sched_class -> rt_sched_class -> fair_sched_class -> idle_sched_class
     */
    const struct sched_class *next;

    /* 将进程加入到运行队列中，即将调度实体（进程）放入红黑树中，并对 nr_running 变量加1 */
    void (*enqueue_task) (struct rq *rq, struct task_struct *p, int flags);
    /* 从运行队列中删除进程，并对 nr_running 变量中减1 */
    void (*dequeue_task) (struct rq *rq, struct task_struct *p, int flags);
    /* 放弃CPU，在 compat_yield sysctl 关闭的情况下，该函数实际上执行先出队后入队；在这种情况下，它将调度实体放在红黑树的最右端 */
    void (*yield_task) (struct rq *rq);
    bool (*yield_to_task) (struct rq *rq, struct task_struct *p, bool preempt);

    /* 检查当前进程是否可被新进程抢占 */
    void (*check_preempt_curr) (struct rq *rq, struct task_struct *p, int flags);

    /*
     * It is the responsibility of the pick_next_task() method that will
     * return the next task to call put_prev_task() on the @prev task or
     * something equivalent.
     *
     * May return RETRY_TASK when it finds a higher prio class has runnable
     * tasks.
     */
    /* 选择下一个应该要运行的进程运行 */
    struct task_struct * (*pick_next_task) (struct rq *rq,
                        struct task_struct *prev);
    /* 将进程放回运行队列 */
    void (*put_prev_task) (struct rq *rq, struct task_struct *p);

#ifdef CONFIG_SMP
    /* 为进程选择一个合适的CPU */
    int (*select_task_rq)(struct task_struct *p, int task_cpu, int sd_flag, int flags);
    /* 迁移任务到另一个CPU */
    void (*migrate_task_rq)(struct task_struct *p, int next_cpu);
    /* 用于上下文切换后 */
    void (*post_schedule) (struct rq *this_rq);
    /* 用于进程唤醒 */
    void (*task_waking) (struct task_struct *task);
    void (*task_woken) (struct rq *this_rq, struct task_struct *task);
    /* 修改进程的CPU亲和力(affinity) */
    void (*set_cpus_allowed)(struct task_struct *p,
                 const struct cpumask *newmask);
    /* 启动运行队列 */
    void (*rq_online)(struct rq *rq);
    /* 禁止运行队列 */
    void (*rq_offline)(struct rq *rq);
#endif
    /* 当进程改变它的调度类或进程组时被调用 */
    void (*set_curr_task) (struct rq *rq);
    /* 该函数通常调用自 time tick 函数；它可能引起进程切换。这将驱动运行时（running）抢占 */
    void (*task_tick) (struct rq *rq, struct task_struct *p, int queued);
    /* 在进程创建时调用，不同调度策略的进程初始化不一样 */
    void (*task_fork) (struct task_struct *p);
    /* 在进程退出时会使用 */
    void (*task_dead) (struct task_struct *p);

    /* 用于进程切换 */
    void (*switched_from) (struct rq *this_rq, struct task_struct *task);
    void (*switched_to) (struct rq *this_rq, struct task_struct *task);
    /* 改变优先级 */
    void (*prio_changed) (struct rq *this_rq, struct task_struct *task,
             int oldprio);

    unsigned int (*get_rr_interval) (struct rq *rq,
                     struct task_struct *task);

    void (*update_curr) (struct rq *rq);

#ifdef CONFIG_FAIR_GROUP_SCHED
    void (*task_move_group) (struct task_struct *p, int on_rq);
#endif
};
```

在内核中的所有现有调度类是按优先级的排序的列表中调度类。被称为该结构的第一个成员下一步是一个指针，指向下一个调度类具有较低的优先级，该列表中。使用列表来优先考虑不同类型在别人面前的任务。在当前Linux 版本中，其初始流程如下所示︰

``` C
stop_sched_class → rt_sched_class → fair_sched_class → idle_sched_class → NULL
```

idle模式----swapper进程

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

PREEMPT_ACTIVE  

进程一旦调用了schedule，如果再次被调度运行，那么有下面几种可能：
1.状态为TASK_RUNNING，处于运行队列，那么它肯定有机会再运行；
2.处于睡眠队列，那么一旦条件满足被唤醒，那么它就会运行。
那么如果一个进程被抢占的话，而且它不在运行队列，那么怎么再让它运行呢？答案是它不能运行了。为了避免这种情况，就必须避免处于非TASK_RUNNING的进程被抢占的进程不被赶出运行队列，也就是下面的代码，schedule的代码：

if (prev->state && !(preempt_count() & PREEMPT_ACTIVE)) {

switch_count = &prev->nvcsw;

if (unlikely((prev->state & TASK_INTERRUPTIBLE) && unlikely(signal_pending(prev))))

prev->state = TASK_RUNNING;

else {

if (prev->state == TASK_UNINTERRUPTIBLE)

rq->nr_uninterruptible++;

deactivate_task(prev, rq);

}

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
遇到的等待队列
drivers/base/dd.c
static DECLARE_WAIT_QUEUE_HEAD(probe_waitqueue); //设备驱动全部加载完成通过probe_waitqueue队列来同步各个驱动的probe



O(1)是多队列调度器，每个处理器都有一条自己的运行队列







## 多核进程管理

## numa

非均匀存储器存取（Nonuniform-Memory-Access，简称NUMA）模型





## 参考

1. [调度器](http://www.cnblogs.com/tolimit/p/4303052.html)
2. http://blog.csdn.net/lsl180236/article/details/51155373
3. http://blog.chinaunix.net/uid-27052262-id-3239260.html
4. http://www.cnblogs.com/Nancy5104/p/5389990.html
