## 启动时间



```
setenv bootargs 'console=ttyS2,115200 mem=256M@0x0 mem=256M@0x30000000 rootfstype=ext4 root=/d
ev/mmcblk0p7 rootdelay=3 rw initcall_debug printk.time=1'

dmesg -s 131072 > ktime
cat ktime | perl scripts/bootgraph.pl > boot.svg
```

1. [嵌入式 Linux 启动时间优化](http://tinylab.org/elinux-org-boot-time-optimization/)
2. [Linux加速启动，启动时间的极限优化](https://blog.csdn.net/reille/article/details/5694155)
3. [怎样在 1 秒内启动 Linux](http://www.codeceo.com/start-linux-1-second.html)
4. [linux启动](http://www.eetop.cn/blog/index.php?uid/1539194/action/viewspace/itemid/403701/php/1)
5. [ Linux Bootup Time](https://blog.csdn.net/younger_china/article/details/51616478)
