## 2.1 字符设备驱动

linux设备分为字符设备、块设备和网络设备三种，字符设备是最常见、最简单的一种。字符设备的访问是以字节流的形式来访问设备的，换句话说，应用程序对它的读取是以字节为单位，而且要按照先后顺序不能随机读取。串口是最常见的字符设备，它在进行收发数据时就是一个字节一个字节进行传输。

cdev结构体


```
struct cdev {
		struct kobject kobj;	/*内嵌的kobject对象*/
		struct module *owner;	/*所属模块*/
		const struct file_operations *ops;	/*文件操作结构体*/
		struct list_head list;	
		dev_t dev;			/*设备号*/
		unsigned int count;
};

```
cdev 结构体的 dev_t 成员定义了设备号,为 32 位,其中高 12 位为主设备号,低20 位为次设备号。
使用下列宏可以从 dev_t 获得主设备号和次设备号。 
MAJOR(dev_t dev)
MINOR(dev_t dev)
而使用下列宏则可以通过主设备号和设备号生成 dev_t
MKDEV(int major, int minor)

Linux内核提供一组函数用于操作cdev结构体：

```
void cdev_init(struct cdev *, struct file_operations *);
用于初始化 cdev 的成员,并建立 cdev 和 file_operations 之间的连接

```

```
struct cdev *cdev_alloc(void);
动态申请一个 cdev 内存

```

```
void cdev_put(struct cdev *p);
int cdev_add(struct cdev *, dev_t, unsigned);
向系统添加一个 cdev,完成字符设备的注册
```

```
void cdev_del(struct cdev *);
向系统删除一个 cdev,完成字符设备的注销

```

```

file_operations 结构体
	struct file_operations
	{
		struct module *owner;
		// 拥有该结构的模块的指针,一般为 THIS_MODULES
		loff_t(*llseek)(struct file *, loff_t, int);
		// 用来修改文件当前的读写位置
		ssize_t(*read)(struct file *, char _ _user *, size_t, loff_t*);
		// 从设备中同步读取数据
		ssize_t(*write)(struct file *, const char _ _user *, size_t,
		loff_t*);
		// 向设备发送数据
		unsigned int(*poll)(struct file *, struct poll_table_struct*);
		// 轮询函数,判断目前是否可以进行非阻塞的读取或写入
		int(*ioctl)(struct inode *, struct file *, unsigned int, unsigned
		long);
		// 执行设备 I/O 控制命令
		int(*mmap)(struct file *, struct vm_area_struct*);
		// 用于请求将设备内存映射到进程地址空间
		int(*open)(struct inode *, struct file*);
		// 打开
		int(*release)(struct inode *, struct file*);
		……
};

```

内核空间数据和用户空间访问函数

unsigned long __must_check copy_from_user(void *to, const void __user *from, unsigned long n)

用户空间缓存区到内核空间的复制

- to：目标地址，内核空间地址
- from：源地址，用户空间地址
- n：拷贝数据的字节数

unsigned long __must_check copy_to_user(void __user *to, const void *from, unsigned long n)
内核空间到用户空间缓存区的复制
- to：目标地址，用户空间地址
- from:源地址，内核空间地址
- n:拷贝数据的字节数
