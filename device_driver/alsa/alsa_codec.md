# CODEC

Codec(解码器)
各解码器驱动必须提供如下特性：
1) Codec DAI and PCM configuration
2) Codec control IO - using I2C, 3 Wire(SPI) or both APIs
3) Mixers and audio controls
4) Codec audio operations
1）解码器数字音频接口和PCM配置。
2）解码器控制IO－使用I2C，3总线（SPI）或两个都有。
3）混音器和音频控制。
4）解码器音频操作。


Optionally, codec drivers can also provide:-
解码器驱动可以选择性提供：

5) DAPM description.
6) DAPM event handler.
7) DAC Digital mute control.
5）动态音频电源管理描述。
6）动态音频电源管理事件控制。
7）数模转换数字消音控制。
SoC DAI Drivers
板级DAI驱动
===============

Each SoC DAI driver must provide the following features:-
每个SoC DAI驱动都必须提供如下性能：

1) Digital audio interface (DAI) description
1)数字音频接口描述
2) Digital audio interface configuration
2)数字音频接口配置
3) PCM's description
3）PCM描述
4) SYSCLK configuration
4）系统时钟配置
5) Suspend and resume (optional)
5）挂起和恢复（可选的）


## Codec简介
在移动设备中，Codec的作用可以归结为4种，分别是：
对PCM等信号进行D/A转换，把数字的音频信号转换为模拟信号
对Mic、Linein或者其他输入源的模拟信号进行A/D转换，把模拟的声音信号转变CPU能够处理的数字信号
对音频通路进行控制，比如播放音乐，收听调频收音机，又或者接听电话时，音频信号在codec内的流通路线是不一样的
对音频信号做出相应的处理，例如音量控制，功率放大，EQ控制等等
