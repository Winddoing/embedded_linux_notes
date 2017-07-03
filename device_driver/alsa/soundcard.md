# 声卡

```
                          linux 
          __________________|________
          |        |
       snd_dev snd_dev  ....        
```

```
struct snd_device {                                                                        
    struct list_head list;      /* list of registered devices */                           
    struct snd_card *card;      /* card which holds this device */                         
    snd_device_state_t state;   /* state of the device */                                  
    snd_device_type_t type;     /* device type */                                          
    void *device_data;      /* device structure */                                         
    struct snd_device_ops *ops; /* operations */                                           
};                                                                                         
```


## 个数

正常使用最多可以实例化8个,使用`CONFIG_SND_DYNAMIC_MINORS`后可以到32个


## 状态

```
//准备好一个声卡所需的所有资源
#define SNDRV_DEV_BUILD     ((__force snd_device_state_t) 0)                  
//
#define SNDRV_DEV_REGISTERED    ((__force snd_device_state_t) 1)              
#define SNDRV_DEV_DISCONNECTED  ((__force snd_device_state_t) 2)              
```

## 类型

```
#define SNDRV_DEV_TOPLEVEL  ((__force snd_device_type_t) 0)                              
#define SNDRV_DEV_CONTROL   ((__force snd_device_type_t) 1)                              
#define SNDRV_DEV_LOWLEVEL_PRE  ((__force snd_device_type_t) 2)                          
#define SNDRV_DEV_LOWLEVEL_NORMAL ((__force snd_device_type_t) 0x1000)                   
#define SNDRV_DEV_PCM       ((__force snd_device_type_t) 0x1001)                         
#define SNDRV_DEV_RAWMIDI   ((__force snd_device_type_t) 0x1002)                         
#define SNDRV_DEV_TIMER     ((__force snd_device_type_t) 0x1003)                         
#define SNDRV_DEV_SEQUENCER ((__force snd_device_type_t) 0x1004)                         
#define SNDRV_DEV_HWDEP     ((__force snd_device_type_t) 0x1005)                         
#define SNDRV_DEV_INFO      ((__force snd_device_type_t) 0x1006)                         
#define SNDRV_DEV_BUS       ((__force snd_device_type_t) 0x1007)                         
#define SNDRV_DEV_CODEC     ((__force snd_device_type_t) 0x1008)                         
#define SNDRV_DEV_JACK          ((__force snd_device_type_t) 0x1009)                     
#define SNDRV_DEV_COMPRESS  ((__force snd_device_type_t) 0x100A)                         
#define SNDRV_DEV_LOWLEVEL  ((__force snd_device_type_t) 0x2000)                         
```

### SNDRV_DEV_CONTROL 



### SNDRV_DEV_PCM

```
snd_pcm_new
    |-> _snd_pcm_new
        |-> snd_device_new(card, SNDRV_DEV_PCM, pcm, &ops))
```



### SNDRV_DEV_TIMER
