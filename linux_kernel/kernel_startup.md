# 内核启动

进入C语言部分

## 流程

start_kernel -> rest_init

## start_kernel


## rest_init

启动内核的第一个线程kernel_init

static noinline void __init_refok rest_init(void)                                   
{                                                                                   
    int pid;                                                                        
                                                                                    
    rcu_scheduler_starting(); //内核RCU锁机制调度启动,因为下面就要用到                                                     

    /*                                                                              
     * We need to spawn init first so that it obtains pid 1, however                
     * the init task will end up wanting to create kthreads, which, if              
     * we schedule it before we create kthreadd, will OOPS.                         
     */          
                                                                   
    kernel_thread(kernel_init, NULL, CLONE_FS | CLONE_SIGHAND);                     
    numa_default_policy();   //设定NUMA系统的内存访问策略为默认                                                        

    pid = kernel_thread(kthreadd, NULL, CLONE_FS | CLONE_FILES);                    
    rcu_read_lock();                                                                
    kthreadd_task = find_task_by_pid_ns(pid, &init_pid_ns);//获取kthreadd的线程信息，获取完成说明kthreadd已经创建成功。                        
    rcu_read_unlock();                                                              
    complete(&kthreadd_done);                                                       
                                                                                    
    /*                                                                              
     * The boot idle thread must execute schedule()                                 
     * at least once to get things moving:                                          
     */                                                                             
    init_idle_bootup_task(current); //设置当前进程为idle（闲置）进程类
    schedule_preempt_disabled(); //执行一次调度,并禁用抢占 
    /* Call into cpu_idle with preempt disabled */                                  
    cpu_startup_entry(CPUHP_ONLINE);                                                
}

1. 创建kernel_init线程

2. 创建kthreadd内核线程，它的作用是管理和调度其它内核线程。
它循环运行一个叫做kthreadd的函数，该函数的作用是运行kthread_create_list全局链表中维护的内核线程。
调用kthread_create创建一个kthread，它会被加入到kthread_create_list 链表中；
被执行过的kthread会从kthread_create_list链表中删除；
且kthreadd会不断调用scheduler函数让出CPU。此线程不可关闭。

### task_struct链表kthread_create_list的管理:(删,增)

a. 判断kthread_create_list链表是否为空,为空让出CPU
b. 链表中存在新的task_struct时,删除task_struct,并创建新的kthread线程处理,它会将所在的线程进入睡眠状态.
c. 使用kthread_create(或kthread_run)创建线程时,将调用kthread_create_on_node对kthread_create_list链表进行操作-添加
(kernel_thread创建的线程是否加入kthread_create_list链表)

### kthreadd线程的管理

为提高CPU利用率,kthread_create_list链表是否为空,为空让出CPU

1. 何时让出

kthread_create_list链表为空

2. 何时进入该线程进行处理

                                                                                    
## 内核启动的线程

0. cpu_idle
1. kernel_init
2. kthreadd

Linux下有3个特殊的进程，idle进程(PID=0), init进程(PID=1)和kthreadd(PID=2)

* idle进程(也称swapper)由系统自动创建, 运行在内核态

idle进程其pid=0，其前身是系统创建的第一个进程，也是唯一一个没有通过fork或者kernel_thread产生的进程。完成加载系统后，演变为进程调度、交换

* init进程由idle通过kernel_thread创建，在内核空间完成初始化后, 加载init程序, 并最终用户空间

由0进程创建，完成系统的初始化. 是系统中所有其它用户进程的祖先进程
Linux中的所有进程都是有init进程创建并运行的。首先Linux内核启动，然后在用户空间中启动init进程，再启动其他系统进程。在系统启动完成完成后，init将变为守护进程监视系统其他进程。

* kthreadd进程由idle通过kernel_thread创建，并始终运行在内核空间, 负责所有内核线程的调度和管理

## init线程

1. 设置CPU和系统相关,如SMP,NUMA
2. 初始化设备驱动 do_basic_setup();
static void __init do_basic_setup(void)                
{   
    cpuset_init_smp();
    usermodehelper_init();
    shmem_init();
    driver_init(); //初始化与设备驱动相关的设备模型等
    init_irq_proc();
    do_ctors();
    usermodehelper_enable();
    do_initcalls();   //调用所有通过module_init静态编译进入内核的设备驱动程序
}   

3. 挂载文件系统prepare_namespace

4. 通过run_init_process启动用户空间的init进程

### ps

=====>$ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0  33736  2024 ?        Ss    9月26   0:02 /sbin/init
root         2  0.0  0.0      0     0 ?        S     9月26   0:01 [kthreadd]


