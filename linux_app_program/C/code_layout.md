# 代码的分布


## 相关概念

### 段

#### 代码段(code segment/text segment)

>通常是指用来存放程序执行代码的一块内存区域。这部分区域的大小在程序运行前就已经确定，并且内存区域通常属于只读, 某些架构也允许代码段为可写，即允许修改程序。在代码段中，也有可能包含一些只读的常数变量，例如字符串常量等。

#### 数据段(data segment)

>通常是指用来存放程序中已初始化的全局变量的一块内存区域。数据段属于静态内存分配。

#### BSS段(bss segment)

>通常是指用来存放程序中未初始化的全局变量的一块内存区域。BSS是英文Block Started by Symbol的简称。BSS段属于静态内存分配。




## 测试代码:

``` C
#include<stdio.h>

int aaa;
int aaaa;

unsigned int wqshao = 123456;

int main()
{
    unsigned int a;
    unsigned int b = 1;

    static unsigned int xxx = 66;

    printf("xxxxxxxxxxxxxxxxxxx\n");
    return 0;
}
```
>环境: ubuntu 14.04
>gcc: mips-linux-gnu-

### 不同段的位置

> $mips-linux-gnu-objdump -x a.out > aaa.s


``` asm
Sections:
Idx Name          Size      VMA       LMA       File off  Algn
  0 .interp       0000000d  00400154  00400154  00000154  2**0
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  1 .note.ABI-tag 00000020  00400164  00400164  00000164  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  2 .MIPS.abiflags 00000018  00400188  00400188  00000188  2**3
                  CONTENTS, ALLOC, LOAD, READONLY, DATA, LINK_ONCE_SAME_SIZE
  3 .reginfo      00000018  004001a0  004001a0  000001a0  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA, LINK_ONCE_SAME_SIZE
  4 .dynamic      00000100  004001b8  004001b8  000001b8  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  5 .hash         00000034  004002b8  004002b8  000002b8  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  6 .dynsym       00000080  004002ec  004002ec  000002ec  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  7 .dynstr       0000006b  0040036c  0040036c  0000036c  2**0
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  8 .gnu.version  00000010  004003d8  004003d8  000003d8  2**1
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  9 .gnu.version_r 00000020  004003e8  004003e8  000003e8  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 10 .rel.plt      00000010  00400408  00400408  00000408  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 11 .init         0000006c  00400418  00400418  00000418  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
 12 .text         00000340  00400490  00400490  00000490  2**4
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
 13 .fini         0000003c  004007d0  004007d0  000007d0  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
 14 .rodata       00000030  00400810  00400810  00000810  2**4
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 15 .eh_frame     00000004  00400840  00400840  00000840  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 16 .plt          00000040  00400860  00400860  00000860  2**5
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
 17 .ctors        00000008  004108a0  004108a0  000008a0  2**2
                  CONTENTS, ALLOC, LOAD, DATA
 18 .dtors        00000008  004108a8  004108a8  000008a8  2**2
                  CONTENTS, ALLOC, LOAD, DATA
 19 .jcr          00000004  004108b0  004108b0  000008b0  2**2
                  CONTENTS, ALLOC, LOAD, DATA
 20 .got.plt      00000010  004108b4  004108b4  000008b4  2**2
                  CONTENTS, ALLOC, LOAD, DATA
 21 .data         00000020  004108d0  004108d0  000008d0  2**4
                  CONTENTS, ALLOC, LOAD, DATA
 22 .rld_map      00000004  004108f0  004108f0  000008f0  2**2
                  CONTENTS, ALLOC, LOAD, DATA
 23 .got          00000018  00410900  00410900  00000900  2**4
                  CONTENTS, ALLOC, LOAD, DATA
 24 .sdata        00000004  00410918  00410918  00000918  2**2
                  CONTENTS, ALLOC, LOAD, DATA
 25 .sbss         00000004  0041091c  0041091c  0000091c  2**2
                  ALLOC
 26 .bss          00000010  00410920  00410920  0000091c  2**4
                  ALLOC
 27 .comment      0000002e  00000000  00000000  0000091c  2**0
                  CONTENTS, READONLY
 28 .pdr          00000060  00000000  00000000  0000094c  2**2
                  CONTENTS, READONLY
 29 .debug_frame  00000050  00000000  00000000  000009ac  2**2
                  CONTENTS, READONLY, DEBUGGING
 30 .gnu.attributes 00000010  00000000  00000000  000009fc  2**0
                  CONTENTS, READONLY
 31 .mdebug.abi32 00000000  00000000  00000000  00000a0c  2**0
                  CONTENTS, READONLY
```

### data段,BSS段和text段

各个段的大小:

``` C
=====>$size a.out
   text	   data	    bss	    dec	    hex	filename
   1816	    100	     24	   1940	    794	a.out
```

>$mips-linux-gnu-objdump -D a.out > aaa.s

1. Data

``` asm
Disassembly of section .data:

004108d0 <__data_start>:
    ...

004108e0 <wqshao>:
  4108e0:   0001e240    sll gp,at,0x9

004108e4 <xxx.1939>:
  4108e4:   00000042    srl zero,zero,0x1
    ...
```

2. BSS

``` asm
Disassembly of section .sbss:

0041091c <__bss_start>:
  41091c:   00000000    nop

00410920 <aaa>:
  410920:   00000000    nop

Disassembly of section .bss:

00410930 <completed.6399>:
  410930:   00000000    nop

00410934 <dtor_idx.6401>:
    ...
```
3. Text

``` asm
Disassembly of section .text:

00400490 <__start>:
  400490:   3c1c0042    lui gp,0x42
  400494:   279c88f0    addiu   gp,gp,-30480
  400498:   0000f825    move    ra,zero
  40049c:   3c040040    lui a0,0x40
  4004a0:   24840670    addiu   a0,a0,1648
  4004a4:   8fa50000    lw  a1,0(sp)
  4004a8:   27a60004    addiu   a2,sp,4
  4004ac:   2401fff8    li  at,-8
  4004b0:   03a1e824    and sp,sp,at
  4004b4:   27bdffe0    addiu   sp,sp,-32

    ...
```

### 地址的分配和内存映射的关系

``` C

    a.out                         进程的空间



------------

    BSS

------------  <-- 0x0041091c

    Data

------------  <-- 0x004108d0

    Text

------------  <-- 0x00400490

```

## 程序的执行

1. 为什么入口是`main`函数,是必须的吗
2. `#include`进的头文件的作用


### 没有main函数的程序

``` C
#include <stdio.h>
#include <stdlib.h>

int swq()
{
    printf("no main function\n");

    exit(0);
}
```
```
$gcc swq.c -e swq -nostartfiles
```
因此,main函数不是必须的,同样也不是入口

``` asm
Disassembly of section .text:

00400360 <swq>:
  400360:   27bdffe0    addiu   sp,sp,-32
  400364:   afbf001c    sw  ra,28(sp)
  400368:   afbe0018    sw  s8,24(sp)
  40036c:   03a0f025    move    s8,sp
  400370:   3c020040    lui v0,0x40
  400374:   24440390    addiu   a0,v0,912
  400378:   0c1000f8    jal 4003e0 <puts@plt>
  40037c:   00000000    nop
  400380:   00002025    move    a0,zero
  400384:   0c1000fc    jal 4003f0 <exit@plt>
  400388:   00000000    nop
  40038c:   00000000    nop
```
### 程序的入口









## 参考

1. [ Linux - 进程(一) 进程空间](http://blog.csdn.net/zhangzhebjut/article/details/39060253)
