### 1.1 对驱动模块进行操作的shell命令

insmod demo.ko 插入驱动模块
lsmod           查看插入正在运行驱动模块
rmmod demo      移除驱动模块
modinfo demo.ko 查看驱动模块信息
dmesg           查看内核缓存信息
dmesg -c        清除内核缓存信息
有依赖关系的
modprobe (智能安装） xx（不加.ko）
modprobe -r 移除(注意:移除需要依赖的模块，同时会移除被依赖的模块）

