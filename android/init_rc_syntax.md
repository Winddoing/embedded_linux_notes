# init.rc语法

## 关键字

1. Section: 语句块，相当于C语言中大括号内的一个块。一个Section以Service或On开头的语句块.以Service开头的Section叫做服务,而以On开头的叫做动作(Action).
2. services: 服务.
3. Action: 动作
4. commands:命令.
5. options:选项.
6. trigger:触发器，或者叫做触发条件.
7. class: 类属，即可以为多个service指定一个相同的类属，方便操作同时启动或停止

## 动作(Action)

动作表示了一组`命令`(commands)组成.动作包含一个`触发器`，决定了何时执行这个动作。*当触发器的条件满足时，这个动作会被加入到已被执行的队列尾*。如果此动作在队列中已经存在，那么它将不会执行.一个动作所包含的命令将被依次执行。

动作的语法如下所示:

```
on <trigger>
   <command>
   <command>
```

## 服务(service)

服务是指那些需要在系统初始化时就启动或退出时自动重启的程序.
语法结构如下所示:

```
service <name> <pathname> [ <argument> ]*
   <option>
   <option>
   ...
```

## 选项(option)

选项是用来修改服务的。它们影响如何及何时运行这个服务.


| 选项 | 描述 |
| :--: | :--: |
| critical | 表明这是一个非常重要的服务。如果该服务4分钟内退出大于4次，系统将会重启并进入 Recovery （恢复）模式。|
| disabled | 服务不会自动运行，必须显式地通过服务器来启动。|
| socket <name><type> <perm> [ <user> [ <group> ] ] | 创建一个unix域的名为/dev/socket/<name> 的套接字，并传递它的文件描述符给已启动的进程。<type> 必须是 "dgram","stream" 或"seqpacket"。用户和组默认是0。|
| user <username> | 在启动这个服务前改变该服务的用户名。此时默认为 root |
| group <groupname> [<groupname> ]* | 在启动这个服务前改变该服务的组名。除了（必需的）第一个组名，附加的组名通常被用于设置进程的补充组（通过setgroups函数），默认是root。|
| oneshot | 服务退出时不重启 |
| onrestart | 当服务重启，执行一个命令, 如:onrestart restart zygote |
| class <name> | 指定一个服务类。所有同一类的服务可以同时启动和停止。如果不通过class选项指定一个类，则默认为"default"类服务 |
| ioprio | IoSchedClass, usage: ioprio <rt|be|idle> <ioprio 0-7> |
| seclabel | 在执行服务之前改变安全级别 |


## 触发器(trigger)

触发器用来描述一个触发条件，当这个触发条件满足时可以执行动作.

| 触发器 | 描述 |
| :--: | :--: |
| boot | 当init程序执行，并载入/init.conf文件时触发. |
| <name>=<value> | 当属性名对应的值设置为指定值时触发 |


## 命令(command)

| 命令 | 描述 |
| :--: | :--: |
| ifup <interface> | 使指定的网络接口"上线",相当激活指定的网络接口 |
| hostname <name> | 设置主机名 |
| domainname <name> | 设置域名 |
| setrlimit <resource> <cur> <max> | 设置资源的rlimit(资源限制) |
| setprop <name> <value> | 设置属性及对应的值. |
| class_start <serviceclass> | 启动指定类属的所有服务，如果服务已经启动，则不再重复启动 |
| class_stop <serviceclass> | 停止指定类属的所胡服务 |
| start <service> | 如果指定的服务未启动，则启动它.|
| stop <service> | 如果指定的服务当前正在运行，则停止它. |


## 参考

1. [Android的init过程（二）：初始化语言（init.rc）解析](http://www.cnblogs.com/nokiaguy/p/3164799.html)
2. [Android init.rc文件浅析](http://blog.csdn.net/flydream0/article/details/7458332)

