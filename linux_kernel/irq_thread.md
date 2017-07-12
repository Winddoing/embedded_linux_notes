# 中断线程


## 解决的问题

提高中断处理的实时性

> 在Linux标准内核中，中断是最高优先级的执行单元，不管内核当时处理什么，只要有中断事件，系统将立即响应该事件并执行相应的中断处理代码，除非当时中断关闭。因此，如果系统有严重的网络或I/O负载，中断将非常频繁，后发生的实时任务将很难有机会运行，也就是说，毫无实时性可言。中断线程化之后，中断将作为内核线程运行而且赋予不同的实时优先级，实时任务可以有比中断线程更高的优先级，这样，实时任务就可以作为最高优先级的执行单元来运行，即使在严重负载下仍有实时性保证。

## 操作接口

一般使用接口

```
extern int __must_check                                                         
request_threaded_irq(unsigned int irq, irq_handler_t handler,                   
             irq_handler_t thread_fn,                                           
             unsigned long flags, const char *name, void *dev);                 
```
| 参数 | 说明 |
| -- | -- |
| irq | 中断号 |
| handler | 在发生中断时，首先要执行的code，非常类似于顶半，该函数最后会return IRQ_WAKE_THREAD来唤醒中断线程，一般设为NULL，用系统提供的默认处理 |
| thread_fn | 在线程里执行的handler，非常类似于底半部 |
| flags | 中断类型 |
| name | dev name |
| dev | thread_fn的参数 |

> irqsflags新增加了一个标志，IRQF_ONESHOT，用来标明是在中断线程执行完后在打开该中断，该标志非常有用，否则中断有可能一直在顶半执行，而不能处理中断线程。例如对于gpio level中断，如果不设置该位，在顶半执行完成后，会打开中断，此时由于电平没有变化，马上有执行中断，永远没有机会处理线程。



## 参考

1. [linux中断线程化（转载）](http://sxw0624.blog.163.com/blog/static/20016803820125282654580/)
