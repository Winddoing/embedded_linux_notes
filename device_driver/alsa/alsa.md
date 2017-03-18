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



## 参考:

1. [ Linux ALSA声卡驱动](http://blog.csdn.net/droidphone/article/details/6271122)
2. [ALSA框架](http://www.360doc.com/content/12/0731/17/10388890_227508444.shtml)
3. [alsa架构音频分析总结](http://blog.csdn.net/shen924/article/details/8775352)
