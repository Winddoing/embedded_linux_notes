# ALSA


## 





ASoC被分为`Machine`、`Platform`和`Codec`三大部分。其中的Machine驱动负责Platform和Codec之间的耦合和设备或板子特定的代码。

Platform驱动的主要作用是完成音频数据的管理，最终通过CPU的数字音频接口（DAI）把音频数据传送给Codec进行处理，最终由Codec输出驱动耳机或者是喇叭的音信信号。

machine


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






## 参考:

1. [ Linux ALSA声卡驱动](http://blog.csdn.net/droidphone/article/details/6271122)
