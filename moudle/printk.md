### 1.1.5内核调试打印 prink

#### printk的打印等级

内核提供的打印函数printk()和C库提供的printf()函数功能几乎相同, printk()就是内核的格式化打印函数。printk()在任何时候、任何地方都能调用它，内核中的printk()到处都是，它可以在中断上下文调用，也可以在进程上下中调用，它可以在持有锁时调用，也可以在多处理器上同时调用，而且调用者连锁都不必使用。总之，它是一个弹性极佳的函数，而这一点尤其重要，printk()之所以这么有用，就在于它随时都在那里而且随时都能用（不过，在系统启动过程中，终端还没有初始化之前，在某些地方是不能使用它的，内核中还提供了一种early_printk的方式来解决这个问题）。

printk()和printf()一个最主要的区别就是前者可以指定一个记录级别。内核根据这个级别来判断是否在终端上打印消息。内核把级别比某个特定值低的所有消息显示在终端上。

可以通过下面这种方式指定一个记录级别：
```
printk(KERN_WARNING “This is a warning!\n”);
printk(KERN_DEBUG “This is a debug notice!\n”);
printk(“I did not specify a loglevel!\n”);
```
KERN_WARING和KERN_DEBUG都是<linux/kernel.h>中的简单宏定义。它们扩展开是像“<4>”或“<7>”这样的字符串，加进printk()函数要打印的消息的开头。内核用这个指定的记录等级和当前终端的记录等级console_loglevel来决定是不是向终端上打印。使用printk()函数中的日志级别为的是使编程人员在编程过程中自定义地进行信息的输出，更加容易地掌握系统当前的状况。对程序的调试起到了很重要的作用。
在<linux/kernel.h>头文件里一共定义了8个级别（0-7）的输出，从高到低分别由如下常量表示：
```
KERN_EMERG "<0>"：    最高级别，一般只用来打印崩溃信息；
KERN_ALERT "<1>"：    需要立即处理的信息；
KERN_CRIT"<2>"：      关键信息，一般用来显示严重的硬件和软件错误；
KERN_ERR"<3>"：       用来显示硬件错误；
KERN_WARNING "<4>"：  显示不会造成严重错误的警告信息；
KERN_NOTICE"<5>"：    显示需要引起注意的信息；
KERN_INFO"<6>"：      显示一般信息，例如驱动所发现的硬件列表；
KERN_DEBUG"<7>"：     用来显示调试信息。
```
printk函数的字符串参数中使用了KERN_ALERT，它实际上扩展为字符串：“<1>”，而这部分信息没有输出到终端。实际上，这部分是内核信息的日志级别，只有超过了当前日志级别的信息才会输出到终端。当前内核的日志级别可以在/proc/sys/kernel/printk文件中看到。这个文件包含了四个整数，其中前两个是控制台的当前日志级别和默认日志级别。我们在printk的参数中使用较高的日志级别就是要保证信息得到输出。
通过读写/proc/sys/kernel/printk文件可读取和修改控制台的日志级别。
```
$ cat /proc/sys/kernel/printk
6   4   1   7
```
上面显示的4个数据分别对应控制台日志级别、默认的消息日志级别、最低的控制台日志级别和默认的控制台日志级别。
可用下面的命令设置当前日志级别：
```
$ echo 8 > /proc/sys/kernel/printk
$ cat  /proc/sys/kernel/printk
8   4   1   7
```
这样所有级别8,(0-7)的消息都可以显示在控制台上。
通常在产品开发阶段，会把系统默认等级设置到最低，以便在开发测试阶段可以暴露出更多问题和调试信息，在产品发布时再把打印级别设置为0或4。
在实际调试中，可把函数名称（func）和代码行号(LINE)加上方便定位发现问题。

```
printk("%s %s is enter!\n",__func__,__LINE__);
```
示例myprintk.c：
```
#include <linux/init.h>
#include <linux/module.h>

static int hellokernel_init(void)
{
    printk("<0>" "level 0!\n");
    printk("<1>" "level 1!\n");
    printk("<2>" "level 2!\n");
    printk("<3>" "level 3!\n");
    printk("<4>" "level 4!\n");
    printk("<5>" "level 5!\n");
    printk("<6>" "level 6!\n");
    printk("<7>" "level 7!\n");
    return 0;
}
static void hellokernel_exit(void)
{
    printk("<0>" "level 0!\n");
    printk("<1>" "level 1!\n");
    printk("<2>" "level 2!\n");
    printk("<3>" "level 3!\n");
    printk("<4>" "level 4!\n");
    printk("<5>" "level 5!\n");
    printk("<6>" "level 6!\n");
    printk("<7>" "level 7!\n");
}
module_init(hellokernel_init);
module_exit(hellokernel_exit);
MODULE_LICENSE("GPL");
```
加快开机启动速度 uboot bootargs 参数中可设置。
```
quiet=		[KNL] Disable log messages
debug=		[KNL] Enable kernel debugging (events log level).
loglevel= 	All Kernel Messages with a loglevel smaller than the
console loglevel will be printed to the console. It can also be changed with klogd or other programs.
The loglevels are defined as follows:
0 (KERN_EMERG)          system is unusable
1 (KERN_ALERT)          action must be taken immediately
2 (KERN_CRIT)           critical conditions
3 (KERN_ERR)            error conditions
4 (KERN_WARNING)        warning conditions
5 (KERN_NOTICE)         normal but significant condition
6 (KERN_INFO)           informational
7 (KERN_DEBUG)          debug-level messages
```

#### printk的记录缓冲区

内核消息都被保存在一个LOG_BUF_LEN大小的环形队列中。该缓冲区大小可以在编译时通过CONFIG_LOG_BUF_SHIFT进行调整。在单处理器的系统上其默认值是16K字节。换句话说，就是内核在同一时间只能保存16K字节的内核消息。如果它已经放满而又接收到了新的消息，旧消息就会被覆盖。这个记录缓冲区之所以称为环形是因为它的读写都是按照环形队列方式进行操作的。

使用环形队列有许多好处。由于读写环形队列时同步问题非常容易解决，所以即使在中断上下文中也可以方便的使用printk()。此外，它使记录维护起来也更容易。如果有大量的消息同时产生，新消息只需覆盖掉旧消息即可。在某个问题引发大量消息的时候，记录只会覆盖掉它本身，而不会因为失控而消耗掉大量内存。而环形缓冲区的唯一缺点——可能会丢失消息——带来的损失和在简单性和健壮性上的好处相比，完全可以忽略不计。

#### printk 打印格式

printk打印格式在使用时，需要注意格式符，否则在编译时会出现很多警告。

| 数据类型 | printk格式符 |
| :----: | :----:|
| int | %d或%x |
| unsigned int | %u或%x |
| long | %ld或%lx |
| long long | %lld或%llx |
| unsigned long long | %llu或%llx |
| size_t | %zu或%zx |
| ssize_t | %zd或%zx |
| 指针 | %p |
| 函数指针 | %pf |