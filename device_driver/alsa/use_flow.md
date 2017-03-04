# 使用流程



## 节点

```
# ls
controlC0  pcmC0D0c   pcmC0D0p   pcmC0D1c   pcmC0D1p   pcmC0D2c   timer
```

## 创建

```
snd_pcm_new
snd_pcm_new_internal
     |
	 |______snd_pcm_new
				|
				|___
					static struct snd_device_ops ops = {
   							 .dev_free = snd_pcm_dev_free,
   							 .dev_register = snd_pcm_dev_register,               
    						 .dev_disconnect = snd_pcm_dev_disconnect,
					};

				| -> snd_device_new  //create an ALSA device component

```

驱动的加载:函数调用

snd_soc_register_card --> snd_soc_instantiate_card --> snd_card_register --> snd_device_register_all --> dev->ops->dev_register(dev)




## dapm
动态音频电源管理(Dynamic Audio Power Management)
