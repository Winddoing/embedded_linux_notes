# DMA

>dma数据传输块的组织和应用



三个指针



## 驱动控制器层的数据关系

* buffer大小: snd_pcm_lib_buffer_bytes(substream) = 32768 = 32KB
* period : snd_pcm_lib_period_bytes(substream) = 8192 = 8KB
* DMA描述符的个数: snd_pcm_lib_buffer_bytes(substream) / snd_pcm_lib_period_bytes(substream) = 4

以上参数的限制:

``` C
static const struct snd_pcm_hardware xxx_pcm_hardware = {
    .info = SNDRV_PCM_INFO_MMAP |
        SNDRV_PCM_INFO_PAUSE |
        SNDRV_PCM_INFO_RESUME |
        SNDRV_PCM_INFO_MMAP_VALID |
        SNDRV_PCM_INFO_INTERLEAVED |
        SNDRV_PCM_INFO_BLOCK_TRANSFER,
    .formats = SNDRV_PCM_FMTBIT_S24_LE |
        SNDRV_PCM_FMTBIT_S20_3LE |
        SNDRV_PCM_FMTBIT_S18_3LE |
        SNDRV_PCM_FMTBIT_S16_LE |
        SNDRV_PCM_FMTBIT_S8,
    .rates                  = SNDRV_PCM_RATE_8000_192000,
    .rate_min               = 8000,
    .rate_max               = 192000,
    .channels_min           = 1,
    .channels_max           = 2,
    .buffer_bytes_max       = 32*1024,       /* 32K */  //snd_pcm_lib_buffer_bytes
    .period_bytes_min       = PAGE_SIZE / 4, /* 1K */   //snd_pcm_lib_period_bytes
    .period_bytes_max       = PAGE_SIZE * 2, /* 8K */
    .periods_min            = 4,
    .periods_max            = 64,
    .fifo_size              = 0,
};

```
* buffer size = 32KB
* DMA desc buffer size = 8KB

| 位宽 | 通道 | 采样率  | 理论速率  | 填满一个描述符的时间 | 填满整个buffer的时间 |
| :-: | :-:  | :--:   | :--:     |  :---:            |   :---:            |
| 16  | 2    | 44100  | 172KB/s  |   46ms            | 186ms              |
| 16  | 2    | 96000  | 375KB/s  |   21ms            | 85ms               |
| 16  | 2    | 192000 | 750KB/s  |   11ms            | 43ms               |
| 32  | 8    | 44100  | 1378KB/s |   6ms             | 23ms               |




