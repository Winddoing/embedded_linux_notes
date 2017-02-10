# Android init启动

系统: `Android 7.0`

## init作用

1. 分析及运行init.rc文件
2. 生成设备驱动节点(与linux下udev和mdev的区别)
3. 处理子进程终止
4. 属性服务

## init执行流程

### 设置log输出

``` C
open_devnull_stdio();   //重定向标准输入/输出/错误输出到/dev/_null_ 
klog_init();            //初始化log
klog_set_level(KLOG_NOTICE_LEVEL); 
``` 
dup2的作用，是复制一个函数的描述符，一般用于重定向进程的stdin，stdout，stderr。

### 事件监控

``` C
epoll_fd = epoll_create1(EPOLL_CLOEXEC); //创建poll事件监控
```

事件注册:

``` C
signal_handler_init();       //处理子进程的终止信号SIGCHLD

...

start_property_service();    //设置相关属性服务

...

am.QueueBuiltinAction(keychord_init_action, "keychord_init");
```

### property_service

定义: bionic/libc/include/sys/_system_properties.h

``` C
#define PROP_PATH_RAMDISK_DEFAULT  "/default.prop"
#define PROP_PATH_SYSTEM_BUILD     "/system/build.prop"                                 
#define PROP_PATH_VENDOR_BUILD     "/vendor/build.prop"
#define PROP_PATH_LOCAL_OVERRIDE   "/data/local.prop"
#define PROP_PATH_FACTORY          "/factory/factory.prop"
```
>系统查看属性设置命令: `getprop`


## init.rc

### init.rc 主要启动的执行流程

init进程的解析流程

``` C
am.QueueEventTrigger("early-init");   //解析执行init.rc中的on early-init

...

am.QueueEventTrigger("init");         //执行on init

...

am.QueueEventTrigger("late-init");    //执行on late-init
```

在init.rc的late-init区段中会触发boot区段,执行boot区段中的命令到`class_start core`,将启动`core`类的服务.

### Action List

on xxx 区段

### Service List

service xxx 区段

### 运行

Action和Service列表的启动流程:

Action list  -->  action(early-init)   -->   action(init)   -->  action(late-init)   -->   action(boot)

Service list  --> service(console)    -->   service(adbd) --->  service(servicemanage)


## 脚本解析

``` C
const BuiltinFunctionMap function_map;
Action::set_function_map(&function_map);

Parser& parser = Parser::GetInstance();
parser.AddSectionParser("service",std::make_unique<ServiceParser>());
parser.AddSectionParser("on", std::make_unique<ActionParser>());
parser.AddSectionParser("import", std::make_unique<ImportParser>());                           
parser.ParseConfig("/init.rc");
```							

## 属性服务

android中的属性服务是全局性的,同样由Key--Value构成.

在系统中运行的所有进程都可以访问属性值,但是不行修改,所有的修改都由init完成.
## 其他

不了解

``` C
// Queue an action that waits for coldboot done so we know ueventd has set up all of /dev...
am.QueueBuiltinAction(wait_for_coldboot_done_action, "wait_for_coldboot_done");               
// ... so that we can start queuing up actions that require stuff from /dev.
am.QueueBuiltinAction(mix_hwrng_into_linux_rng_action, "mix_hwrng_into_linux_rng");
am.QueueBuiltinAction(set_mmap_rnd_bits_action, "set_mmap_rnd_bits");
am.QueueBuiltinAction(keychord_init_action, "keychord_init");
am.QueueBuiltinAction(console_init_action, "console_init");
```

