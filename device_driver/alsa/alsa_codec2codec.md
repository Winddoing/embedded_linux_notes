# Codec to Codec

## 硬件结构

```
+---------------------------------+
|                                 |
|             CPU                 |
|                                 |
+----^---------------------^------+
     |                     |
     |                     |
 +---+---+            +----+----+
 |       +------------>         |
 | codec |            |  codec  |
 |       +------------>         |
 +-------+            +---------+
```
> 两个Codec进行直连,CPU只提供MCLK时钟

## 软件的配置

```
static const struct snd_soc_pcm_stream sub_params = {            
    .formats = SNDRV_PCM_FMTBIT_S32_LE,                          
    .rate_min = SYS_AUDIO_RATE,                                  
    .rate_max = SYS_AUDIO_RATE,                                  
    .channels_min = 2,                                           
    .channels_max = 2,                                           
};                                                               

static struct snd_soc_dai_link bells_dai_wm5110[] = {          
{
    ....
     {                                                                                        
     .name = "Sub",                                                                       
     .stream_name = "Sub",                                                                
     .cpu_dai_name = "wm5110-aif3",                                                       
     .codec_dai_name = "wm9081-hifi",                                                     
     .codec_name = "wm9081.1-006c",                                                       
     .dai_fmt = SND_SOC_DAIFMT_I2S | SND_SOC_DAIFMT_NB_NF                                 
             | SND_SOC_DAIFMT_CBS_CFS,                                                    
     .ignore_suspend = 1,                                                                 
     .params = &sub_params,                                                               
     },                                                                                       
    ....
}                                                          
```
> 在dailink中添加以上结构,将其中一个codec(录入数据)当作cpu dai进行配置

## 基本原理

1. 由于dai link中有`params`对该link的初始化使用`soc_link_dai_widgets`
2. 通过`soc_link_dai_widgets`建立一个内部的wedget将两个codec进行连接,并且该wedget添加的事件最终可以回调到codec驱动中的`startup`和`hw_params`接口进行codec的配置,从而数据传输

## 使用过程

根据实际的需求分别将两个codec的录音和放音通路连通,连通后进行上电,回调每个wedget中的event,两个codec形成的内部wedget同样被回调最终调用`startup`和`hw_params`接口进行codec的配置,进行数据传输.

## 参考

1. 代码: sound/soc/samsung/bells.c
