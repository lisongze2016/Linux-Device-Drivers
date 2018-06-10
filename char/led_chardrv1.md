### 2.2.1 LED字符设备驱动示例

Led驱动：

```

/**
 *4LED 1亮 0灭
 *早期经典的字符设备注册方法
 *register_chrdev
 *write read
 *
 */

#include <linux/init.h>
#include <linux/module.h>
#include <linux/fs.h>
#include <linux/io.h>
#include <linux/uaccess.h>
#include <linux/slab.h>

#define LED_MAJOR 185

#define GPM4BASE 0x11000000
#define GPM4CON  0x2e0
#define GPM4DAT  0x2e4

static char *VGPM4BASE = NULL;

#define VGPM4CON *((u32 *)(VGPM4BASE+GPM4CON))
#define VGPM4DAT *((u32 *)(VGPM4BASE+GPM4DAT))


static void led_init(void)
{
	VGPM4CON &= ~0xffff;
	VGPM4CON |= 0x1111;/*设置输出引脚*/

	VGPM4DAT |= 0x0f;/*led all off*/
}


static void led_stat(char *buf)
{
	u32 tmp;
	int i;
	
	tmp = VGPM4DAT;
	
	for(i = 0; i < 4; i++){
	
		if(tmp & (1<<i)){
			buf[i] = 0;
		}else{
			buf[i] = 1;
		}
	}
}

static void led_ctrl(int num, int stat)
{
	
	if(stat){
		VGPM4DAT &= ~(1 << (num - 1));
	}else{
		VGPM4DAT |= (1 << (num - 1));
	}
}

static int
exynos4412_led_open(struct inode *inodp, struct file *filp)
{
	return 0;
}

static ssize_t 
exynos4412_led_read(struct file *filp, char __user *buf, size_t cnt, loff_t *fpos)
{
	char stat[4] = {0};
	
	led_stat(stat);	

	if(copy_to_user(buf,stat,sizeof(stat)))
	{
		return -EINVAL;
	}
	return sizeof(stat);
}

static ssize_t
exynos4412_led_write(struct file *filp, const char __user *buf, size_t cnt, loff_t *fpos)
{
	u8 cmd;
	u8 num,stat;
	if(copy_from_user(&cmd,buf,cnt)){//get_user(cmd,buf);
		return -EINVAL;
	}

	num = cmd >> 4;stat = cmd & 0xf;//cmd 高4位放led编号 低4位放led状态

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
exynos4412_led_release(struct inode *inodp, struct file *filp)
{
	return 0;
}


/**字符驱动的核心
 *当应用程序操作设备文件时所调用的open read write等函数
 *最终会调用这个结构中对应的函数
 */
static struct file_operations exynos4412_led_fops = {
	.owner   = THIS_MODULE,/*内核使用这个字段以避免在模块的操作正在被使用时卸载该模块*/
	.open    = exynos4412_led_open,
	.read    = exynos4412_led_read,
	.write   = exynos4412_led_write,
	.release = exynos4412_led_release/*当file结构被释放时，将调用这个操作*/
};

static int __init exynos4412_led_init(void)
{
	VGPM4BASE = ioremap(GPM4BASE,SZ_4K);
	if(NULL == VGPM4BASE){
		return -ENOMEM;
	}
	led_init();
	return register_chrdev(LED_MAJOR,"LED",&exynos4412_led_fops);
}
module_init(exynos4412_led_init);

static void __exit exynos4412_led_exit(void)
{
	unregister_chrdev(LED_MAJOR,"LED");
	printk("goodbye! led  driver\n");
}
module_exit(exynos4412_led_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Songze Lee");
MODULE_VERSION("verson 1.0");
MODULE_DESCRIPTION("char driver for led");

```

Makefile:

```
obj-m	:=led.o

OUR_KERNEL :=/home/lisongze/tiny4412/Tiny4412_kernel/

all:
	make -C $(OUR_KERNEL) M=$(shell pwd) modules
clean:
	make -C $(OUR_KERNEL) M=`pwd` clean

```


应用测试程序：
```
#include <sys/types.h>
#include <stdlib.h>
#include <string.h>
#include <stdio.h>
#include <sys/stat.h>
#include <fcntl.h>

void pri_usage(char *info)
{
	printf("-------- usage: ------\n");
	printf("usage1: %s stat\n", info);
	printf("usage2: %s num on/off\n", info);
	printf("           num: <1~4>\n");
}

//a.out led stat
//./a.out led 1 on
int main(int argc, char **argv)
{
	int i;
	int ret;
	int num;
	char buf[4];
	int fd;

	unsigned char cmd = 0;

	if((argc !=  3) && (argc != 4)){
		pri_usage(argv[0]);
		exit(1);
	}

	fd = open(argv[1], O_RDWR | O_NONBLOCK);
	if(fd < 0){
		perror("open");
		exit(1);
	}

	if(argc == 3){
		if(!strncmp("stat", argv[2], 4)){
			ret = read(fd, buf, sizeof(buf));	
			if(ret != 4){
				perror("read");
				exit(1);
			}
			for(i = 0; i < sizeof(buf); i++){
				printf("led %d is %s\n", \
						i+1, buf[i]?"on":"off");
			}
		}else{
			pri_usage(argv[0]);
			exit(1);
		}
	}else{
		num = atoi(argv[2]);
		if(num >= 1 && num <= 4){
			cmd &= ~(0xf << 4);
			cmd = num << 4;
		}else{
			pri_usage(argv[0]);
			exit(1);
		}

		if(!strncmp("on", argv[3], 2)){
			cmd &= ~0xf;
			cmd |= 1;
		}else if(!strncmp("off", argv[3], 3)){
			cmd &= ~0xf;
		}else{
			pri_usage(argv[0]);
			exit(1);
		}
		ret = write(fd, &cmd,sizeof(cmd));
		if(ret != sizeof(cmd)){
			pri_usage(argv[0]);
			exit(1);
		}
	}

	return 0;
}

```

调试结果如下：

```
------------------------------PC机---------------------------------
[root@localhost led1]# arm-linux-gcc led_app.c -o led_test
[root@localhost led1]# make
make -C /ARM/linux-3.5-songze/ M=/ARM/linux-3.5-songze/drivers/
songze_drivers/char/led/led1 modules
make[1]: Entering directory `/ARM/linux-3.5-songze'
  Building modules, stage 2.
  MODPOST 1 modules
make[1]: Leaving directory `/ARM/linux-3.5-songze'
[root@localhost led1]# cp led_test /nfsroot
[root@localhost led1]# cp led.ko /nfsroot
--------------------------------ARM---------------------------------
[projct /]# insmod led.ko 
[projct /]# cat /proc/devices 
Character devices:
  1 mem
  2 pty
  3 ttyp
  4 /dev/vc/0
  4 tty
  4 ttyS
  5 /dev/tty
  5 /dev/console
  5 /dev/ptmx
  7 vcs
 10 misc
 13 input
 14 sound
 21 sg
 29 fb
 81 video4linux
 89 i2c
108 ppp
116 alsa
128 ptm
136 pts
180 usb
185 LED
188 ttyUSB
189 usb_device
204 ttySAC
216 rfcomm
251 ttyGS
252 watchdog
253 media
254 rtc
[projct /]# mknod led c 185 0
[projct /]# ls led -l
crw-r--r--    1 0        0         185,   0 Jan  5  2016 led
[projct /]# ./led_test 
-------- usage: ------
usage1: ./led_test stat
usage2: ./led_test num on/off
           num: <1~4>
[projct /]# ./led_test led stat
led 1 is off
led 2 is off
led 3 is off
led 4 is off
[projct /]# ./led_test led 1 on
[projct /]# ./led_test led 3 on
[projct /]# ./led_test led stat
led 1 is on
led 2 is off
led 3 is on
led 4 is off
[projct /]# rmmod led
[ 3322.195000] goodbye! led  driver
rmmod: module 'led' not found

```










