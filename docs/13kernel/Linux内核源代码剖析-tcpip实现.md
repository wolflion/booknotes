## 《Linux内核源代码剖析-TCP/IP实现》

+ 2.6.20代码
+ https://lxr.missinglinkelectronics.com/
+ 上册（1-19章）

### chap1、预备知识

#### 1.1、应用层配置诊断工具

##### 1.1.1、iputils

##### 1.1.2、net-tools

##### 1.1.3、iproute2

#### 1.2、内核空间与用户空间的接口

##### 1.2.1、procfs

##### 1.2.2、sysctl(/proc/sys目录)

##### 1.2.3、sysfs(/sys文件系统)

##### 1.2.4、ioctl系统调用

##### 1.2.5、netlink套接口

#### 1.3、网络I/O加速

##### 1.3.1、TSO/GSO

##### 1.3.2、I/O AT

#### 1.4、其他

##### 1.4.1、slab分配器

##### 1.4.2、RCU

+ Read-Copy Update，**是一种改进的rwlock**

### chap2、网络体系结构概述

+ *感觉目录，是从上层往下层的，lionel*

2.1、引言

2.2、协议简介

2.3、网络架构

2.4、系统调用接口

2.5、协议无关接口

2.6、传输层协议

2.7、套接口缓存

#### 2.8、设备无关接口

+ 提供一组**通用接口**，供底层网络设备驱动程序和上层协议调用
+ **协议栈向设备发送数据包**时需要调用`dev_queue_xmit()`，该函数对SKB进行排队，真正传输还是底层设备驱动程序
+ **底层设备驱动程序接收报文**是调用`netif_rx()`将SKB上传到网络层。
+ **NAPI技术**

#### 2.9、设备驱动程序

+ net_device结构，有个`hard_start_xmit()`接口

#### 2.10、网络模块源代码组织

+ drivers
  + net
+ net
  + bridge
  + core
  + ethernet
  + ipv4
  + netlink
+ include
  + linux
  + net
  + asm-xxx

### chap3、套接口缓存（sk_buff）

#### 3.1、引言

+ 网络协议中的操作对系统的存储和设计有要求
  + 处理**变长缓存**，接收和发送的数据报长度不是固定的
  + 容器在头尾部添加和移除数据，因为需要在不同网络层次间进行数据传递
  + 在添加和移除数据时能够尽量避免数据的复制
+ 套接口缓存（socket buffer）
+ **SKB的主要用途**：保存在进程和网络接口之间互相传递的用户数据，以及其他一些信息

+ include/linux/[skbuff.h](https://lxr.missinglinkelectronics.com/linux+v2.6.24/include/linux/skbuff.h)，SKB结构定义和宏
+ net/core/[skbuff.c](https://lxr.missinglinkelectronics.com/linux+v2.6.24/net/core/skbuff.c)，操作SKB的函数

#### 3.2、sk_buff结构

##### 0、

+ **已接收或待发送的数据报文消息**，分为以下几类
  + 与SKB组织相关的成员变量
  + 通用成员变量
  + 标志性变量
  + 与特性相关的成员变量
+ **添加首部，比拷贝数据效率高**，`skb_reserve()`先预留

##### 3.2.1、网络参数和内核数据结构

##### 3.2.2、SKB组织相关的变量

+ 用来构成SKB双向链表

3.2.3、数据存储相关的变量

3.2.4、通用的成员变量

3.2.5、标志性变量

3.2.6、特性相关的成员变量

#### 3.3、skb_shared_info结构

##### 0、

##### 3.3.1、“零拷贝”技术

3.3.2、对聚合分散I/O数据的支持

3.3.3、对GSO的支持

3.3.4、访问skb_shared_info结构

#### 3.4、管理函数

##### 0、

+ **不要直接调用`__do_something()`这种，而是用`do_something()`**，*有`__`开头的函数，lionel*

3.4.1、SKB的缓存池

##### 3.4.2、分配SKB

+ 1、
+ 2、dev_alloc_skb()

3.4.3、释放SKB

3.4.4、数据预留和对齐

3.4.5、克隆和复制SKB

3.4.6、链表管理函数

3.4.7、添加或删除尾部数据

3.4.8、拆分数据：skb_split()

3.4.9、重新分配SKB的线性数据区：pskb_expand_head()

3.4.10、其他函数

### chap4、网络模块初始化

#### 4.1、引言

+ 网络设备驱动程序是如何初始化的
+ 各个协议是如何初始化的
+ include/linux/init.h
+ include/asm-generic/vmlinux.lds.h，**编译链接相关的宏定义**
+ init/main.c，启动时的高级初始化
+ net/core/dev.c，网络设备注册、输入和输出等接口
+ drivers/net/e100.c，e100驱动程序

#### 4.2、网络模块初始化顺序

+ **图4-1**

#### 4.3、优化基于宏的标记

+ 模块的初始化，通过`module_init`宏

#### 4.4、网络设备处理层初始化

+ 表4-3
  + net/socket.c，`core_initcall(sock_init)`，套接口层的初始化函数
  + net/core/sock.c，`subsys_initcall(proto_init)`，传输层的初始化函数
  + net/ipv4/af_inet.c，`fs_initall(inet_init)`，Internet协议族的初始化函数
  + net/core/dev.c，`subsys_initcall(net_dev_init)`，设备处理层的初始化函数
  + drivers/net/e100.c，`module_init(e100_init_module)`，e100型号的网络设备驱动的初始化函数
+ 本地暂时讲的是net_dev_init和e100_init_module

### chap5、网络设备

#### 5.1、PCI设备

##### 5.1.1、PCI驱动程序相关结构

##### 5.1.2、注册PCI驱动程序

#### 5.2、与网络设备有关的数据结构

##### 5.2.1、net_device结构

##### 5.2.2、网络设备有关的结构的组织

##### 5.2.3、相关函数

#### 5.3、网络设备的注册

##### 5.3.1、设备注册的时机

##### 5.3.2、分配net_device结构空间

##### 5.3.3、网络设备注册过程

##### 5.3.4、注册设备的状态迁移

##### 5.3.5、设备注册状态通知

+ 1、netdev_chain通知链
+ 2、netlink链接通知

##### 5.3.6、引用计数

#### 5.4、网络设备的注销

##### 5.4.1、设备注销的时机

+ 两种情形
  + 1、卸载网络设备驱动程序
  + 2、移除热插拔网络设备

##### 5.4.2、网络设备注销过程

+ 1、unregister_netdevice()
+ 2、衔接操作：netdev_run_todo()

#### 5.5、网络设备的启用

+ `dev_open()`，发送一个**NETDEV_UP消息**到网络设备状态改变通知链上

#### 5.6、网络设备的禁用

+ `dev_close()`

#### 5.7、与电池管理交互

##### 5.7.1、挂起设备

+ `e100_suspend()`

##### 5.7.2、唤醒设备

#### 5.8、侦测连接状态改变

+ 设备驱动侦测到设备传递信号时调用`netif_carrier_on()`
+ 设备驱动侦测到设备丢失信号时调用`netif_carrier_off()`
  + `__LINK_STATE_NOCARRIER`标志位

##### 5.8.1、调度处理连接状态改变事件

##### 5.8.2、linkwatch标志

#### 5.9、从用户空间配置设备相关信息

##### 5.9.1、ethtool

##### 5.9.2、媒体独立接口

#### 5.10、虚拟网络设备

+ 虚拟设备与物理设备的关系
  + 1对1
  + 多对1
  + 1对多

+ 表5-8，虚拟网络设备类型
  + Bonding
  + 802.1Q
  + Bridging
  + Aliasing interfaces
  + Tunnel interfaces
+ 表5-9，虚拟设备与物理设备在与内核交互的方式上的区别
  + 初始化
  + 配置
  + 外部接口
  + 发送
  + 接收
  + 外部通知
  + 注册
  + 注销

### chap6、IP编址

#### 6.1、接口和IP地址

##### 6.1.3、in_device结构

##### 6.1.4、in_ifaddr结构

### chap7、接口层的输入

7.1、系统参数

#### 7.2、接口层的ioctl

7.2.1、SIOCxIFxxx类命令

### chap8、接口层的输出

#### 8.1、输出接口

8.1.1、dev_queue_xmit()

8.1.2、dev_hard_start_xmit()

##### 8.1.3、e100的输出接口：e100_xmit_frame()

8.2、网络输出软中断

8.2.1、netif_schedule()

8.2.2、net_tx_action()

### chap9、流量控制

#### 9.1、通过流量控制后输出

9.1.1、dev_queue_xmit()

9.1.2、qdisc_restart()

9.2、构成流量控制的三种元素

9.2.1、排队规则

9.2.2、类

9.2.3、过滤器

9.3、默认的FIFO排队规则

9.3.1、pfifo_fast_init()

9.3.2、pfifo_fast_reset()

9.3.1、pfifo_fast_enqueue()

9.3.1、pfifo_fast_dequeue()

9.3.1、pfifo_fast_requeue()

9.4、netlink的tc接口

9.5、排队规则的创建接口

9.5.1、类的创建接口

9.5.2、过滤器的创建接口

### chap10、Internet协议族

+ 支持32种协议族，`net_families[NPROTO];`
+ include/linux/net.h，套接口层的结构、宏和函数原型
+ include/linux/protocol.h，注册传输层协议的结构、宏和函数原型
+ net/ipv4/af_inet.c，网络层和传输层接口
+ net/ipv4/ip_input.c，IP数据报的输入

#### 10.1、net_proto_family结构

+ net_proto_family屏蔽了（不同的协议族，传输层结构和实现的差异，各自套接口创建函数也有差异）差异，统一用`sock_register()`注册到net_families数组中，**提供了协议族到套接口创建**之前的接口

#### 10.2、inet_protosw结构

+ **只在套接口层起作用**
+ inetsw_array数组包含了三个inet_protosw结构的实例：**TCP，UDP和原始套接口**

#### 10.3、net_protocol结构

+ 定义了协议族中支持的传输层协议以及传输层的报文接收例程。是**网络层和传输层之间的桥梁**。

#### 10.4、Internet协议族的初始化

+ 初始化函数是`inet_init()`，通过fs_initcall(inet_init)，将inet_init加到内核的初始化列表中，保证了此函数会在系统启动时被调用

### chap11、IP：网际协议

#### 11.1、引言

##### 11.1.1、IP首部

+ 计算一份数据报IP检验和的方法
  + 首先把检验和字段置为0
  + 然后将整个首部看成由一串16位字组成，对其中的每个16位进行二进制反码求和，结果存在检验和字段中

##### 11.1.2、IP数据报的输入与输出

+ 图11-3，**IP层主要函数调用关系**

#### 11.2、IP的私有信息控制块

11.3、系统参数

11.4、初始化

11.5、IP层套接口选项

11.6、ipv4_devconf结构

11.7、套接口的错误队列

11.8、报文控制信息

11.9、对端信息块

11.10、IP数据报的输入处理

11.11、IP数据报的输出处理

11.12、IP层对GSO的支持

### chap12、IP选项处理

12.1、IP选项

12.2、ip_options结构

12.3、在IP数据报中构建IP选项

12.4、复制IP数据报中选项到指定的ip_options结构

12.5、处理待发送IP分片中的选项

12.6、解析IP选项

12.7、还原在检验IP选项时修改的IP选项

12.8、

12.9、

12.10、

12.11、

12.12、由控制信息生成IP选项信息块

### chap13、IP的分片与组装

#### 13.1、系统参数

13.2、分片

13.3、组装

13.3.1、ipq结构

13.3.2、ipq散列表和链表的维护

13.3.3、ipq散列表的重组

13.3.4、

13.3.5、

13.3.6、

13.3.7、

### chap14、ICMP：Internet控制报文协议

#### 14.1、ICMP报文结构

14.2、注册ICMP报文类型

14.3、系统参数

14.4、ICMP的初始化

14.5、输入处理

14.5.1、

14.5.2、

14.5.3、

14.5.4、

14.5.5、

14.6、输出处理

14.6.1、发送ICMP报文

14.6.2、发送回显应答和时间戳应答报文

### chap15、IP组播

15.1、初始化

15.2、虚拟接口

15.3、组播转发缓存

15.4、临时组播转发缓存

15.5、外部事件

15.6、组播套接口选项

15.7、组播选路套接口选项

15.8、组播的ioctl

15.9、组播报文的输入

15.10、组播报文的转发

15.11、组播报文的输出