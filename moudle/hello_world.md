### 1.1.2 一个最简单的linux内核模块代码（hello world）

#### A．hello world驱动模块
demo.c
```
#include <linux/init.h>
#include <linux/module.h>

static int __init demo_init(void)
{
	printk("hello,word! driver module is inserted!\n");

	return 0;
}
module_init(demo_init);/*指定驱动模块入口点*/


static void __exit demo_exit(void)
{
	printk("goodbye, word! driver is removed!\n");
}
module_exit(demo_exit);/*指定驱动模块出口*/

MODULE_LICENSE("GPL");/*公共许可声明*/
MODULE_AUTHOR("Songze Lee");/*作者*/
MODULE_VERSION("Verson 1.0");/*版本号*/
MODULE_DESCRIPTION("It is a simple demo for driver module");/*描述*/

```
内核模块中用于输出的函数的内核空间的printk(),用法和printf基本相似，但可以定义输出级别。
Makefile编写如下：
```
obj-m	:= demo.o

OUR_KERNEL := /ARM/linux-3.5-songze/

all:
	make -C $(OUR_KERNEL) M=$(shell pwd) modules
#	进入到变量目录(内核源码目录)下编译  M 编译pwd路径下的模块
clean:
	make -C $(OUR_KERNEL) M=`pwd` clean
```
调试结果如下：
```
[projct /]# insmod demo.ko
[ 1670.390000] hello,word! driver module is inserted!
[projct /]# rmmod demo
[ 1681.215000] goodbye, word! driver is removed!
rmmod: module 'demo' not found

```
#### B.驱动模块调用子函数
demo.c
```
/**
 * 驱动模块调用子函数
 * 注意Makefile的编写
 */

#include <linux/init.h>
#include <linux/module.h>

extern void call_func0(void);

static int __init demo_init(void)
{
	call_func0();
	printk("hello,word! driver module is inserted!\n");

	return 0;
}
module_init(demo_init);/*指定驱动模块入口点*/

static void __exit demo_exit(void)
{
	printk("goodbye, word! driver is removed!\n");
}
module_exit(demo_exit);/*指定驱动模块出口*/

MODULE_LICENSE("GPL");/*公共许可声明*/
MODULE_AUTHOR("Songze Lee");/*作者*/
MODULE_VERSION("Verson 1.0");/*版本号*/
MODULE_DESCRIPTION("It is a simple demo for driver module");/*描述*/
```
fun0.c
```
#include <linux/init.h>
#include <linux/module.h>

extern void call_func1(void);

void call_func0(void)
{
	call_func1();
}
```
fun1.c
```
#include <linux/init.h>
#include <linux/module.h>

void call_func1(void)
{
	printk("you are a good boy.\n");
}
```
Makefile(重点注意)
```
obj-m	:= team_hehe.o
team_hehe-objs	:= demo.o fun0.o fun1.o
# team_hehe.o依赖demo.o fun0.o fun1.o 编译成1个ko
OUR_KERNEL := /ARM/linux-3.5-songze/

all:
	make -C $(OUR_KERNEL) M=$(shell pwd) modules
clean:
	make -C $(OUR_KERNEL) M=`pwd` clean
```
调试结果如下：
```
[projct /]# insmod team_hehe.ko
[ 1920.880000] you are a good boy.
[ 1920.880000] hello,word! driver module is inserted!
[projct /]# rmmod team_hehe
[ 1932.020000] goodbye, word! driver is removed!
rmmod: module 'team_hehe' not found
```
