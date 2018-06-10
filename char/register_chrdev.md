## 2.2 早期注册字符设备驱动的经典方式


int register_chrdev(unsigned int major, const char *name,
			const struct file_operations *fops)
- major:设备的主设备号   0 内核会自动分配主设备号
- name：驱动程序的名词(出现在/proc/devices)
- fops：file_operations结构

对register_chrdev的调用将为给定的主设备号注册0-255作为次设备号，并为每个设备建立一个对应默认cdev结构。

移除设备函数

void unregister_chrdev(unsigned int major, const char *name)

- major和name必须与传递给register_chrdev函数的值保持一致，否则该调用会失败。


