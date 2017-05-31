# 系统调用--read


``` C
static ssize_t safe_read(int fd, void *buf, size_t count)                   
{                                                                           
    ssize_t result = 0, res;                                                
                                                                            
    while (count > 0) {                                                     
        if ((res = read(fd, buf, count)) == 0)                              
            break;                                                          
        if (res < 0)                                                        
            return result > 0 ? result : res;                               
        count -= res;                                                       
        result += res;                                                      
        buf = (char *)buf + res;                                            
    }                                                                       
    return result;                                                          
}                                                                           
```
