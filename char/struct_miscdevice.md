#### 2.3.1 miscdevice 结构体

```
struct miscdevice  {
	int minor;                              /* 次设备号*/
	const char *name;                       /*设备名称*/
	const struct file_operations *fops;     /*文件操作结构体*/
	struct list_head list;                  /*misc_list的链表头*/
	struct device *parent;                  /*父设备*/
	struct device *this_device;             /*当前设备*/
	const char *nodename;
	umode_t mode;
};

```

