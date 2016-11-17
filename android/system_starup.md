# Android系统启动




## NFS启动Android


设置bootargs和bootcmd参数：

set bootcmd tftpboot c0008000 192.168.1.14:kernel.img\; bootm c0008000

set bootargs root=/dev/nfs rwnfsroot=192.168.1.14:/nfsboot/root ip=192.168.1.20:192.168.1.14:192.168.1.1  console=ttySAC2,115200




## 参考

1. 
