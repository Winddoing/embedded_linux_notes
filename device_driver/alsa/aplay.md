# 用户层测试

aplay,tinyplay


## 录(arecord)

> arecord -Dhw:0,1 -r 8000 -f cd a.wav

参数含义:
    -Dhw:0:1   //表示设备节点`pcmC0D1c`
    -r sample  //采样率
    -f format  //格式
 
### 常用参数:
 
| 参数  | 含义 |
| :--: | :--: |
| -l, --list-devices | list all soundcards and digital audio devices  |
| -L, --list-pcms    | list device names |
| -D, --device=NAME  | select PCM by name| 
| -t, --file-type TYPE  |  file type (voc, wav, raw or au)  |
| -c, --channels=#   |     channels |
| -f, --format=FORMAT|     sample format (case insensitive) |
| -r, --rate=#       |     sample rate |
| -v, --verbose      |     show PCM structure and setup (accumulative) |
| -d, --duration=#   |     interrupt after # seconds |

> 详细: arecord -h

### 采样数据格式

>Recognized sample formats are: S8 U8 S16_LE S16_BE U16_LE U16_BE S24_LE S24_BE U24_LE U24_BE S32_LE S32_BE U32_LE U32_BE FLOAT_LE FLOAT_BE FLOAT64_LE FLOAT64_BE IEC958_SUBFRAME_LE IEC958_SUBFRAME_BE MU_LAW A_LAW IMA_ADPCM MPEG GSM 
SPECIAL S24_3LE S24_3BE U24_3LE U24_3BE S20_3LE S20_3BE U20_3LE U20_3BE S18_3LE S18_3BE U18_3LE

| 符号 | 说明 |
| :--: | :-: |
| S | 有符号(Signed)  |
| U | 无符号(Unsigned)   |
| LE |  小端(Little Endian)  |
| BE |  大端(Big Endian) |
| FLOAT_BE | Float 32 bit Big Endian |
| IEC958_SUBFRAME_LE | IEC-958 Little Endian |
| S18_3BE| Signed 18 bit Big Endian in 3bytes |

## 放(aplay)

> aplay -Dhw:0,0 music/caiqin_short_48khz_16bit_2.wav



## 参考

1. [SPDIF S24_LE S24_3LE调试小结](http://blog.csdn.net/u012769691/article/details/46728233)
2. [S8 U8 S16_LE S16_BE U16_LE U16_BE格式](http://blog.csdn.net/qingkongyeyue/article/details/52829886)


