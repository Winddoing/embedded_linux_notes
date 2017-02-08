

## handle_ri_rdhwr

应用程序进入异常循环原因:

>file: arch/mips/kernel/traps.c

根据ExcCode域的值对通用异常处理表进行初始化，注册各个通用异常处理程序。可以查看`CACUS`寄存器`ExcCode`域的值所对应的异常种类。

set_except_vector(10, rdhwr_noopt ? handle_ri :
           (cpu_has_vtag_icache ?                                              
            handle_ri_rdhwr_vivt : handle_ri_rdhwr));

>Excode [10 ---  RI]  一条CPU没有定义的指令(或非法指令)

异常处理:handle_ri_rdhwr

>file: arch/mips/kernel/genex.S


### cpu_has_vtag_icache

>file: arch/mips/xburst2/soc-x2000/include/cpu-feature-overrides.h
#define cpu_has_vtag_icache             0 


### rdhwr_noopt

uboot传入参数决定, 当前没有使用该参数.

>#define CONFIG_BOOTARGS BOOTARGS_COMMON xxxx

## RI

 一条CPU没有定义的指令(或非法指令)

## rdhwr

mips指令(read hardware register),读取硬件寄存器

rdhwr向用户态提供了对某些具体CPU信息的只读访问.

