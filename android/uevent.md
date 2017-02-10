# uevent

Uevent是内核通知android有状态变化的一种`异步通信`方法，比如USB线插入、拔出，电池电量变化等等。其本质是内核发送（可以通过socket）一个字符串，应用层（android）接收并解释该字符串，获取相应信息。

## kernel实现

file: lib/kobject_uevent.c

### action

``` C
enum kobject_action {
    KOBJ_ADD,
    KOBJ_REMOVE,
    KOBJ_CHANGE,                   
    KOBJ_MOVE,
    KOBJ_ONLINE,
    KOBJ_OFFLINE,
    KOBJ_MAX
};
```

``` C
static const char *kobject_actions[] = {
    [KOBJ_ADD] =        "add",
    [KOBJ_REMOVE] =     "remove",
    [KOBJ_CHANGE] =     "change",
    [KOBJ_MOVE] =       "move",
    [KOBJ_ONLINE] =     "online",
    [KOBJ_OFFLINE] =    "offline",
};
```
### socket

``` C
socket(PF_NETLINK, SOCK_DGRAM | SOCK_CLOEXEC, NETLINK_KOBJECT_UEVENT);
```

>#define NETLINK_KOBJECT_UEVENT    15      /* Kernel messages to userspace */

PF_NETLINK

NETLINK_KOBJECT_UEVENT


### 示例

kobject_uevent

电源管理: drivers/power/charger-manager.c

``` C
kobject_uevent(&cm->dev->kobj, KOBJ_CHANGE); 
```



## Android管理

file: hardware/libhardware_legacy/uevent/uevent.c


## Netlink

Netlink socket是一种Linux特有的socket，用于实现用户进程与内核进程之间通信的一种特殊的进程间通信方式(IPC)，也是网络应用程序与内核通信的最常用的接口。

Netlink是一种在内核和用户应用间进行`双向`数据传输的非常好的方式，用户态应用使用标准的socket API就能使用Netlink提供的强大功能，内核态需要使用专门的内核API来使用Netlink

## 其他

uevent与设备模型中的event的关系


## 参考

1. [Netlink实现热拔插监控 ](http://blog.chinaunix.net/uid-24943863-id-3223000.html)



