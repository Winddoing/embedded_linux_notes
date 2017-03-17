# Platform



Platform驱动的主要作用是完成音频数据的管理，最终通过CPU的数字音频接口（DAI）把音频数据传送给Codec进行处理，最终由Codec输出驱动耳机或者是喇叭的音信信号。

实现上，ASoC有把Platform驱动分为两个部分：snd_soc_platform_driver和snd_soc_dai_driver。
	platform_driver负责管理音频数据，把音频数据通过dma或其他操作传送至cpu dai中，
	dai_driver则主要完成cpu一侧的dai的参数配置，同时也会通过一定的途径把必要的dma等参数与snd_soc_platform_driver进行交互


## 注册

```
int snd_soc_register_platform(struct device *dev,
        const struct snd_soc_platform_driver *platform_drv) 
{
	...
}
```
实现:
为snd_soc_platform实例申请内存；
从platform_device中获得它的名字，用于Machine驱动的匹配工作；
初始化snd_soc_platform的字段；
把snd_soc_platform实例连接到全局链表platform_list中；
调用snd_soc_instantiate_cards，触发声卡的machine、platform、codec、dai等的匹配工作；

### struct snd_soc_platform_driver 

```
struct snd_soc_platform_driver {
                                                                                          
	...
    /* pcm creation and destruction */
    int (*pcm_new)(struct snd_soc_pcm_runtime *);
    void (*pcm_free)(struct snd_pcm *);
 	...
    /* platform stream pcm ops */
    const struct snd_pcm_ops *ops;
};
```



### struct snd_pcm_ops

```
struct snd_pcm_ops {
    int (*open)(struct snd_pcm_substream *substream);
    int (*close)(struct snd_pcm_substream *substream);
    int (*ioctl)(struct snd_pcm_substream * substream,
             unsigned int cmd, void *arg);
    int (*hw_params)(struct snd_pcm_substream *substream,
             struct snd_pcm_hw_params *params);			 //驱动的hw_params阶段，该函数会被调用
    int (*hw_free)(struct snd_pcm_substream *substream);
    int (*prepare)(struct snd_pcm_substream *substream); //开始数据传送之前会调用该函数，该函数通常会完成dma操作的必要准备工作
    int (*trigger)(struct snd_pcm_substream *substream, int cmd);  //数据传送的开始，暂停，恢复和停止时，该函数会被调用
    snd_pcm_uframes_t (*pointer)(struct snd_pcm_substream *substream);  //该函数返回传送数据的当前位置。
  	...
};
```

### struct snd_soc_pcm_runtime

``` C
/* SoC machine DAI configuration, glues a codec and cpu DAI together */                    
struct snd_soc_pcm_runtime {                                                               
    struct device *dev;                                                                    
    struct snd_soc_card *card;                                                             
    struct snd_soc_dai_link *dai_link;                                                     
    struct mutex pcm_mutex;                                                                
    enum snd_soc_pcm_subclass pcm_subclass;                                                
    struct snd_pcm_ops ops;                                                                
                                                                                           
    unsigned int dev_registered:1;                                                         
                                                                                           
    /* Dynamic PCM BE runtime data */                                                      
    struct snd_soc_dpcm_runtime dpcm[2];                                                   
                                                                                           
    long pmdown_time;                                                                      
    unsigned char pop_wait:1;                                                              
                                                                                           
    /* runtime devices */                                                                  
    struct snd_pcm *pcm;                                                                   
    struct snd_compr *compr;                                                               
    struct snd_soc_codec *codec;                                                           
    struct snd_soc_platform *platform;                                                     
    struct snd_soc_dai *codec_dai;                                                         
    struct snd_soc_dai *cpu_dai;                                                           
                                                                                           
    struct delayed_work delayed_work;                                                      
#ifdef CONFIG_DEBUG_FS                                                                     
    struct dentry *debugfs_dpcm_root;                                                      
    struct dentry *debugfs_dpcm_state;                                                     
#endif                                                                                     
};                                                                                         

```
## 注册

``` C
static struct snd_soc_platform_driver jz_pcm_platform = {               
    .ops            = &jz_pcm_ops,                                      
    .pcm_new        = jz_pcm_new,                                       
    .pcm_free       = jz_pcm_free,                                      
};                                                                      
```

```
snd_soc_register_platform(&pdev->dev, &jz_pcm_platform)
    |
    |-> platform->driver = platform_drv
    |-> list_add(&platform->list, &platform_list)
```

## 绑定

```
snd_soc_register_card
    |
    |-> snd_soc_instantiate_card
        |
        |-> soc_bind_dai_link
            |
            |/* find one from the set of registered platforms */    
            |-> list_for_each_entry(platform, &platform_list, list) {  
                    if (dai_link->platform_of_node) {                  
                        if (platform->dev->of_node !=                  
                            dai_link->platform_of_node)                
                            continue;                                  
                    } else {                                           
                        if (strcmp(platform->name, platform_name))     
                            continue;                                  
                    }                                                  
                                                                       
                    rtd->platform = platform;                          
                }                                                         

```

## 回调

```
snd_soc_instantiate_card
    |
    |-> soc_probe_link_dais
        |
        |-> soc_new_pcm
            |
            |-> 第二次注册 { rtd->ops.open       = soc_pcm_open; ...}
            |-> platform->driver->pcm_new(rtd)
```

