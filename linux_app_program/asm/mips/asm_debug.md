# mips汇编


## 通过指令码得到汇编指令

```
800104b8:   00641821    addu    v1,v1,a0     
800104bc:   00604021    move    t0,v1        
```

通过'00641821'得到'addu    v1,v1,a0'
### 查找CPU手册

### 通过编译器dump

``` C
#include<stdio.h>                              
            
int main()                                     
{                                              
    __asm__ __volatile__ (                     
            "   .word   0x00641821     \n\t"   
            "   .word   0x00604021   \n\t"     
            );                                 
                                               
    return 0;                                  
}                                              
```

#### 编译

```
mips-linux-gnu-gcc a.c
```

#### 反汇编

```
mips-linux-gnu-objdump -D a.out > aa.S
```

#### 结果

``` asm
    ...                                             
                                                    
00400640 <main>:                                    
  400640:   27bdfff8    addiu   sp,sp,-8            
  400644:   afbe0004    sw  s8,4(sp)                
  400648:   03a0f025    move    s8,sp               
  
  40064c:   00641821    addu    v1,v1,a0            # 指令码
  400650:   00604021    move    t0,v1               #
  
  400654:   00001025    move    v0,zero             
  400658:   03c0e825    move    sp,s8               
  40065c:   8fbe0004    lw  s8,4(sp)                
  400660:   27bd0008    addiu   sp,sp,8             
  400664:   03e00008    jr  ra                      
  400668:   00000000    nop                         
  40066c:   00000000    nop                         
                                                    
    ...
```

## 原因

