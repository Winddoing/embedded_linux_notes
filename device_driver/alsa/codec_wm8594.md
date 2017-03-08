# ALSA codec 驱动


## 数据类型

``` C
struct snd_kcontrol_new {
    snd_ctl_elem_iface_t iface; /* interface identifier */
    unsigned int device;        /* device/client number */
    unsigned int subdevice;     /* subdevice (substream) number */
    const unsigned char *name;  /* ASCII name of item */
    unsigned int index;     /* index of item */
    unsigned int access;        /* access rights */
    unsigned int count;     /* count of same elements */
    snd_kcontrol_info_t *info;
    snd_kcontrol_get_t *get;
    snd_kcontrol_put_t *put;
    union {
        snd_kcontrol_tlv_rw_t *c;
        const unsigned int *p;
    } tlv;
    unsigned long private_value;                                                                             
};
```
用`snd_kcontrol_new`结构体了codec可供控制的部分，包括：通道切换（switch/mixer），音量调整（volume）

* `iface` 字段定义了control的类型，形式为SNDRV_CTL_ELEM_IFACE_XXX，对于mixer是`SNDRV_CTL_ELEM_IFACE_MIXER`，对于不属于mixer的全局控制，使用CARD；如果关联到某类设备，则是PCM、RAWMIDI、TIMER或SEQUENCER。在这里，我们主要关注mixer

* `name` 字段是名称标识，这个字段非常重要，因为control的作用由名称来区分，对于名称相同的control，则使用index区分。
下面会详细介绍上层应用如何根据name名称标识来找到底层相应的control。
name定义的标准是`“SOURCE DIRECTION FUNCTION”`即“源 方向 功能”
`SOURCE` 定义了control的源，如“Master”、“PCM”等；
`DIRECTION` 则为“Playback”、“Capture”等，如果DIRECTION忽略，意味着Playback和capture双向；
`FUNCTION` 则可以是“Switch”、“Volume”和“Route”等。

* `access` 字段是访问控制权限。SNDRV_CTL_ELEM_ACCESS_READ意味着只读，这时put()函数不必实现；SNDRV_CTL_ELEM_ACCESS_WRITE意味着只写，这时get()函数不必实现。若control值频繁变化，则需定义 VOLATILE标志。当control处于非激活状态时，应设置INACTIVE标志。

* `private_value` 字段包含1个长整型值，可以通过它给info()、get()和put()函数传递参数。

## 驱动初始化加载

snd_soc_register_codec()   //注册一个codec
	|->snd_soc_codec_driver结构体包含snd_kcontrol_new


## 常用接口


###  SOC_DOUBLE_R_TLV

``` C
#define SOC_DOUBLE_R_TLV(xname, reg_left, reg_right, xshift, xmax, xinvert, tlv_array) \         
{   .iface = SNDRV_CTL_ELEM_IFACE_MIXER, .name = (xname),\
    .access = SNDRV_CTL_ELEM_ACCESS_TLV_READ |\
         SNDRV_CTL_ELEM_ACCESS_READWRITE,\
    .tlv.p = (tlv_array), \
    .info = snd_soc_info_volsw, \
    .get = snd_soc_get_volsw, .put = snd_soc_put_volsw, \
    .private_value = SOC_DOUBLE_R_VALUE(reg_left, reg_right, xshift, \
                        xmax, xinvert) }
```

作用:  对两个寄存器reg_left和reg_right进行同一操作，Codec芯片中左右声道的寄存器配置一般来说是差不多的，这就是这个宏存在的意义。

###  SOC_SINGLE

``` C
#define SOC_SINGLE(xname, reg, shift, max, invert) \                                         
{   .iface = SNDRV_CTL_ELEM_IFACE_MIXER, .name = xname, \
    .info = snd_soc_info_volsw, .get = snd_soc_get_volsw,\
    .put = snd_soc_put_volsw, \
    .private_value =  SOC_SINGLE_VALUE(reg, shift, max, invert) }
```
作用: 寄存器reg的位偏移shift可以设置0-max的数值

SOC_SINGLE定义最简单的控件，这种控件只有一个控制量，比如一个开关，或者是数值的变化(比如codec中的某个频率，FIFO大小等) 
参数：xname(该控件的名字)，reg(该控件对应的寄存器的地址)，shift(控制位在寄存器中的位移)，max(控件可设置的最大值)，invert(设定值是否取反)

### SOC_SINGLE_VALUE


## 应用

上层也可以根据numid来找到对应的control，snd_ctl_find_id()也是优先判断上层是否传递了numid，是则直接返回这个numid对应的control。用户层设置numid和control的关联时，可用alsa-lib的snd_mixer_selem_set_enum_item()函数。snd_kcontrol_new结构体并没有numid这个成员，是因为numid是系统自动管理的，原则是该control的注册次序，保存到snd_ctl_elem_value结构体中。


## 参考

1. [ Asoc dapm(一) - kcontrol](http://blog.csdn.net/luckywang1103/article/details/49095655)

