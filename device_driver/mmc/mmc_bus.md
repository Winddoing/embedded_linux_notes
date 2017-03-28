# MMC bus


## 注册mmc bus

``` C
static int __init mmc_init(void)
{
    int ret;

    workqueue = alloc_ordered_workqueue("kmmcd", 0);
    if (!workqueue)
        return -ENOMEM;

    ret = mmc_register_bus();     //MMC BUS
    if (ret)
        goto destroy_workqueue;

    ret = mmc_register_host_class();
    if (ret)
        goto unregister_bus;

    ret = sdio_register_bus();
    if (ret)
        goto unregister_host_class;

    return 0;
}
```
>file: drivers/mmc/core/core.c

``` C
static struct bus_type mmc_bus_type = {
    .name       = "mmc",
    .dev_attrs  = mmc_dev_attrs,
    .match      = mmc_bus_match,
    .uevent     = mmc_bus_uevent,
    .probe      = mmc_bus_probe,
    .remove     = mmc_bus_remove,
    .pm     = &mmc_bus_pm_ops,
};

int mmc_register_bus(void)
{
    return bus_register(&mmc_bus_type); //设备模型接口
}
```

## 注册驱动mmc_driver

``` C
static int __init mmc_blk_init(void)
{

    res = register_blkdev(MMC_BLOCK_MAJOR, "mmc");
    if (res)
        goto out;

    res = mmc_register_driver(&mmc_driver);  //mmc_driver
    if (res)
        goto out2;

    return 0;
}
```
>file: drivers/mmc/card/block.c

``` C
int mmc_register_driver(struct mmc_driver *drv)
{
    drv->drv.bus = &mmc_bus_type;
    return driver_register(&drv->drv);  //设备模型接口
}
```

## 注册设备mmc_card

``` C
int mmc_add_card(struct mmc_card *card)
{
    ...
    ret = device_add(&card->dev); //设备模型接口
    ...
}
```
>file: drivers/mmc/core/bus.c

## bind和unbind

``` C
cd sys/bus/mmc/drivers/mmcblk
echo mmc0:e624 > unbind

cd sys/bus/mmc/drivers/mmc_test
echo mmc0:e624 > bind
```



## 向卡中写入数据时,怎么判断卡中空间是否可写



