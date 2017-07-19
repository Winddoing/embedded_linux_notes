# Kernel Panic


## hard lockup

> Kernel panic - not syncing: Watchdog detected hard LOCKUP on cpu 1

**中断被长时间关闭而导致更严重的问题（hard lockup）**

hard lock_up是因为中断被禁掉了，长时间（默认应该是5S）没有打开，这时NMI（Non-maskable interrupt，不可屏蔽中断，简称NMI）会中断当前进程，这时就要分析，当前进程为什么长时间关闭中断，目前我遇到一种情况：

其他进程拿了锁，忘记放锁，另一个线程调用spin_lock_irqsave()，这个函数是先关闭中断，再去申请锁，如果一直拿不到锁，就会触发NMI。这种问题的分析过程是这样的，在crash工具中执行foreach bt，把所有的进程栈都打印出来，看是否有进程在锁内的情况，如果有，分析为什么长时间没有放锁，如果没有进程在锁内，则需要遍历代码，查找所有使用该锁的地方，看是否有忘记放锁的流程。

## soft lockup

**抢占被长时间关闭而导致进程无法调度（soft lockup）**



## 参考

1. [soft lockup和hard lockup介绍](http://blog.csdn.net/panzhenjie/article/details/10074551/)

