# 电源管理-REGULATOR

>以下接口可根据Asoc系统查看使用原理

>**注**: 这些接口的使用需要内核配置`CONFIG_REGULATOR`

## 数据结构

``` C
 struct regulator_bulk_data {                  
     const char *supply;                       
     struct regulator *consumer;               
                                               
     /* private: Internal use */               
     int ret;                                  
 };                                            
```

## API

### devm_regulator_bulk_get

### regulator_bulk_enable

### regulator_bulk_disable


