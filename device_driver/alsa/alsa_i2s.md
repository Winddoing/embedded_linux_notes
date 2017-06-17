# I2S


## 作用

主要是控制I2S接口的时钟和I2S接口相关操作实现.

## 注册


```
int snd_soc_register_component(struct device *dev,
             const struct snd_soc_component_driver *cmpnt_drv,
             struct snd_soc_dai_driver *dai_drv,
             int num_dai)
```

参数:



## 数据结构

### struct snd_soc_component_driver

```
struct snd_soc_component_driver {
    const char *name;
};
```

### struct snd_soc_dai_driver

```
struct snd_soc_dai_driver {
    /* DAI description */
    const char *name;
    unsigned int id;
    int ac97_control;
    unsigned int base;

    /* DAI driver callbacks */
    int (*probe)(struct snd_soc_dai *dai);
    int (*remove)(struct snd_soc_dai *dai);
    int (*suspend)(struct snd_soc_dai *dai);
    int (*resume)(struct snd_soc_dai *dai);
    /* compress dai */
    bool compress_dai;

    /* ops */
    const struct snd_soc_dai_ops *ops;		//dai的操作接口

    /* DAI capabilities */
    struct snd_soc_pcm_stream capture;
    struct snd_soc_pcm_stream playback;
    unsigned int symmetric_rates:1;

    /* probe ordering - for components with runtime dependencies */
    int probe_order;
    int remove_order;
};
```

### struct snd_soc_dai_ops

```
struct snd_soc_dai_ops {
    /*
     * DAI clocking configuration, all optional.
     * Called by soc_card drivers, normally in their hw_params.
     */
    int (*set_sysclk)(struct snd_soc_dai *dai,
        int clk_id, unsigned int freq, int dir);
    int (*set_pll)(struct snd_soc_dai *dai, int pll_id, int source,
        unsigned int freq_in, unsigned int freq_out);
    int (*set_clkdiv)(struct snd_soc_dai *dai, int div_id, int div);

    /*
     * DAI format configuration
     * Called by soc_card drivers, normally in their hw_params.
     */
    int (*set_fmt)(struct snd_soc_dai *dai, unsigned int fmt);
    int (*set_tdm_slot)(struct snd_soc_dai *dai,
        unsigned int tx_mask, unsigned int rx_mask,
        int slots, int slot_width);
    int (*set_channel_map)(struct snd_soc_dai *dai,
        unsigned int tx_num, unsigned int *tx_slot,
        unsigned int rx_num, unsigned int *rx_slot);
    int (*set_tristate)(struct snd_soc_dai *dai, int tristate);

    /*
     * DAI digital mute - optional.
     * Called by soc-core to minimise any pops.
     */
    int (*digital_mute)(struct snd_soc_dai *dai, int mute);
    int (*mute_stream)(struct snd_soc_dai *dai, int mute, int stream);

    /*
     * ALSA PCM audio operations - all optional.
     * Called by soc-core during audio PCM operations.
     */
    int (*startup)(struct snd_pcm_substream *,
        struct snd_soc_dai *);
    void (*shutdown)(struct snd_pcm_substream *,
        struct snd_soc_dai *);
    int (*hw_params)(struct snd_pcm_substream *,
        struct snd_pcm_hw_params *, struct snd_soc_dai *);
    int (*hw_free)(struct snd_pcm_substream *,
        struct snd_soc_dai *);
    int (*prepare)(struct snd_pcm_substream *,
        struct snd_soc_dai *);
    int (*trigger)(struct snd_pcm_substream *, int,
        struct snd_soc_dai *);
    int (*bespoke_trigger)(struct snd_pcm_substream *, int,
        struct snd_soc_dai *);
    /*
     * For hardware based FIFO caused delay reporting.
     * Optional.
     */
    snd_pcm_sframes_t (*delay)(struct snd_pcm_substream *,
        struct snd_soc_dai *);
};
```
