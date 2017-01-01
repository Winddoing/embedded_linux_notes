# 开发环境

## 目标

* 简单，高效，快速
* 配置/编译 Linux Kernel
* 配置/编译 文件系统

## 开发环境

* 架构： MIPS

## 备份

只为后续使用可以更便捷

https://github.com/Winddoing/linux-lab-wdg.git

## 搭建过程

首先把 Linux Lab 下载下来：

``` shell
$ git clone https://github.com/tinyclub/cloud-lab.git
$ cd cloud-lab
$ tools/docker/choose linux-lab //下载
```

## 使用

进入linux-labs，参考README

### 查看列表
``` shell
$ make list                   # 查看支持的列表
```
### 下载

需要下载linux uboot qemu 源码

Makefile中可修改下载路径

``` shell
$ make core-source -j4
```
### 选择板级

``` shell
$ make BOARD=malta             # 这里选择一块 MIPS 板子：malta
```

### 启动

``` shell
$ make boot
```


## 遇见错误

### Cannot connect to the Docker daemon. Is the docker daemon running on this host?

权限不够，使用root权限

## 参考

* http://tinylab.org/linux-lab/
