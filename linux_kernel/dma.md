
#一致性

## dma_alloc_coherent

## dma_free_coherent

# 流式映射

## dma_map_single

单个已经分配的缓冲区而言，使用dma_map_single()可实现流式DMA映射

dma_addr_t dma_map_single(struct device *dev, void *buffer, size_t size, enum dma_data_direction direction);  
如果映射成功，返回的是总线地址，否则返回NULL.最后一个参数DMA的方向，可能取DMA_TO_DEVICE, DMA_FORM_DEVICE, DMA_BIDIRECTIONAL和DMA_NONE;

buffer 虚拟地址必须是kmalloc获取的


## dma_unmap_single

void dma_unmap_single(struct device *dev,dma_addr_t *dma_addrp,size_t size,enum dma_data_direction direction);

## dma_sync_single_for_cpu

数据从设备到主存,也就是写操作.在DMA写完成后,CPU进行读取前,为了使CPU不会读到cache中的旧数据,使用该函数使cache无效,这样CPU就直接可以从主存中拿到数据

## dma_sync_single_for_device

# 分散/集中映射(scatter/gather map)
是流式 DMA 映射的一个特例


# dma_map_sg

void dma_map_sg(struct device *dev, struct scatterlist *sg, int nents, enum dma_data_direction direction);   

其中nents是散列表入口的数量，该函数的返回值是DMA缓冲区的数量，可能小于nents。对于scatterlist中的每个项目，dma_map_sg()为设备产生恰当的总线地址，它会合并物理上临近的内存区域。

SG映射属于流式DMA映射，与单一缓冲区情况下流式DMA映射类似，

## dma_unmap_sg

## dma_sync_sg_for_cpu

## dma_sync_sg_for_device




