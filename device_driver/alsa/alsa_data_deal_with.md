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

## 数据传递

用户空间和内核空间的数据转移,以及数据分片的处理

### 用户空间传递数据

``` C
do {
     num_read = fread(buffer, 1, size, file);
     if (num_read > 0) {
     if (pcm_write(pcm, buffer, num_read)) {
             fprintf(stderr, "Error playing sample\n");
             break;
         }
     }else if(num_read == 0) {
     memset(buffer, 0, size);
     if (pcm_write(pcm, buffer, size)) {
             fprintf(stderr, "Error playing sample\n");
             break;
         }
    }
} while (!close && num_read > 0);
```
通过pcm_write将数据传入内核,将数据打包`snd_xferi`,使用ioctl写入.

``` C
int pcm_write(struct pcm *pcm, const void *data, unsigned int count)
{
    struct snd_xferi x;

    if (pcm->flags & PCM_IN)
        return -EINVAL;

    x.buf = (void*)data;
    x.frames = count / (pcm->config.channels *
                        pcm_format_to_bits(pcm->config.format) / 8);

    for (;;) {
        if (!pcm->running) {
            int prepare_error = pcm_prepare(pcm);           //SNDRV_PCM_IOCTL_PREPARE
            if (prepare_error)
                return prepare_error;
            if (ioctl(pcm->fd, SNDRV_PCM_IOCTL_WRITEI_FRAMES, &x))
                return oops(pcm, errno, "cannot write initial data");
            pcm->running = 1;
            return 0;
        }
        if (ioctl(pcm->fd, SNDRV_PCM_IOCTL_WRITEI_FRAMES, &x)) {
            pcm->prepared = 0;
            pcm->running = 0;
            if (errno == EPIPE) {
                /* we failed to make our window -- try to restart if we are
                 * allowed to do so.  Otherwise, simply allow the EPIPE error to
                 * propagate up to the app level */
                pcm->underruns++;
                if (pcm->flags & PCM_NORESTART)
                    return -EPIPE;
                continue;
            }
            return oops(pcm, errno, "cannot write stream data");
        }
        return 0;
    }
}
```

### 内核空间数据传递

此时内核中alsa框架初始化完成,已建立完成一条substream数据传输流

写处理:
``` C
snd_pcm_sframes_t snd_pcm_lib_write(struct snd_pcm_substream *substream, const void __user *buf, snd_pcm_uframes_t size)
{
    struct snd_pcm_runtime *runtime;
    int nonblock;
    int err;

    err = pcm_sanity_check(substream);
    if (err < 0)
        return err;
    runtime = substream->runtime;
    nonblock = !!(substream->f_flags & O_NONBLOCK);

    if (runtime->access != SNDRV_PCM_ACCESS_RW_INTERLEAVED &&
        runtime->channels > 1)
        return -EINVAL;
    return snd_pcm_lib_write1(substream, (unsigned long)buf, size, nonblock,
                  snd_pcm_lib_write_transfer);
}

```
#### 检测substream

检测当前的各种状态是否准备完成

1. 检测当前substream中是否有正在执行的pcm
2. 检测当前substream中数据的处理方式是否准备完成
3. 检测当前substream中的pcm是否已经打开

#### 写处理

1. 判断当前substream中的pcm是否处于running状态
2. 如果当前substream中的pcm处于running状态,进行`snd_pcm_update_hw_ptr`
3. 判断当前substream中的pcm是否可以播放??????
4. 判断当前buffer中可用于传输的有效数据大小是否大于0
5.


#### 数据同步

是什么保证了数据同步??????

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

>处理方式: `trigger` <-> `soc_pcm_trigger`, (相当于开始,触发,发射)

```
Codec -->  Platform  -->  CPU(BAIC)
```
驱动实现:
|   模块    |   接口  |   作用  |
|   :--:    |  :--:   |   :-: |
| Codec     | trigger |  debug |
| Platform  | trigger |       |
| CPU(BAIC) | trigger |       |


### Codec

注: X1000使用内部codec,trigger没有做出任何动作,只是添加debug接口

### DMA buffer

Audio中数据的处理是通过DMA进行数据传输,因此数据传输中DMA工作流程.

#### 配置DMA通道

在数据传输中DMA通道的选择和配置工作,主要在`初始化`时完成. 因此在驱动初始化时,需要对当前使用控制器BAIC的DMA通道进行选择配置,ALSA中数据的处理主要集中中`Platform`模块

根据platform模块的接口函数,对DMA的初始化工作在`pcm_new(rtd)`中实现.

驱动实现:
|   模块    |   接口  |   作用  |
|   :--:    |  :--:   |   :-: |
| Platform  | pcm_new | 初始化工作,DMA通道的选择 |

#### DMA数据的组织

在DMA的数据传输中,数据的传输单元以DMA desc为主.


#### DMA数据的传输

根据内核对Audio数据的处理流程进行分析:
主要结合`snd_pcm_start`和音频播放的说明.

1. Platform进行pointer

> 主要作用时检测数据传输时的数据量

实现:
``` C
/**
 * snd_pcm_period_elapsed - update the pcm status for the next period
 * @substream: the pcm substream instance
 *
 * This function is called from the interrupt handler when the
 * PCM has processed the period size.  It will update the current
 * pointer, wake up sleepers, etc.
 *
 * Even if more than one periods have elapsed since the last call, you
 * have to call this only once.
 */

snd_pcm_period_elapsed
    |
    |-> snd_pcm_update_hw_ptr0
        |
        |-> substream->ops->pointer(substream)
```


2. Platform进行trigger

>主要的作用时根据TRIGGER的条件判断,并做出相关处理,

#### 相关API

1. frames_to_bytes

>将当前的帧计数数据量转化byte, 帧计数=size * frame_bit / 8(frame_bit = sample * channel)

2. bytes_to_frames

> size * 8 / frame_bit

3. snd_pcm_lib_buffer_bytes

> 获取buffer大小, 单位`byte`

4. snd_pcm_lib_period_bytes

> 获取buffer大小,单位`period`

### 控制器 -- I2S

主要作用是数据传输的开关(start/stop)

1. enable FIFO
2. enable DMA

## "啊" 怎么出来的

### 用户空间

1. open打开音频文件
2. read取出"啊"(BUFFER)
3. ioctl为BUFFER进行铺路
4. write将BUFFER给内核

### 内核空间


## 参考

1. [【关于alsa buffer】ALSA编程细节分析](http://blog.sina.com.cn/s/blog_533074eb0101dayo.html)
2. [ALSA中PCM参数配置](http://blog.sina.com.cn/s/blog_533074eb0101dav1.html)
3. [ALSA缓存的理解](http://blog.csdn.net/aguei868/article/details/52588010)





