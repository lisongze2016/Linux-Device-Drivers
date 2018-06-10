### 2.1.2 自动创建设备节点
	
在上一个程序中通mknod创建字符设备，内核提供了一组函数来自动创建。如下所示：

(1).class_create()创建类

```
#define class_create(owner, name)               \
({                                              \
	static struct lock_class_key __key;     \
	__class_create(owner, name, &__key);    \
})
```
宏class_create()用于动态创建设备的逻辑类，并完成部分字段的初始化，然后将其添加进linux内核系统中。
此函数的执行效果就是在目录/sys/class/下创建一个新的文件夹，此文件夹的名字为此函数的第二个输入参数，文件夹是空的。

宏输入参数说明：

- 宏owner是一个struct module结构体类型的指针，指向函数__class_create()即将创建的struct class类型对象的拥有者，一般赋值为THIS_MODULE，详细查看：src/include/linux/module.h
- name是char类型的指针，代表即将创建的struct class变量的名字
- 返回参数：
    宏class_create()函数返回 struct class*的逻辑类

(2).class_destroy()销毁结构体类

```
void class_destroy(struct class *cls)
cls: 指向要被销毁的结构体类，指针指向的必须时已经被创建的结构体
```

(3).device_create()创建一个设备，并在sysfs中注册

```
struct device *device_create(struct class *class, struct device *parent,
					dev_t devt, void *drvdata, const char *fmt, ...)
```

函数功能：

函数device_create()用于动态的建立逻辑设备，并对新的逻辑设备类进行相应初始化，将其与函数的第一个参数所代表的逻辑类关联起来，然后将此逻辑设备加到linux内核系统的设备驱动程序模型中。函数能够自动在/sys/devices/virtual目录下创建新的逻辑设备目录，在/dev目录下创建于逻辑类对应的设备文件

参数说明：

- struct class *class：即将创建逻辑设备相关的逻辑类
- dev_t dev:设备号
- void *drvdata: void类型的指针，代表回调函数的输入参数
- const char *fmt: 逻辑设备的设备名,即在目录 /sys/devices/virtual创建的逻辑设备目录的目录名。

4.函数device_unregister()移除设备

```
void device_unregister(struct device *dev)
```

class_create() 和 device_create()关系

很多时候都是利用mknod命令手动创建设备节点，实际上Linux内核为我们提供了一组函数，可以用来在模块加载的时候自动在/dev目录下创建相应设备节点，并在卸载模块时删除该节点，当然前提条件是用户空间移植了udev。

内核中定义了struct class结构体，顾名思义，一个struct class结构体类型变量对应一个类，内核同时提供了class_create(…)函数，可以用它来创建一个类，这个类存放于sysfs下面，一旦创建好了这个类，再调用device_create(…)函数来在/dev目录下创建相应的设备节点。这样，加载模块的时候，用户空间中的udev会自动响应device_create(…)函数，去/sysfs下寻找对应的类从而创建设备节点。

修改上一个程序部分代码如下，insmod led.ko可在/dev/看见相应的设备节点。

```
static struct class *led_class = NULL;
static struct device *led_device = NULL;

 static int __init exynos4412_led_init(void)
{
	int ret = 0;
	VGPM4BASE = ioremap(GPM4BASE,SZ_4K);
	if(NULL == VGPM4BASE){
		return -ENOMEM;
	}
	led_init();
	ret = register_chrdev(LED_MAJOR,"LED",&exynos4412_led_fops);

	/*自动创建设备文件节点*/
	/*1.创建LED类*/
	led_class =  class_create(THIS_MODULE,"LED");
	/*2.创建一个设备，并在sysfs中注册*/
	led_device = device_create(led_class,NULL,MKDEV(LED_MAJOR,0),NULL,"led");

	return ret;
}
module_init(exynos4412_led_init);

static void __exit exynos4412_led_exit(void)
{
	unregister_chrdev(LED_MAJOR,"LED");
	/*卸载类下的设备*/
	device_unregister(led_device);
	/*卸载类*/
	class_destroy(led_class);
	/*解除映射*/
	iounmap(VGPM4BASE);
	printk("goodbye! led  driver\n");
}
```
