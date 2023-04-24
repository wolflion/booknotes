## 《深入Linux设备驱动程序内核机制》

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

+ include/linux/cdev.h中的`struct cdev{};`
+ fs/char_dev.c中的`cdev_alloc()`

#### 2.4、设备号的构成与分配

##### 2.4.1、设备号的构成

+ include/linux/tyes.h中的`typedef __u32 __kernel_dev_t`

##### 2.4.2、设备号的分配与管理

+ fs/char_dev.c中的`register_chrdev_region()`，`alloc_chrdev_region()`

#### 2.5、字符设备的注册

+ fs/char_dev.c中的`cdev_add()`
+ drivers/base/map.c中的`struct kobj_map{};`

#### 2.6、设备文件节点的生成

+ **用户空间的应用程序要想使用驱动程序提供的服务，需要经过设备文件来达成**。
+ fs/node.c中的`init_special_inode();`
+ fs/block_dev.c中的`def_blk_fops={};`

#### 2.7、字符设备文件的打开操作

+ 图2-8，sys_open到chrdev_open调用流程
+ fs/file_table.c中的`fput()`

#### 2.8、本章小结

### chap3、分配内存  87（102/538）
#### 3.1、物理内存的管理
##### 3.1.1、内存节点node
+ UMA模型
+ NUMA模型（Non-Uniform Memory Access）
##### 3.1.2、内存区域zone
+ include/linux/mmzone.h中的`enum zone_type{};`
##### 3.1.3、内存页
+ struct page对象。
#### 3.2、页面分配器（page allocator）
##### 0、
+ 图3-2、**mem_map、物理页面及虚地址空间关系**
##### 3.2.1、gfp_mask
+ include/linux/gfp.h
##### 3.2.2、alloc_pages
+ include/linux/gfp.h中的`alloc_pages()`
##### 3.2.3、`_get_free_pages`
+ mm/page_alloc.c中`__get_free_pages()`
##### 3.2.4、get_zeroed_page
+ mm/page_alloc.c中`get_zeroed_page()`
##### 3.2.5、`_get_dma_pages`
#### 3.3、slab分配器（slab allocator）
##### 3.3.1、管理slab的数据结构
+ **struct kmem_cache和struct slab**是slab分配器的基石
+ include/linux/slab_def.h中的`struct kmem_cache{};`
+ mm/slab.c中的`struct slab{};`
+ include/linux/slab_def.h中的`struct cache_sizes{};`
+ mm/slab.c中的`kernel_cache_init()`
##### 3.3.2、kmalloc与kzalloc
+ kmalloc分配出来的内存空间在物理上是连续的，**不负责把分配出的内存空间中的内容清零**，建立在**slab分配器基础之上**
+ mm/slab.c中的`cache_grow()`
+ include/linux/gfp.h中的
##### 3.3.3、kmem_cache_create与kmem_cache_alloc
+ mm/slab.c中的`__cache_shrink()`
#### 3.4、内存池
#### 3.5、虚拟内存的管理
+ **内核代码中用`PAGE_OFFSET`宏来标示虚拟地址空间中内核部分的起始地址**。
##### 3.5.1、内核虚拟地址空间构成
+ 内核会将1GB的内核大体上分为三部分：
	+ 第一部分位于1GB空间的开头，用于对系统物理内存的直接映射（线性映射），内核用全局变量high_memory来表示这段空间的上界
	+ 第二部分，主要用于vmalloc函数
	+ 第三部分，位于1GB空间的结尾部分
##### 3.5.2、vmalloc与vfree
+ vmalloc特点是分配的虚拟地址空间是连续的，但这段虚拟地址空间所映射的物理地址可能是不连续的。
+ **驱动程序中并不鼓励使用vmalloc函数**：
	+ 首先，vmalloc的实现机制决定了它的使用效率没有kmalloc这样的函数高
+ include/linux/vmalloc.h中的`struct vm_struct{};`
+ mm/vmalloc.c中的`map_vm_area()`
##### 3.5.3、ioremap
#### 3.6、per-CPU变量
##### 0、
##### 3.6.1、静态per-CPU变量的声明与定义
+ include/linux/percpu_defs.h
##### 3.6.2、静态per-CPU变量的链接脚本
+ kernel/vmlinux.lds
##### 3.6.3、setup_per_cpu_areas函数
##### 3.6.4、使用per-CPU变量
+ mm/percpu.c中的`__per_cpu_offset()`
#### 3.7、本章小结
+ 基于伙伴系统的页面分配器
+ 基于slab分配器的kmalloc函数和kmem_cache_alloc函数
+ vmalloc函数
+ 用在多处理器系统中，即per-CPU内存分配器

### chap7、设备文件的高级操作  231（246/538）
#### 7.1、ioctl文件操作
+ ioctl一般用来在用户空间的应用程序和驱动程序模块之间传递控制参数。
##### 7.1.1、ioctl的系统调用
+ include/linux/fs.h中的`struct file_operations{};`
+ fs/ioctl.c中有`sys_ioctl()`，`do_vfs_ioctl()`，`vfs_ioctl()`
+ **大内核锁，虽然还在2.6版本的内核中，但应该避免使用**
##### 7.1.2、ioctl的命令编码
##### 7.1.3、copy_from_user和copy_to_user
+ include/asm-generic/uaccess.h中的`copy_from_user()`
	+ arch/x86/lib/usercopy_32.c中的`__copy_user_zeroing()`
#### 7.2、字符设备的I/O模型
+ 4种
#### 7.3、同步阻塞型I/O
##### 7.3.1、wait_event_interruptible
+ include/linux/wait.h中的
##### 7.3.2、wait_up_interruptible
+ include/linux/wait.h中的
+ kernel/sched.c中的`__wake_up()`，`__wake_up_common()`，`try_to_wake_up()`
#### 7.4、同步非阻塞型I/O
+ `filp->f_flags`的`O_NONBLOCK`位
#### 7.5、异步阻塞型I/O
+ **在设备驱动程序中如何实现poll方法**
+ include/asm/poll.h中的`struct pollfd{};`，`poll_wait()`
+ fs/select.c中的`__pollwait()`，`pollwake()`
#### 7.6、异步非阻塞型I/O
+ **实现file_operations对象中的`aio_read()`和`aio_write()`**
+ include/linux/aio.h中
#### 7.7、驱动程序的fsync例程
+ 字符设备没有实现，**块设备，使用通用的`block_fsync()`作为fsync例程的实现
#### 7.8、fasync例程
+ fs/fcntl.c中的`fasync_helper()`，`kill_fasync()`
+ *lionel，这部分没太看懂*
#### 7.9、llseek例程
+ fs/read-write.c中的`lseek()`，`vfs_llseek()`
+ fs/open.c中的`nonseekable_open()`
#### 7.10、访问权能
+ kernel/capabiltiy.c中的`capable()`
+ include/linux/security.h中的`security_capable()`
+ security/commoncap.c中的`cap_capable()`
#### 7.11、本章小结
+ 字符型设备驱动程序最常用的是ioctl例程。
+ fasync实现一个异步通知机制，**主要依赖内核的fasync_helper和kill_fasync**，前者将需要通知的进程加入一个链表，后者在应用程序关注的事件发生时通过信号发送的方式来通知应用程序。

### chap11、块设备驱动程序  407（422/538）
#### 11.1、块子系统初始化
#### 11.2、ramdisk源码实例
##### 11.2.1、make_request版本的RAM DISK源码
##### 11.2.2、request版本的RAM DISK源码
##### 11.2.3、ramdisk的使用
#### 11.3、块设备号的注册与管理
+ block/genhd.c中的`register_blkdev()`
#### 11.4、block_device
+ include/linux/fs.h中的`struct block_device{};`
#### 11.5、struct gendisk
+ include/linux/genhd.h中的`struct gendisk{};`
#### 11.6、struct hd_struct
+ include/linux/genhd.h中的`struct hd_struct{};`
#### 11.7、用alloc_disk分配gendisk对象
#### 11.8、向系统添加个块设备add_disk
+ fs/partitions/check.c中的`register_disk()`
+ fs/block_dev.c中的`bdget()`
#### 11.9、block_device_operations
+ include/linux/blkdev.h中的`block_device_operations{};`
#### 11.10、块设备文件的打开
+ fs/inode.c中的`init_special_inode()`
+ fs/block_dev.c中的`def_blk_fops=`
#### 11.11、blk_init_queue
+ block/blk-core.c
+ include/linux/blkdev.h中的`struct request_queue{};`
+ include/linux/blkdev.h中的`struct request{};`
#### 11.12、blk_queue_make_request
+ block/blk-settings.c中的`blk_queue_make_request()`
#### 11.13、向队列提交请求
+ block/blk-core.c中的`submit_bio()`，`generic_make_request()`
+ drivers/block/brd.c中的`ramdisk`
#### 11.14、块设备的请求处理函数
#### 11.15、bio结构
+ include/linux/bio.h中的`struct bio{};`，`struct bio_vec{};`

#### 11.16、本章小结