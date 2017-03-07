# regmap


## 数据结构

``` C
struct regmap_config {
    const char *name;

    int reg_bits;			// 寄存器地址的位数，必须配置，例如I2C寄存器地址位数为 8  
    int reg_stride; 
    int pad_bits;			// 寄存器值的位数，必须配置 
    int val_bits; 
    
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
    const struct reg_default *reg_defaults;
    unsigned int num_reg_defaults;
    enum regcache_type cache_type;
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

## 常用接口



## 配置寄存器默认值

### 数据结构

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
