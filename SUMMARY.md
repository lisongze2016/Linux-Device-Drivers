# Summary

* [简介](README.md)
* [Linux 驱动模块](moudle/README.md)
  * [驱动模块框架](moudle/moudle.md)
    * [对驱动模块进行操作的shell命令](moudle/moudle_shell.md)
    * [一个最简单的linux内核模块代码 hello world](moudle/hello_world.md)
    * [导出符号 EXPORT_SYMBOL](moudle/export_symbol.md)
    * [模块参数 module_param](moudle/module_param.md)
    * [内核调试打印 prink](moudle/printk.md)
    * [Kconfig和Makefile](moudle/Kconfig_Makefile.md)
* [Linux 字符设备](char/char_dev.md)
    * [早期注册字符设备驱动的经典方式](char/register_chrdev.md)
        * [LED字符设备驱动示例](char/led_chardrv1.md)
        * [自动创建设备节点](char/class_create.md)
    * [注册设备和释放设备号](char/register_chrdev_region.md)
        * [LED字符设备驱动示例](char/led_chardrv2.md)
    * [混杂设备](char/misc.md)
        * [miscdevice 结构体](char/struct_miscdevice.md)
        * [字符设备驱动和混杂设备驱动的使用区别](char/misc_char_usage_diff.md)
        * [混杂设备实现机制](char/miscdev_implement.md)
        * [内核中的gpio口操作](char/gpio.md)
        * [混杂设备led驱动示例](char/led_misc.md)
* [Linux 内核数据结构](data_structure/README.md)
  * [内核链表](data_structure/list.md)
  * [环形buffer](data_structure/ringbuffer.md)
* [Linux 并发和竞争](linux-bing-fa-he-jing-zheng.md)
  * 原子操作
  * 自旋锁
  * 信号量
  * 互斥体
  * 完成量
* Linux 时间管理
  * HZ、jiffies
  * timer
* Platform 平台总线
* Led 子系统
* Input 子系统
* proc、sysfs文件系统
* i2c
* spi

