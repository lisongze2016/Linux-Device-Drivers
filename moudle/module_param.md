### 1.1.4 模块参数
在用户态下编程可以通过main()来传递命令行参数，而编写一个内核模块则可通过module_param()来传递命令行参数。
module_param宏是Linux 2.6内核中新增的,该宏被定义在include/linux/
moduleparam.h文件中，具体定义如下：
```
#define module_param(name, type, perm)
module_param_named(name, name, type, perm)
```
由此可知 module_param的实现是通过module_param_named(name, name, type, perm)的。
module_param(name, type, perm)
name：参数名 type：参数类型 perm：参数读写权限
参数类型可以是byte、short、ushort、int、uint、long、ulong、charp（字符指针）、bool或invbool（布尔的反），在模块被编译时会将module_param中声明的类型与变量定义的类型进行比较、判断是否一致。
```
module_param_array(name,type,num,perm)
```
name：数组名 type：数组类型 num：数组元素个数指针 perm:参数读写权限
perm参数的作用是什么？
最后的 module_param 字段是一个权限值,表示此参数在sysfs文件系统中所对应的文件节点的属性。你应当使用 <linux/stat.h> 中定义的值. 这个值控制谁可以存取这些模块参数在 sysfs 中的表示.
当perm为0时，表示此参数不存在 sysfs文件系统下对应的文件节点。 否则, 模块被加载后，在/sys/module/ 目录下将出现以此模块名命名的目录, 带有给定的权限。
权限在include/linux/stat.h中有定义
比如：
```
	#define S_IRWXU 00700
	#define S_IRUSR 00400
	#define S_IWUSR 00200
	#define S_IXUSR 00100
	#define S_IRWXG 00070
	#define S_IRGRP 00040
	#define S_IWGRP 00020
	#define S_IXGRP 00010
	#define S_IRWXO 00007
	#define S_IROTH 00004
	#define S_IWOTH 00002
	#define S_IXOTH 00001
```

使用 S_IRUGO 参数可以被所有人读取, 但是不能改变; S_IRUGO|S_IWUSR 允许 root 来改变参数.注意, 如果一个参数被 sysfs 修改, 你的模块看到的参数值也改变了, 但是你的模块没有任何其他的通知. 你应当不要使模块参数可写, 除非你准备好检测这个改变并且因而作出反应. 
示例A：module_param(name, type, perm)


```
/**
 *module_param(name, type, perm) 内核模块传参
 *向当前模块传参
 */

#include <linux/init.h>
#include <linux/module.h>

static int x_rel = 480, y_rel = 272;

module_param(x_rel, int, 0);
module_param(y_rel, int, 0);

static char *info = "mdg: lol?";

static int num=10;
module_param(num,int,S_IRUGO);
module_param(info, charp, 0);

static int __init demo_init(void)
{
	printk("hello,word! driver module is inserted!\n");

	printk("x: %d, y: %d\n %s\n", x_rel, y_rel, info);
	printk("num=%d\n",num);
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
MODULE_VERSION("version 1.0");
MODULE_DESCRIPTION("It is a simple demo for driver module");
```
调试结果：
```
[projct /]# insmod demo.ko num=110 x_rel=123 y_rel=345 info="hello world"
[ 4140.560000] hello,word! driver module is inserted!
[ 4140.560000] x: 123, y: 345
[ 4140.560000]  hello world
[ 4140.560000] num=110
[projct /]# cat /sys/module/demo/parameters/num
110
[projct /]# ls -l /sys/module/demo/parameters/num
-r--r--r--    1 0        0             4096 Jan  1 14:06 /sys/module/demo/parameters/num
```
可发现num为只读文件

示例B：module_param_array(name,type,num,perm)
```
#include <linux/init.h>
#include <linux/module.h>

#define CNT 16
static int num = CNT;
static int array[CNT] = {1,2,3,4,5,6,7};
module_param_array(array, int, &num, 0);

static int __init demo_init(void)
{
	int i;
	printk("Insert module ok!\n");
	for(i = 0; i < num; i++){
		printk("array[%d] = %d\n", i, array[i]);
	}
	return 0;
}
module_init(demo_init);

static void __exit demo_exit(void)
{
	printk("goodbye, word! driver is removed!\n");
}

module_exit(demo_exit);
MODULE_LICENSE("GPL");
MODULE_AUTHOR("Songze Lee ");
MODULE_VERSION("version 1.0");
MODULE_DESCRIPTION("It is a simple demo for driver module");
```
调试结果：
```
[projct /]# insmod demo.ko array=11,22,33,44,55,66,77,88,99
[ 4965.605000] Insert module ok!
[ 4965.605000] array[0] = 11
[ 4965.605000] array[1] = 22
[ 4965.605000] array[2] = 33
[ 4965.605000] array[3] = 44
[ 4965.605000] array[4] = 55
[ 4965.605000] array[5] = 66
[ 4965.605000] array[6] = 77
[ 4965.605000] array[7] = 88
[ 4965.605000] array[8] = 99
```


