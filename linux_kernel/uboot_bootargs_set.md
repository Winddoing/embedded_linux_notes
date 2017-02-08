# uboot传入的启动参数


## __setup

``` C
#define __setup_param(str, unique_id, fn, early)            \
    static const char __setup_str_##unique_id[] __initconst \
        __aligned(1) = str; \
    static struct obs_kernel_param __setup_##unique_id  \
        __used __section(.init.setup)           \
        __attribute__((aligned((sizeof(long)))))    \
        = { __setup_str_##unique_id, fn, early }

#define __setup(str, fn)                    \                           
    __setup_param(str, fn, fn, 0)
```

处理kernel启动参数的函数和数据结构

>通过`__setup`宏定义obs_kernel_param结构变量都被放入`.init.setup`段中，这样一来实际是使.init.setup段变成一张表，Kernel在处理每一个启动参数时，都会来查找这张表，与每一个数据项中的成员str进行比较，如果完全相同，就会调用该数据项的函数指针成员setup_func所指向的函数（该函数是在使用__setup宏定义该变量时传入的函数参数），并将启动参数如root=后面的内容传给该处理函数。

## 应用

``` C
__setup("root=", root_dev_setup);       #./init/do_mounts.c
__setup("rdinit=", rdinit_setup);       #./init/main.c
__setup("rootfstype=", fs_names_setup); #./init/do_mounts.c

```




