#### 2.3.5 混杂设备led驱动示例

led misc驱动：

```
/**
 *混杂设备misc 
 *主设备号为10的特殊的字符设备
 *
 */

#include <linux/init.h>
#include <linux/module.h>
#include <linux/fs.h>
#include <linux/uaccess.h>
#include <linux/slab.h>
#include <linux/io.h>
#include <linux/gpio.h>
#include <linux/miscdevice.h>

enum{
	ON,OFF
};

static int ledgpio[] = {
	EXYNOS4X12_GPM4(0),
	EXYNOS4X12_GPM4(1),
	EXYNOS4X12_GPM4(2),
	EXYNOS4X12_GPM4(3),
};

static void led_ctrl(int num, int stat)
{
	if(stat){
		 gpio_set_value(ledgpio[num - 1], ON);
	}else{
		gpio_set_value(ledgpio[num - 1], OFF);
	}
}

static void led_init(void)
{
	int i;

	for(i = 0; i < ARRAY_SIZE(ledgpio); i++){
		/*设置gpio方向为输出，电平为1*/
		gpio_direction_output(ledgpio[i], OFF);
	}
}

static void led_stat(char *buf)
{
	int i;

	for(i = 0; i < ARRAY_SIZE(ledgpio); i++){
		/*读取gpio端口的值*/
		if(gpio_get_value(ledgpio[i]) == ON){
			buf[i] = !ON;
		}else{
			buf[i] = !OFF;
		}
	}
}

static int exynos4412_led_open (struct inode *inodp, struct file *filp)
{

	return 0;
}

static ssize_t 
exynos4412_led_read (struct file *filp, char __user *buf, size_t cnt, loff_t *fpos)
{
	char stat[4] = {0};

	led_stat(stat);

	if(copy_to_user(buf, stat, sizeof(stat))){
		return -EINVAL;
	}

	return sizeof(stat);
}


static ssize_t 
exynos4412_led_write (struct file *filp, const char __user *buf, size_t cnt, loff_t *fpos)
{
	u8 cmd;	
	u8 num, stat;

	get_user(cmd, buf);

	num = cmd >> 4; stat = cmd & 0xf;

	if(( num < 1 ) || ( num > 4 ) ){
		return -EINVAL;
	}

	if(( stat != 0 ) && ( stat != 1 )){
		return -EINVAL;
	}

	led_ctrl(num, stat);

	return 1;
}

static int     
exynos4412_led_release (struct inode *inodp, struct file *filp)
{
	return 0;
}

static struct file_operations fops = {
	.owner = THIS_MODULE,
	.open = exynos4412_led_open,	
	.read = exynos4412_led_read,
	.write = exynos4412_led_write,
	.release = exynos4412_led_release,
};

static struct miscdevice miscdev = {
	.minor = MISC_DYNAMIC_MINOR,	/*动态分配次设备号*/
	.name = "LED",			/*设备名 在/dev/可见*/
	.fops = &fops, 			/*操作函数集*/
};

static void led_exit(void)
{
	int i;

	for(i = 0; i < ARRAY_SIZE(ledgpio); i++){
		/*设置gpio端口为1*/
		gpio_set_value(ledgpio[i], OFF);
		/*释放之前声明申请的gpio*/
		gpio_free(ledgpio[i]);
	}	
}

static int __init exynos4412_led_init(void)
{
	int ret;
	int i;

	for(i = 0; i < ARRAY_SIZE(ledgpio); i++){
		/*申请分配gpio*/
		ret = gpio_request(ledgpio[i], "led");
		if(ret < 0){
			goto error0;
		}
	}
	
	led_init();
	
	ret = misc_register(&miscdev);

	if(ret < 0){
		led_exit();
		return ret;
	}

	return	0;

error0:
	while(i--){
		gpio_free(ledgpio[i]);
	}

	return ret;
}

module_init(exynos4412_led_init);


static void __exit exynos4412_led_exit(void)
{
	misc_deregister(&miscdev);
	led_exit();
} 

module_exit(exynos4412_led_exit);

MODULE_LICENSE("GPL");

MODULE_AUTHOR("Songze Lee");
MODULE_VERSION("verson 1.0");
MODULE_DESCRIPTION("misc for led driver");

```

应用层调试程序，Makefile同上。

```
[projct /]# insmod led.ko
[projct /]# ./led_test /dev/led stat
led 1 is off
led 2 is off
led 3 is off
led 4 is off
[projct /]# ./led_test /dev/led 2 on
[projct /]# ./led_test /dev/led 4 on
[projct /]# ./led_test /dev/led stat
led 1 is off
led 2 is on
led 3 is off
led 4 is on
[projct /]# rmmod led
[ 3520.355000] goodbye! led  driver
rmmod: module 'led' not found

```




