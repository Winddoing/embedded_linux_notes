# ext4文件系统


## ext4

EXT4(Fourth extended filesystem，缩写为 ext4)已经被许多发行版Linux作为默认的文件系统。EXT4文件系统在一定程度上向下兼容EXT2、EXT3，但是与原有的文件系统已经有很大的区别。

## 可扩展性

为了支持更大的文件系统，ext4 决定采用 48 位的块号取代 ext3 原来的 32 位块号，并采用 extent 映射来取代 ext3 所采用的间接数据块映射的方法。这样既可以增大文件系统的容量，又可以改进大文件的访问效率。在使用 4KB 大小的数据块时，ext4 可以支持最大 2^48 * 2^12 = 2^60 B（1 EB）的文件系统。之所以采用 48 位的块号而不是直接将其扩展到 64 位是因为，ext4 的开发者认为 1 EB 实际上（按照目前的速度，要对，大小的文件系统对未来很多年都足够了 1 EB 大小的文件系统执行一次完整的 fsck 检查，大约需要 119 年的时间），与其耗费心机去完全支持 64 位的文件系统，还不如先花些精力来解决更加棘手的可靠性问题。

## ext1, ext2, ext3, ext4区别

### 文件系统容量

1. ext1支持最大文件系统2GB
2. ext2支持最大文件系统32T
3. ext4支持最大文件系统1024PiB

### 日志

相对而言ext3以后,包括ext3和ext4的文件系统中增加量日志系统.

ext4文件系统为日志数据添加量检查和(checksum)功能,来提高可靠性和性能.

## 配置只读文件系统

在uboot使用`CONFIG_BOOTARGS`进行内核传入参数设置.

``` C
#define CONFIG_BOOTARGS BOOTARGS_COMMON " rootfstype=ext4 root=/dev/mmcblk0p7 rootdelay=3 ro"
```
>参考[uboot传入的启动参数](/linux_kernel/uboot_bootargs_set.md)

## 参考

[ext4文件介绍](!http://blog.csdn.net/robinlovesnow/article/details/7567037)
