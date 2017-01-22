# Android系统启动

## NFS启动Android


设置bootargs和bootcmd参数：

set bootcmd tftpboot c0008000 192.168.1.14:kernel.img\; bootm c0008000

set bootargs root=/dev/nfs rwnfsroot=192.168.1.14:/nfsboot/root ip=192.168.1.20:192.168.1.14:192.168.1.1  console=ttySAC2,115200


## 系统启动流程和相关服务

### 启动过程

Android系统完整的启动过程，可分为Linux系统层、Android系统服务层、Zygote进程模型三个阶段，从开机到启动Home Launcher完成具体的任务细节可分为七个步骤。

1. 启动BootLoader
2. 加载系统内核
3. 启动Init和其它重要守护进程
4. 启动Zygote进程
5. 启动Runtime进程，初始化Service Manager。Service Manager用于binder通讯，负责绑定服务的注册与查找。
6. 启动SystemService
7. 启动Home Laucher
8. 启动其它应用程序



## 参考

1. 
