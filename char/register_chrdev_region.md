### 2.2 注册设备和释放设备号

静态注册设备：

```
int register_chrdev_region(dev_t from, unsigned count, const char *name)
```

- from：要分配设备编号范围的起始值
- count：连续设备编号的个数
- name：设备名称

动态注册设备：

```
int alloc_chrdev_region(dev_t *dev, unsigned baseminor, unsigned count,
				const char *name)

```
- dev:仅用于输出的参数，在成功完成调用后将保存已分配范围的第一个编号
- baseminor:要使用的被请求的第一个次设备号
- count:连续设备编号的个数
- name:设备名称

注销设备

```
void unregister_chrdev_region(dev_t from, unsigned count)
```

- from：要分配设备编号范围的起始值
- count：连续设备编号的个数
