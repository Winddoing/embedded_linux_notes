# do_div

## 头文件:

    arch/arm/include/asm/div64.h

## 用法:

``` C
#include <asm/div64.h>

unsigned long long x,y,result;
unsigned long mod;

mod = do_div(x,y);
result = x; 
```

>64 bit division 结果保存在x中；余数保存在返回结果中。
