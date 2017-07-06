# amixer

帮助:
```
# amixer --help
Usage: amixer <options> [command]

Available options:
  -h,--help       this help
  -c,--card N     select the card
  -D,--device N   select the device, default 'default'
  -d,--debug      debug mode
  -n,--nocheck    do not perform range checking
  -v,--version    print version of this program
  -q,--quiet      be quiet
  -i,--inactive   show also inactive controls
  -a,--abstract L select abstraction level (none or basic)
  -s,--stdin      Read and execute commands from stdin sequentially
  -R,--raw-volume Use the raw value (default)
  -M,--mapped-volume Use the mapped volume

Available commands:
  scontrols       show all mixer simple controls
  scontents       show contents of all mixer simple controls (default command)
  sset sID P      set contents for one mixer simple control
  sget sID        get contents for one mixer simple control
  controls        show all controls for given card
  contents        show contents of all controls for given card
  cset cID P      set control contents for one control
  cget cID        get control contents for one control
```
## 操作接口

### amixer controls

> 查看当前的音频驱动提供哪些可操作接口

### amixer contents

> 查看当前所有控件的配置属性(相关参数的值)

### amixer cget

> 查看单个控件的属性,相关参数的范围和定义

```
# amixer cget numid=1,iface=MIXER,name='MERCURY Playback Volume'
numid=1,iface=MIXER,name='MERCURY Playback Volume'
  ; type=INTEGER,access=rw---R--,values=2,min=0,max=63,step=0
  : values=31,31
  | dBscale-min=-31.00dB,step=1.00dB,mute=0
```

### amixer cset

> 修改控件的属性,比如设置音量

```
amixer cset numid=1,iface=MIXER,name='MERCURY Playback Volume' 25
amixer cset numid=1 25
amixer cset name='MERCURY Playback Volume' 25
```


## 简易操作接口

对简单的控件的另一组操作接口,与上面类似.

### amixer scontrols

### amixer sget

> amixer sget 'MERCURY AIDAC MIXER Mux'

### amixer sset

> amixer sget 'MERCURY AIDAC MIXER Mux' Mixed Inputs



## 实现

重新配置空间属性

```
amixer
    |-> snd_ctl_ioctl (SNDRV_CTL_IOCTL_ELEM_WRITE)
        |-> snd_ctl_elem_write_user
            |-> snd_ctl_elem_write
                |-> snd_ctl_find_id(card, &control->id) //遍历kcontrol链表匹配name字段的kctl
                |-> kctl->put(kctl, control)    //回调kctl的成员函数put()        
```




