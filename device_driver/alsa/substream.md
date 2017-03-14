# substream


## substream


## substream的状态

``` C
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


