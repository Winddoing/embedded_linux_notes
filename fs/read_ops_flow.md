# read流程---文件系统

linux: 3.10

## 系统框架

用户空间   ----系统调用-----   内核空间

read                         sys_read


## 数据结构

### struct fd

``` C
struct fd {                     
    struct file *file;
    int need_put;
};
```
>file: include/linux/file.h


### struct file

``` C
struct file {
	/*
	 * fu_list becomes invalid after file_free is called and queued via
	 * fu_rcuhead for RCU freeing
	 */
	union {
		struct list_head	fu_list;
		struct rcu_head 	fu_rcuhead;
	} f_u;
	struct path		f_path;
#define f_dentry	f_path.dentry
	struct inode		*f_inode;	/* cached value */
	const struct file_operations	*f_op;

	/*
	 * Protects f_ep_links, f_flags, f_pos vs i_size in lseek SEEK_CUR.
	 * Must not be taken from IRQ context.
	 */
	spinlock_t		f_lock;
#ifdef CONFIG_SMP
	int			f_sb_list_cpu;
#endif
	atomic_long_t		f_count;
	unsigned int 		f_flags;
	fmode_t			f_mode;
	loff_t			f_pos;
	struct fown_struct	f_owner;
	const struct cred	*f_cred;
	struct file_ra_state	f_ra;

	u64			f_version;
#ifdef CONFIG_SECURITY
	void			*f_security;
#endif
	/* needed for tty driver, and maybe others */
	void			*private_data;

#ifdef CONFIG_EPOLL
	/* Used by fs/eventpoll.c to link all the hooks to this file */
	struct list_head	f_ep_links;
	struct list_head	f_tfile_llink;
#endif /* #ifdef CONFIG_EPOLL */
	struct address_space	*f_mapping;
#ifdef CONFIG_DEBUG_WRITECOUNT
	unsigned long f_mnt_write_state;
#endif
};
```
>file: include/linux/fs.h

### struct file_operations

``` C
struct file_operations {                                                                                                 
    struct module *owner;
    loff_t (*llseek) (struct file *, loff_t, int);
    ssize_t (*read) (struct file *, char __user *, size_t, loff_t *);
    ssize_t (*write) (struct file *, const char __user *, size_t, loff_t *);
    ssize_t (*aio_read) (struct kiocb *, const struct iovec *, unsigned long, loff_t);
    ssize_t (*aio_write) (struct kiocb *, const struct iovec *, unsigned long, loff_t);
    int (*readdir) (struct file *, void *, filldir_t);
    unsigned int (*poll) (struct file *, struct poll_table_struct *);
    long (*unlocked_ioctl) (struct file *, unsigned int, unsigned long);
    long (*compat_ioctl) (struct file *, unsigned int, unsigned long);
    int (*mmap) (struct file *, struct vm_area_struct *);
    int (*open) (struct inode *, struct file *);
    int (*flush) (struct file *, fl_owner_t id);
    int (*release) (struct inode *, struct file *);
    int (*fsync) (struct file *, loff_t, loff_t, int datasync);
    int (*aio_fsync) (struct kiocb *, int datasync);
    int (*fasync) (int, struct file *, int);
    int (*lock) (struct file *, int, struct file_lock *);
    ssize_t (*sendpage) (struct file *, struct page *, int, size_t, loff_t *, int);
    unsigned long (*get_unmapped_area)(struct file *, unsigned long, unsigned long, unsigned long, unsigned long);
    int (*check_flags)(int);
    int (*flock) (struct file *, int, struct file_lock *);
    ssize_t (*splice_write)(struct pipe_inode_info *, struct file *, loff_t *, size_t, unsigned int);
    ssize_t (*splice_read)(struct file *, loff_t *, struct pipe_inode_info *, size_t, unsigned int);
    int (*setlease)(struct file *, long, struct file_lock **);
    long (*fallocate)(struct file *file, int mode, loff_t offset,
              loff_t len);
    int (*show_fdinfo)(struct seq_file *m, struct file *f);
};
```
>file:include/linux/fs.h

### struct seq_file

``` C
struct seq_file {                                                               
    char *buf;
    size_t size;
    size_t from;
    size_t count;
    loff_t index;
    loff_t read_pos;
    u64 version;
    struct mutex lock;
    const struct seq_operations *op;
    int poll_event;
#ifdef CONFIG_USER_NS
    struct user_namespace *user_ns;
#endif
    void *private;
};      
```
>file:include/linux/seq_file.h

## 流程

### sys_read

``` C
SYSCALL_DEFINE3(read, unsigned int, fd, char __user *, buf, size_t, count)
{
	...
	ret = vfs_read(f.file, buf, count, &pos);
	...	
}
```
>file:fs/read_write.c

### vfs_read

``` C
ssize_t vfs_read(struct file *file, char __user *buf, size_t count, loff_t *pos)
{
	...
	 if (file->f_op->read)
	     ret = file->f_op->read(file, buf, count, pos);
	 else
	     ret = do_sync_read(file, buf, count, pos);
	...
}
```
>file:fs/read_write.c

在vfs层,需要匹配具体的文件系统的ops操作(read)

### 文件系统匹配----ext4


``` C
static const struct file_operations ext4_seq_options_fops = {
    .owner = THIS_MODULE,
    .open = options_open_fs,
    .read = seq_read,
    .llseek = seq_lseek,
    .release = single_release,                                                    
};
```
>file:

### seq_read

``` C
ssize_t seq_read(struct file *file, char __user *buf, size_t size, loff_t *ppos)
{ ... }
```
>file:fs/seq_file.c

### 文件系统绑定block层

``` C
static const struct seq_operations partitions_op = {
    .start  = show_partition_start,
    .next   = disk_seqf_next,
    .stop   = disk_seqf_stop,
    .show   = show_partition
};
```
>file:block/genhd.c


### 


### block层





