# 常用工具

## 修改只读的system分区

remount system

``` shell
mount -o rw,remount -t ext4 /dev/block/platform/driver_name/by-name/system /system
```


