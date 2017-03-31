# regmap


## 数据结构

### 配置

``` C
struct regmap_config {
    const char *name;

    int reg_bits;			// 寄存器地址的位数，必须配置，例如I2C寄存器地址位数为 8  
    int reg_stride; 
    int pad_bits;			 
    int val_bits;           // 寄存器值的位数，必须配置
    
    bool (*writeable_reg)(struct device *dev, unsigned int reg); // 可写寄存器回调，maintain一个可写寄存器表                              
    bool (*readable_reg)(struct device *dev, unsigned int reg);  // 可读寄存器回调, maintain一个可读寄存器表
    bool (*volatile_reg)(struct device *dev, unsigned int reg);  // 可要求读写立即生效的寄存器回调，不可以被cache,maintain一个可立即生效寄存器表  
    bool (*precious_reg)(struct device *dev, unsigned int reg);  // 要求寄存器数值维持在一个数值范围才正确，maintain一个数值准确表  
    regmap_lock lock;
    regmap_unlock unlock;
    void *lock_arg;

    int (*reg_read)(void *context, unsigned int reg, unsigned int *val);
    int (*reg_write)(void *context, unsigned int reg, unsigned int val);
    
    bool fast_io;

    unsigned int max_register;		// max_register: 最大寄存器地址  
    const struct regmap_access_table *wr_table;
    const struct regmap_access_table *rd_table;
    const struct regmap_access_table *volatile_table;
    const struct regmap_access_table *precious_table;
    const struct reg_default *reg_defaults;    //配置默认寄存器的相关初始值
    unsigned int num_reg_defaults;
    enum regcache_type cache_type;              //缓存类型
    const void *reg_defaults_raw;
    unsigned int num_reg_defaults_raw;

    u8 read_flag_mask;
    u8 write_flag_mask;

    bool use_single_rw;

    enum regmap_endian reg_format_endian;
    enum regmap_endian val_format_endian;

    const struct regmap_range_cfg *ranges;
    unsigned int num_ranges;
};

```

### 寄存器

``` C
/** 
 * Default value for a register.  We use an array of structs rather
 * than a simple array as many modern devices have very sparse
 * register maps.
 *  
 * @reg: Register address.
 * @def: Register default value.
 */
struct reg_default {                                                          
    unsigned int reg;
    unsigned int def;
};  
```

## 常用接口



## 配置寄存器默认值


### 初始化配置

``` C
static const struct reg_default wm8594_reg_defaults[] = {                  
    { 0x02, 0x0102 },  //R2 DAC1_CRTL1                                     
    { 0x03, 0x001d },  //R3 DAC1_CRTL2                                     
    { 0x04, 0x0000 },  //R4 DAC1_CRTL3                                     
}
```
### 设备寄存器配置信息 -- I2C

通过`struct regmap_config`结构进行配置
``` C
 static const struct regmap_config wm8594_regmap = {                 
     .reg_bits = 8,                                                  
     .val_bits = 16,                                                 
     .max_register = 0x04, //最后一个寄存器                               
                                                                     
     .reg_defaults = wm8594_reg_defaults,                            
     .num_reg_defaults = ARRAY_SIZE(wm8594_reg_defaults),            
     .cache_type = REGCACHE_RBTREE,                                  
                                                                     
     .volatile_reg = wm8594_volatile_register,                       
 };                                                                  

``` 

## 初始化

进行初始化配置

接口: `devm_regmap_init_i2c`

``` C
/**                                                                         
 * devm_regmap_init_i2c(): Initialise managed register map                  
 *                                                                          
 * @i2c: Device that will be interacted with                                
 * @config: Configuration for register map                                  
 *                                                                          
 * The return value will be an ERR_PTR() on error or a valid pointer        
 * to a struct regmap.  The regmap will be automatically freed by the       
 * device management code.                                                  
 */                                                                         
struct regmap *devm_regmap_init_i2c(struct i2c_client *i2c,               
                    const struct regmap_config *config)                   
{                                                                         
    return devm_regmap_init(&i2c->dev, &regmap_i2c, &i2c->dev, config);   
}                                                                         
EXPORT_SYMBOL_GPL(devm_regmap_init_i2c);                                  
```
>file: drivers/base/regmap/regmap-i2c.c


## 配置

在将默认值设置完成后,将外部模块(Codec wm8594)上电后进行初始值的配置

接口: `regcache_sync`


## 实现


![regmap](images/regmap.jpg)





## 参考

1. [内核探索：Regmap 框架：简化慢速 I/O 接口优化性能](http://www.tinylab.org/kernel-explore-regmap-framework/#)


