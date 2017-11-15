# FrameBuffer

## 基本概念

### 分辨率

> 指每行和第列的像素数的描述。比如480*272.表示每行分布着480点,每列分布着272点.分辩率越高越意味显示越清

### bpp (bits per pixel)

> 表示RGB的bits数

嵌入式LINUX常用 16bpp,18bpp,24bpp. bpp值越大,能表示颜色越丰富.但是占用空间越多.单色的bpp就是1.一个点占的字节数移为(BPP) bytes per pixel,BPP与bpp的对应换算并不是一个简单除8的关系,要根据硬件换算。一般是16bpp占用2byte,18,24,32均是4个字节. bpp还一个说法是,**色深(Color Deepth)**



## 结构体

### fb_info

```
struct fb_info
{
   int node;
   int flags;
   struct fb_var_screeninfo var; /*可变参数 */
   struct fb_fix_screeninfo fix; /*固定参数 */
   struct fb_monspecs monspecs; /*显示器标准 */
   struct work_struct queue; /* 帧缓冲事件队列 */
   struct fb_pixmap pixmap; /* 图像硬件 mapper */
   struct fb_pixmap sprite; /* 光标硬件 mapper */
   struct fb_cmap cmap; /* 目前的颜色表*/
   struct list_head modelist;
   struct fb_videomode *mode; /* 目前的 video 模式 */

   #ifdef CONFIG_FB_BACKLIGHT
      struct mutex bl_mutex;
      /*  对应的背光设备  */
      struct backlight_device *bl_dev;
      /* 背光调整 */
      u8 bl_curve[FB_BACKLIGHT_LEVELS];
    #endif

    struct fb_ops *fbops; /* fb_ops，帧缓冲操作 */
    struct device *device;
    struct class_device *class_device; /
    int class_flag; /* 私有 sysfs 标志 */
    #ifdef CONFIG_FB_TILEBLITTING
        struct fb_tile_ops *tileops; /* 图块 Blitting */
    #endif
    char _ _iomem *screen_base; /* 虚拟基地址 */
    unsigned long screen_size; /* ioremapped 的虚拟内存大小 */
    void *pseudo_palette; /* 伪 16 色颜色表 */
    #define FBINFO_STATE_RUNNING 0
    #define FBINFO_STATE_SUSPENDED         1
    u32 state; /* 硬件状态，如挂起 */
    void *fbcon_par;
    void *par;
};
```
### fb_ops

```
struct fb_ops
{
	struct module *owner;
	/* 打开/释放 */
	int(*fb_open)(struct fb_info *info, int user);
	int(*fb_release)(struct fb_info *info, int user);

	/* 对于非线性布局的/常规内存映射无法工作的帧缓冲设备需要 */
	ssize_t(*fb_read)(struct file *file, char _ _user *buf, size_t count,
		loff_t*ppos);
	ssize_t(*fb_write)(struct file *file, const char _ _user *buf, size_t count,
		loff_t *ppos);

	/* 检测可变参数，并调整到支持的值*/
	int(*fb_check_var)(struct  fb_var_screeninfo  *var,  struct fb_info *info);

	/* 根据 info->var 设置 video 模式 */
	int(*fb_set_par)(struct fb_info *info);

	/* 设置 color 寄存器 */
	int(*fb_setcolreg)(unsigned regno, unsigned red, unsigned green, unsigned
		blue, unsigned transp, struct fb_info *info);

	/* 批量设置 color 寄存器，设置颜色表 */
	int(*fb_setcmap)(struct fb_cmap *cmap, struct fb_info *info);

	/*显示空白 */
	int(*fb_blank)(int blank, struct fb_info *info);

	/* pan 显示 */
	int(*fb_pan_display)(struct  fb_var_screeninfo  *var,  struct fb_info *info);

	/* 矩形填充 */
	void(*fb_fillrect)(struct  fb_info  *info,  const  struct fb_fillrect *rect);
	/* 数据复制 */
	void(*fb_copyarea)(struct  fb_info  *info,  const  struct fb_copyarea *region);
	/* 图形填充 */
	void(*fb_imageblit)(struct fb_info *info, const struct fb_image *image);

	/* 绘制光标 */
	int(*fb_cursor)(struct fb_info *info, struct fb_cursor *cursor);

	/* 旋转显示 */
	void(*fb_rotate)(struct fb_info *info, int angle);

	/* 等待 blit 空闲 (可选) */
	int(*fb_sync)(struct fb_info *info);

	/* fb 特定的 ioctl (可选) */
	int(*fb_ioctl)(struct fb_info *info, unsigned int cmd, unsigned long arg);

	/* 处理 32 位的 compat ioctl (可选) */
	int(*fb_compat_ioctl)(struct  fb_info  *info,  unsigned  cmd, unsigned long arg);

	/* fb 特定的 mmap */
	int(*fb_mmap)(struct fb_info *info, struct vm_area_struct *vma);

	/* 保存目前的硬件状态 */
	void(*fb_save_state)(struct fb_info *info);

	/* 恢复被保存的硬件状态 */
	void(*fb_restore_state)(struct fb_info *info);
};
```

### fb_var_screeninfo

```
struct fb_var_screeninfo
{
   /* 可见解析度   */
   u32 xres;
   u32 yres;
   /* 虚拟解析度    */
   u32 xres_virtual;
   u32 yres_virtual;
   /* 虚拟到可见之间的偏移 */
   u32 xoffset;
   u32 yoffset;

   u32 bits_per_pixel; /* 每像素位数，BPP */
   u32 grayscale; /非 0 时指灰度 */

   /* fb 缓存的 R\G\B 位域 */
   struct fb_bitfield red;
   struct fb_bitfield green;
   struct fb_bitfield blue;
   struct fb_bitfield transp; /* 透明度 */

   u32 nonstd; /* != 0 非标准像素格式 */

   u32 activate;

   u32 height; /*高度 */
   u32 width; /*宽度 */

   u32 accel_flags; /* 看 fb_info.flags */

   /* 定时: 除了 pixclock 本身外，其他的都以像素时钟为单位 */
   u32 pixclock; /* 像素时钟（皮秒） */
   u32 left_margin; /* 行切换：从同步到绘图之间的延迟    */
   u32 right_margin; /* 行切换：从绘图到同步之间的延迟   */
   u32 upper_margin; /* 帧切换：从同步到绘图之间的延迟  */
   u32 lower_margin; /* 帧切换：从绘图到同步之间的延迟  */
   u32 hsync_len; /* 水平同步的长度     */
   u32 vsync_len; /* 垂直同步的长度     */
   u32 sync;
   u32 vmode;
   u32 rotate; /* 顺时钟旋转的角度 */
   u32 reserved[5]; /* 保留 */
};
```
### fb_fix_screeninfo

```
 struct fb_fix_screeninfo
 {
     char id[16]; /* 字符串形式的标识符 */
     unsigned long smem_start; /* fb 缓存的开始位置 */
     u33 smem_len; /* fb 缓存的长度 */
     u32 type; /* FB_TYPE_*        */
     u32 type_aux; /* 分界 */
     u32 visual; /* FB_VISUAL_* */
     u16 xpanstep; /* 如果没有硬件 panning ，赋 0 */
     u16 ypanstep;
     u16 ywrapstep;/
     u32 line_length; /* 1 行的字节数 */
     unsigned long mmio_start; /* 内存映射 I/O 的开始位置 */
     u32 mmio_len; /* 内存映射 I/O 的长度  */
     u32 accel;
     u16 reserved[3]; /* 保留*/
 };
```
