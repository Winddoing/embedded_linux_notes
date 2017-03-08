# I2C

## I2C设备的使用

### client(相当于device端)

``` C
module_i2c_driver(wm8594_i2c_driver); 
```
相当于
``` C
i2c_add_driver(&wm8523_i2c_driver); 

i2c_del_driver(&wm8523_i2c_driver);
```

### i2c_board_info

``` C
struct i2c_board_info jz_i2c2_devs[] __initdata = {
    {
        I2C_BOARD_INFO("ecdc-wm8594", 0x1a),
    },
};
```
``` C
i2c_register_board_info(2, jz_i2c2_devs, ARRAY_SIZE(jz_i2c2_devs));
```

##  设备匹配I2C控制器

### i2c_register_board_info

``` C
int __init
i2c_register_board_info(int busnum,                                                         
    struct i2c_board_info const *info, unsigned len)
{
	...
    for (status = 0; len; len--, info++) {
        struct i2c_devinfo  *devinfo;
		...
        devinfo->busnum = busnum;
        devinfo->board_info = *info;
        list_add_tail(&devinfo->list, &__i2c_board_list);
    }
}
```

将板级信息注册到I2C bus,其实就是将板级的`i2c_decvinfo`添加到`__i2c_board_list`链表

#### struct i2c_devinfo

### 设备注册到I2C BUS

接口:
``` C
 /* use a define to avoid include chaining to get THIS_MODULE */
 #define i2c_add_driver(driver) \
     i2c_register_driver(THIS_MODULE, driver)                                     
```
``` C
/* 
 * An i2c_driver is used with one or more i2c_client (device) nodes to access
 * i2c slave chips, on a bus instance associated with some i2c_adapter.
 */

int i2c_register_driver(struct module *owner, struct i2c_driver *driver)
```


## 参考

1. [ i2c子系统之 adapter driver注册——i2c_dev_init()](http://blog.csdn.net/rockrockwu/article/details/7460407)


