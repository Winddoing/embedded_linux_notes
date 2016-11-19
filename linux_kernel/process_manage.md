# 进程管理

## 进程

在linux中，进程主要分为两种，一种为实时进程，一种为普通进程

* 实时进程：对系统的响应时间要求很高，它们需要短的响应时间，并且这个时间的变化非常小，典型的实时进程有音乐播放器，视频播放器等。
* 普通进程：包括交互进程和非交互进程，交互进程如文本编辑器，它会不断的休眠，又不断地通过鼠标键盘进行唤醒，而非交互进程就如后台维护进程，他们对IO，响应时间没有很高的要求，比如编译器。
　　
它们在linux内核运行时是共存的，实时进程的优先级为0~99，实时进程优先级不会在运行期间改变(静态优先级)，而普通进程的优先级为100~139，普通进程的优先级会在内核运行期间进行相应的改变(动态优先级)。

## 优先级

严格地说，对于优先级对于实时进程和普通进程的意义是不一样的。

1. 在一定程度上，实时进程优先级高，实时进程存在，就没有普通进程占用CPU的机会

2. 对于实时进程而言，高优先级的进程存在，低优先级的进程是轮不上的，没机会跑在CPU上，所谓实时进程的调度策略,指的是相同优先级之间的调度策略。如果是FIFO实时进程在占用CPU，除非出现以下事情，否则FIFO一条道跑到黑。

 * FIFO进程良心发现，调用了系统调用sched_yield 自愿让出CPU
 * 更高优先级的进程横空出世，抢占FIFO进程的CPU。有些人觉得很奇怪，怎么FIFO占着CPU，为啥还能有更高优先级的进程出现呢。别忘记，我们是多核多CPU ,如果其他CPU上出现了一个比FIFO优先级高的进程，可能会push到FIFO进程所在的CPU上。
 * FIFO进程停止（TASK_STOPPED or TASK_TRACED状态）或者被杀死（EXIT_ZOMBIE or EXIT_DEAD状态）
 * FIFO进程执行了阻塞调用并进入睡眠（TASK_INTERRUPTIBLE OR TASK_UNINTERRUPTIBLE）。
    
如果是进程的调度策略是时间片轮转RR，那么，除了前面提到的abcd，RR实时进程耗尽自己的时间片后，自动退到对应优先级实时队列的队尾，重新调度。


在用户层或者应用层，1表示优先级最低，99表示优先级最高。但是在内核中，[0,99]表示的实时进程的优先级，0最高，99最低。[100,139]是普通进程折腾的范围。应用层比较天真率直，就看大小，数字大，则优先级高。ps查看进程的优先级也是如此。有意思的是，应用层实时进程最高优先级的99，在ps看进程优先级的时候，输出的是139.



对于普通进程，是通过nice系统调用来调整优先级的。从内核角度讲[100,139]是普通进程的优先级的范围，100最高，139最低，默认是120。普通进程的优先级的作用和实时进程不同，普通进程优先级表示的是占的CPU时间。深入linux内核架构中提到，普通优先级越高（100最高，139最低），享受的CPU time越多，相邻的两个优先级，高一级的进程比低一级的进程多占用10%的CPU，比如内核优先级数值为120的进程要比数值是121的进程多占用10%的CPU。


## 生命周期

1. 诞生

2. 调度

3. 消亡




## do_fork


1. 实时拷贝的实现 --- 写时复制  
2. 父进程返回子进程ID


什么时候复制完成,什么时候执行

const struct cred __rcu *real_cred;  //cred负责保存进程安全上下文

### 流程

1. do_fork
long do_fork(unsigned long clone_flags,
          unsigned long stack_start,
          unsigned long stack_size,
          int __user *parent_tidptr,
          int __user *child_tidptr)

参数:
   clone_flags: 进程的相关标志,属性
   stack_start: 子进程用户态的堆栈地址????
   stack_size: 堆栈大小
   parent_tidptr: 父进程在用户态下pid的地址，该参数在CLONE_PARENT_SETTID标志被设定时有意义
   child_tidptr: 子进程在用户态下pid的地址，该参数在CLONE_CHILD_SETTID标志被设定时有意义
2. copy_process()
主要完成进程的拷贝,包括task_struct,thread_info,内核栈

3. dup_task_struct()

``` C
 int node = tsk_fork_get_node(orig);
 int err;

 tsk = alloc_task_struct_node(node);
 if (!tsk)
     return NULL;

 ti = alloc_thread_info_node(tsk, node); //进程的栈空间是独立的
 if (!ti)
     goto free_tsk;
 
 err = arch_dup_task_struct(tsk, orig);
```
开始拷贝,子进程拥有自己独立内核栈和thread_info,而task_struct中的其他资源均为父进程,此时还包括进程的地址空间.

子进程中的栈空间是拷贝父进程的栈空间,setup_thread_stack(tsk, orig)

stackend = end_of_stack(tsk);
*stackend = STACK_END_MAGIC;  
设置栈底的检测,主要方式栈底回退时溢出.

account_kernel_stack(ti, 1);

systemtap


**由于每个进程的地址空间时独享的,因此子进程何时修改自己进程的地址空间**

4. copy_thread()
copy_thread设置子进程的栈信息，并将子进程的返回值置为0(返回值保存在v0中)

## 参考

[linux进程调度之 FIFO 和 RR 调度策略](http://blog.chinaunix.net/uid-24774106-id-3379478.html)

