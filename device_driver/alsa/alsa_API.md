# API


## snd_pcm_hw_constraint_integer






## 宏定义


## trigger command

``` C
#define SNDRV_PCM_TRIGGER_STOP      0     //停止
#define SNDRV_PCM_TRIGGER_START     1     //开始      
#define SNDRV_PCM_TRIGGER_PAUSE_PUSH    3 //暂停 开始           
#define SNDRV_PCM_TRIGGER_PAUSE_RELEASE 4 //暂停 结束  
#define SNDRV_PCM_TRIGGER_SUSPEND   5     //挂起      
#define SNDRV_PCM_TRIGGER_RESUME    6     //唤起            
```
>include/sound/pcm.h 

对应接口:

``` C
snd_pcm_action_stop 
snd_pcm_action_start 
snd_pcm_action_pause         
snd_pcm_action_resume    
snd_pcm_action_suspend   
```
>sound/core/pcm_native.c  

