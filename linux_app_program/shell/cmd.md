# 命令

## tee

## xargs

```
 find . -type f -name "*" | xargs grep "root/init.sh"
```
> -type f 表示只找文件
> -name “xxx” 表示查找特定文件；也可以不写，表示找所有文件

## ldd

``` shell
$ldd adb 
	linux-gate.so.1 =>  (0xb771c000)
	librt.so.1 => /lib/i386-linux-gnu/librt.so.1 (0xb75ba000)
	libdl.so.2 => /lib/i386-linux-gnu/libdl.so.2 (0xb75b5000)
	libpthread.so.0 => /lib/i386-linux-gnu/libpthread.so.0 (0xb7598000)
	libstdc++.so.6 => /usr/lib/i386-linux-gnu/libstdc++.so.6 (0xb74af000)
	libm.so.6 => /lib/i386-linux-gnu/libm.so.6 (0xb7469000)
	libgcc_s.so.1 => /lib/i386-linux-gnu/libgcc_s.so.1 (0xb744c000)
	libc.so.6 => /lib/i386-linux-gnu/libc.so.6 (0xb729d000)
	/lib/ld-linux.so.2 (0xb771d000)
```

## tail

* 查看实时日志

```
tail -f  audio_test_20170814-14.log
```
> -f 循环读取

