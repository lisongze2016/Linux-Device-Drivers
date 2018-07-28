
## 5.2 timer

### 5.2.1 Struct timer_list结构体

```c
struct timer_list {
	/*
	 * All fields that change during normal runtime grouped to the
	 * same cacheline
	 */
	struct list_head entry;		//定时器链表的入口
	unsigned long expires;		//以jiffies为单位的定时值
	struct tvec_base *base;		//定时器内部值，用户不要使用

	void (*function)(unsigned long);//定时器处理函数
	unsigned long data;

	int slack;

#ifdef CONFIG_TIMER_STATS
	int start_pid;
	void *start_site;
	char start_comm[16];
#endif
#ifdef CONFIG_LOCKDEP
	struct lockdep_map lockdep_map;
#endif
};
```
### 5.2.2 定时器使用步骤

1）分配定时器
```c
struct timer_list mytimer;
int mydata = 0x1234;
```
2）初始化定时器对象
```c
init_timer(&mytimer);
```
3）指定定时器的超时处理函数
```c
mytimer.function = mytimer_function;
```
4）指定定时器的超时时间
```c
mytimer.expires = jiffies + msecs_to_jiffies(2000);
//jiffies+2*HZ;2s
```
5）给超时处理函数传递参数
```c
mytimer.data = (unsigned long)&mydata;
```
其中步骤2-5等价于set_timer(&mytimer.mytimer_function,(unsigned long)&mydata);

6）向内核注册定时器对象并且启动定时器
```c
add_timer(&mytimer);
```
7)删除定时器
```c
del_timer(&mytimer);
```

### timer demo程序
```c
#include <linux/module.h>
#include <linux/init.h>
#include <linux/timer.h>

/* 1.分配定时器 */
struct timer_list mytimer;
int mydata = 0x1234;
char buf[] = "hello timer!";

void mytimer_function(unsigned long data)
{
	/* int *pdata = (int *)data; */
	char *pdata = (char *)data;
	/* printk("lsz mydata=%#x\n",*pdata); */
	printk("lsz mydata=%s\n",pdata);
	/* 定时器到期，处理函数只执行一次；如果循环执行，必须重启定时器*/
	mod_timer(&mytimer,jiffies+msecs_to_jiffies(2000));
}

static void timer_start(void)
{
	/* char buf[]= "hello"; */
	/* 2.初始化定时器对象 */
	init_timer(&mytimer);
	/* 3.指定定时器的超时处理函数 */
	mytimer.function = mytimer_function;
	/* 4.指定定时器的超时时间 */
	mytimer.expires = jiffies + msecs_to_jiffies(2000);
	/* jiffies+2*HZ;2s */
	/* 5.给超时处理函数传递参数 */
	/* mytimer.data = (unsigned long)&mydata; */
	mytimer.data = (unsigned long)buf;
	/* setup_timer(&mytimer.mytimer_function,(unsigned long)&mydata);等价2-5 */
	/* 6.向内核注册定时器对象并且启动定时器 */
	add_timer(&mytimer);
}

static int __init mytimer_init(void)
{
	printk("hello,word! driver module is inserted!\n");
	timer_start();

	return 0;
}
module_init(mytimer_init);/*指定驱动模块入口点*/

static void __exit mytimer_exit(void)
{
	printk("goodbye, word! driver is removed!\n");
	del_timer(&mytim:wqer);
}
module_exit(mytimer_exit);/*指定驱动模块出口*/

MODULE_LICENSE("GPL");/*公共许可声明*/
MODULE_AUTHOR("Songze Lee <Songze_Lee@163.com>");/*作者*/
MODULE_VERSION("Verson 1.0");/*版本号*/
MODULE_DESCRIPTION("It is a simple demo for timer");/*描述*/
```
