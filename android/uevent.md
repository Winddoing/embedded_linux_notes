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

### 示例

kobject_uevent


## Android管理

file: hardware/libhardware_legacy/uevent/uevent.c






