# 编译

## source

## lunch

## make


制作Androidrootfs
复制out\target\product\imx51_bbg\root\*到\nfs\rootfs
cp -dar out/target/product/imx51_bbg/root/* /nfs/rootfs
复制out/target/product/imx51_bbg/system目录到/nfs/rootfs/
cp -dar out/target/product/imx51_bbg/system /nfs/rootfs/
更改init.rc脚本,使android可以在nfs上跑.
注释掉所有的mount,因为我们不需要从SD卡或者Nand启动.
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


## error

1. libEGL  -  SurfaceFlinger




## 



## 参考

1. [android开发](http://www.kancloud.cn/digest/imx6-android/148862)
