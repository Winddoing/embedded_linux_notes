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

//第一次打开`pcmC0D0p`设备节点, 重新设置硬件参数
open("/dev/snd/pcmC0D0p", O_RDWR)       = 4
//ioctl - cmd=SNDRV_PCM_IOCTL_HW_REFINE
ioctl(4, 0xc25c4110, 0x412178)          = 0
close(4)                                = 0

//第二次打开`pcmC0D0p`设备节点, 进行音频播放的准备工作和播放
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

1. 第一次打开`pcmC0D0p`,主要为了重新规范硬件
``` C
struct snd_pcm_hw_params {
    unsigned int flags;
    struct snd_mask masks[SNDRV_PCM_HW_PARAM_LAST_MASK -
                   SNDRV_PCM_HW_PARAM_FIRST_MASK + 1];
    struct snd_mask mres[5];    /* reserved masks */
    struct snd_interval intervals[SNDRV_PCM_HW_PARAM_LAST_INTERVAL -
                        SNDRV_PCM_HW_PARAM_FIRST_INTERVAL + 1];
    struct snd_interval ires[9];    /* reserved intervals */
    unsigned int rmask;     /* W: requested masks */
    unsigned int cmask;     /* R: changed masks */
    unsigned int info;      /* R: Info flags for returned setup */
    unsigned int msbits;        /* R: used most significant bits */
    unsigned int rate_num;      /* R: rate numerator */
    unsigned int rate_den;      /* R: rate denominator */
    snd_pcm_uframes_t fifo_size;    /* R: chip FIFO size in frames */
    unsigned char reserved[64]; /* reserved for future */
};
```
>file: include/uapi/sound/asound.h

主要是将用户空间的snd_pcm_hw_params信息和内核空间的进行对比和规范化

2. 第一次打开`pcmC0D0p`,主要为了进行音频播放的准备和播放音频信号


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
在进行strace时,一次播放进行了两次的read系统调用,将每一次read数据的大小相加(12288+4096=16384Byte),正好与malloc的buffer大小一致.因此两次的read是由用户空间的函数进行数据分割的.

>用户空间的buffer和内核得到的buffer大小一致,分两次read的具体原因以后在细究

fread函数:

### 用户空间申请buffer大小的依据

在播放当前歌曲时,所申请的buffer大小为16KB,为什么申请16K?

音频信息:

| 采样率 | 通道 | 位宽(format) |
|:----: |:----:|:-----------:|
| 44100Hz | 2  | 16bit		|


>4KB的buffer大小为`tinyplay`默认大小,`period_size = 1024`, `period_count = 4`决定了buffer大小,而`period_size`可以进行修改默认大小.

需要申请buffer的大小: 1024 * 4 * 2 * (16 / 8) = 16384



## ioctl幻数

``` C
//获取声卡信息返回给用户空间
#define SNDRV_PCM_IOCTL_INFO _IOR('A', 0x01, struct snd_pcm_info)
//硬件参数重新规范
#define SNDRV_PCM_IOCTL_HW_REFINE _IOWR('A', 0x10, struct snd_pcm_hw_params)
//设置硬件参数
#define SNDRV_PCM_IOCTL_HW_PARAMS _IOWR('A', 0x11, struct snd_pcm_hw_params)
//设置软件参数
#define SNDRV_PCM_IOCTL_SW_PARAMS _IOWR('A', 0x13, struct snd_pcm_sw_params)
//准备操作
#define SNDRV_PCM_IOCTL_PREPARE _IO('A', 0x40)
//从用户空间把音频数据拿过来，从wav文件中读出数据
#define SNDRV_PCM_IOCTL_WRITEI_FRAMES _IOW('A', 0x50, struct snd_xferi)
```


## open

``` C
const struct file_operations snd_pcm_f_ops[2] = {
    {
		...
		.release =      snd_pcm_release
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
			  \_**snd_soc_register_codec** -> snd_soc_dai_driver -> snd_soc_dai_ops (.startup = jz_icdc_startup)
			|-> rtd->dai_link->ops->startup(substream);
			  \_ **snd_soc_register_card** -> snd_soc_dai_link -> snd_soc_ops (.startup = phoenix_spk_sup)
```


> ASOC接口: ops->open <---> `soc_pcm_open`

### struct snd_pcm_str

``` C
 struct snd_pcm_str {
     int stream;             /* stream (direction) */
     struct snd_pcm *pcm;
     /* -- substreams -- */
     unsigned int substream_count;
     unsigned int substream_opened;
     struct snd_pcm_substream *substream;
	 ...
     struct snd_kcontrol *chmap_kctl; /* channel-mapping controls */
 };
```
### struct snd_pcm_substream

``` C
struct snd_pcm_substream {
    struct snd_pcm *pcm;
    struct snd_pcm_str *pstr;
    void *private_data;     /* copied from pcm->private_data */
    int number;
    char name[32];          /* substream name */
    int stream;         /* stream (direction) */
    struct pm_qos_request latency_pm_qos_req; /* pm_qos request */
    size_t buffer_bytes_max;    /* limit ring buffer size */
    struct snd_dma_buffer dma_buffer;
    unsigned int dma_buf_id;
    size_t dma_max;
    /* -- hardware operations -- */
    struct snd_pcm_ops *ops;       //OPS
    /* -- runtime information -- */
    struct snd_pcm_runtime *runtime;
        /* -- timer section -- */
    struct snd_timer *timer;        /* timer */
    unsigned timer_running: 1;  /* time is running */
    /* -- next substream -- */
    struct snd_pcm_substream *next;
    /* -- linked substreams -- */
}
```

### struct snd_soc_pcm_runtime

``` C
struct snd_soc_pcm_runtime {
    struct device *dev;
    struct snd_soc_card *card;
    struct snd_soc_dai_link *dai_link;
    struct mutex pcm_mutex;
    enum snd_soc_pcm_subclass pcm_subclass;
    struct snd_pcm_ops ops;                   //OPS
	...
}
```
### struct snd_soc_dai_driver

``` C
struct snd_soc_dai_driver {
     /* DAI description */
     const char *name;
     unsigned int id;
     int ac97_control;
     unsigned int base;

     /* DAI driver callbacks */
     int (*probe)(struct snd_soc_dai *dai);
     int (*remove)(struct snd_soc_dai *dai);
     int (*suspend)(struct snd_soc_dai *dai);
     int (*resume)(struct snd_soc_dai *dai);
     /* compress dai */
     bool compress_dai;

     /* ops */
     const struct snd_soc_dai_ops *ops;     //OPS

     /* DAI capabilities */
     struct snd_soc_pcm_stream capture;
     struct snd_soc_pcm_stream playback;
     unsigned int symmetric_rates:1;

     /* probe ordering - for components with runtime dependencies */
     int probe_order;
     int remove_order;
 };                                                                         =
```

驱动层的实现接口:cpu_dai


### struct snd_soc_dai

``` C
struct snd_soc_dai {
    const char *name;

    /* driver ops */
    struct snd_soc_dai_driver *driver;


    /* DAI DMA data */
    void *playback_dma_data;
    void *capture_dma_data;

    /* Symmetry data - only valid if symmetry is being enforced */
    unsigned int rate;

    /* parent platform/codec */
    struct snd_soc_platform *platform;
    struct snd_soc_codec *codec;

    struct snd_soc_card *card;

    struct list_head list;
    struct list_head card_list;
};

```
### struct snd_soc_dai 和 snd_soc_pcm_runtime

#### 注册`struct snd_soc_dai`的功能接口到`LIST_HEAD(dai_list)`

``` C
static struct snd_soc_dai_driver jz_i2s_dai = {
        .probe   = jz_i2s_probe,
 		...
        .ops = &jz_i2s_dai_ops,
};
```
>file: soc/ingenic/asoc-v13/asoc-i2s-v13.c


``` C
jz_i2s_platfrom_probe
  |
  |-> snd_soc_register_component(&pdev->dev, &jz_i2s_component, &jz_i2s_dai, 1);
	|
	|-> snd_soc_register_dai //1. 将dai和codec进行匹配(dai->codec = codec)
                             //2. 将`snd_soc_dai`的结构添加到`dai_list`(list_add(&dai->list, &dai_list))
```
#### 绑定 --- soc_bind_dai_link



``` C
	struct snd_soc_pcm_runtime *rtd = &card->rtd[num];
	struct snd_soc_dai *codec_dai, *cpu_dai;

 /* Find CPU DAI from registered DAIs*/
 list_for_each_entry(cpu_dai, &dai_list, list) {
     if (dai_link->cpu_of_node &&
         (cpu_dai->dev->of_node != dai_link->cpu_of_node))
         continue;
     if (dai_link->cpu_name &&
         strcmp(dev_name(cpu_dai->dev), dai_link->cpu_name))
         continue;
     if (dai_link->cpu_dai_name &&
         strcmp(cpu_dai->name, dai_link->cpu_dai_name))
         continue;

     rtd->cpu_dai = cpu_dai;
 }
```
>file: sound/soc/soc-core.c


### snd_pcm_substream 和 snd_soc_pcm_runtime

machine函数调用关系:
``` C
snd_soc_register_card
  |
  |-> snd_soc_instantiate_card
	|
	|-> soc_probe_link_dais
	  |
	  |-> soc_new_pcm //create the pcm
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
		//回调函数
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

``` C
void snd_pcm_set_ops(struct snd_pcm *pcm, int direction, struct snd_pcm_ops *ops)
{
    struct snd_pcm_str *stream = &pcm->streams[direction];
    struct snd_pcm_substream *substream;

    for (substream = stream->substream; substream != NULL; substream = substream->next)
        substream->ops = ops;
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
|-> snd_pcm_playback_ioctl1 --> 判断cmd <SNDRV_PCM_IOCTL_WRITEI_FRAMES>
|(`sound/core/pcm_lib.c`)
|-> snd_pcm_lib_write --- > struct snd_pcm_substream *substream
|
|-> snd_pcm_lib_write1
		|_call_back-->snd_pcm_lib_write_transfer(数据传输:copy和map)
			|_.(内存和DMA之间的数据传递, 循环搬送直到播放完毕)
					char *hwbuf = runtime->dma_area + frames_to_bytes(runtime, hwoff);
					if (copy_from_user(hwbuf, buf, frames_to_bytes(runtime, frames)))
		|
		|-> snd_pcm_start(substream) //**启动传输(只是在开始时,调用一次)**
			|
			|-> snd_pcm_action
				|
				|-> snd_pcm_action_single
					|
					|-> {
							res = ops->pre_action(substream, state);
							if (res < 0)
 						   		return res;
							res = ops->do_action(substream, state);
							if (res == 0)
 						   		ops->post_action(substream, state);
							else if (ops->undo_action)
						    	ops->undo_action(substream, state);
						}
```


### snd_pcm_start

``` C
int snd_pcm_start(struct snd_pcm_substream *substream)
{
    return snd_pcm_action(&snd_pcm_action_start, substream,
                  SNDRV_PCM_STATE_RUNNING);
}
```

回调注册:

``` C
static struct action_ops snd_pcm_action_start = {
    .pre_action = snd_pcm_pre_start,
    .do_action = snd_pcm_do_start,
    .undo_action = snd_pcm_undo_start,
    .post_action = snd_pcm_post_start
};
```

```C
static int snd_pcm_do_start(struct snd_pcm_substream *substream, int state)
{
    if (substream->runtime->trigger_master != substream)
        return 0;
    return substream->ops->trigger(substream, SNDRV_PCM_TRIGGER_START);
}
```
>file: sound/core/pcm_native.c


>ASOC 接口: ops->trigger <--->  `soc_pcm_trigger`


### SNDRV_PCM_IOCTL_HW_REFINE

> 硬件参数重新规范

```
snd_pcm_hw_refine_user
  |
  |-> snd_pcm_hw_refine
```
>该函数的功能主要是通过软件进行实现,在驱动添加时可默认此功能正常,不细究

### SNDRV_PCM_IOCTL_INFO

``` C
struct snd_pcm_info {
    unsigned int device;        /* RO/WR (control): device number */
    unsigned int subdevice;     /* RO/WR (control): subdevice number */
    int stream;         /* RO/WR (control): stream direction */
    int card;           /* R: card number */
    unsigned char id[64];       /* ID (user selectable) */
    unsigned char name[80];     /* name of this device */
    unsigned char subname[32];  /* subdevice name */
    int dev_class;          /* SNDRV_PCM_CLASS_* */
    int dev_subclass;       /* SNDRV_PCM_SUBCLASS_* */
    unsigned int subdevices_count;
    unsigned int subdevices_avail;
    union snd_pcm_sync_id sync; /* hardware synchronization ID */
    unsigned char reserved[64]; /* reserved for future... */
};
```
返回用户空间snd_pcm_info的信息.


### SNDRV_PCM_IOCTL_HW_PARAMS

>配置硬件参数:
> 1. 数据的采样格式(8bit, 16bit等)
> 2. 时钟配置
> 3. 设置通道数
> 4. 配置dma

```
snd_pcm_hw_params_user
  | <SNDRV_PCM_IOCTL_HW_PARAMS>
  |-> snd_pcm_hw_params
	|
	|-> substream->ops->hw_params(substream, params)
```

> ASOC 接口: ops->hw_params <---> `soc_pcm_hw_params`

### SNDRV_PCM_IOCTL_SW_PARAMS

>配置软件参数

```
snd_pcm_common_ioctl1
  | <SNDRV_PCM_IOCTL_SW_PARAMS>
  |-> snd_pcm_sw_params_user
	|
	|-> snd_pcm_sw_params
```
这里主要进行软件相关的设置,目前不做细究.


### SNDRV_PCM_IOCTL_PREPARE

> 数据传输的准备工作

```
snd_pcm_common_ioctl1
  | <SNDRV_PCM_IOCTL_PREPARE>
  |-> snd_pcm_prepare
	|
	|-> snd_power_wait  //wait until the power-state is changed
	|-> snd_pcm_action_nonatomic
	  |
	  |-> snd_pcm_action_nonatomic
		|
		|-> snd_pcm_action_single
		  |
          |-> ops->pre_action(substream, state)
		  |-> ops->do_action(substream, state)
		  |-> ops->post_action(substream, state)
```

回调函数注册:
``` C
static struct action_ops snd_pcm_action_prepare = {
    .pre_action = snd_pcm_pre_prepare,
    .do_action = snd_pcm_do_prepare,
    .post_action = snd_pcm_post_prepare
};
```

驱动实现:
``` C
static int snd_pcm_do_prepare(struct snd_pcm_substream *substream, int state)
{
    err = substream->ops->prepare(substream);
}
```

>ASOC 接口: ops->prepare <---> `soc_pcm_prepare`


### SNDRV_PCM_IOCTL_WRITEI_FRAMES

将audio控制器BAIC的FIFO写数据,即播放音频文件,开头分析部分

#### 写数据的组织结构

``` C
typedef unsigned long snd_pcm_uframes_t;
typedef signed long snd_pcm_sframes_t;

struct snd_xferi {
    snd_pcm_sframes_t result;  //?
    void __user *buf;          //写的真实数据
    snd_pcm_uframes_t frames;  //写数据的大小(4096) 4K
};

```

#### 数据传输dma_area

``` C
char *hwbuf = runtime->dma_area + frames_to_bytes(runtime, hwoff);
if (copy_from_user(hwbuf, buf, frames_to_bytes(runtime, frames)))
```

使用`tinyplay`播放,数据传输采用`map`方式,默认配置.


## close

通过系统调用close, 到release进行关闭

```
.release =      snd_pcm_release

snd_pcm_release
  |
  |-> snd_pcm_release_substream
	|
	|-> snd_pcm_drop
	  |
	  |-> snd_pcm_stop
		|
		|-> snd_pcm_action(&snd_pcm_action_stop, substream, state)
	|
	|-> substream->ops->hw_free(substream)
	|-> substream->ops->close(substream)
```

### snd_pcm_action_stop回调接口

```C
 static struct action_ops snd_pcm_action_stop = {
     .pre_action = snd_pcm_pre_stop,
     .do_action = snd_pcm_do_stop,
     .post_action = snd_pcm_post_stop
 };
```

### ASOC接口

1. snd_pcm_do_stop  <---> substream->ops->trigger  <---> `soc_pcm_trigger`
2. substream->ops->hw_free <--->  `soc_pcm_hw_free`
3. substream->ops->close  <--->  `soc_pcm_close`


## PCM的action函数 -- snd_pcm_action

### 函数实现:
``` C
static int snd_pcm_action(struct action_ops *ops,
              struct snd_pcm_substream *substream,
              int state)
{
   	...
    res = snd_pcm_action_single(ops, substream, state);
}
```

>此处的`struct action_ops`ops均是以回调注册的形式进行函数参数传入.

### 调用关系:
```
snd_pcm_action
  |
  |-> snd_pcm_action_single
	|
	|-> ops->pre_action(substream, state);		//1. 准备
    |-> ops->do_action(substream, state);       //2. 执行
 	|-> ops->post_action(substream, state);     //3. 下一步
	|-> ops->undo_action(substream, state);		//4. 撤销还原
```


## ASOC接口的具体相关实现:

>所谓ASOC接口, 根据自己理解主要是为soc层的各个芯片厂商的IP提供一个统一的驱动实现接口

### ASOC接口的回调注册

``` C
rtd->ops.open       = soc_pcm_open;
rtd->ops.hw_params  = soc_pcm_hw_params;
rtd->ops.prepare    = soc_pcm_prepare;
rtd->ops.trigger    = soc_pcm_trigger;
rtd->ops.hw_free    = soc_pcm_hw_free;
rtd->ops.close      = soc_pcm_close;
rtd->ops.pointer    = soc_pcm_pointer;
rtd->ops.ioctl      = soc_pcm_ioctl;
```
>func: `soc_new_pcm`  file: sound/soc/soc-pcm.c

>注: X1000 Phoenix Audio asoc-v13

### soc_pcm_open

```
static int soc_pcm_open(struct snd_pcm_substream *substream)
{
	...
	// CPU <I2S> : jz_i2s_startup
	if (cpu_dai->driver->ops->startup) {
		 ret = cpu_dai->driver->ops->startup(substream, cpu_dai);
	}
	// Platform <DMA> : jz_pcm_open
	if (platform->driver->ops && platform->driver->ops->open) {
		 ret = platform->driver->ops->open(substream);
	}
	// Codec <idec_d3> : jz_icdc_startup
	if (codec_dai->driver->ops->startup) {
		 ret = codec_dai->driver->ops->startup(substream, codec_dai);
	}
 	// Machine <link> : phoenix_spk_sup  file:sound/soc/ingenic/asoc-board/phoenix_icdc.c
	if (rtd->dai_link->ops && rtd->dai_link->ops->startup) {
		 ret = rtd->dai_link->ops->startup(substream);
	}
	...
}
```

### soc_pcm_hw_params

``` C
static int soc_pcm_hw_params(struct snd_pcm_substream *substream,
                struct snd_pcm_hw_params *params)
{
	 ...
	 // Machine <link> : phoenix_i2s_hw_params
	 if (rtd->dai_link->ops && rtd->dai_link->ops->hw_params) {
		 ret = rtd->dai_link->ops->hw_params(substream, params);
	 }
	 // Codec <idec_d3> : icdc_d3_hw_params
	 if (codec_dai->driver->ops->hw_params) {
		 ret = codec_dai->driver->ops->hw_params(substream, params, codec_dai);
	 }
	 // CPU <I2S> : jz_i2s_hw_params
	 if (cpu_dai->driver->ops->hw_params) {
		 ret = cpu_dai->driver->ops->hw_params(substream, params, cpu_dai);
	 }
	 // Platform <DMA> : jz_pcm_hw_params
	 if (platform->driver->ops && platform->driver->ops->hw_params) {
		 ret = platform->driver->ops->hw_params(substream, params);
 	}
 	...
}
```
### soc_pcm_prepare

``` C
static int soc_pcm_prepare(struct snd_pcm_substream *substream)
{
	...
	// Machine <link> : phoenix_i2s_hw_params
	if (rtd->dai_link->ops && rtd->dai_link->ops->prepare) {
		ret = rtd->dai_link->ops->prepare(substream);
	}
	// Platform <DMA> :	jz_pcm_prepare
	if (platform->driver->ops && platform->driver->ops->prepare) {
		ret = platform->driver->ops->prepare(substream);
	}
   	// Codec <idec_d3> : 默认函数
	if (codec_dai->driver->ops->prepare) {
		ret = codec_dai->driver->ops->prepare(substream, codec_dai);
	}
	// CPU <I2S> : 默认函数
	if (cpu_dai->driver->ops->prepare) {
		ret = cpu_dai->driver->ops->prepare(substream, cpu_dai);
	}
    ...
}
```

### soc_pcm_trigger

``` C
static int soc_pcm_trigger(struct snd_pcm_substream *substream, int cmd)
{
	// Codec <idec_d3> : icdc_d3_trigger
	if (codec_dai->driver->ops->trigger) {
		ret = codec_dai->driver->ops->trigger(substream, cmd, codec_dai);
	}
	// Platform <DMA> :	jz_pcm_trigger
	if (platform->driver->ops && platform->driver->ops->trigger) {
		ret = platform->driver->ops->trigger(substream, cmd);
	}
	// CPU <I2S> : jz_i2s_trigger
	if (cpu_dai->driver->ops->trigger) {
		ret = cpu_dai->driver->ops->trigger(substream, cmd, cpu_dai);
	}
    ...
}
```

### soc_pcm_hw_free

``` C
static int soc_pcm_hw_free(struct snd_pcm_substream *substream)
{
	/* free any machine hw params */
	// Machine <link> : phoenix_i2s_hw_free
	if (rtd->dai_link->ops && rtd->dai_link->ops->hw_free)
		rtd->dai_link->ops->hw_free(substream);

	/* free any DMA resources */
	// Platform <DMA> : snd_pcm_lib_free_pages
	if (platform->driver->ops && platform->driver->ops->hw_free)
		platform->driver->ops->hw_free(substream);

	/* now free hw params for the DAIs  */
	// Codec <idec_d3> : 默认函数
	if (codec_dai->driver->ops->hw_free)
		codec_dai->driver->ops->hw_free(substream, codec_dai);
	// CPU <I2S> : 默认函数
	if (cpu_dai->driver->ops->hw_free)
		cpu_dai->driver->ops->hw_free(substream, cpu_dai);
    ...
}
```

### soc_pcm_close

``` C
static int soc_pcm_close(struct snd_pcm_substream *substream)
{
	 // CPU <I2S> : jz_i2s_shutdown
	if (cpu_dai->driver->ops->shutdown)
		cpu_dai->driver->ops->shutdown(substream, cpu_dai);
	// Codec <idec_d3> : jz_icdc_shutdown
	if (codec_dai->driver->ops->shutdown)
		codec_dai->driver->ops->shutdown(substream, codec_dai);
	// Machine <link> : phoenix_spk_sdown
	if (rtd->dai_link->ops && rtd->dai_link->ops->shutdown)
		rtd->dai_link->ops->shutdown(substream);
	// Platform <DMA> :	jz_pcm_close
	if (platform->driver->ops && platform->driver->ops->close)
		platform->driver->ops->close(substream);
	...
}
```
## 参考

1. [linux_sound_alsa_ALSA体系SOC子系统中数据流分析 ](http://blog.chinaunix.net/uid-20776117-id-3085423.html)
2. [linux_sound_alsa_ALSA体系SOC子系统中hw_params逻辑 ](http://blog.chinaunix.net/uid-20776117-id-3085420.html)
3. [ ALSA声卡07_分析调用过程_学习笔记](http://blog.csdn.net/qingkongyeyue/article/details/54617950)
