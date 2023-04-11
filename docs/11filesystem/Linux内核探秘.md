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