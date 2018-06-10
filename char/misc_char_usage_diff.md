#### 2.3.2 字符设备驱动和混杂设备驱动的使用区别

字符驱动程序完成初始化顺序如下:

1. register_chrdev_region()/register_chrdev_region()注册分配主次设备号
2. class_create()和device_create()创建/dev和/sys设备节点
3. 使用cdev_init()和cdev_add()将自身注册为字符设备驱动

字符驱动程序注销顺序如下:

1. unregister_chrdev()和device_unregister()卸载类下的设备和类
2. cdev_del(&cdev);删除一个 cdev，完成字符设备的注销
3. unregister_chrdev_region()注销字符设备

混杂设备只需调用misc_register()和misc_deregister()完成初始化和注销设备。

```
static struct miscdevice miscdev = {
	.minor = MISC_DYNAMIC_MINOR,
	.name = "LED",
	.fops = &fops,
};
misc_register(&miscdev);
misc_deregister(&miscdev);
```

```
int misc_register(struct miscdevice * misc)
```

misc_register()函数用于注册一个混杂设备，其主设备号为10，如果次设备号指定为MISC_DYNAMIC_MINOR将由系统去指定一个次设备号，调用device_create创建设备节点。

```
misc_deregister(struct miscdevice * misc)
```

misc_deregister()用于注销这个混杂设备，其中调用了device_destroy删除设备节点。

混杂设备就是主设备号为10，所有的混杂设备形成一个链表，次设备号由注册时指定，类名为misc，自动创建设备节点的字符设备，是一种节约主设备号，为驱动程序员简化的字符设备驱动注册方式。
