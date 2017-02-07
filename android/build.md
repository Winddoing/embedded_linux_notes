# 编译

## source

## lunch

## make


## 制作Android rootfs

复制out\target\product\imx51_bbg\root\*到\nfs\rootfs

``` shell
cp -dar out/target/product/imx51_bbg/root/* /nfs/rootfs
```
复制out/target/product/imx51_bbg/system目录到/nfs/rootfs/

``` shell
cp -dar out/target/product/imx51_bbg/system /nfs/rootfs/
```
更改init.rc脚本,使android可以在nfs上跑.
注释掉所有的mount,因为我们不需要从SD卡或者Nand启动.
``` shell
#on fs
#mount ext4 partitions
#Mount /system rw first to give the filesystem a chance to save acheckpoint
#mount ext4 /dev/block/mmcblk0p2 /system
#mount ext4 /dev/block/mmcblk0p2 /system ro remount
#mount ext4 /dev/block/mmcblk0p5 /data nosuid nodev
#mount ext4 /dev/block/mmcblk0p6 /cache nosuid nodev

#on post-fs
#once everything is setup, no need to modify /
#mount rootfs rootfs / ro remount
```
**data分区需要进行挂载,否则系统无法正常启动,data分区的挂载方式有:loop, vram, sd卡**

## 挂载data

### loop

**以下修改是为量mount data分区,该方法是通过loop挂载实现的,可以正常挂载并启动,但是由于网络环境不是很良好,可能会导致系统启动时,进行android虚拟机所需的相关核心包,无法进行解压,可能是android系统判断相关jar包,已经解压完成,而直接启动虚拟机所致.**

1. 注释掉mount文件系统的相关操作
```
on fs
    #mount ext4 /dev/block/mmcblk2p1 /system
    #write /proc/exec /system/bin/sh:/system/etc/disk_preparing.sh
    # create /data and /cache filesystem if necessary
    #mount -t ext4 -o loop /system.img /system
    chmod 0755 /system/bin/
    class_start test_data_class
    # wait /dev/disk_preparing_done 100
    mount_all /fstab.jz4785_fpga
    setprop ro.crypto.fuse_sdcard true

    service disk_prepare /system/bin/disk_preparing.sh
    class test_data_class
    oneshot
```

2. disk_preparing.sh

``` shell
#!/system/bin/sh
/system/bin/sh /system/bin/mount -o loop -t ext4 userdata.img /data

```

3. loop节点

```
 --- Block devices                                             
 <*>   Loopback device support                                 
 (8)     Number of loop devices to pre-create at init time     

```

### vram

申请一块大的内存空间并制作为一个block设备节点,在将该节点进行格式化和mount操作.

如果系统启动直接进行data分区的mount需要fs_mgr进行自动的格式化,格式化流程如下:


1. libEGL  -  SurfaceFlinger




## 



## 参考

1. [android开发](http://www.kancloud.cn/digest/imx6-android/148862)
