## 《深入Linux内核架构》

### chap8、虚拟文件系统

#### 8.4、处理VFS对象

##### 8.4.1、文件系统操作

###### 8.4.1.1、注册文件系统

+ fs.h中用于描述文件系统的结构`struct file_system_type {};`

###### 8.4.1.2、装载与卸载

+ 共享子树
+ 伪文件系统

##### 8.4.2、文件操作

###### 8.4.2.1、查找inode

+ do_lookup的实现
+ do_follow_link的实现

###### 8.4.2.2、打开文件

###### 8.4.2.3、读取和写入

+ 0
  + fs/read_write.c
+ read
+ write

#### 8.5、标准函数

8.5.1、

8.5.2、失效机制

8.5.3、权限检查

#### 8.6、小结

### chap9、Ext文件系统族

#### 9.2、Ext文件系统

##### 9.2.1、物理结构

+ 1、结构概观
  + 一些语义
    + 超级块是用于存储文件系统自身元数据的核心结构。
+ 2、间接
  + 数据块的长度是4KiB
  + 使用间接，在inode中仅需要耗费少量字节存储块号，刚好够用来表示平均意义上长度较小的文件。
  + 文件系统在硬盘上分配一个数据块，不存储文件数据，专门用于存储块号。**该块称为一次间接块（single indirect block）**，可以容纳数百个块号。
+ 3、碎片

##### 9.2.2、数据结构

+ 1、超级块
  + **超级块的数据使用ext2_read_super例程读取（fs/ext2/super.c），内核通常借助file_system_type结构中的read_super函数指针来调用该函数**。
  + ext2_fs.h中的`struct ext2_super_block{};`
  + 感兴趣的成员
    + s_log_block_size：
+ 2、组描述符
  + ext2_fs.h中`struct ext2_group_desc{};`
+ 3、inode
  + ext2_fs.h中的`struct ext2_inode{};`
+ 4、目录和文件
+ 5、内存中的数据结构
  + ext2_fs.h中的`struct ext2_sb_info{};`

##### 9.2.3、创建文件系统

+ 文件系统并非由内核自身创建，是由`mke2fs`用户空间工具创建的。

##### 9.2.4、文件系统操作

+ 0
  + 用于操作文件内容的操作保存在file_operations中
  + 用于此类文件对象自身的操作保存在inode_operations中
  + 用于一般地址空间的操作保存在address_space_operations中
  + fs/ext2/file.c中的`ext2_file_operations`定义，**大多数项指向了VFS的标准函数**
  + 目录文件，fs/ext2/dir.c中的`ext2_dir_operations`定义
  + 普通文件，fs/ext2/file.c中的 `ext2_file_inode_operations`定义
  + 目录有更多可用的inode操作，fs/ext2/namei.c中的`ext2_dir_inode_operations`
  + **文件系统和块层通过`address_space_operations`关联**，fs/ext2/inode.c中的`ext2_aops`定义
  + **`super_operations`与超级块交互**，fs/ext2/super.c中的`ext2_sops`定义
  + **后面只是讨论了几个重要的函数**，没有全部说，*lionel，我可以通过这种思路，扩展一下，其它函数的学习*
+ 1、装载和卸载
  + 结构的定义，fs/ext2/super.c中的`struct file_sytem_type ext2_fs_type`定义
  + mount系统调用通过get_sb来读取文件系统超级块的内容，`get_sb_bdev()`，fs/ext2/super.c中的`ext2_get_sb()`
  + `ext2_fill_super()`的一个函数指针作为参数传递给`get_sb_bdev`，该函数用数据填充一个超级块对象。
  + **文件系统的元信息应该一直驻留在内存中，并由ext2_sb_info数据结构保存**
  + 逐块读取组描述符，并使用`ext2_check_descriptors`检查一致性
  + 填充超级块信息的最后一步是`ext2_count_free_blocks、ext2_count_free_inodes、ext2_count_dirs`分别计算空闲块数目、空闲inode数目和目录数目。
  + 控制权转移到ext2_setup_super，进行几项最后的检查并输出适当的警告信息
  + 最后一步是ext2_write_super，它将超级块的内容写回到底层的存储介质
+ 2、读取并产生数据块和间接块
  + `ext2_get_blocks()`流程 【*lionel，这里有个勘误*】
+ 3、创建和删除inode
  + fs/ext2/namei.c中的`ext2_mkdir()`
+ 4、注册inode
+ 5、删除inode
+ 6、删除数据块
+ 7、地址空间操作

#### 9.3、Ext3文件系统

##### 9.3.1、概念

##### 9.3.2、数据结构

+ ext3_fs_sb.h中`struct ext3_sb_info{};`新增了几个日志
  + `struct inode *s_journal_inode; struct journal_s *s_journal;`

### chap12、网络

### chap13、系统调用

#### 13.1、系统程序设计基础

##### 13.1.1、追踪系统调用

##### 13.1.2、支持的标准

##### 13.1.3、重启系统调用

#### 13.2、可用的系统调用

#### 13.3、系统调用的实现

##### 13.3.1、系统调用的结构

##### 13.3.2、访问用户空间

##### 13.3.3、追踪系统调用

#### 13.4、小结

### 附录

#### 附录A、体系结构相关知识

#### 附录B、使用源代码

##### B.1、内核源代码的组织

#### 附录E、ELF二进制格式

##### E.1、布局和结构