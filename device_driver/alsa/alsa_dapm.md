# DAPM


>所谓widget，其实可以理解为是kcontrol的进一步升级和封装，她同样是指音频系统中的某个部件，比如mixer，mux，输入输出引脚，电源供应器等等，甚至，我们可以定义虚拟的widget，例如playback stream widget。widget把kcontrol和动态电源管理进行了有机的结合，同时还具备音频路径的连结功能，一个widget可以与它相邻的widget有某种动态的连结关系。在DAPM框架中，widget用结构体snd_soc_dapm_widget来描述：

## 数据结构

``` C
/* dapm widget */
struct snd_soc_dapm_widget {
    enum snd_soc_dapm_type id;   //该widget的类型值，比如snd_soc_dapm_output，snd_soc_dapm_mixer等等。
    const char *name;       /* widget name */
    const char *sname;  /* stream name */
    struct snd_soc_codec *codec;
    struct snd_soc_platform *platform;
    struct list_head list;		//所有注册到系统中的widget都会通过该list，链接到代表声卡的snd_soc_card结构的widgets链表头字段中
    struct snd_soc_dapm_context *dapm;	//snd_soc_dapm_context结构指针，ASoc把系统划分为多个dapm域，每个widget属于某个dapm域，同一个域代表着同样的偏置电压供电策略，
										//比如，同一个codec中的widget通常位于同一个dapm域，而平台上的widget可能又会位于另外一个platform域中。

    void *priv;             /* widget specific data */
    struct regulator *regulator;        /* attached regulator */	// 对于snd_soc_dapm_regulator_supply类型的widget，该字段指向与之相关的regulator结构指针。
    const struct snd_soc_pcm_stream *params; /* params for dai links */ //目前对于snd_soc_dapm_dai_link类型的widget，指向该dai的配置信息的snd_soc_pcm_stream结构

    /* dapm control */
    int reg;                /* negative reg = no direct dapm */
    unsigned char shift;            /* bits to shift */
    unsigned int value;             /* widget current value */
    unsigned int mask;          /* non-shifted mask */
    unsigned int on_val;            /* on state value */
    unsigned int off_val;           /* off state value */
    unsigned char power:1;          /* block power status */
    unsigned char invert:1;         /* invert the power bit */
    unsigned char active:1;         /* active stream on DAC, ADC's */
    unsigned char connected:1;      /* connected codec pin */
    unsigned char new:1;            /* cnew complete */
    unsigned char ext:1;            /* has external widgets */
    unsigned char force:1;          /* force state */
    unsigned char ignore_suspend:1;         /* kept enabled over suspend */
    unsigned char new_power:1;      /* power from this run */
    unsigned char power_checked:1;      /* power checked this run */
    int subseq;             /* sort within widget type */

    int (*power_check)(struct snd_soc_dapm_widget *w);

    /* external events */
    unsigned short event_flags;     /* flags to specify event types */
    int (*event)(struct snd_soc_dapm_widget*, struct snd_kcontrol *, int);

    /* kcontrols that relate to this widget */
    int num_kcontrols;
    const struct snd_kcontrol_new *kcontrol_news;
    struct snd_kcontrol **kcontrols;

    /* widget input and outputs */
    struct list_head sources;
    struct list_head sinks;

    /* used during DAPM updates */
    struct list_head power_list;
    struct list_head dirty;
    int inputs;			//该widget的所有有效路径中，连接到输入端的路径数量。
    int outputs;		//该widget的所有有效路径中，连接到输出端的路径数量。

    struct clk *clk;
};
```



## 参考

1. [ALSA声卡驱动中的DAPM详解之二：widget-具备路径和电源管理信息的kcontrol](http://blog.csdn.net/droidphone/article/details/12906139)
