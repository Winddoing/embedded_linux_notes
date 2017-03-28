# mmc测试



## 方法:

### 配置驱动

```
    ->DeviceDriver
        -> MMC/SD/SDIO card support (MMC [=y])
        [*]MMC host test driver
```


源码文件:drivers/mmc/card/mmc_test.c

### 绑定

* 进入mmcblk解除绑定

```
cd sys/bus/mmc/drivers/mmcblk
echo mmc0:e624 > unbind
```
* 绑定测试

```
cd sys/bus/mmc/drivers/mmc_test
echo mmc0:e624 > bind
[  17.243808] mmc_test mmc0:e624: Card claimed for testing.
```

### 挂载debugfs

```
mount -t debugfs none /debug/
```
### 测试

* 进入测试目录mmc0:e624

```
cd /debug/mmc0/mmc0:e624
```
* 查看测试列表

``` shell
#cat testlist
1:      Basic write (no data verification)
2:      Basic read (no data verification)
3:      Basic write (with data verification)
4:      Basic read (with data verification)
5:      Multi-block write
6:      Multi-block read
7:      Power of two block writes
8:      Power of two block reads
9:      Weird sized block writes
10:     Weird sized block reads
11:     Badly aligned write
12:     Badly aligned read
13:     Badly aligned multi-block write
14:     Badly aligned multi-block read
15:     Correct xfer_size at write (start failure)
16:     Correct xfer_size at read (start failure)
17:     Correct xfer_size at write (midway failure)
18:     Correct xfer_size at read (midway failure)
19:     Highmem write
20:     Highmem read
21:     Multi-block highmem write
22:     Multi-block highmem read
23:     Best-case read performance
24:     Best-case write performance
25:     Best-case read performance into scattered pages
26:     Best-case write performance from scattered pages
27:     Single read performance by transfer size
28:     Single write performance by transfer size
29:     Single trim performance by transfer size
30:     Consecutive read performance by transfer size
31:     Consecutive write performance by transfer size
32:     Consecutive trim performance by transfer size
33:     Random read performance by transfer size
34:     Random write performance by transfer size
35:     Large sequential read into scattered pages
36:     Large sequential write from scattered pages
37:     Write performance with blocking req 4k to 4MB
38:     Write performance with non-blocking req 4k to 4MB
39:     Read performance with blocking req 4k to 4MB
40:     Read performance with non-blocking req 4k to 4MB
41:     Write performance blocking req 1 to 512 sg elems
42:     Write performance non-blocking req 1 to 512 sg elems
43:     Read performance blocking req 1 to 512 sg elems
44:     Read performance non-blocking req 1 to 512 sg elems
45:     eMMC hardware reset
```


* 执行测试

```
echo 34 > test
```
