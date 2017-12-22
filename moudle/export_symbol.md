### 1.1.3 导出符号

Linux的“/proc/kallsyms”文件对应着内核符号表，它记录了符号以及符号所在的内存地址。
模块可以使用如下宏导出符号到内核符号表中：
EXPORT_SYMBOL(符号表);
EXPORT_SYMBOL_GPL(符号表);
导出的符号（变量或函数）可以被其他模块使用，只需要使用前声明一下即可。EXPORT_SYMBOL_GPL()只适用于包含GPL许可权的模块。
示例代码如下：
demo1.c
```
#include <linux/init.h>
#include <linux/module.h>

extern int sure;
extern void pri_value(int);
extern void call_func0(void);

static int __init demo_init(void)
{
	sure = 5678;
	pri_value(sure);

	printk("hello,word! driver module is inserted!\n");

	return 0;
}
module_init(demo_init);

static void __exit demo_exit(void)
{
	printk("goodbye, word! driver is removed!\n");
} 
module_exit(demo_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Songze Lee");
MODULE_VERSION("verson 1.0");
MODULE_DESCRIPTION("It is a simple demo for driver module");
```
demo2.c
```
#include <linux/init.h>
#include <linux/module.h>

static int sure = 1234;

/*符号导出 static定义的变量或函数其他模块可以调用*/
EXPORT_SYMBOL_GPL(sure);

static void pri_value(int val)
{
	printk("In %s: sure = %d\n", __FILE__, sure);
}
EXPORT_SYMBOL_GPL(pri_value);

static int __init demo_init(void)
{
	pri_value(sure);

	printk("hello,word! driver module is inserted!\n");

	return 0;
}
module_init(demo_init);

static void __exit demo_exit(void)
{
	printk("goodbye, word! driver is removed!\n");
}
module_exit(demo_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Songze Lee");
MODULE_VERSION("verson 1.0");
MODULE_DESCRIPTION("It is a simple demo for driver module");
```
调试结果如下：
```
[projct /]# insmod demo02.ko 
[ 2484.680000] In /ARM/linux-3.5-songze/drivers/songze_drivers/module/
demo05/demo02.c: sure = 1234
[ 2484.680000] hello,word! driver module is inserted!
[projct /]# insmod demo01.ko 
[ 2491.690000] In /ARM/linux-3.5-songze/drivers/songze_drivers/module/
demo05/demo02.c: sure = 5678
[ 2491.690000] hello,word! driver module is inserted!
[projct /]# rmmod demo02
rmmod: remove 'demo02': Resource temporarily unavailabl两者有依赖关系
[projct /]# rmmod msb
[ 2531.345000] goodbye, word! driver is removed!
rmmod: module 'msb' not found
[projct /]# rmmod demo02
[ 2542.910000] goodbye, word! driver is removed!
rmmod: module 'demo02' not found
```