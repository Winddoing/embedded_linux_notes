# DMA

主要应用与内存和设备之间的数据交互.

分类:`一致性DMA`和`流式DMA`

CPU --  DDR  --  device  关系图

## 一致性

一致性DMA的申请使用,主要时所需要的DMA缓存区与该驱动的生命周期一致.

### dma_alloc_coherent

原型:
``` C
#define dma_alloc_coherent(d,s,h,f) dma_alloc_attrs(d,s,h,f,NULL)    

static inline void *dma_alloc_attrs(struct device *dev, size_t size,
                    dma_addr_t *dma_handle, gfp_t gfp,
                    struct dma_attrs *attrs)
```
参数:

* `dev`:设备对象指针
* `size`:缓存区大小
* `dma_handle`:分配的一致性缓存区的总线地址(dma地址)
* `gfp`: 分配内存的类型,GFP_KERNEL

### dma_free_coherent

原型:
``` C
#define dma_free_coherent(d,s,c,h) dma_free_attrs(d,s,c,h,NULL)                         

static inline void dma_free_attrs(struct device *dev, size_t size,
                  void *vaddr, dma_addr_t dma_handle,
                  struct dma_attrs *attrs)
```

释放DMA缓存区

## 流式映射

流式DMA的使用,主要是驱动使用到的DMA缓存区并非由自己所分配,而是来自其他驱动模块.

### dma_map_single

单个已经分配的缓冲区而言，使用dma_map_single()可实现流式DMA映射

dma_addr_t dma_map_single(struct device *dev, void *buffer, size_t size, enum dma_data_direction direction);  
如果映射成功，返回的是总线地址，否则返回NULL.最后一个参数DMA的方向，可能取DMA_TO_DEVICE, DMA_FORM_DEVICE, DMA_BIDIRECTIONAL和DMA_NONE;

buffer 虚拟地址必须是kmalloc获取的


### dma_unmap_single

原型:

``` C
void dma_unmap_single(struct device *dev,dma_addr_t *dma_addrp,size_t size,enum dma_data_direction direction);
```

取消流式映射

## cache一致性

### dma_sync_single_for_cpu

原型:
``` C
static inline void dma_sync_single_for_cpu(struct device *dev, dma_addr_t addr,
                       size_t size,
                       enum dma_data_direction dir)
```

数据从设备到主存,也就是`写操作`.在DMA写完成后,CPU进行读取前,为了使CPU不会读到cache中的旧数据,使用该函数使cache无效,这样CPU就直接可以从主存中拿到数据

不同的体系结构有不同的实现.

``` C
static void mips_dma_sync_single_for_cpu(struct device *dev,
    dma_addr_t dma_handle, size_t size, enum dma_data_direction direction)
{
    if (cpu_is_noncoherent_r10000(dev))
        __dma_sync(dma_addr_to_page(dev, dma_handle),
               dma_handle & ~PAGE_MASK, size, direction);
}

```

### dma_sync_single_for_device

原型:

``` C
static inline void dma_sync_single_for_device(struct device *dev,
                          dma_addr_t addr, size_t size,
                          enum dma_data_direction dir)
```


数据从主存到设备,也就是`读操作`.在DMA操作前,CPU需要将数据从主存放到DMA缓存区,为了防止write buffer的介入,导致数据只写到临时的write buffer中,驱动程序需要在CPU往主存写数据之后,启动DMA之前调用该函数.相当于"flush/clean",就是将write buffer中的数据冲入主存.

``` C
static void mips_dma_sync_single_for_device(struct device *dev,                             
    dma_addr_t dma_handle, size_t size, enum dma_data_direction direction)
{
    plat_extra_sync_for_device(dev);
    if (!plat_device_is_coherent(dev))
        __dma_sync(dma_addr_to_page(dev, dma_handle),
               dma_handle & ~PAGE_MASK, size, direction);
}
```

### __dma_sync ---> __dma_sync_virtual


``` C
static inline void __dma_sync_virtual(void *addr, size_t size,                    
    enum dma_data_direction direction)
{
    switch (direction) {
    case DMA_TO_DEVICE:
        dma_cache_wback((unsigned long)addr, size);
        break;

    case DMA_FROM_DEVICE:
        dma_cache_inv((unsigned long)addr, size);
        break;

    case DMA_BIDIRECTIONAL:
        dma_cache_wback_inv((unsigned long)addr, size);
        break;

    default:
        BUG();
    }
}

```
## 分散/聚集映射(scatter/gather map)

流式 DMA 映射的一个特例

分散/聚集映射通过将虚拟地址上分散的DMA缓存区,通过一个类型为struct scatterlist的数组或链表组织起来,然后通过一次DMA传输进行主存和设备之间传输数据.

``` C
struct scatterlist {                                     
#ifdef CONFIG_DEBUG_SG
    unsigned long   sg_magic;
#endif
    unsigned long   page_link;  //指明虚拟地址所对应的物理页面struct page对象的地址
    unsigned int    offset;     //数据在DMA缓存区的偏移地址
    unsigned int    length;     //传输的数据块大小
    dma_addr_t  dma_address;    //设备DMA操作要使用的DMA地址(物理地址)
#ifdef CONFIG_NEED_SG_DMA_LENGTH
    unsigned int    dma_length;
#endif
};
```
*scatterlist操作接口:*

``` C
#define sg_dma_address(sg)  ((sg)->dma_address)              
#define sg_dma_len(sg)      ((sg)->dma_length)
```
### dma_map_sg

原型:
``` C
void dma_map_sg(struct device *dev, struct scatterlist *sg,
			int nents, enum dma_data_direction direction);   
```

* `dev`:设备对象指针
* `sg`:struct scatterlist类型数组首地址
* `nents`:当前分散/聚集中的单一流式映射的个数,也是struct scatterlist数组或链表中的元素个数
* `direction`:DMA传输中数据流的方向

nents是散列表入口的数量，该函数的返回值是DMA缓冲区的数量，可能小于nents。对于scatterlist中的每个项目，dma_map_sg()为设备产生恰当的总线地址，它会合并物理上临近的内存区域。

SG映射属于流式DMA映射，与单一缓冲区情况下流式DMA映射类似，

实现:

``` C
static int mips_dma_map_sg(struct device *dev, struct scatterlist *sg,
    int nents, enum dma_data_direction direction, struct dma_attrs *attrs)
{
    int i;

    for (i = 0; i < nents; i++, sg++) {
        if (!plat_device_is_coherent(dev))
            __dma_sync(sg_page(sg), sg->offset, sg->length,
                   direction);                                                             
        sg->dma_address = plat_map_dma_mem_page(dev, sg_page(sg)) +
                  sg->offset;
    }

    return nents;
}

```
### dma_unmap_sg

原型:
``` C
#define dma_unmap_sg(d, s, n, r) dma_unmap_sg_attrs(d, s, n, r, NULL)
```

取消分散/聚集映射

### dma_sync_sg_for_cpu

原型:
``` C
static inline void
dma_sync_sg_for_cpu(struct device *dev, struct scatterlist *sg,
            int nelems, enum dma_data_direction dir)
```

遍历了对单个流式缓存区的`dma_sync_single_for_cpu`操作.

### dma_sync_sg_for_device

原型:
``` C
static inline void
dma_sync_sg_for_device(struct device *dev, struct scatterlist *sg,
               int nelems, enum dma_data_direction dir)                                        
```

遍历了对单个流式缓存区的`dma_sync_single_for_device`操作.


## 通用API

### 接口:

文件: include/asm-generic/dma-mapping-common.h

``` C
#define dma_map_single(d, a, s, r) dma_map_single_attrs(d, a, s, r, NULL)
#define dma_unmap_single(d, a, s, r) dma_unmap_single_attrs(d, a, s, r, NULL)
#define dma_map_sg(d, s, n, r) dma_map_sg_attrs(d, s, n, r, NULL)                       
#define dma_unmap_sg(d, s, n, r) dma_unmap_sg_attrs(d, s, n, r, NULL)
```
### 实现:

文件: include/asm-generic/dma-mapping-common.h

``` C
static inline dma_addr_t dma_map_single_attrs(struct device *dev, void *ptr,
                          size_t size,
                          enum dma_data_direction dir,
                          struct dma_attrs *attrs)
{
    struct dma_map_ops *ops = get_dma_ops(dev);
    dma_addr_t addr;

    kmemcheck_mark_initialized(ptr, size);                                                    
    BUG_ON(!valid_dma_direction(dir));
    addr = ops->map_page(dev, virt_to_page(ptr),
                 (unsigned long)ptr & ~PAGE_MASK, size,
                 dir, attrs);
    debug_dma_map_page(dev, virt_to_page(ptr),
               (unsigned long)ptr & ~PAGE_MASK, size,
               dir, addr, true);
    return addr;
}
```
相关操作的具体实现与体系结构相关

### MIPS

文件: arch/mips/mm/dma-default.c

``` C
 static struct dma_map_ops mips_default_dma_map_ops = {
     .alloc = mips_dma_alloc_coherent,
     .free = mips_dma_free_coherent,
     .map_page = mips_dma_map_page,
     .unmap_page = mips_dma_unmap_page,
     .map_sg = mips_dma_map_sg,
     .unmap_sg = mips_dma_unmap_sg,
     .sync_single_for_cpu = mips_dma_sync_single_for_cpu,
     .sync_single_for_device = mips_dma_sync_single_for_device,
     .sync_sg_for_cpu = mips_dma_sync_sg_for_cpu,
     .sync_sg_for_device = mips_dma_sync_sg_for_device,
     .mapping_error = mips_dma_mapping_error,
     .dma_supported = mips_dma_supported
 };

 struct dma_map_ops *mips_dma_map_ops = &mips_default_dma_map_ops;
 EXPORT_SYMBOL(mips_dma_map_ops);
```
