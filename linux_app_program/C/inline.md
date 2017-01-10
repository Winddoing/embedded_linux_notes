# 内联函数

内联函数从源代码层看，有函数的结构，而在编译后，却不具备函数的性质。编译时，类似宏替换，使用函数体替换调用处的函数名。一般在代码中用`inline`修饰，但是否能形成内联函数，需要看编译器对该函数定义的具体处理。

## 优点:

* 内联扩展是用来消除函数调用时的时间开销。它通常用于频繁执行的函数。 一个小内存空间的函数非常受益。
* 提高函数调用效率

>函数是一种更高级的抽象。它的引入使得编程者只关心函数的功能和使用方法，而不必关心函数功能的具体实现；函数的引入可以减少程序的目标代码，实现程序代码和数据的共享。但是，函数调用也会带来降低效率的问题，因为调用函数实际上将程序执行顺序转移到函数所存放在内存中某个地址，将函数的程序内容执行完后，再返回到转去执行该函数前的地方。这种转移操作要求在转去前要保护现场并记忆执行的地址，转回后先要恢复现场，并按原来保存地址继续执行。因此，函数调用要有一定的时间和空间方面的开销，于是将影响其效率。特别是对于一些函数体代码不是很大，但又频繁地被调用的函数来讲，解决其效率问题更为重要。引入内联函数实际上就是为了解决这一问题。 

## static inline

促使编译程序尝试着将其代码插入到所有调用它的程序中。

## 示例:

### 源码:
``` C
inline unsigned int test1(unsigned int a, unsigned int b)                           
{
    unsigned int ret;

    ret = a + b;

    return ret;
}

static inline unsigned int test2(unsigned int a, unsigned int b)
{
    unsigned int ret;

    ret = a * b;

    return ret;
}

int main(int argc, char const* argv[])
{

    unsigned a, b;

    a = test1(3, 6);

    b = test2(2, 5);

    printf("a=%d, b=%d\n", a, b);

    return 0;
}                                                                           
```
### 编译:
``` shell
$mips-linux-gnu-gcc inline_test.c -o inline_test+O2 -O2
$mips-linux-gnu-objdump -d inline_test+O2 > a.s
```
### 反汇编:
``` asm
inline_test+O2:     file format elf32-tradlittlemips


Disassembly of section .text:

00000000 <test1>:
   0:   03e00008    jr  ra
   4:   00a41021    addu    v0,a1,a0
    ...

Disassembly of section .text.startup:

00000000 <main>:
   0:   3c040000    lui a0,0x0
   4:   27bdffe0    addiu   sp,sp,-32                                  
   8:   24840000    addiu   a0,a0,0
   c:   24050009    li  a1,9
  10:   afbf001c    sw  ra,28(sp)
  14:   0c000000    jal 0 <main>
  18:   2406000a    li  a2,10
  1c:   8fbf001c    lw  ra,28(sp)
  20:   00001021    move    v0,zero
  24:   03e00008    jr  ra
  28:   27bd0020    addiu   sp,sp,32
```
使用了系统调用printf
