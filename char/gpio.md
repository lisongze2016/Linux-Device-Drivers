2.3.4 内核中的gpio API

在Documentation/gpio.txt中有内核中gpio的使用详细说明，下面先介绍比较常用的函数，如下：

- 使用 GPIO

对于一个GPIO，系统应该做的第一件事情就是通过 gpio_request() 函数声明申请分配他

```
int gpio_request(unsigned gpio, const char *label)
```

- 设置GPIO的方向

```
/*设置为输入或输出, 成功返回 0 失败返回负的错误代码*/
int gpio_direction_input(unsigned gpio);
int gpio_direction_output(unsigned gpio, int value);
```

- 访问自旋锁安全的GPIO

```
/* GPIO 读取：返回零或非零 */
int gpio_get_value(unsigned gpio);
	返回值是布尔值，零表示低电平，非零表示高电平
/* GPIO 设置 */
void gpio_set_value(unsigned gpio, int value);
```

- 释放之前声明的GPIO

```
void gpio_free(unsigned gpio)；
```


