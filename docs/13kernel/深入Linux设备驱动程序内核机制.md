### chap1、内核模块

#### 0、

+ 内核模块可以在系统运行期间动态扩展系统功能而无须重新启动系统。
+ `dmesg`看下内核报啥错

#### 1.1、内核模块的文件格式

+ `file demodev.ko`，**ELF（Executable Linkable Format）文件格式**
+ *linux源码中定义了ELF格式？我要找一下在哪？*

#### 1.2、EXPORT_SYMBOL的内核实现

+ *了解一下，这个宏定义的细节？*

#### 1.3、模块的加载过程

##### 0、

+ 当调用“insmod demodev.ko”来安装demodev.ko这样的内核模块时，insmod会利用文件系统的接口将其数据读取到用户空间的一段内存中，然后通过系统调用sys_init_module让内核去处理模块加载的整个过程。

##### 1.3.1、sys_init_module（第一部分）

+ 原型是：`long sys_init_module(void _user *umod, unsigned len, const char _user *uargs)`
  + umod：是指向用户空间demodev.ko文件映射数据的内存地址
  + len：该文件的数据大小
  + uargs：是传给模块的参数在用户空间下的内存地址
+ 真实是调用了kernel/module.c中的load_module()，`static struct module* load_module(void _user *umod, unsigned len, const char _user *uargs)`，参数同上

##### 1.3.2、struct module

+ include/linux/module.h

##### 1.3.3、load_module（**本节重点**）

+ *自己没看*

##### 1.3.4、sys_init_module（第二部分）

##### 1.3.5、模块的卸载

+ 原型`long sys_delete_module(const char _user* name_user, unsigned int flags);`

#### 1.4、本章小结

+ mod utils的工具包
+ 内核引入了版本控制机制，不同内核版本加载.ko文件的问题

### chap2、字符设备驱动程序

#### 2.1、应用程序与设备驱动程序互动实例

#### 2.2、struct file_operations

+ include/linux/fs.h
+ file_operations的成员变量几乎全是函数指针，**字符设备驱动程序的编写，基本是围绕这些函数指针成员而展开的**
+ 非函数指针是`struct module *owner`

#### 2.3、字符设备的内核抽象

#### 2.4、设备号的构成与分配

##### 2.4.1、设备号的构成

##### 2.4.2、设备号的分配与管理

#### 2.5、字符设备的注册

#### 2.6、设备文件节点的生成

#### 2.7、字符设备文件的打开操作

#### 2.8、本章小结