# 线程

## 线程创建的不同接口

1. kernel_thread()
2. kthread_create()
3. kthread_run()


## 创建

int kernel_thread (int ( * fn )( void * ), void * arg, unsigned long flags);

　　kernel_thread函数的作用是产生一个新的线程
　　内核线程实际上就是一个共享父进程地址空间的进程,它有自己的系统堆栈.
　　内核线程和进程都是通过do_fork()函数来产生的，系统中规定的最大进程数与线程数由fork_init来决定:

此函数用于创建一个新的内核线程，即完成在内核态创建一个子进程。 首先在内核地址空间为此线程分配内存空间，然后初始化与此线程相关的变量，最后调用do_fork( )函数创建一个新的线程，并返回新进程的进程号。

 

参数int (*fn) (void *) 是一个函数指针，即第一个参数是一个函数，此函数是此进程执行时执行的函数，此函数返回值为int型，参数是一个void　指针。

参数 void * arg是一个void 型指针，是传递给第一个参数所代表函数的参数，即子进程执行时函数的参数。

参数 flags 是创建新进程的标志位，在内核文件对此有定义，可能取值如下所列：

	/include/sched.h:

	#define CSIGNAL  0x000000ff     /*进程退出信号*/

	#define CLONE_VM     0x00000100    /*进程间共享虚拟区 */

	#define CLONE_FS      0x00000200    /*进程间共享文件系统信息*/

	#define CLONE_FILES 0x00000400    /*进程间共享文件*/

	#define CLONE_SIGHAND  0x00000800    /*共享信号处理函数及阻塞信号*/

	#define CLONE_PTRACE 0x00002000         /*持续追踪子进程* /

	#define CLONE_VFORK  0x00004000        /*子进程内存空间释放时唤醒父进程 */

	#define CLONE_PARENT    0x00008000    /* 克隆进程共用同一父进程*/

	#define CLONE_THREAD   0x00010000    /*相同的进程组*/

	#define CLONE_NEWNS     0x00020000    /*新命名空间组*/

	#define CLONE_SYSVSEM 0x00040000    /*共享系统的V SEM_UNDO变量*/

	#define CLONE_SETTLS     0x00080000    /*为子进程创建新的TLS值*/

	#define CLONE_PARENT_SETTID    0x00100000    /*设置父进程的TID值 */

	#define CLONE_CHILD_CLEARTID  0x00200000    /*清除子进程的TID值*/

	#define CLONE_DETACHED             0x00400000    /*此标志位没有被使用 */

	#define CLONE_UNTRACED             0x00800000    /*进程禁止追踪*/

	#define CLONE_CHILD_SETTID       0x01000000    /*设置子进程的TID值*/

	#define CLONE_STOPPED         0x02000000    /*从停止状态启动*/

	#define CLONE_NEWUTS          0x04000000    /*新uts命名组*/

	#define CLONE_NEWIPC           0x08000000    /*新ipcs*/

	#define CLONE_NEWUSER              0x10000000    /*新用户命名空间*/

	#define CLONE_NEWPID           0x20000000    /*新pid命名空间*/

	#define CLONE_NEWNET          0x40000000    /*新网络命名空间*/

	#define CLONE_IO              0x80000000          /*克隆输入输出上下文内容 */


## do_fork

	long do_fork(unsigned long clone_flags, 
		  unsigned long stack_start, 
		  unsigned long stack_size, 
		  int __user *parent_tidptr, 
		  int __user *child_tidptr);

stack_start: fork进程的函数指针

### 分配资源


### 启动线程

### 进程号


