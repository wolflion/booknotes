## 《Linux内核探秘》

### chap2、文件系统   12（24/232）

#### 2.1、文件系统的基本概念

##### 2.1.1、什么是VFS

+ **VFS本身只存在于内存中**，它需要将硬盘上的文件系统抽象到内存中，通过几个**重要的结构**（`dentry、inode、super_block`）

##### 2.1.2、超级块super_block

+ 代表了**整个文件系统本身**，
  + 给出了文件系统的全局信息
  + 包含一些函数指针

##### 2.1.3、目录项dentry

+ 目录项反映了文件系统的这种树状关系

##### 2.1.4、索引节点inode

+ **inode代表一个文件**

##### 2.1.5、文件

+ 文件对象的作用是**描述进程和文件交互的关系**。 【**文件对象**指的是内存中的文件】
+ 进程打开一个文件，内核就动态创建一个文件对象。同一个文件，在不同的进程中有不同的文件对象。

#### 2.2、文件系统的架构

##### 2.2.1、超级块作用分析

##### 2.2.2、dentry作用分析

##### 2.2.3、inode作用分析

+ 系统内核提供了一个hash链表数组`inode_hashtable`
+ inode还有一个重要用作用是**缓存文件的数据内容**，通过成员`i_mapping`实现的。

##### 2.2.4、文件作用分析

#### 2.3、从代码层次深入分析文件系统

##### 2.3.1、一个最简单的文件系统aufs

```cpp
#include <linux/moudle.h>
#include <linux/fs.h>
#include <linux/pagemap.h>
#include <linux/mount.h>
#include <linut/init.h>
#include <linux/nami.h>

#define AUFS_MAIGC 0x64668735
static struct vfsmount* aufs_mount;
static int aufs_mount_count;

static struct inode* aufs_get_inode(struct super_block* sb, int mode, dev_t dev)
{
	struct inode* inode = new_inode(sb);
	if (inode) {
		inode->i_mode = mode;
		inode->i_uid = current->fsuid;
		inode->i_gid = current->fsgid;
		inode->i_blksize = PAGE_CACHE_SIZE;
		inode->i_blocks = 0;
		inode->i_atime = inode->i_mtime = inode->i_ctime = CURRENT_TIME;
		switch (mode & S_IFMT) {
		default:
			int_special_inode(inode, mode, dev);
			break;
		case S_IFREG:
			printk("create a file \n");
			break;
		case S_IFDIR:
			inode->i_op = &simple_dir_inode_operations;
			inode->i_fop = &simple_dir_operations;
			printk("create a dir file \n");

			inode->i_nlink++;
			break;
		}
	}
	return inode;
}

//SMP-safe
static int aufs_mknod(struct inode* dir, struct dentry* dentry, int mode, dev_t dev)
{
	struct inode* inode;
	int error = -EPERM;

	if (dentry->d_inode)
		return -EEXIST;

	inode = aufs_get_inode(dir->i_sb, mode, dev);
	if (inode) {
		d_instantiate(dentry, inode);
		dget(dentry);
		error = 0;
	}
	return error;
}

static int aufs_mkdir(struct inode* dir, struct dentry* dentry, int mode)
{
	int res;
	res = aufs_mknode(dir, dentry, mode | S_IFDIR, 0);
	if (!res)
		dir->i_nlink++;
	return res;
}

static int aufs_create(struct inode* dir, struct dentry* dentry, int mode)
{
	return aufs_mknode(dir, entry, mode | S_IFREG, 0);
}

static int aufs_fill_super(struct super_block* sb, void* data, int silent)
{
	static struct tree_descr debug_files[] = { {""} };
	return simple_fill_super(sb, AUFS_MAIGC, debug_files);//lionel，也是系统提供的啊，还是fs目录下libfs.c中的
}

static struct super_block* aufs_get_sb(struct file_system_type* fs_type,
	int flags, const char* dev_name, void* data)
{
	return get_sb_single(fs_type, flags, data, aufs_fill_super);
}

static struct file_system_type au_fs_type = {
	.owner = THIS_MODULE,
.name = "aufs",
.get_sb = aufs_get_sb,
.kill_sb = kill_litter_super,
};

static int aufs_create_by_name(const char* name, mode_t mode,
	struct dentry* parent,
	struct dentry** dentry)
{
	int error = 0;
	/* If the parent is not specified, we create it in the root.
	* We need the root dentry to do this, which is in the super
	* block. A pointer to that is in the struct vfsmount that we
	* have around.
	*/
	if (!parent) {
		if (aufs_mount && aufs_mount->mnt_sb) {
			parent = aufs_mount->mnt_sb->s_root;
		}
	}
	if (!parent) {
		printk("Ah! can not find a parent!\n");
		return -EFAULT;
	}

	*dentry = NULL;
	mutex_lock(&parent->d_inode->i_mutex);
	&dentry = lookup_one_len(name, parent, strlen(name));
	if (!IS_ERR(dentry)) {
		if ((mode & S_IFMT) == S_IFDIR)
			error = aufs_mkdir(parent->d_inode, *dentry, mode);
		else
			error = aufs_create(parent->d_inode, *dentry, mode);
	}
	else
		error = PTR_ERR(dentry);
	mutex_unlock(&parent->d_inode->i_mutex);
	return error;
}

struct dentry* aufs_create_file(const char* name, mode_t mode,
	struct dentry* parent, void* data,
	struct file_operations* fops)
{
	struct dentry* dentry = NULL;
	int error;
	printk("aufs: creating file '%s'\n", name);
	error = aufs_create_by_name(name, mode, parent, &dentry);
	if (error) {
		dentry = NULL;
		goto exit;
	}
	if (dentry->d_inode) {
		if (data)
			dentry->d_inode->u.generic_ip = data;
		if (fops)
			dentry->d_inode->i_fop = fops;
	}
exit:
	return dentry;
}

struct dentry* aufs_create_dir(const char* name, struct dentry* parent)
{
	return aufs_create_file(name, S_IFDIR | S_IRWXU | S_IRUGO | S_IXUGO, parent, NULL, NULL);
}

static int __init aufs_init(void)
{
	int retval;
	struct dentry* pslot;

	retval = register_filesystem(&au_fs_type);

	if (!retval) {
		aufs_mount = kern_mount(&au_fs_type);  //lionel，kern_mount()【这个在super.c中】与do_mount()【这个在fs里的namespace.c中】有关联不？
		if (IS_ERROR(aufs_mount)) {
			printk(KERN_ERR "aufs: could not mount!\n");
			unregister_filesystem(&au_fs_type);
			return retval;
		}
	}

	pslot = aufs_create_dir("woman star", NULL);
	aufs_create_file("lbb", S_IFREG | S_IRUGO, pslot, NULL, NULL);
	aufs_create_file("fbb", S_IFREG | S_IRUGO, pslot, NULL, NULL);
	aufs_create_file("ljl", S_IFREG | S_IRUGO, pslot, NULL, NULL);

	pslot = aufs_create_dir("man star", NULL);
	aufs_create_file("ldh", S_IFREG | S_IRUGO, pslot, NULL, NULL);
	aufs_create_file("lcw", S_IFREG | S_IRUGO, pslot, NULL, NULL);
	aufs_create_file("jw", S_IFREG | S_IRUGO, pslot, NULL, NULL);

	return retval;
}

static void __exit aufs_exit(void)
{
	simple_release_fs(&aufs_mount, &aufs_mount_count);
	unregister_filesystem(&au_fs_type);
}

module_init(aufs_init);
module_exit(aufs_exit);
MODULE_LICENSE("GPL");
MOUDLE_DESCRIPTION("This is a simple module");
MODULE_VERSION("Ver 0.1");


/*
* mkdir au
* mount -t aufs none /au
* ls
*/

```

##### 2.3.2、文件系统如何管理目录和文件

+ 代码三部分
  + 1）`register_filesystem`函数，把aufs文件系统登记到系统
  + 2）调用`kern_mount`函数为文件系统申请必备的数据结构
  + 3）最后在aufs文件系统内创建两目录，每个目录下面创建3个文件

##### 2.3.3、文件系统的挂载过程

##### 2.3.4、文件打开的代码分析

#### 2.4、本章小结

### chap10、文件系统读写   143（155/232）

#### 10.1、page cache机制

+ 其存储的数据在I/O完成后并不回收，而是一直保留在内存中，除非内存紧张，才开始回收。

##### 10.1.1、buffer I/O和direct I/O

+ buffer I/O，是内核缓存
+ direct I/O，是应用提供的内存
+ **应用提供的`read`和`write`接口，是同步I/O接口**。

##### 10.1.2、buffer head和块缓存

+ `struct buffer_head{}`

##### 10.1.3、page cache的管理

+ **通过数据结构address_space管理page cache**
+ `add_to_page_cache()`，插入页面到page cache

##### 10.1.4、page cache的状态

+ page cache的状态：

  + PG_dirty：
+ 块缓存
  + BH_Mapped

#### 10.2、文件预读

+ readhead.c中`struct backing_dev_info{}`

#### 10.3、文件锁

+ 实现机制不同：
  + 建议锁
  + 强制锁
+ 访问方式不同：
  + 读锁（共享锁）
  + 写锁（互斥锁）

#### 10.4、文件读过程代码分析

+ `sys_read()`
+ read_write.c中的`sys_read()`
  + 根据fd，得到file结构；取得文件当前位置
+ read_write.c中的`vfs_read()`
  + 校验权限；看是否有自定义的read
+ 1.generic_file_read函数
  + `vfs_read()`->filemap.c中的`generic_file_read()`    **解决文件同步和异步操作问题**，`kiocb`结构
  + `kiocb`结构定义在include/aio.h中，实现在aio.c中
+ 2.`__generic_file_aio_read`函数
  + file_map.c中，函数比较长，分3部分
  	+ 第一部分：校验
  	+ 第二部分：处理direct I/O
  	+ 第三部分：使用了读描述符结构desc
+ 3.do_generic_file_read函数
  + file_map.c中，**内核提供的通用读函数**
+ 4.do_generic_mapping_read函数
  + file_map.c中，计算文件读操作涉及的页面参数，**比较复杂**，分7个部分：
  	+ 第一部分：
  	+ 第三部分：处理页面状态是最新的情况
  	+ 第七部分：更新文件的位置，修改文件的预读状态。
  + **从硬盘读数据是通过文件系统提供的readpage函数实现的**。
  + ext2文件系统提供的读页面函数是 inode.c中的`ext2_readpage()`，直接调用`mpage_readpage()`**内核提供的一个通用函数**，它调用`do_mpage_readpage`将读请求转换为一个bio结构，如果bio有效，则提交bio给底层去执行读操作。
+ **文件系统的读写请求，最终要转换成对块设备的读写请求**，这涉及几个问题：
	+ 文件对用户呈现了一个连续的读写接口，但是文件在真正物理设备硬盘上的存储可能并不是连续的，如果是不连续的，对文件的读写就不能用同一个I/O完成，而是需要拆分。
	+ 内核通过`submit_bio`来提交一个I/O给底层。同时内核又提供了一个函数`submit_bh`来提交块。submit_bh最终也是通过submit_bio来实现，**它只是多了将块地址转换为硬盘物理扇区地址的过程**。
+ 5.do_mpage_readpage函数
  + 也分了几个部分
  	+ 第六部分：confused分支代码
+ 6.block_read_full_page函数
  + 前面与do_mpage_readpage前面部分流程类似
  + arr数组，保存了页面内的每一个文件块的物理块号

#### 10.5、读过程返回

+ **文件系统通过mpage_bio_submit提交一个I/O**。这个I/O什么时候返回？通过什么机制通知上层？这涉及内核I/O过程的阻塞点设计。
+ mpage.c中的`mpage_bio_submit()`，调用`mpage_bio_read()`【在I/O完成后的中断过程中调用的】
+ mpage.c中的`mpage_end_io_read()`要遍历bio结构的每个向量，看相关页面是否获得最新的数据。

#### 10.6、文件写过程代码分析

+ `sys_write()`
+ 1.generic_file_wirte函数
  + filemap.c中的，buffer I/O读写对象是page cache，**当用户希望真正写到硬盘时，文件就需要设置SYNC标志**，调用`sync_page_range`把page cache的页面写入硬盘。
+ 2.generic_file_buffered_write函数
	+ `__generic_file_io_write_nolock()`
	+ 分为4部分
		+ 第四部分：检查文件是否具有SYNC标志
+ 3.获得文件块的物理块号
	+ `prepare_write()`和`commit_write()`功能，对于ext2而言就是`ext2_prepare_write()`和`generic_commit_write()`
	+ `ext2_prepare_write()`真正实现的是buffer.c中的`__block_preapre_write()`
		+ 第一部分
		+ 第二部分
		+ 第三部分：检查前面的处理过程是否产生了读请求
	+ `generic_commit_write()`作用是，逐个遍历所有文件块，检查块缓存是否最新。

#### 10.7、本章小结

+ 涉及文件中一些复杂参数的计算和page cache中页面缓存的状态处理。

### chap11、通用块层和scsi层   170（182/232）

+ *lionel，这里面会涉及到一些I/O管理？*

+ 上接文件系统的VFS层，下接硬盘驱动。**处理I/O的合并或者排序**

#### 11.1、块设备队列

##### 11.1.1、scsi块设备队列处理函数

+ `scsi_alloc_queue()`
+ 为scsi块设备创建队列结构时调用。
+ 默认为scsi设备提供了出队列函数`scsi_request_fn()`和软中断完成函数`scsi_softirq_done()`

##### 11.1.2、电梯算法和对象

+ `struct elevator_type{}`
  
#### 11.2、硬盘HBA抽象层

+ `struct scsi_host_template{}`

+ `static struct elevator_type elevator_noop = {};`

#### 11.3、I/O的顺序控制

#### 11.4、I/O调度算法

##### 11.4.1、noop调度算法

+ `noop_add_request()`

+ `noop_dispatch()`

+ noop，基本什么都没做，等同于一个先进先出的调度算法，本身没对I/O进行真正的排序。入队列函数决定了I/O是如何插入电梯队列的，**加入noop的链表**。

+ I/O调度算法的出队列函数决定了I/O是如何被挑选，然后由硬盘驱动执行。
	
	+ **I/O从块设备队列到驱动执行的过程，是由scsi层控制的**
	
##### 11.4.2、deadline调度算法

+ `struct deadline_data{}`

+ `deadline_add_request()`

+ 内部启用了一个红黑树结构来对I/O进行排序，保证出队列时，**已经按照扇区地址排好顺序了**。block/deadline-iosched.c中的

+ `deadline_data`结构有2个队列，**一个是红黑树sort_list，另一个是链表fifo_list**（读写请求分别保存在不同的队列中）

+ 出队列函数是`deadline_dispatch_requests()`
	+ 第一部分，判断是否连续的批模式
	+ 第二部分，检查读写请求的饥饿时间
	+ 第三部分，挑选一个出队列的I/O
	+ 最后，重新开启一轮循环，`deadline_find_first_drq()`

#### 11.5、I/O的处理过程

##### 11.5.1、I/O插入队列的过程分析

+ 1、submit_bio函数
	+ 是文件系统VFS层和通用块层的衔接点，调用generic_make_request函数
+ 2、generic_make_request函数
	+ 把入参的bio结构转换为底层处理的请求结构
		+ 第一部分，检查最大扇区限制
		+ 第二部分，检查I/O的大小不应该超过块设备的最大可处理扇区
+ 3、`__generic_make_request`函数
	+ 第一部分，从bio结构获得I/O的起始扇区地址和以扇区度量的I/O大小
	+ 第二部分，首先检查是否需要bounce，其次检查I/O是否barrier I/O
	+ 第三部分，处理后向合并
	+ 第四部分，处理前向合并
	+ 第五部分，申请请求
+ 4、elv_merge函数
	+ 判断两个I/O能否合并，通过两步：
		+ 第一步，判断能否和块设备队列的最后一个请求合并
		+ 第二步，调用电梯队列提供的函数判断能否合并
+ 5、`__elv_add_request`函数
	+ 对于不能合并的I/O，需要把一个请求插入到队列。
+ 6、elv_insert函数
	+ 第一部分，处理前向插入
	+ 第二部分，处理后向插入
	+ 第三部分，处理按顺序插入和重插
	+ 第四部分，检查队列的限制

##### 11.5.2、I/O出队列的过程分析

+ 0
	+ I/O从队列出来真正下发到硬盘？以下几个条件
		+ 第一个I/O启动了3毫秒的定时器，时间到了，会执行unplug函数，开始下发I/O
		+ 请求数目超过设定的限制（默认是4），执行unplug函数，开始下发
		+ 带有sync标志的I/O，立即执行unplug函数，开始下发
		+ barrier I/O，需要先清空电梯队列，然后执行unplug函数，开始下发
		+ 当硬盘执行完毕一个I/O，也要unplug队列，检查是否有I/O可以执行
+ 1、unplug函数
	+ 先检查队列是否设置了stop标志，其次清除队列的plug标志，然后调用队列的request_fn函数
+ 2、scsi_request_fn函数
	+ 第一部分，调用`elv_next_request()`从块设备队列获得一个请求
	+ 第二部分，检查各种异常情况
	+ 第三部分，初始化保存scsi错误返回值的数据buffer，然后调用`scsi_dispatch_cmd()`将I/O下发到驱动
	+ 第四部分，两个处理错误分支
+ 3、elv_next_request函数
	+ 从块设备队列获得一个I/O
	+ 第一部分，调用`__elv_next_request()`从队列中获得一个I/O请求
	+ 第二部分，调用队列的`prep_rq_fn()`来预处理I/O
+ 4、blk_do_ordered函数
	+ 检查I/O的顺序，**为了处理barrier I/O**
	+ 第一部分，
	+ 第二部分，处理特殊情况
+ 5、start_ordered函数
	+ 根据要求设置同步cache命令，保证同步cache命令之前的I/O必须完成
	+ 第一部分，设置I/O请求的必要参数
	+ 第二部分，设置同步cache的命令
+ 6、scsi_dispatch_cmd函数

##### 11.5.3、I/O返回路径

+ 0
	+ `scsi_done()`是I/O返回路径上的第一个函数
	+ `blk_complete_request()`
	+ `scsi_softirq_done()`
+ **`scsi_decide_disposition()`值得仔细研究，总结了所有的scsi错误类型**，并给出了处理措施
+ 1、sd_rw_intr函数
	+ 对scsi命令的各种错误进行处理
	+ 第一部分，如果scsi命令的返回结果result不为0，说明命令执行中产生了错误。
	+ 第二部分，检查scsi命令的sense key
+ 2、scsi_io_completion函数
	+ 第一部分，
	+ 第二部分
	+ 第三部分
+ 3、scsi_end_request函数
+ **bio设置的回调函数是`mpage_end_io_read()`**.

#### 11.6、本章小结

+ **通用块层、I/O调度算法和scsi层，共同构建了文件系统之下的I/O处理流程**。

### chap12、内核回写机制   204（216/232）

#### 12.1、内核的触发条件

+ 文件系统的写函数`generic_file_buffered_write()`要调用`balance_dirty_pages_ratelimited()`
+ 内核定时器触发
+ 当系统申请内存失败，或者文件系统执行同步（sync）操作，或者内存管理模块试图释放更多内存的时候，都可能触发**pdflush线程回写页面**。

#### 12.2、内核回写控制参数

#### 12.3、定时器触发回写

+ `start_kernel()`启动了一个定时器

##### 12.3.1、启动定时器

+ 调用`page_writeback_init()`函数启动定时器

##### 12.3.2、执行回写操作

+ `wb_kupdate()`

##### 12.3.3、检查需要回写的页面

+ `writeback_inodes()`扫描系统的超级块，检查需要回写的页面

##### 12.3.4、回写超级块内的inode

+ `sync_sb_inodes()`

##### 12.3.4.1、

##### 12.3.4.2、

##### 12.3.4.3、

##### 12.3.4.4、

#### 12.4、平衡写

+ `balance_dirty_pages_ratelimited()`检查脏页面是否到达限值，是否进行平衡写操作

+ `balance_dirty_pages()`

##### 12.4.1、检查直接回写的条件

+ `balance_dirty_pages_ratelimited_nr()`

##### 12.4.2、回写系统脏页面的条件

##### 12.4.3、检查计算机模式

#### 12.5、本章小结

+ **Linux内核的回写机制提供了一种缓写机制，真实的写并不是直接写入硬盘，而是在page cache中缓存，等待合适的时机才真正执行写入**。

### chap13、一个真实文件系统ext2   217（229/232）

#### 13.1、ext2的硬盘布局

#### 13.2、ext2文件系统目录树

+ `struct ext2_dir_entry_2{};`
+ **根目录是一个固定的inode，它的inode号是2**。

#### 13.3、ext2文件内容管理

+ ext2的inode信息可以存放15个块的地址，用户数据就存在这些块中。
+ 15块中前12块是直接数据块，**第13块是一级索引块，第14块是二级索引块，第15块是三级索引块**

#### 13.4、ext2文件系统读写

+ 通过文件系统的目录树，可以获得文件所在目录的上级目录。
+ 读上级目录的内容，就可以获得文件的inode号和文件名称。
+ 根据文件的inode号，可以获得文件的inode信息，也就获得了文件的类型、创建修改时间、文件大小等信息，完成打开文件的过程。

#### 13.5、本章小结