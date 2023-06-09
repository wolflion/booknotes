## 《深入理解Linux内核》

### chap3、进程

#### 3.1、进程、轻量级进程、线程

#### 3.2、进程描述符

##### 0、

##### 3.2.1、进程状态

+ state字段

##### 3.2.2、标识一个进程

+ 0
  + pid字段，**pidmap-array位图来表示当前已分配的PID号和闲置的PID号**
+ 3.2.2.1、进程描述符处理
+ 3.2.2.2、标识当前进程
+ 3.2.2.3、双向链表
+ 3.2.2.4、进程链表
+ 3.2.2.5、TASK_RUNNING状态的进程链表

##### 3.2.3、进程间的关系

+ 0
+ 3.2.3.1、pidhash表及链表

##### 3.2.4、如何组织进程

+ 0
+ 3.2.4.1、等待队列
+ 3.2.4.2、等待队列的操作

##### 3.2.5、进程资源限制

+ `current->signal->rlim`字段

#### 3.3、进程切换

##### 3.3.1、硬件上下文

##### 3.3.2、任务状态段

+ 0
+ 3.3.2.1、thread字段

##### 3.3.3、执行进程切换

##### 3.3.4、保存和加载FPU、MMU及XMM寄存器

+ 0
+ 3.3.4.1、保存FPU寄存器
+ 3.3.4.2、装载FPU寄存器

#### 3.4、创建进程

##### 0、

+ *lionel，OS是怎么样的？*（shell创建一个新进程，新进程执行shell的另一个拷贝）

+ 传统的Unix操作系统

+ 现代Unix的引入三种机制
  + 写时复制技术，允许父子进程读相同的物理页。**两者任一试图写一个物理页，内核就把这个页的内容拷贝到一个新的物理页**
  + 轻量级进程
  + vfork()创建的进程能共享父进程的内存地址空间

##### 3.4.1、clone()、fork()及vfork()系统调用

+ 0
  + 轻量级进程是由名为clone()的函数创建的
  + fork()在Linux中是用clone()实现的
  + vfork()在Linux中也是用clone()实现的
+ 3.4.1.1、do_fork()函数
  + **主要步骤**
+ 3.4.1.2、copy_process()函数
  + **主要步骤**

##### 3.4.2、内核线程

+ 0
  + **内核线程不受不必要的用户态上下文的拖累**
+ 3.4.2.1、创建一个内核线程
  + `kernel_thread()`
+ 3.4.2.2、进程0
  + **所有进程的祖先**
+ 3.4.2.3、进程1
  + init()调用execve()系统调用装入可执行程序init。结果，**init内核线程变为一个普通进程**
+ 3.4.2.4、其他内核线程

#### 3.5、撤销进程

##### 3.5.1、进程终止

+ 3.5.1.1、do_group_exit()函数
+ 3.5.1.2、do_exit()函数

##### 3.5.2、进程删除

### chap4、中断和异常

4.1、中断信号的作用

### chap18、ext2和ext3文件系统

#### 18.1、Ext2的一般特征

+ 下列特点有助于Ext2的效率
  + 当创建Ext2文件系统时，系统管理员可以根据预期的文件平均长度来选择最佳块大小（从1024B~4096B）
+ Ext2还包含一些特点：
  + 在启动时支持对文件系统的状态进行自动的一致性检查。**检查是由外部程序e2fsck完成的**
+ 考虑引入另外几个特性
  + 块片（block fragmentation）
  + 透明地处理压缩和加密文件
  + 逻辑删除
  + 日志

#### 18.2、Ext2磁盘数据结构

##### 0、

+ 块组中的每个块包含下列信息之一：

  + 文件系统的超级块的一个拷贝

  + 一组块组描述符的拷贝

  + 一个数据块位图

  + 一个索引节点位图

  + 一个索引节表

  + 属于文件的一大块数据，即数据块

##### 18.2.1、超级块

##### 18.2.2、组描述符和位图

+ `ext2_group_desc`结构

##### 18.2.3、索引节点表

+ `bg_inode_table`字段中

+ `ext2_inode`结构

##### 18.2.4、索引节点的增强属性（extended attribute）

##### 18.2.5、访问控制列表

##### 18.2.6、各种文件类型如何使用磁盘块

+ 普通文件

+ 目录

+ 符号链接

+ 设备文件、管道和套接字

#### 18.3、Ext2的内存数据结构

##### 0、

##### 18.3.1、Ext2的超级块对象

##### 18.3.2、Ext2的索引节点对象

#### 18.4、创建Ext2文件系统

+ Ext2文件系统是由`mke2fs`创建的
+ 执行下列操作
+ 命令,`dd if=/dev/fd0`

#### 18.5、Ext2的方法

##### 18.5.1、Ext2超级块的操作

##### 18.5.2、Ext2索引节点的操作

##### 18.5.3、Ext2的文件操作

#### 18.6、管理Ext2磁盘空间

##### 0、

##### 18.6.1、创建索引节点

+ `ext2_new_inode()`

##### 18.6.2、删除索引节点

+ `ext2_free_inode()`

##### 18.6.3、数据块寻址

##### 18.6.4、文件的洞

##### 18.6.5、分配数据块

+ `ext2_get_block()`

##### 18.6.6、释放数据块

+ `ext2_free_blocks()`

#### 18.7、Ext3文件系统

0、

##### 18.7.1、日志文件系统

+ Ext2文件系统的状态存放在磁盘上超级块的s_mount_state字段中。

##### 18.7.2、Ext3日志文件系统

+ 当从系统故障中恢复时，e2fsck程序区分下列两种情况：

  + 提交到日志之前系统故障发生

  + 提交到日志之后系统故障发生

##### 18.7.3、日志块设备层

+ 0

  + Ext3日志通常存放名为`.journal`的隐藏文件中。该文件位于文件系统的根目录。
  + **Ext3文件系统本身不处理日志，而是利用所谓日志块设备（Journaling Block Device，JBD）的通用内核层**。
  + **JBD也必须保护自己免受任何系统故障引起的日志损坏**
  + Ext3与JDB之间的交互本质上基于三个基本单元
    + 日志记录
    + 原子操作处理
    + 事务

+ 日志记录

+ 原子操作处理

+ 事务

##### 18.7.4、日志如何工作

+ 1、write()系统调用，是由generic_file_write()实现的

+ 2、几次调用address_space对象的prepare_write方法，对于Ext3来说，是由ext3_prepare_write()实现的

+ 3、ext3_prepare_write()调用journal_start()

+ 4、ext3_prepare_write()调用block_prepare_write()，传递给它的参数为ext3_get_block()函数的地址

+ 5、当内核必须确定Ext3文件系统的逻辑块号时，就执行ext3_get_block()函数

+ 6、如果Ext3文件系统已经以“日志”模式安装，则ext3_prepare_write()函数在写操作触及的每个缓冲区上也调用journal_get_write_access()

+ 7、

+ 8、

+ 9、

+ 10、

+ 11、内核周期性地为日志中每个完成的事务激活检查活动。检查点主要验证由journal_commit_transaction()触发的I/O数据传送是否已经成功结束。