# regmap


## 数据结构

``` C
struct regmap {                                                       
    struct mutex mutex;                                               
    spinlock_t spinlock;                                              
    regmap_lock lock;                                                 
    regmap_unlock unlock;                                             
    void *lock_arg; /* This is passed to lock/unlock functions */     
                                                                      
    struct device *dev; /* Device we do I/O on */                     
    void *work_buf;     /* Scratch buffer used to format I/O */       
    struct regmap_format format;  /* Buffer format */                 
    const struct regmap_bus *bus;                                     
    void *bus_context;                                                
    const char *name;                                                 

    ... 
    
}
```
> drivers/base/regmap/internal.h
