# TDM


## TDM1 (A/B) 

> A --- aligned(对齐)
> B --- unaligned (LF)(不对齐)

通道数可调


## TDM2 (A/B)

八个通道






























## 位宽转换

音频文件24bit使用仅支持32bit的codec进行播放

此时,BAIC需要将24bit的数据转化为32bit的数据

```
原数据	:           xxxx xxxx xxxx xxxx xxxx xxxx				#24bit的数据单元
方式1	: 0000 0000 xxxx xxxx xxxx xxxx xxxx xxxx 				#进行高8位补0操作	
方式2	:	        xxxx xxxx xxxx xxxx xxxx xxxx 0000 0000		#进行低8位补0操作
```

### 高8位补0操作

高8位进行补0,相当于将原有音频信号的声音减小,高音部分比较明显.

### 低8位补0操作

低8位进行补0,会出现嘈音.
