# 自旋锁(spin_lock)

```
struct A {
    ...
    spinlock_t lock;
    ...
}
```
## 数据结构

### spinlock_t

``` C
typedef struct spinlock {
    union {
        struct raw_spinlock rlock;

#ifdef CONFIG_DEBUG_LOCK_ALLOC
# define LOCK_PADSIZE (offsetof(struct raw_spinlock, dep_map))
        struct {
            u8 __padding[LOCK_PADSIZE];
            struct lockdep_map dep_map;
        };
#endif
    };
} spinlock_t;
```
>include/linux/spinlock_types.h

### raw_spinlock

``` C
typedef struct raw_spinlock {
    arch_spinlock_t raw_lock;
#ifdef CONFIG_GENERIC_LOCKBREAK
    unsigned int break_lock;
#endif
#ifdef CONFIG_DEBUG_SPINLOCK
    unsigned int magic, owner_cpu;
    void *owner;
#endif
#ifdef CONFIG_DEBUG_LOCK_ALLOC
    struct lockdep_map dep_map;
#endif
} raw_spinlock_t;
```
> include/linux/spinlock_types.h

### arch_spinlock_t

``` C
typedef union {
    /*
     * bits  0..15 : serving_now
     * bits 16..31 : ticket
     */
    u32 lock;
    struct {
#ifdef __BIG_ENDIAN
        u16 ticket;
        u16 serving_now;
#else
        u16 serving_now;
        u16 ticket;
#endif
    } h;
} arch_spinlock_t;
```
> arch/mips/include/asm/spinlock_types.h


## 初始化


### 静态

> static DEFINE_SPINLOCK(lock);

```
#define __SPIN_LOCK_INITIALIZER(lockname) \
    { { .rlock = __RAW_SPIN_LOCK_INITIALIZER(lockname) } }

#define __SPIN_LOCK_UNLOCKED(lockname) \
    (spinlock_t ) __SPIN_LOCK_INITIALIZER(lockname)

#define DEFINE_SPINLOCK(x)  spinlock_t x = __SPIN_LOCK_UNLOCKED(x)
```
> include/linux/spinlock_types.h

### 动态

> spin_lock_init(&lock);

``` C
#define spin_lock_init(_lock)               \
do {                            \
    spinlock_check(_lock);              \
    raw_spin_lock_init(&(_lock)->rlock);        \
} while (0)
```
> include/linux/spinlock.h

```
# define raw_spin_lock_init(lock)               \
    do { *(lock) = __RAW_SPIN_LOCK_UNLOCKED(lock); } while (0)
```

```
#define __RAW_SPIN_LOCK_INITIALIZER(lockname)   \
    {                   \
    .raw_lock = __ARCH_SPIN_LOCK_UNLOCKED,  \   //  #define __ARCH_SPIN_LOCK_UNLOCKED   { .lock = 0 }
    SPIN_DEBUG_INIT(lockname)       \
    SPIN_DEP_MAP_INIT(lockname) }

#define __RAW_SPIN_LOCK_UNLOCKED(lockname)  \
    (raw_spinlock_t) __RAW_SPIN_LOCK_INITIALIZER(lockname)
```
> include/linux/spinlock_types.h

```
#define __ARCH_SPIN_LOCK_UNLOCKED   { .lock = 0 }
```
> arch/mips/include/asm/spinlock_types.h

### 目的

> 将lock初始化为0,表示锁未被占用

## spin_lock


```
spin_lock
    |-> raw_spin_lock(&lock->rlock)
```

``` C

#define raw_spin_lock(lock) _raw_spin_lock(lock)

void __lockfunc _raw_spin_lock(raw_spinlock_t *lock)
{
    __raw_spin_lock(lock);
}
EXPORT_SYMBOL(_raw_spin_lock);

static inline void __raw_spin_lock(raw_spinlock_t *lock)
{
    preempt_disable();           //关闭内核抢占
    spin_acquire(&lock->dep_map, 0, 0, _RET_IP_);   //debug, CONFIG_DEBUG_LOCK_ALLOC
    LOCK_CONTENDED(lock, do_raw_spin_trylock, do_raw_spin_lock);
}

#define LOCK_CONTENDED(_lock, try, lock) \
    lock(_lock)

void do_raw_spin_lock(raw_spinlock_t *lock)
{
    debug_spin_lock_before(lock);
    if (unlikely(!arch_spin_trylock(&lock->raw_lock)))  //实现
        __spin_lock_debug(lock);
    debug_spin_lock_after(lock);
}

```

### MIPS实现

``` C
static inline unsigned int arch_spin_trylock(arch_spinlock_t *lock)
{
	int tmp, tmp2, tmp3;
	int inc = 0x10000;

	__asm__ __volatile__ (
			"   .set push       # arch_spin_trylock \n"
			"   .set noreorder                  \n"
			"                           \n"
			"1: ll  %[ticket], %[ticket_ptr]        \n"
			"   srl %[my_ticket], %[ticket], 16     \n"
			"   andi    %[now_serving], %[ticket], 0xffff   \n"
			"   bne %[my_ticket], %[now_serving], 3f    \n"
			"    addu   %[ticket], %[ticket], %[inc]        \n"
			"   sc  %[ticket], %[ticket_ptr]        \n"
			"   beqz    %[ticket], 1b               \n"
			"    li %[ticket], 1                \n"
			"2:                         \n"
			"   .subsection 2                   \n"
			"3: b   2b                  \n"
			"    li %[ticket], 0                \n"
			"   .previous                   \n"
			"   .set pop                    \n"
			: [ticket_ptr] "+m" (lock->lock),
			[ticket] "=&r" (tmp),
			[my_ticket] "=&r" (tmp2),
			[now_serving] "=&r" (tmp3)
				: [inc] "r" (inc));

	smp_llsc_mb();
}
```
> `R10000_LLSC_WAR`未定义,其值为0, `arch/mips/xburst2/soc-x2000/include/war.h`

## spin_unlock


```
spin_unlock
    |-> raw_spin_unlock(&lock->rlock);
        |-> #define raw_spin_unlock(lock)       _raw_spin_unlock(lock)
        |-> #define _raw_spin_unlock(lock) __raw_spin_unlock(lock)
        |-> __raw_spin_unlock(raw_spinlock_t *lock)
            |   { spin_release(&lock->dep_map, 1, _RET_IP_);      //debug , CONFIG_DEBUG_LOCK_ALLOC
            |->   do_raw_spin_unlock(lock);                       //
                  preempt_enable();                               //开启内核抢占
                }

            |-> do_raw_spin_unlock
                |-> arch_spin_unlock(&lock->raw_lock);
```

### MIPS实现

``` C
static inline void arch_spin_unlock(arch_spinlock_t *lock)
{
    unsigned int serving_now = lock->h.serving_now + 1;
    wmb();
    lock->h.serving_now = (u16)serving_now;
    nudge_writes();
}
```
> arch/mips/include/asm/spinlock.h



## spin_lock的特点

1. 单处理器非抢占内核下：自旋锁会在编译时被忽略；
2. 单处理器抢占内核下：自旋锁仅仅当作一个设置内核抢占的开关；
3. 多处理器下：此时才能完全发挥出自旋锁的作用，自旋锁在内核中主要用来防止多处理器中并发访问临界区，防止内核抢占造成的竞争。


> 内核中通过`CONFIG_SMP`控制不同的头文件(`<linux/spinlock_api_smp.h>`, `<linux/spinlock_api_up.h>`),来改变接口的具体实现


## spin_lock的dubug


### 内核配置

```
Kernel hacking  ---> 
     -*- Spinlock and rw-lock debugging: basic checks    [CONFIG_DEBUG_SPINLOCK]
     -*- Mutex debugging: basic checks                   [CONFIG_DEBUG_MUTEXES]
     -*- Lock debugging: detect incorrect freeing of live locks       [CONFIG_DEBUG_LOCK_ALLOC]
     [*] Lock debugging: prove locking correctness                    [CONFIG_PROVE_LOCKING]

```




## 参考

1. [spinlock与linux内核调度的关系](http://www.chinabaike.com/2011/0116/175835.html)
2. [Linux 内核的排队自旋锁(FIFO Ticket Spinlock)](https://www.ibm.com/developerworks/cn/linux/l-cn-spinlock/index.html)
3. [Linux内核spin_lock与spin_lock_irq分析](http://blog.csdn.net/zhanglei4214/article/details/6837697)
