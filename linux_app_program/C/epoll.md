# epoll机制



## epoll

为处理大批量句柄而作了改进的`poll`.

## 相关系统调用

### epoll_create

int epoll_create(int size);

### epoll_ctl

int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);

### epoll_wait

int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);

## epoll与select

在linux的网络编程中，很长的时间都在使用select来做事件触发。在linux新的内核中，有了一种替换它的机制，就是epoll。相比于select，epoll最大的好处在于它不会随着监听fd数目的增长而降低效率。因为在内核中的select实现中，它是采用轮询来处理的，轮询的fd数目越多，自然耗时越多。并且，linux/posix_types.h头文件有这样的声明：

>#define__FD_SETSIZE   1024
>表示select最多同时监听1024个fd，当然，可以通过修改头文件再重编译内核来扩大这个数目，但这似乎并不治本。

## 参考

1. [【Linux学习】epoll详解](http://blog.csdn.net/xiajun07061225/article/details/9250579)
2. [ epoll机制:epoll_create、epoll_ctl、epoll_wait、close](http://blog.csdn.net/yusiguyuan/article/details/15027821)
