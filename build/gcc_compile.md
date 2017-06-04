# 编译、汇编、链接

## 编译

### 预处理（Pre-processing）
将所有的头文件和宏定义展开
``` shell
cpp
```
> gcc参数： `-E`

### 汇编阶段（Compiling）
将代码转成相应的汇编代码
``` shell
cc1
```
> gcc参数： `-S`

### 汇编阶段(Assembling)
将汇编代码编译成二进制目标文件
``` shell
as
```
> gcc参数： `-c`

### 链接阶段(Link)
将所有的目标文件链接成可执行文件
``` shell
ld
```
> gcc参数： `-o`

## 链接
