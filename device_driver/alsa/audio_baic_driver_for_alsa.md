# 设计

## 初始化：

1.硬件通路的建立
a)上电
b)时钟使能
c)初始化Codec
2.控制器内部逻辑通路的建立
a)选择DMA、FORMAT和FIFO通道，原则一致性，方便操作

使各个模块达到工作状态
原则：将数据传输中不需要修改在驱动加载时的初始化中做

## 数据传输：

1.相关属性的配置
2.DMA的配置
3.数据的循环搬运





## 参考:

1. [Advanced_Linux_Sound_Architecture](https://en.wikipedia.org/wiki/Advanced_Linux_Sound_Architecture)
