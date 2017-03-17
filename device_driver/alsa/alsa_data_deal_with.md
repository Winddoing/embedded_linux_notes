# DMA数据处理

音频数据流

## mmap
### 用户空间

采用tinyplay进行数据播放.

``` C
static int pcm_hw_mmap_status(struct pcm *pcm) {                                                              
                                                                                                                                                               
    int page_size = sysconf(_SC_PAGE_SIZE);                                                                   
    pcm->mmap_status = mmap(NULL, page_size, PROT_READ, MAP_FILE | MAP_SHARED,                                
                            pcm->fd, SNDRV_PCM_MMAP_OFFSET_STATUS);                                           
    //mmap 失败
    if (pcm->mmap_status == MAP_FAILED)                                                                       
        pcm->mmap_status = NULL;                                                                              
    if (!pcm->mmap_status)                                                                                            
        goto mmap_error;        
    ...                                                                              
}
```

### 内核空间

''' C
static int snd_pcm_mmap(struct file *file, struct vm_area_struct *area)                                
{                                                                                                      
    struct snd_pcm_file * pcm_file;                                                                    
    ...
    pcm_file = file->private_data;                                                                     
    substream = pcm_file->substream;                                                                   
    ...                                 
                                                                                                       
    offset = area->vm_pgoff << PAGE_SHIFT;                                                             
    //用户空间传入SNDRV_PCM_MMAP_OFFSET_STATUS,但是用户空间没有对no_compat_mmap使能(用户空间不支持),所以返回error      
    switch (offset) {                                                                                  
    case SNDRV_PCM_MMAP_OFFSET_STATUS:                                                                 
        if (pcm_file->no_compat_mmap)                                                                  
            return -ENXIO;                                                                             
        return snd_pcm_mmap_status(substream, file, area);                                             
    case SNDRV_PCM_MMAP_OFFSET_CONTROL:                                                                
        if (pcm_file->no_compat_mmap)                                                                  
            return -ENXIO;                                                                             
        return snd_pcm_mmap_control(substream, file, area);                                            
    default:                                                                                           
        return snd_pcm_mmap_data(substream, file, area);                                               
    }                                                                                                  
    return 0;                                                                                          
}                                                                                                      

```


## DMA

### 音频数据的播放流程

采用tinyplay播放时strace的部分过程:
```
read(3, "\201\21\301\27q\30\261\25\301\20Q\6x\375h\373\370\373\230\374\210\374x\374p\374\220\374\30\375 \375"..., 12288) = 12288
read(3, "\201\35\1%\241'\301\32\341\t@\377\250\374\220\372\20\373\30\374\340\373X\374H\374X\376\201\v\321\32"..., 4096) = 4096
ioctl(4, 0x800c4150, 0x7f83f648)        = 0
```

>这是数据搬运时对一块数据和一个buffer的处理过程,详细的含义可参考音频文件播放

流程:

1. 在用户空间读取音频文件的填写到16KB的buffer
2. 通过ioctl将该buffer拷贝到内核空间中的进行播放


### 播放时的数据搬运

在用户空间通过read读取音频文件的数据,此时通过`copy_to_user`将用户空间的数据拷贝到内核空间

``` C
static int snd_pcm_lib_write_transfer(struct snd_pcm_substream *substream,                                         
                      unsigned int hwoff,                                                                     
                      unsigned long data, unsigned int off,                                                   
                      snd_pcm_uframes_t frames)                                                               
{                                                                                                             
    struct snd_pcm_runtime *runtime = substream->runtime;                                                     
    int err;                                                                                                  
    char __user *buf = (char __user *) data + frames_to_bytes(runtime, off); //                                 
    ...                                                                             
        char *hwbuf = runtime->dma_area + frames_to_bytes(runtime, hwoff);                                    
        if (copy_from_user(hwbuf, buf, frames_to_bytes(runtime, frames)))                                     
            return -EFAULT;                                                                                   
    ...                                                                                                        
    return 0;                                                                                                 
}                                                                                                             
```
>file: sound/core/pcm_lib.c

在用户空间通过ioctl(SNDRV_PCM_IOCTL_WRITEI_FRAMES),进行数据填写时,通过回调函数`snd_pcm_lib_write_transfer`,将用户空间的音频数据搬运到内核的DMA buffer


### 内核空间处理该buffer

用户空间的buffer拿到后,进行数据处理.
``` C
int snd_pcm_start(struct snd_pcm_substream *substream)                             
{                                                                                             
    return snd_pcm_action(&snd_pcm_action_start, substream,                        
                  SNDRV_PCM_STATE_RUNNING);                                        
}                                                                                  
```
>file: sound/core/pcm_native.c 

>该函数的具体调用流程参考音频播放

#### 数据的处理流程:

>处理方式: `trigger`, (相当于开始,触发,发射)

```
Codec -->  Platform  -->  CPU(BAIC)
```
驱动实现:
|   模块    |   接口  |   作用  |
|   :--:    |  :--:   |   :-: |
| Platform  | trigger |       |
| CPU(BAIC) | trigger |       |

### DMA buffer

Audio中数据的处理是通过DMA进行数据传输,因此数据传输中DMA工作流程.

#### 配置DMA通道

在数据传输中DMA通道的选择和配置工作,主要在`初始化`时完成. 因此在驱动初始化时,需要对当前使用控制器BAIC的DMA通道进行选择配置,ALSA中数据的处理主要集中中`Platform`模块

根据platform模块的接口函数,对DMA的初始化工作在`pcm_new(rtd)`中实现.

驱动实现:
|   模块    |   接口  |   作用  |
|   :--:    |  :--:   |   :-: |
| Platform  | pcm_new | 初始化工作,DMA通道的选择 |

####
## codec层
