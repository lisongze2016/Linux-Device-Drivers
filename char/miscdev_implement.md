#### 2.3.3 混杂设备实现机制

misc_init()通过class_create(THIS_MODULE, "misc")在/sys/class/目录下创建一个名为misc,调用register_chrdev(MISC_MAJOR,"misc",&misc_fops)注册设备，其中设备的主设备号为MISC_MAJOR，为10。设备名为misc，misc_fops是操作函数的集合。通过subsys_initcall(misc_init)函数将misc作为一个子系统被注册到linux内核中。

misc_register（&miscdev）中先初始化misc_list链表，遍历misc_list链表，看这个次设备号是否已被使用如果次设备号已被占有则退出。然后 判断misc->minor == MISC_DYNAMIC_MINOR，如果相等，系统动态分配次设备号，MKDEV(MISC_MAJOR, misc->minor)计算出设备号，device_create(misc_class, misc->parent,dev, NULL, "%s", misc->name)在/dev下创建设备节点，list_add(&misc->list, &misc_list)将这个miscdevice添加到misc_list链中。(初始化和遍历misc_list链表->匹配次设备号->找到一个没有占用的次设备号(如果需要动态分配的话)->计算设备号->创建设备节点->miscdevice结构体添加到misc_list链表中。) misc_deregister list_del(&misc->list)在misc_list链表中删除miscdevice设备,device_destroy(misc_class, MKDEV(MISC_MAJOR,misc->min
or))删除设备节点(从mist_list中删miscdevice->删除设备文件。)
