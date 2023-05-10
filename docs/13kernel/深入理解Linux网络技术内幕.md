## 《深入理解Linux网络技术内幕》

+ P1、基础背景（1-3）
+ P2、系统初始化（4-8）
+ P3、传输和接收（9-13）

### chap1、简介

#### 1.1、基本术语

+ RX（接收），TX（传输）

#### 1.2、常见编码模式

+ 内存缓存
  + kmalloc和kfree
  + skb_init()
  + kmem_cache_create/kmem_cache_destroy
  + kmem_cache_alloc/kmem_cache_free
+ 缓存和hash表
+ 引用计数
  + xxx_hold/xxx_release
  + xxx_put有时也是释放
+ 垃圾收集
  + 异步
  + 同步
+ 函数指针和虚拟函数表（VFT）
  + **指向一个数据结构的一组函数指针**通常就被称为VFT
+ goto语句
+ 向量定义
+ 条件指示指令（#ifdef及其系列指令）
+ 条件检查的编译期间最优化
  + likely或unlikely宏
+ 互斥
  + 回转锁（Spin lock）
  + 读-写回转锁
  + 读取（拷贝）更新（Read-Copy-Update,RCU）
+ 主机和网络之间的尾端转换
  + 小尾端（*小弟弟*），**最低字节存储在最低内存地址**
+ 捕获bug
+ 统计数据
+ 测量时间

#### 1.3、用户空间工具

+ iputils
+ net-tools
+ IPROUTE2

#### 1.4、浏览源代码

+ cscope
+ **死代码**（已经不在使用了）

#### 1.5、当功能以补丁形式提供时

### chap2、关键数据结构

#### 2.1、套接字缓冲区：sk_buff结构

##### 0

##### 2.1.1、网络选项以及内核结构

+ `struct sk_buff{}`

##### 2.1.2、布局字段

+ `struct sk_buff_head{}`，**哑元元素**
+ sk_buff其它感兴趣的字段
  + struct sock *sk
  + unsigned int len
  + unsigned int data_len
  + unsigned int mac_len
  + atomic_t users
  + unsigned int truesize
  + unsigned int *head
  + unsigned char *end
  + unsigned char *data
  + unsigned char *tail
  + void (*destructor)(...)

##### 2.1.3、通用字段

+ **即与内核无关**

##### 2.1.4、功能专用字段

##### 2.1.5、管理函数

+ 0
+ 分配内存：alloc_skb和dev_alloc_skb
  + net/core/skbuff.c中
+ 释放内存：kfree_skb和dev_kfree_skb
+ 数据预留及对齐：skb_reserve, skb_put,skb_push以及skb_pull
+ skb_shared_info结构和skb_shinfo函数
+ 缓冲区的克隆和拷贝
+ 列表管理函数

#### 2.2、net_device结构

##### 0

##### 2.2.1、标识符

+ int ifindex  **dev_new_index注册时分派**
+ int iflink  **（虚拟）隧道设备**
+ unsigned short dev_id

##### 2.2.2、配置

+ 0
+ 接口类型与端口

##### 2.2.3、统计数据

##### 2.2.4、设备状态

##### 2.2.5、列表管理

##### 2.2.6、链路层多播

##### 2.2.7、流量管理

##### 2.2.8、功能专用

##### 2.2.9、通用

##### 2.2.10、函数指针（或VFT）

#### 2.3、本章涉及的文件

+ include/linux
  + if.h
  + if_packet.h
  + skbuff.h
  + netdevice.h
  + slab.h
  + if_arp.h
+ kernel
  + time.c
  + dma.c
+ mm
  + slab.c
+ drivers
  + 3c59x.c
  + tulip.c
  + sys9000.c
+ net/core
  + skbuff.c

### chap3、用户空间与内核的接口

#### 3.1、概论

+ 把内部信息传输给用户空间，除了**系统调用**外，还有3种
  + procfs **虚拟文件系统**，/proc文件系统
  + sysctl /proc/sys目录
  + sysfs  /sys文件系统

#### 3.2、procfs与sysctl

##### 3.2.1、procfs

##### 3.2.2、sysctl：目录/proc/sys

+ 0
+ ctl_table初始化的实例
+ 在/proc/sys中注册文件
+ 核心网络文件和目录

#### 3.3、ioctl

#### 3.4、Netlink

#### 3.5、配置改变串行化