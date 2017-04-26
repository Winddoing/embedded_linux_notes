# buffer的数据处理

控制器和buffer之间的数据关系


## 操作场景一

简单粗暴的播放测试: 

1. 用户空间申请一定大小的buffer,将音源全部读入
2. 将用户空间buffer,拷贝到内核空间
3. 使用DMA描述符对该buffer进行描述

DMA描述符对其所描述buffer大小的要求:
假设: 
* DMA描述符个数 4
* DMA数据传输brust 16word(64Byte)
* 音源的采样位宽n(16, 24, 32)

buffer的大小必须是n/8, 64, 4三个数的最小公倍数的倍数.


## alsa中buffer的大小


在播放当前歌曲时,所申请的buffer大小为16KB,为什么申请16K?

音频信息:

| 采样率 | 通道 | 位宽(format) |
|:----: |:----:|:-----------:|
| 44100Hz | 2  | 16bit		|


>4KB的buffer大小为`tinyplay`默认大小,`period_size = 1024`, `period_count = 4`决定了buffer大小,而`period_size`可以进行修改默认大小.

需要申请buffer的大小: 1024 * 4 * 2 * (16 / 8) = 16384


>!!!为什么buffer的默认基础大小是4KB


