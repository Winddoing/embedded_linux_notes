# 字符串


## 截取

``` shell
a="/dev/mmcnlk0p1"
echo "a=$a"

a="/dev/mmcnlk0p1"
a=${a#*/}
echo "a=$a"

a="/dev/mmcnlk0p1"
a=${a##*/}
echo "a=$a"

a="/dev/mmcnlk0p1"
a=${a%/*}
echo "a=$a"

a="/dev/mmcnlk0p1"
a=${a%%/*}
echo "a=$a"

a="/dev/mmcnlk0p1"
a=${a:0:7}
echo "a=$a"

a="/dev/mmcnlk0p1"
a=${a:0-7}
echo "a=$a"

a="/dev/mmcnlk0p1"
a=${a:0-7:3}
echo "a=$a"
```
结果:

``` shell
a=/dev/mmcnlk0p1
a=dev/mmcnlk0p1
a=mmcnlk0p1
a=/dev
a=
a=/dev/mm
a=cnlk0p1
a=cnl
```
