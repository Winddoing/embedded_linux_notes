# Android系统板级相关配置


## WITH_DEXPREOPT

使能system image中的所有东西都被提前优化（pre-optimized）。这可能导致system image非常大

>WITH_DEXPREOPT := true

## BOARD_SYSTEMIMAGE_PARTITION_SIZE

设置文件系统system.img的大小

>BOARD_SYSTEMIMAGE_PARTITION_SIZE := 419430400 #400MB

## PRODUCT_PROPERTY_OVERRIDES

添加相关配置属性

>PRODUCT_PROPERTY_OVERRIDES += \
>    ro.kernel.qemu=1 \
>    ro.kernel.qemu.gles=0

编译后的最终显示结果:
文件: out/target/product/xxxx/system/build.prop
``` 
66 ro.kernel.qemu=1
67 ro.kernel.qemu.gles=0


```
