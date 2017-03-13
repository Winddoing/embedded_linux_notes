# alsa 音频播放

`tinyplay`播放音乐

``` C
# strace  tinyplay  pcmrec.wav
execve("/usr/bin/tinyplay", ["tinyplay", "pcmrec.wav"], [/* 16 vars */]) = 0

...

open("pcmrec.wav", O_RDONLY)            = 3
...
//读取wav格式的音频文件的头数据
read(3, "RIFF$\342\4\0WAVEfmt \20\0\0\0\1\0\2\0@\37\0\0\0}\0\0"..., 4096) = 4096

//第一次打开`pcmC0D0p`设备节点
open("/dev/snd/pcmC0D0p", O_RDWR)       = 4
//ioctl - cmd=SNDRV_PCM_IOCTL_HW_REFINE
ioctl(4, 0xc25c4110, 0x412178)          = 0
close(4)                                = 0

open("/dev/snd/pcmC0D0p", O_RDWR)       = 4
//ioctl - cmd=`SNDRV_PCM_IOCTL_INFO`
ioctl(4, AGPIOC_ACQUIRE or APM_IOC_STANDBY, 0x7f83f3cc) = 0
//ioctl - cmd=`SNDRV_PCM_IOCTL_HW_PARAMS`
ioctl(4, 0xc25c4111, 0x7f83f170)        = 0
//ioctl - cmd=`SNDRV_PCM_IOCTL_SW_PARAMS`
ioctl(4, 0xc0684113, 0x7f83f5ec)        = 0

//在播放期间响应Ctrl+C的中断信号
rt_sigaction(SIGINT, {0x10000000, [RT_65 RT_67], 0x401240 /* SA_??? */}, {SIG_DFL, [RT_67 RT_68 RT_72 RT_74 RT_75 RT_77 RT_81 RT_89 RT_90 RT_91 RT_93 RT_94], 0}, 16) = 0

read(3, "\320\367\200\367\370\370`\370\220\370\330\370@\372h\371\240\371\320\374\230\373\240\374\341\5\301\1\241\5\221\25"..., 12288) = 12288
read(3, "a\0361\36\241\f\10\376\300\374\320\375\30\375\360\375\340\375\0\377\320\377(\377\370\376p\375p\374\321\0"..., 4096) = 4096
//ioctl - cmd=`SNDRV_PCM_IOCTL_PREPARE`
ioctl(4, 0x20004140, 0x7f83f648)        = 0
//ioctl - cmd=`SNDRV_PCM_IOCTL_WRITEI_FRAMES`
ioctl(4, 0x800c4150, 0x7f83f648)        = 0
read(3, "\201\21\301\27q\30\261\25\301\20Q\6x\375h\373\370\373\230\374\210\374x\374p\374\220\374\30\375 \375"..., 12288) = 12288
read(3, "\201\35\1%\241'\301\32\341\t@\377\250\374\220\372\20\373\30\374\340\373X\374H\374X\376\201\v\321\32"..., 4096) = 4096
ioctl(4, 0x800c4150, 0x7f83f648)        = 0

... //while(){ 循环读取播放 }

read(3, "\370\375\10\376 \376\210\376X\376x\376\250\376\350\376\360\376\260\376(\377H\377q\0\301\5\1\vq\21"..., 12288) = 12288
read(3, "\221\0021\n\21\f\241\5x\376\30\377\300\377(\377\1\1!\3q\4Q\3\301\4\240\377h\376\210\377"..., 4096) = 4096
ioctl(4, 0x800c4150, 0x7f83f648)        = 0
read(3, "P\377\0\377!\0\361\6Q\t\230\377@\376\250\377X\377\361\3\1\16\241\n!\0!\6A\16\241\v"..., 12288) = 4652
read(3, "", 4096)                       = 0
ioctl(4, 0x800c4150, 0x7f83f648)        = 0
read(3, "", 16384)                      = 0
ioctl(4, 0x800c4150, 0x7f83f648)        = 0
close(4)                                = 0
close(3)                                = 0
munmap(0x76fe9000, 65536)               = 0
write(1, "Playing sample: 2 ch, 8000 hz, 1"..., 38) = 38  //printf
exit_group(0) 
```

## 疑惑

### 为什么open两次pcmC0D0p设备节点

### 为什么read音频文件两次,并且读的数据大小不一致

tinyplay中播放时,每次只读取一部分(16KB)的音频文件进行播放
``` C
size = pcm_frames_to_bytes(pcm, pcm_get_buffer_size(pcm)); //size=16384Byte=16KB
buffer = malloc(size); 
...
do {                             
	//buffer 临时存放音频文件的数据的buf
	//size   一次读取的大小(16384Byte)
	//file   打开的音频文件描述符                                   
    num_read = fread(buffer, 1, size, file);                        
    if (num_read > 0) {                                             
        if (pcm_write(pcm, buffer, num_read)) {                     
            fprintf(stderr, "Error playing sample\n");              
            break;                                                  
        }                                                           
    }else if(num_read == 0) {                                       
        memset(buffer, 0, size);                                    
        if(pcm_write(pcm, buffer, size)){                           
            fprintf(stderr, "Error playing sample\n");              
            break;                                                  
        }                                                           
    }                                                               
} while (!close && num_read > 0);                                   
```
>在进行strace时,一次播放进行了两次的read系统调用,将每一次read数据的大小相加(12288+4096=16384Byte),正好与malloc的buffer大小一致.因此两次的read是由用户空间的函数进行数据分割的.

fread函数:


## ioctl幻数

``` C
#define SNDRV_PCM_IOCTL_INFO _IOR('A', 0x01, struct snd_pcm_info)                                                            
#define SNDRV_PCM_IOCTL_HW_REFINE _IOWR('A', 0x10, struct snd_pcm_hw_params)             
#define SNDRV_PCM_IOCTL_HW_PARAMS _IOWR('A', 0x11, struct snd_pcm_hw_params)             
#define SNDRV_PCM_IOCTL_SW_PARAMS _IOWR('A', 0x13, struct snd_pcm_sw_params)
#define SNDRV_PCM_IOCTL_PREPARE _IO('A', 0x40) 
#define SNDRV_PCM_IOCTL_WRITEI_FRAMES _IOW('A', 0x50, struct snd_xferi)  
```


## open

``` C
const struct file_operations snd_pcm_f_ops[2] = {                           
    {                                                                       
 		...                      
        .open =         snd_pcm_playback_open,                                                          
        .unlocked_ioctl =   snd_pcm_playback_ioctl,                         
      ...                 
    },                    
}                                                  
```
>file: core/pcm_native.c

打开音频文件和设备节点(/dev/snd/pcmC0D0p)

函数栈:
``` 
[  481.324677] Call Trace:
[  481.327216] [<8001fe8c>] show_stack+0x48/0x70
[  481.332639] [<804b59cc>] dump_stack+0x20/0x2c
[  481.337157] [<8036e970>] jz_i2s_startup+0x74/0x498
[  481.342595] [<803698ec>] soc_pcm_open+0x9c/0x570
[  481.347371] [<80353f60>] snd_pcm_open_substream+0x80/0xfc
[  481.353410] [<80354094>] snd_pcm_open+0xb8/0x1fc
[  481.358183] [<803542a4>] snd_pcm_playback_open+0x50/0x7c
[  481.364131] [<803442a0>] snd_open+0x180/0x210
[  481.368641] [<800d7140>] chrdev_open+0x12c/0x174
[  481.373873] [<800d0e88>] do_dentry_open.isra.19+0x1e0/0x2b0
[  481.379630] [<800d0f80>] finish_open+0x28/0x4c
[  481.384687] [<800e08a4>] do_last.isra.50+0x988/0xb6c
[  481.389817] [<800e0b48>] path_openat+0xc0/0x448
[  481.394959] [<800e1230>] do_filp_open+0x3c/0xa4
[  481.399646] [<800d2254>] do_sys_open+0x148/0x198
[  481.404879] [<80022e9c>] stack_done+0x20/0x44
```

函数调用:
``` C
**open**
|(sound/core/pcm_native.c )
|-> snd_pcm_playback_open
  \
  |-> snd_pcm_open(file, pcm, SNDRV_PCM_STREAM_PLAYBACK); 
    \
    |-> while(1){ snd_pcm_open_file(file, pcm, stream); schedule(); }
      \
      |-> snd_pcm_open_substream
		\
		|-> substream->ops->open(substream)
		  |(sound/soc/soc-pcm.c)
		  |-> soc_pcm_open
			\
			|-> cpu_dai->driver->ops->startup(substream, cpu_dai);
			  \_**snd_soc_register_component** -> snd_soc_dai_driver -> snd_soc_dai_ops (.startup = jz_i2s_startup)
			|-> codec_dai->driver->ops->startup(substream, codec_dai);
             |\_**snd_soc_register_codec** -> snd_soc_dai_driver -> snd_soc_dai_ops (.startup = jz_icdc_startup)
			|-> rtd->dai_link->ops->startup(substream);
			  \_ **snd_soc_register_card** -> snd_soc_dai_link -> snd_soc_ops (.startup = phoenix_spk_sup)
```

### struct snd_pcm_ops

``` C
void snd_pcm_set_ops(struct snd_pcm *pcm, int direction, struct snd_pcm_ops *ops)             
{                                                                                             
    struct snd_pcm_str *stream = &pcm->streams[direction];                                    
    struct snd_pcm_substream *substream;                                                      
                                                                                              
    for (substream = stream->substream; substream != NULL; substream = substream->next)       
        substream->ops = ops;                                                                 
}                                                                                             
```

实现:
``` C
/* create a new pcm */                                               
int soc_new_pcm(struct snd_soc_pcm_runtime *rtd, int num)            
{
	...
	/* ASoC PCM operations */                                                                                 
	if (rtd->dai_link->dynamic) {                                                                             
		rtd->ops.open       = dpcm_fe_dai_open;                                                               
		rtd->ops.hw_params  = dpcm_fe_dai_hw_params;                                                          
		rtd->ops.prepare    = dpcm_fe_dai_prepare;                                                            
		rtd->ops.trigger    = dpcm_fe_dai_trigger;                                                            
		rtd->ops.hw_free    = dpcm_fe_dai_hw_free;                                                            
		rtd->ops.close      = dpcm_fe_dai_close;                                                              
		rtd->ops.pointer    = soc_pcm_pointer;                                                                
		rtd->ops.ioctl      = soc_pcm_ioctl;                                                                  
	} else {                                                                                                  
		rtd->ops.open       = soc_pcm_open;                                                                   
		rtd->ops.hw_params  = soc_pcm_hw_params;                                                              
		rtd->ops.prepare    = soc_pcm_prepare;                                                                
		rtd->ops.trigger    = soc_pcm_trigger;                                                                
		rtd->ops.hw_free    = soc_pcm_hw_free;                                                                
		rtd->ops.close      = soc_pcm_close;                                                                  
		rtd->ops.pointer    = soc_pcm_pointer;                                                                
		rtd->ops.ioctl      = soc_pcm_ioctl;                                                                  
	}                                                                                                         
		                                                                                                      
	if (platform->driver->ops) {                                                                              
		rtd->ops.ack        = platform->driver->ops->ack;                                                     
		rtd->ops.copy       = platform->driver->ops->copy;                                                    
		rtd->ops.silence    = platform->driver->ops->silence;                                                 
		rtd->ops.page       = platform->driver->ops->page;                                                    
		rtd->ops.mmap       = platform->driver->ops->mmap;                                                    
	}                                                                                                         
		                                                                                                      
	if (playback)                                                                                             
		snd_pcm_set_ops(pcm, SNDRV_PCM_STREAM_PLAYBACK, &rtd->ops);                                           
		                                                                                                      
	if (capture)                                                                                              
		snd_pcm_set_ops(pcm, SNDRV_PCM_STREAM_CAPTURE, &rtd->ops);                                            
	...                                                                    
}
```
### struct snd_soc_dai_ops


## ioctl



播放:
```
ioctl:
	(`sound/core/pcm_native.c`)	
|->	snd_pcm_playback_ioctl
|
|-> snd_pcm_playback_ioctl1 --> cmd:<SNDRV_PCM_IOCTL_WRITEI_FRAMES>
|(`sound/core/pcm_lib.c`)
|-> snd_pcm_lib_write --- > struct snd_pcm_substream *substream
|
|-> snd_pcm_lib_write1
		|_call_back-->snd_pcm_lib_write_transfer(数据传输:copy和map)
				|_.(内存和DMA之间的数据传递)
					char *hwbuf = runtime->dma_area + frames_to_bytes(runtime, hwoff); 
					if (copy_from_user(hwbuf, buf, frames_to_bytes(runtime, frames))) 
									
```

### SNDRV_PCM_IOCTL_WRITEI_FRAMES

写数据

### 写数据的组织结构

``` C
typedef unsigned long snd_pcm_uframes_t;        
typedef signed long snd_pcm_sframes_t;          

struct snd_xferi {                        
    snd_pcm_sframes_t result;  //?
    void __user *buf;          //写的真实数据
    snd_pcm_uframes_t frames;  //写数据的大小(4096) 4K
};                                        

```
### struct snd_pcm_substream



### 数据传输dma_area

``` C
char *hwbuf = runtime->dma_area + frames_to_bytes(runtime, hwoff); 
if (copy_from_user(hwbuf, buf, frames_to_bytes(runtime, frames)))  
```

使用`tinyplay`播放,数据传输采用`map`方式,默认配置.



## 参考

1. [linux_sound_alsa_ALSA体系SOC子系统中数据流分析 ](http://blog.chinaunix.net/uid-20776117-id-3085423.html)
2. [linux_sound_alsa_ALSA体系SOC子系统中hw_params逻辑 ](http://blog.chinaunix.net/uid-20776117-id-3085420.html)

