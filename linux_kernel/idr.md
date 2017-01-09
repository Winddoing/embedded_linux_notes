# idr机制

``` C
drivers/mmc/core/host.c:57:static DEFINE_IDR(mmc_host_idr);
drivers/mmc/core/host.c:440:	err = idr_alloc(&mmc_host_idr, host, 0, 0, GFP_NOWAIT);
drivers/mmc/core/host.c:561:	idr_remove(&mmc_host_idr, host->index);
```

## idr

IDR机制在Linux内核中指的是整数ID管理机制。实质上来讲，这就是一种将一个整数ID号和一个指针关联在一起的机制。

所谓IDR，其实就是和身份证的含义差不多，我们知道，每个人有一个身份证，身份证只是一串数字，从数字，我们就能知道这个人的信息。同样道理，idr的要完成的任务是给要管理的对象分配一个数字，可以通过这个数字找到要管理的对象。

## struct idr

``` C
struct idr_layer {
    int         prefix; /* the ID prefix of this idr_layer */
    DECLARE_BITMAP(bitmap, IDR_SIZE); /* A zero bit means "space here" */
    struct idr_layer __rcu  *ary[1<<IDR_BITS];
    int         count;  /* When zero, we can release it */
    int         layer;  /* distance from leaf */
    struct rcu_head     rcu_head;
};

struct idr {
    struct idr_layer __rcu  *hint;  /* the last layer allocated from */
    struct idr_layer __rcu  *top;
    struct idr_layer    *id_free;
    int         layers; /* only valid w/o concurrent changes */
    int         id_free_cnt;
    int         cur;    /* current pos for cyclic allocation */
    spinlock_t      lock;
};

#define IDR_INIT(name)                          \
{                                   \
    .lock           = __SPIN_LOCK_UNLOCKED(name.lock),  \
}
#define DEFINE_IDR(name)    struct idr name = IDR_INIT(name)                                                         
```

## 使用

### DEFINE_IDR

### idr_alloc

### idr_remove

