# ALSA


##





ASoC被分为`Machine`、`Platform`和`Codec`三大部分。其中的Machine驱动负责Platform和Codec之间的耦合和设备或板子特定的代码。

Platform驱动的主要作用是完成音频数据的管理，最终通过CPU的数字音频接口（DAI）把音频数据传送给Codec进行处理，最终由Codec输出驱动耳机或者是喇叭的音信信号。

### machine

>是指某一款机器，可以是某款设备，某款开发板，又或者是某款智能手机，由此可以看出Machine几乎是不可重用的，每个Machine上的硬件实现可能都不一样，CPU不一样，Codec不一样，音频的输入、输出设备也不一样，Machine为CPU、Codec、输入输出设备提供了一个`载体`。

### Platform

> 一般是指某一个SoC平台，比如pxaxxx,s3cxxxx,omapxxx等等，与音频相关的通常包含该SoC中的时钟、DMA、I2S、PCM等等，只要指定了SoC，那么我们可以认为它会有一个对应的Platform，它只与SoC相关，与Machine无关，这样我们就可以把Platform抽象出来，使得同一款SoC不用做任何的改动，就可以用在不同的Machine中。实际上，把Platform认为是某个SoC更好理解。

### Codec

> 字面上的意思就是编解码器，Codec里面包含了I2S接口、D/A、A/D、Mixer、PA（功放），通常包含多种输入（Mic、Line-in、I2S、PCM）和多个输出（耳机、喇叭、听筒，Line-out），Codec和Platform一样，是可重用的部件，同一个Codec可以被不同的Machine使用。嵌入式Codec通常通过I2C对内部的寄存器进行控制。


### struct snd_soc_dai_link


### struct snd_soc_pcm_runtime


snd_soc_aux_dev  -- 可选的辅助设备如放大器和编解码器codec

##

ASoC定义了三个全局的链表头变量：codec_list、dai_list、platform_list，系统中所有的Codec、DAI、Platform都在注册时连接到这三个全局链表上。

## dai

static LIST_HEAD(dai_list);


dai 分为 cpu_dai 和 codec_dai



注册:

snd_soc_register_component



snd_soc_register_codec


## platform

LIST_HEAD(platform_list);

注册:

snd_soc_add_platform   //Add a platform to the ASoC core

snd_soc_register_platform


## 初始化声卡


snd_soc_instantiate_card(card);
card->instantiated = 0;   //卡是否实例化


## 音频

样本长度(sample)：样本是记录音频数据最基本的单位，常见的有8位和16位。

通道数(channel)：该参数为1表示单声道，2则是立体声。

桢(frame)：桢记录了一个声音单元，其长度为样本长度与通道数的乘积。(16 * 2)

采样率(rate)：每秒钟采样次数，该次数是针对桢而言。

周期(period)：音频设备一次处理所需要的桢数，对于音频设备的数据访问以及音频数据的存储，都是以此为单位。


* Sample：样本长度，音频数据最基本的单位，常见的有8位和16位。

* Channel：声道数，分为单声道mono和立体声stereo。

* Frame：帧，构成一个声音单元，Frame = Sample * channel, sample*channel/8 Byte。

* Rate：又称Sample rate，采样率，即每秒的采样次数，针对帧而言。

* Interleaved：交错模式，一种音频数据的记录方式，在交错模式下，数据以连续桢的形式存放，即首先记录完桢1的左声道样本和右声道样本（假设为立体声），再开始桢2的记录。而在非交错模式下，首先记录的是一个周期内所有桢的左声道样本，再记录右声道样本，数据是以连续通道的方式存储。多数情况下使用交错模式。

* Period size：周期，每次硬件中断处理音频数据的帧数，对于音频设备的数据读写，以此为单位。

* Buffer size：数据缓冲区大小，这里特指runtime的buffer size，而不是snd_pcm_hardware定义的buffer_bytes_max。


>**Period**
>
>The interval between interrupts from the hardware. This defines the input latency, since the CPU will not have any idea that there is data waiting until the audio interface interrupts it.
>
>The audio interface has a "pointer" that marks the current position for read/write in its h/w buffer. The pointer circles around the buffer as long as the interface is running.
>
>Typically, there are an integral number of periods per traversal of the h/w buffer, but not always. There is at least one card (ymfpci)
>that generates interrupts at a fixed rate indepedent of the buffer size (which can be changed), resulting in some "odd" effects compared to more traditional designs.
>
>Note: h/w generally defines the interrupt in frames, though not always.
>
>Alsa's period size setting will affect how much work the CPU does. if you set the period size low, there will be more interrupts and the work that is done every interrupt will be done more often. So, if you don't care about low latency,
>set the period size large as possible and you'll have more CPU cycles for other things. The defaults that ALSA provides are in the middle of the range, typically.
>
>(from an old AlsaDevel thread[1], quoting Paul Davis)
>
>Retrieved from "http://alsa.opensrc.org/Period"
>
>来自：http://alsa.opensrc.org/Period
>
>**FramesPeriods**
>
>A frame is equivalent of one sample being played, irrespective of the number of channels or the number of bits. e.g.
>  * 1 frame of a Stereo 48khz 16bit PCM stream is 4 bytes.
>  * 1 frame of a 5.1 48khz 16bit PCM stream is 12 bytes.
>A period is the number of frames in between each hardware interrupt. The poll() will return once a period.
>The buffer is a ring buffer. The buffer size always has to be greater than one period size. Commonly this is 2*period size, but some hardware can do 8 periods per buffer. It is also possible for the buffer size to not be an integer multiple of the period size.
>Now, if the hardware has been set to 48000Hz , 2 periods, of 1024 frames each, making a buffer size of 2048 frames. The hardware will interrupt 2 times per buffer. ALSA will endeavor to keep the buffer as full as possible. Once the first period of samples has
>been played, the third period of samples is transfered into the space the first one occupied while the second period of samples is being played. (normal ring buffer behaviour).
>
>
>Additional example
>
>Here is an alternative example for the above discussion.
>Say we want to work with a stereo, 16-bit, 44.1 KHz stream, one-way (meaning, either in playback or in capture direction). Then we have:
>  * 'stereo' = number of channels: 2
>  * 1 analog sample is represented with 16 bits = 2 bytes
>  * 1 frame represents 1 analog sample from all channels; here we have 2 channels, and so:
>      * 1 frame = (num_channels) * (1 sample in bytes) = (2 channels) * (2 bytes (16 bits) per sample) = 4 bytes (32 bits)
>  * To sustain 2x 44.1 KHz analog rate - the system must be capable of data transfer rate, in Bytes/sec:
>      * Bps_rate = (num_channels) * (1 sample in bytes) * (analog_rate) = (1 frame) * (analog_rate) = ( 2 channels ) * (2 bytes/sample) * (44100 samples/sec) = 2*2*44100 = 176400 Bytes/sec
>Now, if ALSA would interrupt each second, asking for bytes - we'd need to have 176400 bytes ready for it (at end of each second), in order to sustain analog 16-bit stereo @ 44.1Khz.
>  * If it would interrupt each half a second, correspondingly for the same stream we'd need 176400/2 = 88200 bytes ready, at each interrupt;
>  * if the interrupt hits each 100 ms, we'd need to have 176400*(0.1/1) = 17640 bytes ready, at each interrupt.
>We can control when this PCM interrupt is generated, by setting a period size, which is set in frames.
>  * Thus, if we set 16-bit stereo @ 44.1Khz, and the period_size to 4410 frames => (for 16-bit stereo @ 44.1Khz, 1 frame equals 4 bytes - so 4410 frames equal 4410*4 = 17640 bytes) => an interrupt will be generated each 17640 bytes - that is, each 100 ms.
>  * Correspondingly, buffer_size should be at least 2*period_size = 2*4410 = 8820 frames (or 8820*4 = 35280 bytes).
>It seems (writing-an-alsa-driver.pdf), however, that it is the ALSA runtime that decides on the actual buffer_size and period_size, depending on: the requested number of channels, and their respective properties (rate and sampling resolution) - as well as the
>parameters set in the snd_pcm_hardware structure (in the driver).
>Also, the following quote may be relevant, from http://mailman.alsa-project.org/pipermail/alsa-devel/2007-April/000474.html:
>
>> > The "frame" represents the unit, 1 frame = # channels x sample_bytes.
>> > In your case, 1 frame corresponds to 2 channels x 16 bits = 4 bytes.
>> >
>> > The periods is the number of periods in a ring-buffer.  In OSS, called
>> > as "fragments".
>> >
>> > So,
>> >  - buffer_size = period_size * periods
>> >  - period_bytes = period_size * bytes_per_frame
>> >  - bytes_per_frame = channels * bytes_per_sample
>> >
>>
>> I still don't understand what 'period_size' and a 'period' is?
>
>
>The "period" defines the frequency to update the status, usually viathe invokation of interrupts.  The "period_size" defines the frame sizes corresponding to the "period time".  This term corresponds to the "fragment size" on OSS.  On major sound hardwares,
>a ring-buffer is divided to several parts and an irq is issued on each boundary. The period_size defines the size of this chunk.
>
>On some hardwares, the irq is controlled on the basis of a timer.  In this case, the period is defined as the timer frequency to invoke an irq.
>
>来自：http://alsa-project.org/main/index.php/FramesPeriods
>

## 音频处理软件

>  Audacity 2.0.5


## 驱动相关定义

```
 #define JZ_DMA_BUFFERSIZE (32*1024)
 static const struct snd_pcm_hardware jz_pcm_hardware = {
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
     .buffer_bytes_max       = JZ_DMA_BUFFERSIZE,
     .period_bytes_min       = PAGE_SIZE / 4, /* 1K */
     .period_bytes_max       = PAGE_SIZE * 2, /* 64K */
     .periods_min            = 4,
     .periods_max            = 64,
     .fifo_size              = 0,
 };
```
SNDRV_PCM_FMTBIT_S24_LE

LE  --   小端
BE  --   大端

S24 --
U24 --


## 状态机


```
typedef int __bitwise snd_pcm_state_t;                                                                        
#define SNDRV_PCM_STATE_OPEN        ((__force snd_pcm_state_t) 0) /* stream is open */                        
#define SNDRV_PCM_STATE_SETUP       ((__force snd_pcm_state_t) 1) /* stream has a setup */                    
#define SNDRV_PCM_STATE_PREPARED    ((__force snd_pcm_state_t) 2) /* stream is ready to start */              
#define SNDRV_PCM_STATE_RUNNING     ((__force snd_pcm_state_t) 3) /* stream is running */                     
#define SNDRV_PCM_STATE_XRUN        ((__force snd_pcm_state_t) 4) /* stream reached an xrun */                
#define SNDRV_PCM_STATE_DRAINING    ((__force snd_pcm_state_t) 5) /* stream is draining */                           
#define SNDRV_PCM_STATE_PAUSED      ((__force snd_pcm_state_t) 6) /* stream is paused */                      
#define SNDRV_PCM_STATE_SUSPENDED   ((__force snd_pcm_state_t) 7) /* hardware is suspended */                 
#define SNDRV_PCM_STATE_DISCONNECTED    ((__force snd_pcm_state_t) 8) /* hardware is disconnected */          
#define SNDRV_PCM_STATE_LAST        SNDRV_PCM_STATE_DISCONNECTED                                              
```

## 其他

* 采样率和实际的分频误差在5%左右



## 参考:

1. [ Linux ALSA声卡驱动](http://blog.csdn.net/droidphone/article/details/6271122)
2. [ALSA框架](http://www.360doc.com/content/12/0731/17/10388890_227508444.shtml)
3. [alsa架构音频分析总结](http://blog.csdn.net/shen924/article/details/8775352)
4. [alsa相关](http://www.alivepea.me/)
5. [Linux ALSA 音频系统：物理链路篇](http://www.itwendao.com/article/detail/290711.html)
6. [alsa driver](http://www.alsa-project.org/main/index.php/Minivosc)
7. [PCM data flow之二：Frames and Periods](http://www.xuebuyuan.com/1519752.html)
8. [alsa-project FramesPeriods](http://alsa-project.org/main/index.php/FramesPeriods)
9. [内核Alsa之pcm (转载)  ](http://kuafu80.blog.163.com/blog/static/12264718020148511458729/)
10. [声卡设备alsa音频架构1](http://www.cnblogs.com/jiangu66/archive/2013/05/06/3063406.html)
