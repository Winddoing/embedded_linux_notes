# Codec init


machine将codec绑定后,需要对其进行初始化


## 检测绑定

检测将在注册过程中完成
``` C
snd_soc_register_card
    |
    |-> snd_soc_instantiate_card
        |
        |-> for->soc_probe_link_components
            |<order = 0>
            |-> soc_probe_codec
            |-> soc_probe_platform
```

## Codec初始化

```
soc_probe_codec
    |
    |-> driver->probe(codec)
```



## regmap和snd_soc_write关系


