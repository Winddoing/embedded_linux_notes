# 环形buffer


>软件与硬件的无锁同步


```
数据方向: --->
|-start     |<-(w)soft pointer
| U | U | U | ... | U |
|-end             |<-(r)hardware pointer
```

对于数据的读写均以buffer中的每个小buffer为数据单元U

## Buffer满
```
数据方向: --->
    |<-(w)soft pointer
| U | U | U | ... | U |
        |<-(r)hardware pointer
```

>r - w = 1U  表示环形buffer以及写满

## Buffer空
```
数据方向: --->
        |<-(w)soft pointer
| U | U | U | ... | U |
        |<-(r)hardware pointer
```

> r - w = 0 表示环形buffer为空


## 获取当前可用buffer的数据单元

根据剩余空间的大小减去不足一个单元的buffer大小

> (r - w) - (r - w)%Unit







