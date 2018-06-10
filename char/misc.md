### 2.3 混杂设备

Misc驱动是一些拥有着共同特性的简单字符设备驱动。内核抽象出这些特性而形成了一些API（在文件drivers/char/misc.c中实现），以简化这些设备驱动程序的初始化。所有misc设备被分配同一个主设备号MISC_MAJOR(10),但每次可以选择一个单独的次设备号。如果一个字符设备驱动要驱动多个设备，那么它就不应给用misc设备来实现。