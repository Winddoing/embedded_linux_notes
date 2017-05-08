# linker

>以Android6.0中的bionic的linker源码


## ELF文件

>ELF(Executable and Linking Format)是一种对象文件的格式


## linker的整体框架

以MIPS架构为例:

1. 加载linker

``` C
 96     /* call linker_init */                                                                                    
 97     move    $a0, $sp                                                                                                                                                                                                               
 98     addiu    $sp, -4*4        /* space for arg saves in linker_init */                                        
 99     la    $t9, __linker_init                                                                                  
100     jalr    $t9                                                                                               
101     move    $t9, $v0                                                                                          
102     addu    $sp, 4*4        /* restore sp */                                                                  
103     j    $t9                                                                                           
```
>android-6.0/bionic/linker/arch/mips/begin.S



## 参考

1. [可执行文件（ELF）格式的理解](http://www.cnblogs.com/xmphoenix/archive/2011/10/23/2221879.html)
