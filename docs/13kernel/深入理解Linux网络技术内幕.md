## 《深入理解Linux网络技术内幕》

+ *是不是看看2.2或2.4的代码内核*

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

### chap4、通知链

#### 4.1、使用通知链的原因

4.2、概论

4.3、定义链

4.4、链注册

4.5、链上的通知事件

4.6、网络子系统的通知链

4.7、通过/proc文件系统的调整

4.8、本章涉及的函数和变量

4.9、本章涉及的文件和目录

### chap5、网络设备初始化

### chap9、中断和网络驱动程序

+ *lionel，本章讲的是，处理L2层的帧的函数，是由中断事件驱动的*

#### 9.1、决策与流量方向

+ 网络协议栈所走向路径，会根据，封包会分为几种情况，**接收的、传输的、转发的**，而不同。
+ 虚拟设备呢，*lionel想一下*
+ 图9.2怎么看懂，*lionel*

#### 9.2、接收到帧时通知驱动程序

+ 设备与内核使用两种技术来交换数据，**轮询和中断**

##### 9.2.1、轮询

##### 9.2.2、中断

##### 9.2.3、在中断期间处理多帧

+ **当驱动程序运行时中断功能会关闭**
+ *lionel，为何采用NAPI*

##### 9.2.4、定时器驱动的中断事件

##### 9.2.5、组合

##### 9.2.6、范例

+ drivers/net/3c59x.c中的`vortex_interrupt()`

#### 9.3、中断处理函数

##### 9.3.1、下半部函数存在的原因

9.3.2、下半部解决方案

9.3.3、并发和上锁

9.3.4、抢占功能

9.3.5、下半部函数

##### 9.3.6、微任务

+ include/linux/interrupt.h中的`tasklet_struct`结构

9.3.7、软IRQ初始化

9.3.8、未决软IRQ的处理

9.3.9、依体系结构处理软IRQ

9.3.10、微任务的处理

##### 9.3.11、网络代码如何使用软IRQ

+ 在net_dev_init中注册
  + `open_softirq(NET_TX_SOFTIRQ, net_tx_action, NULL);`，处理出去的流程
  + `open_softirq(NET_RX_SOFTIRQ, net_rx_action, NULL);`，处理进来的流程

#### 9.4、softnet_data结构

+ **每个CPU都有其队列，用来接收进来的帧**
+ include/linux/netdevice.h中的

##### 9.4.1、softnet_data的字段

##### 9.4.2、softnet_data的初始化

+ 每个CPU的softnet_data结构是由dev.c中的`net_dev_init`在引导期间执行初始化的

### chap10、帧的接收

#### 10.1、与其他功能交互

#### 10.2、设备的开启与关闭

+ net_device->state中的`__LINK_STATE_START`标识

#### 10.3、队列

+ 回环设备，不需要队列就能工作
+ **流量控制（QoS层）会为每个设备都定义一个队列**

#### 10.4、通知内核帧已接收：NAPI和netif_rx

##### 0、

##### 10.4.1、NAPI简介

+ NAPI背后的主要想法：**混合使用中断事件和轮询，而不使用纯粹的中断事件驱动模型**。

##### 10.4.2、NAPI所用之net_device字段

10.4.3、net_rx_action和NAPI

10.4.4、新旧驱动程序接口

10.4.5、操作poll_list

#### 10.5、设备驱动程序与内核间的旧接口：netif_rx的第一部分

10.5.1、net_rx的初始任务

10.5.2、管理队列以及下半部调度

#### 10.6、拥塞管理

##### 10.6.1、netif_rx中的拥塞管理

##### 10.6.2、平均队列长度及拥塞等级计算

#### 10.7、处理NET_RX_SOFTIRQ：net_rx_action

##### 10.7.1、积压的处理：process_backlog轮询虚拟函数

##### 10.7.2、入口帧的处理

### chap11、帧的传输

#### 11.1、传输的开启和关闭

##### 11.1.1、设备调度以准备传输

##### 11.1.2、队列规则接口

##### 11.1.3、dev_queue_xmit函数

##### 11.1.4、处理NET_TX_SOFTIRQ：net_tx_action

### chap12、中断事件一般性参考数据

12.1、统计数据

12.2、通过/proc和sysfs文件系统调整

#### 12.3、本章涉及的函数和变量

#### 12.4、本章涉及的文件和目录

### chap13、协议处理函数

#### 13.1、网络协议栈概论

#### 13.2、执行正确的协议处理函数

#### 13.3、协议处理函数的组织

#### 13.4、协议处理函数的注册

#### 13.5、Ethernet与IEEE802.3帧

#### 13.6、通过/proc文件系统进行调整

#### 13.7、本章涉及的函数和变量

#### 13.8、本章涉及的文件和目录

### chap14、桥接：概念

### chap15、桥接：生成树协议

### chap16、桥接：Linux实现

chap17、桥接：其它主题

### chap18、IPv4：概念

#### 18.1、IP协议：大蓝图

+ **图18.1**，IP内核协议栈的关键函数
+ IP协议的任务
  + 健康检查
  + 防火墙
  + 处理选项
  + 分段/重组
  + 接收、传输以及转发操作

#### 18.2、IP报头

#### 18.3、IP选项

##### 0、

+ 单字节
+ 多字节

##### 18.3.1、End of Option List和No Operation选项

##### 18.3.2、“Source Route"选项

##### 18.3.3、Record Route选项

18.3.4、“Timestamp“选项

18.3.5、Router Alert选项

#### 18.4、封包的分段/重组

18.4.1、分段对较高分层的效应

18.4.2、分段/重组所用的IP报头字段

18.4.3、分段/重组的问题实例

+ 重新传输
+ 把片段和其IP封包相关联
+ IP ID生成范例
+ 无法解决的重组问题范例：NAT

18.4.4、路径MTU发现

#### 18.5、检验和

##### 18.5.1、用于校验和计算的API

##### 18.5.2、对L4校验和所做的修改

### chap19、IPv4：Linux的原理和功能

#### 19.1、主要IPv4数据结构

##### 19.1.1、sk_buff和net_device结构里与检验和相关的字段

+ net_device结构
+ sk_buff结构

#### 19.2、封包的一般性处理

##### 19.2.1、协议初始化

19.2.2、和Netfilter互动

19.2.3、与路由子系统的交互

19.2.4、处理输入IP封包

#### 19.3、IP选项

##### 19.3.0、0

+ 处理IP选项相当耗时，因此IP选项一直用得不多
+ api定义在 net/ipv4/ip_options.c中，**并非一个封包的所有IP选项都必须在其所有片段中重复**
  + ip_options_compile
  + ip_options_build

##### 19.3.1、选项的处理

+ **入口IP封包的选项首先会由ip_options_compile()解析**
+ 图19-3

##### 19.3.2、选项的解析

+ **解析**：是指把IP选项从存储在IP封包报头里的格式中抽取出来，然后将其存储在一个名为ip_options的结构中，使得程序代码更方便处理。
+ 19.3.2.1、选项：严格和松散源路由
+ 19.3.2.2、选项：Record Route
+ 19.3.2.3、选项：Timestamp
+ 19.3.2.4、选项：Router Alert
+ 19.3.2.5、处理解析错误

### chap20、IPv4：转发和本地传递

#### 20.1、转发

##### 20.1.0、0

+ 转发分割成两个函数
  + ip_forward
  + ip_forward_finish
+ 转发的步骤：
  + 1、处理IP选项
  + 2、确定封包可以被转发
  + 3、递减IP报头的TTL字段，然后，TTL字段变为零，就丢弃该包
  + 4、根据路径相关MTU，必要时处理分段工作
  + 5、把封包传送至外出设备

##### 20.1.1、ICMP

##### 20.1.2、ip_forward函数

+ 由ip_rcv_finish调用

##### 20.1.3、ip_forward_finish函数

##### 20.1.4、dst_output函数

+ 以抵达目的地主机
+ **会调用虚拟函数output**

#### 20.2、本地传递

+ 重组工作是在ip_defrag函数内进行的
+ ip_local_deliver_finish函数

### chap21、IPv4：传输

+ L3层的封包传输
+ 中央传递封包的函数是dst_output
+ 本章讨论的是在dst_output之前，在此阶段，内核的任务
  + 查询下一个跳点
  + 初始化IP报头
  + 处理选项
  + 分段
  + 校验和
  + 由Netfilter检查
  + 更新统计数据

#### 21.1、进行传输的重要参数

##### 21.1.0、0

##### 21.1.1、多播流量

##### 21.1.2、本地流量相关的套接字数据结构

##### 21.1.3、ip_queue_xmit函数

+ `int ip_queue_xmit(struct sk_buff *skb, int ipfragok)`
+ 21.1.3.1、设定路径
+ 21.1.3.2、构建IP报头

##### 21.1.4、ip_append_data函数

+ 21.1.4.1、针对ip_append_data的基本内存分配和缓冲区组织
+ 21.1.4.2、针对ip_append_data（有分散/聚集IO）的内存分配和缓冲区组织
+ 21.1.4.3、处理片段缓冲区的重要函数
+ 21.1.4.4、缓冲区的后续处理
+ 21.1.4.5、设定上下文
+ 准备产生片段
+ 把数据拷贝到片段：getfrag
+ 缓冲区分配
+ 主要循环
+ L4校验和

##### 21.1.5、ip_append_page函数

##### 21.1.6、ip_push_pending_frames函数

##### 21.1.7、整合传输函数

##### 21.1.8、raw套接字

#### 21.2、衔接邻居子系统

### chap22、IPv4：处理分段

#### 22.1、IP分段

##### 22.1.0、0

22.1.1、牵涉IP分段的函数

22.1.2、ip_fragment函数

22.1.3、慢速分段

22.1.4、快速分段

#### 22.2、IP重组

22.2.1、IP分段hash表的组织

22.2.2、重组的重要议题

22.2.3、涉及重组的函数

##### 22.2.4、新ipq实例的初始化

22.2.5、ip_defrag函数

##### 22.2.6、ip_frag_queue函数

22.2.6.1、处理重迭

22.2.6.2、L4校验和

##### 22.2.7、垃圾收集

22.2.7.1、hash表重新组织

### chap23、IPv4：其它主题

#### 23.1、长效IP端点信息

### chap24、L4协议和Raw IP的处理

#### 24.1、可用的L4协议

### chap25、因特网控制消息协议（ICMPv4）

#### 25.1、ICMP报头

