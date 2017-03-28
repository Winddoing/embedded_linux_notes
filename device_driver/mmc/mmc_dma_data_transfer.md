# DMA


## mmc数据的传输形式

```
CMD + DATA
```
因此数据的组织将在发送命令之前准备

## 数据组织

采用DMA描述符的形式进行传输

特点:
1. DMA数据传输必须使用`物理地址`
2. DMA描述符的申请也必须是`物理地址`

## 数据流程

### 申请buffer

``` C
//申请DMA描述符表
host->adma_table = dma_alloc_coherent(mmc_dev(mmc),
                      host->adma_table_sz,
                      &host->adma_addr,
                      GFP_KERNEL);
//申请数据buffer
host->align_buffer = kmalloc(host->align_buffer_sz, GFP_KERNEL);
```

#### 填充DMA描述符

将block层一次传输的数据buffer填入DMA描述符.
