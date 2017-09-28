# OSS

声卡注册：
```
register_sound_special(&xb_snd_fops, ddata->minor);  
```

```
const struct file_operations xb_snd_fops = {                    
    .owner = THIS_MODULE,                                       
    .llseek = xb_snd_llseek,                                    
    .read = xb_snd_read,                                        
    .write = xb_snd_write,                                      
    .poll = xb_snd_poll,                                        
    .unlocked_ioctl = xb_snd_ioctl,                             
    .mmap = xb_snd_mmap,                                        
    .open = xb_snd_open,                                        
    .release = xb_snd_release,                                  
};                                                              
```

## OPEN







## READ


## WIRTE
