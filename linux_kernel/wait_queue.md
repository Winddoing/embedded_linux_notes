# 等待队列


## 数据结构

```
struct __wait_queue {                             
    unsigned int flags;                           
#define WQ_FLAG_EXCLUSIVE   0x01                  
    void *private;                                
    wait_queue_func_t func;                       
    struct list_head task_list;                   
};                                                
``` 








## 参考

1. [Linux内核的等待队列](http://linux.chinaunix.net/techdoc/system/2009/05/03/1109864.shtml)
