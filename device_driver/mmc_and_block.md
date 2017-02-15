# mmc子系统与block层关系

## 整体结构

block  -->   mmc


## 注册block设备

### block层API接口

#### 设备gendisk结构

``` C
struct gendisk *alloc_disk(int minors)
```

#### 生成请求队列

``` C
struct request_queue *blk_init_queue(request_fn_proc *rfn, spinlock_t *lock) 
```
生成一个请求队列。其中rfn函数就是我们用户自己的request函数。生成的这个队列会放到gendisk结构里面，gendisk是来表示一个独立的磁盘设备或分区

#### 配置相关gendisk参数

``` C
err = elevator_change(vram->queue, "noop");      //设置电梯调度算法类型                                    
if (err) {
    printk(KERN_ERR "%s:%d (elevator_init) fail\n",
           __func__, __LINE__);
    goto change_err;
}

blk_queue_max_hw_sectors(vram->queue, 1024);     //  
blk_queue_bounce_limit(vram->queue, BLK_BOUNCE_ANY);

vram->gendisk->major = vram_major;
vram->gendisk->first_minor = atomic_inc_return(&vram_index) - 1;
vram->gendisk->fops = &vram_fops;
sprintf(vram->gendisk->disk_name, "vram");

vram->gendisk->queue = vram->queue;
vram->gendisk->private_data = vram;
set_capacity(vram->gendisk,vram->size / SECTOR_SIZE);
add_disk(vram->gendisk);    //块设备注册入内核

```
### mmc子系统实现


