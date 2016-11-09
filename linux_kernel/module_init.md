# 内核模块编译

	module_init

## 动态加载

	#define module_init(initfn)                 \                   
	    static inline initcall_t __inittest(void)       \
	    { return initfn; }                  \
	    int init_module(void) __attribute__((alias(#initfn)));


## 静态编译

	#define module_init(x)  __initcall(x); 
	#define __initcall(fn) device_initcall(fn)
	#define device_initcall(fn)     __define_initcall(fn, 6)
	#define __define_initcall(fn, id) \                            
	    static initcall_t __initcall_##fn##id __used \
	    __attribute__((__section__(".initcall" #id ".init"))) = fn
	

将加载的驱动__init函数指定到.initcall6段

## 运行

内核启动后,调用do_pre_smp_initcalls遍历.initcall段

start_kernel->rest_init->kernel_init->kernel_init_freeable->do_pre_smp_initcalls

static void __init do_pre_smp_initcalls(void)                             
{
    initcall_t *fn;

    for (fn = __initcall_start; fn < __initcall0_start; fn++)
        do_one_initcall(*fn);
}

