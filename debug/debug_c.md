# 调试


## GDB

core dump
### 查看core设置

ulimit -a


### 开启core file

limit -c unlimited

### 使用

* 异常程序(段错误)
``` C
int main(int argc, char *argv[])
{
    int *a;

    *a = 1;                                                         

    while(1)
    {

    }
    return EXIT_SUCCESS;
}

```

* 运行异常程序后,生成core文件

* 使用gdb查看异常位置

gdb ./a.out core


