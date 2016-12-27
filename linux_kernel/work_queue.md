# 工作队列

工作队列是一种特殊的线程,主要在初始化一个工作队列的过程中体现.

/*
 * The externally visible workqueue abstraction is an array of
 * per-CPU workqueues:
 */
struct workqueue_struct {                                                              
    unsigned int        flags;      /* I: WQ_* flags */
    union {
        struct cpu_workqueue_struct __percpu    *pcpu;
        struct cpu_workqueue_struct     *single;
        unsigned long               v;
    } cpu_wq;               /* I: cwq's */
    struct list_head    list;       /* W: list of all workqueues */

    struct mutex        flush_mutex;    /* protects wq flushing */
    int         work_color; /* F: current work color */
    int         flush_color;    /* F: current flush color */
    atomic_t        nr_cwqs_to_flush; /* flush in progress */
    struct wq_flusher   *first_flusher; /* F: first flusher */
    struct list_head    flusher_queue;  /* F: flush waiters */
    struct list_head    flusher_overflow; /* F: flush overflow list */

    mayday_mask_t       mayday_mask;    /* cpus requesting rescue */
    struct worker       *rescuer;   /* I: rescue worker */

    int         saved_max_active; /* W: saved cwq max_active */
    const char      *name;      /* I: workqueue name */
#ifdef CONFIG_LOCKDEP
    struct lockdep_map  lockdep_map;
#endif
};

线程池: per cpu thread pool和unbound thread pool

WQ_UNBOUND

如果一个workqueue有WQ_UNBOUND这样的flag，则说明该workqueue上挂入的work处理是考虑到power saving的。如果workqueue没有WQ_UNBOUND flag，则说明该workqueue是per cpu的，这时候，调度哪一个CPU core运行worker thread来处理work已经不是scheduler可以控制的了，这样，也就间接影响了功耗。



* 工作队列的维护者
* workqueue何时被调度


内核中的workqueue,个数,作用

## 延时队列




