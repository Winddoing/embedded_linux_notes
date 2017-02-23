# Android系统板级相关配置


## WITH_DEXPREOPT

使能system image中的所有东西都被提前优化（pre-optimized）。这可能导致system image非常大

>WITH_DEXPREOPT := true

## BOARD_SYSTEMIMAGE_PARTITION_SIZE

设置文件系统system.img的大小

>BOARD_SYSTEMIMAGE_PARTITION_SIZE := 419430400 #400MB

## PRODUCT_PROPERTY_OVERRIDES

添加相关配置属性

``` C
# Fix USE_OPENGL_RENDERER by ro.kernel.qemu && ro.kernel.qemu.gles
#
# Code location: You can check the exactly meanings in these files.
# <android_dir>/frameworks/native/opengl/libs/EGL/Loader.cpp
# <android_dir>/frameworks/base/core/jni/android_view_GLES20Canvas.cpp
#
# Usages:
#   ro.kernel.qemu = <number>      -> tells us that we run inside the emulator
#      ro.kernel.qemu = 1          #have emulator
#
#   ro.kernel.qemu.gles=<number>   -> choose one of them, tells us the GLES GPU emulation status
#      ro.kernel.qemu.gles = 0     #SurfaceFlinger && HardwareRender not use GPU
#      ro.kernel.qemu.gles = 1     #SurfaceFlinger && HardwareRender use GPU
#      ro.kernel.qemu.gles = 2     #SurfaceFlinger use GPU && HardwareRender not use GPU
#
# Examples: We can just use two methods.
#   1. App Not use OpenGL_Render
#      ro.kernel.qemu=1
#      ro.kernel.qemu.gles=2
#
#   2. App use OpenGL_Render
#      ro.kernel.qemu=1
#      ro.kernel.qemu.gles=1
```

>PRODUCT_PROPERTY_OVERRIDES += \
>    ro.kernel.qemu=1 \
>    ro.kernel.qemu.gles=0

编译后的最终显示结果:
文件: out/target/product/xxxx/system/build.prop
``` 
66 ro.kernel.qemu=1
67 ro.kernel.qemu.gles=0
```
