# 进程退出的exitcode

## 错误信息

内核打印

```
 Kernel panic - not syncing: Attempted to kill init! exitcode=0x0000000b
```

## 分析

### 出错位置

``` C
 panic("Attempted to kill init! exitcode=0x%08x\n",
     father->signal->group_exit_code ?:
         father->exit_code);                                 
```
> kernel/exit.c

### exit_cede赋值

``` C
void do_exit(long code)
{
	...
	 tsk->exit_code = code;
	...
	exit_notify(tsk, group_dead);
	...
}
```

函数调用关系:
```
exit_notify
	|-> forget_original_parent(tsk);
				|-> find_new_reaper(father);
							|-> "Attempted to kill init! exitcode=0x%08x\n"
```

## 错误来源

在Android系统中,linux内核启动过程中,进入`用户空间`后,init进程执行过程中出现该错误

**由于在用户空间引起的内核错误,因此只能通过系统调用产生**

``` C
SYSCALL_DEFINE1(exit, int, error_code)                 
{
    do_exit((error_code&0xff)<<8);
}
```
在进入内核是do_exit取了用户空间传入的错误码的`低8位`

## 进程退出的错误码

在系统中的进程在正常和非正常退出时，都有一个表示当前进程退出状态的标识，即`退出码`

### 查看进程退出码

退出码代表的是一个进程退出的状态码, 可以使用wait函数进行查看。
``` C
void _exit(int status)，
```
>status表明了进程终止时的状态。当子进程使用_exit()后，父进程如果在用wait()等待子进程，那么wait()将会返回status状态，注意只有status的低8位（0~255）会返回给父进程

``` c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <unistd.h>
#include <errno.h>

int main(void)
{
    int count = 1;
    int pid;
    int status;

    pid = fork( );
    printf("pid=%d\n", pid);

    if(pid < 0) {
        perror("fork error : ");
    } else if(pid == 0) {
        printf("This is son, his count is: %d (%p). and his pid is: %d\n",
                ++count, &count, getpid());
        sleep(3);
        _exit(0);
    } else {
        pid = wait(&status);

        printf("This is father, his count is: %d (%p), his pid is: %d, son exit status: %d[%08x]\n",
                count, &count, getpid(), status, status);
    }

    return 0;
}                                                                                                                                 
```
正常退出结果：
``` shell
=====>$./a.out
pid=4018
pid=0
This is son, his count is: 2 (0x7fff19658714). and his pid is: 4018
This is father, his count is: 1 (0x7fff19658714), his pid is: 4017, son exit status: 0[00000000]
```
在子进程sleep时将其kill掉的结果：
``` shell
=====>$./a.out &
[1] 4066
00:11 [wqshao@machine]~/work/MyCode/systemcall/test
=====>$pid=4067
pid=0
This is son, his count is: 2 (0x7ffe19987d04). and his pid is: 4067

00:11 [wqshao@machine]~/work/MyCode/systemcall/test
=====>$kill 4067
This is father, his count is: 1 (0x7ffe19987d04), his pid is: 4066, son exit status: 15[0000000f]
```
在进程正常退出时，子进程的状态码是`0`，而kill掉后变为了`15`.

>注：此时如果在linux终端下使用`echo $?`,获取的仅仅该进程的main函数的返回值。

### 退出码的含义

根据前面分析，在进程调用_exit退出时,是通过exit系统调用实现的，而这里的`0`和`15`,就是系统调用exit的参数`error_code`

**进程的退出状态不等于退出码，程退出时候的状态码是8位，高4位存储退出码，低4位存储导致进程退出的信号标志位**

>网上有人说16位，分别是高八位和低八位，还需确认

根据这段话的描述，之前测试中子进程的退出状态`0`和`15`中，退出码均为`0`,而退出时的singal不同，正常退出时为0，kill掉后变为15

### 制造断错误

在测试case中的子进程中，制造一个段错误，根据此时的分析子进程退出的状态码中的signal应该代表段错误
子进程中添加：
``` C
int *a;
*a = 3;
```
x86测试结果：
``` shell
=====>$./a.out
pid=4500
pid=0
This is son, his count is: 2 (0x7fff54e86d1c). and his pid is: 4500
This is father, his count is: 1 (0x7fff54e86d1c), his pid is: 4499, son exit status: 139[0000008b]
```
此时子进程的`退出码=8`，而`signal=b`

mips下测试:
``` shell
# a.out 
pid=109
pid=0
This is son, his count is: 2 (0x7f85e578). and his pid is: 109
This is father, his count is: 1 (0x7f85e578), his pid is: 108, son exit status: 11[0000000b]
```
### 信号

linux内核中x86的信号列表：
``` C
#define SIGSEGV     11
#define SIGTERM     15
```
>arch/x86/include/uapi/asm/signal.h

| 信号 | 行为 | 产生原因 |
| ---- | ---- | --- |
| SIGTERM | 请求中断 | kill() 可以发 SIGTERM 过去；kill 命令默认也使用 SIGTERM 信号 |
| SIGSEGV | 无效内存引用| 段错误|

## 总结

进程在退出时都会将自己当前的状态告诉内核，而此时的`状态码`包含两种含义：

* 高4位代表当前进程的退出码
* 低4位代表使当前进程退出所使用的信号

**在本文最开始提到的错误也是由于`SIGSEGV`无效内存引用引起的。**

mips架构下的信号列表：

``` C
#define SIGTERM     15  /* Termination (ANSI).  */
```
>arch/mips/include/uapi/asm/signal.h

## 遗留问题

### 段错误时的系统调用

mips下:

``` shell
# strace a.out > a.txt
execve("/bin/a.out", ["a.out"], [/* 17 vars */]) = 0
brk(0)                                  = 0x411000
old_mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7793d000
uname({sys="Linux", node="buildroot", ...}) = 0
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
open("/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
fstat64(3, {st_mode=S_IFREG|0664, st_size=2812, ...}) = 0
old_mmap(NULL, 2812, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7793c000
close(3)                                = 0
open("/lib/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\1\1\1\0\0\0\0\0\0\0\0\0\3\0\10\0\1\0\0\0\244\247\1\0004\0\0\0"..., 512) = 512
lseek(3, 700, SEEK_SET)                 = 700
read(3, "\4\0\0\0\20\0\0\0\1\0\0\0GNU\0\0\0\0\0\2\0\0\0\6\0\0\0\f\0\0\0", 32) = 32
fstat64(3, {st_mode=S_IFREG|0775, st_size=1552564, ...}) = 0
old_mmap(NULL, 1531744, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x77798000
mprotect(0x778f6000, 65536, PROT_NONE)  = 0
old_mmap(0x77906000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x15e000) = 0x77906000
old_mmap(0x7790c000, 8032, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7790c000
close(3)                                = 0
old_mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7793b000
set_thread_area(0x77942490)             = 0
mprotect(0x77906000, 12288, PROT_READ)  = 0
mprotect(0x7793e000, 4096, PROT_READ)   = 0
munmap(0x7793c000, 2812)                = 0
clone(child_stack=0, flags=CLONE_CHILD_CLEARTID|CLONE_CHILD_SETTID|SIGCHLD, child_tidptr=0x7793b068) = 137
fstat64(1, {st_mode=S_IFREG|0644, st_size=0, ...}) = 0
old_mmap(NULL, 65536, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x77788000
wait4(-1, [{WIFSIGNALED(s) && WTERMSIG(s) == SIGSEGV}], 0, NULL) = 137
--- SIGCHLD (Child exited) @ 0 (0) ---
getpid()                                = 136
write(1, "pid=137\nThis is father, his coun"..., 101) = 101
exit_group(0)                           = ?

```

此时没有调用exit系统调用,


## 参考：

1. [ linux子进程退出状态值解析：waitpid() status意义解析](http://blog.csdn.net/eqiang8271/article/details/8225468)
