# dex2oat

`dex2oat` 顾名思义 dex file to oat file，将dex字节码编译成oat格式的可执行文件的工具

dex2oat 对所有 apk 进行编译并保存在 dalvik-cache 目录里。PMS 会持续扫描安装目录，如果有新的 App 安装则马上调用 dex2oat 进行编译。

## dex

Dex是Dalvik VM executes的全称，即Android Dalvik执行程序，通常执行时都进行优化 。

## oat

OAT文件是一种Android私有ELF文件格式，它不仅包含有从DEX文件翻译而来的本地机器指令，还包含有原来的DEX文件内容。这使得我们无需重新编译原有的APK就可以让它正常地在ART里面运行，也就是我们不需要改变原来的APK编程接口。

## Art模式

ART模式英文全称为：Android runtime,谷歌`Android 4.4`系统新增的一种应用运行模式，与传统的Dalvik模式不同，ART模式可以实现更为流畅的安卓系统体验

在启用ART模式后，系统在安装应用的时候会进行一次预编译，在安装应用程序时会先将代码转换为机器语言存储在本地，这样在运行程序时就不会每次都进行一次编译了，执行效率也大大提升。

**ART是一个AOT编译器。所谓AOT (Ahead of Time)是指在运行以前就把中间代码静态编译成本地代码, 而JIT (Just inTime)则是在运行时动态编译**

## Dalvik模式

在`每次`执行应用的时候Dalvik虚拟机都会将程序的语言由高级语言编译为机器语言，这样当前设备才能够运行这一应用。


## 模式与文件

Dalvik   ----->      dex

ART      ----->      oat

## 参考

1. [Android ART运行时无缝替换Dalvik虚拟机的过程分析](http://blog.csdn.net/luoshengyang/article/details/18006645)



