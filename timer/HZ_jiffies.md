
## 5.1 HZ, jiffies

HZ:定时器频率（节拍率），即为1s定时器产生的中断个数为HZ个。

jiffies：全局变量jiffies用来记录自系统启动以来产生的节拍个数（定时器中断个数），1s内增加HZ个，系统运行时间以s为单位即为jiffies/HZ。jiffies定义在linux/jiffies.h文件中，extern unsigned long volatile jiffies;

在代码中经常需要设置一些将来的时间：

```c
unsigned long time_stamp = jiffies;	//现在
unsigned long next_tick = jiffies+1;	//从现在开始1个节拍
unsigned long later = jiffies+5*HZ;	//从现在开始5s
unsigned long fraction = jiffies+HZ/10;	//从现在开始1/10s
```
linux 内核提供四个宏帮助比较节拍计数，它们能正确的处理节拍计数回绕问题。

```c
#define time_after(a,b)		\			//a>b返回真
	(typecheck(unsigned long, a) && \
	 typecheck(unsigned long, b) && \
	 ((long)(b) - (long)(a) < 0))
#define time_before(a,b)	time_after(b,a)		//a<b返回真

#define time_after_eq(a,b)	\
	(typecheck(unsigned long, a) && \
	 typecheck(unsigned long, b) && \
	 ((long)(a) - (long)(b) >= 0))
#define time_before_eq(a,b)	time_after_eq(b,a)
```

delay延时函数

```c
void delay（int sec）
{
	unsigned long end_ticker = jiffies+sec*HZ;
	while(timer_before(jiffies,end_ticker))//超过sec返回假，退出
		schedule();
}
```
