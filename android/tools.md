# 常用工具

## 修改只读的system分区

remount system

``` shell
mount -o rw,remount -t ext4 /dev/block/platform/driver_name/by-name/system /system
```

## 库文件的dump信息定位

命令: addr2line

使用:
``` shell
=====>$mips-linux-gnu-addr2line -e out/target/product/xxxxx/symbols/system/lib/libdvm.so 23452
/work/android-4.3-fpga/dalvik/vm/mterp/out/InterpAsm-mips.S:1335
```
>23452 --> 异常PC



