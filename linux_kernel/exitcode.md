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

## 系统exit退出码

``` C

```
