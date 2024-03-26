## 《Linux内核源码剖析-TCP/IP实现》

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

#### 2.7、套接口缓存

+ *skb具体结构啥意思？*
+ 可能大部分都在chap3里讲了

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
+ 套接口缓存（socket buffer），结构体是`sk_buffer`
+ **SKB的主要用途**：保存在进程和网络接口之间互相传递的用户数据，以及其他一些信息
+ 操作SKB的函数
+ *写代码的时候，怎么操作skb*
+ 源码路径
  + include/linux/[skbuff.h](https://lxr.missinglinkelectronics.com/linux+v2.6.24/include/linux/skbuff.h)，SKB结构定义和宏
  + net/core/[skbuff.c](https://lxr.missinglinkelectronics.com/linux+v2.6.24/net/core/skbuff.c)，操作SKB的函数

#### 3.2、sk_buff结构

##### 0、

+ **已接收或待发送的数据报文消息**，分为以下几类
  + 与SKB组织相关的成员变量
  + 通用成员变量
  + 标志性变量
  + 与特性相关的成员变量
+ **SKB在不同网络协议层之间传递**，可被用于不同的网络协议，**添加首部，比在不同层之间复制数据效率更高**，由于在数据缓冲区的头部添加数据意味着要修改指向数据缓存区的指针，这是个复杂的操作，所以提供了`skb_reserve()`先预留出空间
  + 向上层协议传递SKB，下层协议层的首部信息就没有用了。二层首部在三层没有用，**但内核没有删除，只是把有效载荷指针指向三层首部**
  + 向下层传递SKB，需要预留head空间

##### 3.2.1、网络参数和内核数据结构

+ *如何启用编译选项，以及编译选项的宏对应哪个？*
+ **SKB有两个部分**，
  + 一部分为SKB描述符（sk_buffer结构本身）
  + 另一部分为数据缓存区（参见sk_buff结构的head成员）

##### 3.2.2、SKB组织相关的变量

+ 用来构成SKB双向链表，`struct sk_buff *next; struct sk_buff *prev;`
  + **每个SKB必须能被整个链表的头部快速找到**，在第一个SKB结点前面会插入另一个辅助的sk_buff_head结构的头结点，**可以认为该sk_buff_head结构就是SKB链表的头结点**。
  + 相当于就是`sk_buff_head->skb1->skb2`【*还是书上的图画得更准确*】

```c
struct sk_buff_head {
	/* These two members must be first. */
    struct sk_buff  *next;
    struct sk_buff  *prev;

    __u32           qlen;  //SKB链表中结点数，即队列长度
    spinlock_t      lock;  //控制对SKB链表并发操作的自旋锁
};
```



##### 3.2.3、数据存储相关的变量

+ *不确定，书上的解读对不对了*-想找个例子看一下

```c
 326        /* These elements must be at the end, see alloc_skb() for details.  */
 327        sk_buff_data_t          tail;
 328        sk_buff_data_t          end;
 329        unsigned char           *head,
 330                                *data;
 331        unsigned int            truesize;
 332        atomic_t                users;
```



##### 3.2.4、通用的成员变量

+ **与内核功能无关，主要跟网络协议、网络设备**等有关

##### 3.2.5、标志性变量

+ **标识payload是否被单独引用，是否允许分片，当前克隆的状态**

##### 3.2.6、特性相关的成员变量

#### 3.3、skb_shared_info结构

+ **end指针所指向地址起紧跟着一个skb_shared_info结构**，作用是**保存了数据块的附加信息**。

+ *确实看到了，不知道啥意思*
+ *本节，看纸质书，好好学下*

##### 0、

##### 3.3.1、“零拷贝”技术

##### 3.3.2、对聚合分散I/O数据的支持

##### 3.3.3、对GSO的支持

##### 3.3.4、访问skb_shared_info结构

#### 3.4、管理函数

+ *这一部分，是不是也可以通过man skb_init()来看到操作，要确认一下*

##### 0、

+ **不要直接调用`__do_something()`这种，而是用`do_something()`**，*有`__`开头的函数，lionel*，加过了合法性校验或者获取锁操作

##### 3.4.1、SKB的缓存池

+ `skb_init()`，调用了`kmem_cache_create()`
+ *skbuff_head_cache和skbuff_fclone_cache，有什么区别？*

##### 3.4.2、分配SKB

+ 1、alloc_skb()
+ 2、dev_alloc_skb()
  + **alloc_skb()的封装函数**

##### 3.4.3、释放SKB

+ dev_kfree_skb()
+ kfree_skb()

##### 3.4.4、数据预留和对齐

+ 1、skb_reserve()
+ 2、skb_put()
+ 3、skb_push()
+ 4、skb_pull()

##### 3.4.5、克隆和复制SKB

+ 1、skb_clone()
+ 2、pskb_copy()
+ 3、skb_copy()

##### 3.4.6、链表管理函数

##### 3.4.7、添加或删除尾部数据

+ 1、skb_add_data()

##### 3.4.8、拆分数据：skb_split()

##### 3.4.9、重新分配SKB的线性数据区：pskb_expand_head()

##### 3.4.10、其他函数

+ pskb_may_pull()
+ skb_queue_empty()
+ skb_realloc_headroom()
+ skb_get()
+ skb_shared()
+ skb_share_check()
+ skb_unshare()
+ skb_orphan()
+ skb_cow()
+ skb_pagelen()

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
+ 分**静态**和**动态**加载
  + 判断标准是**函数地址是否为NULL**

#### 4.3、优化基于宏的标记

+ 模块的初始化，通过`module_init`宏（*可以自动根据条件选不同初始化方法*）

#### 4.4、网络设备处理层初始化

+ 表4-3
  + net/socket.c，`core_initcall(sock_init)`，套接口层的初始化函数
  + net/core/sock.c，`subsys_initcall(proto_init)`，传输层的初始化函数
  + net/ipv4/af_inet.c，`fs_initall(inet_init)`，Internet协议族的初始化函数
  + net/core/dev.c，`subsys_initcall(net_dev_init)`，设备处理层的初始化函数
  + drivers/net/e100.c，`module_init(e100_init_module)`，e100型号的网络设备驱动的初始化函数
+ 本地暂时讲的是net_dev_init和e100_init_module

### chap5、网络设备  （65/560）

#### 5.1、PCI设备

+ 除了PCI能插网卡，还有哪些接口可以插网卡
  + PCIe
  + USB
  + Express Card
  + thunderbolt 
  + pcncia

+ 之前用ISA（Industry Standard Architecutre）接口，现在都是PCI接口了
  + ISA是慢速设备
+ 涉及到的文件
  + include/linux/mod_devicetable.h

##### 5.1.1、PCI驱动程序相关结构

+ 1、pci_device_id结构，**配置寄存器**
+ 2、pci_driver结构，**描述一个PCI设备，因此所有的PCI驱动都必须创建一个pci_driver结构的实例**

##### 5.1.2、注册PCI驱动程序

+ *这一套，其实是有实现模板的*

+ **以e100为例**
  + 首先是`module_init(e100_init_module)`，表示e100_init_module()是e100驱动的初始化接口，**在模块装载到内核时被调用**
  + `e100_init_module()`再调用`pci_register_driver()`，**为e100网络设备进行PCI驱动的注册**

#### 5.2、与网络设备有关的数据结构

##### 5.2.1、net_device结构

+ **网络驱动及接口层**中最重要的结构，不但描述了**接口方面的信息**，还包括**硬件信息**，*容易被诟病*，大概分为以下几类：
  + 硬件信息
  + 接口信息
  + 设备操作接口变量
  + 辅助成员变量
+ *这部分内容有点多，但主要还是要搞懂，这个结构体被谁用，具体字段是啥意思，这属于细节*

##### 5.2.2、网络设备有关的结构的组织

+ *net_device与in_device*的的关系

##### 5.2.3、相关函数

#### 5.3、网络设备的注册

##### 5.3.1、设备注册的时机

+ （1）加载网络设备驱动程序
+ （2）插入可热插拔网络设备

##### 5.3.2、分配net_device结构空间

+ 1、alloc_netdev()
+ 2、ether_setup()

##### 5.3.3、网络设备注册过程

+ *没细看呢*

##### 5.3.4、注册设备的状态迁移

+ 图5-5

##### 5.3.5、设备注册状态通知

+ 1、netdev_chain通知链
+ 2、netlink链接通知

##### 5.3.6、引用计数

#### 5.4、网络设备的注销

##### 5.4.1、设备注销的时机

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

+ 1、linkwatch_fire_event()
  + *图5-9*的流程
+ 2、linkwatch_event()
+ 3、linkwatch_run_queue()

##### 5.8.2、linkwatch标志

+ net/core/linkwatch.c中的定义的
  + LW_RUNNING
  + LW_SE_USED

#### 5.9、从用户空间配置设备相关信息

##### 5.9.1、ethtool

+ 图5-10，设备配置ioctl接口

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

### chap6、IP编址   （109/560）

+ [rfc790](https://datatracker.ietf.org/doc/html/rfc790)，提到了a,b,c类地址

#### 6.1、接口和IP地址

##### 6.1.1、主IP地址，从属IP地址和IP别名

##### 6.1.2、IP地址的组织

##### 6.1.3、in_device结构

+ **IP配置块**（网络适配层和IPv4相关的配置），通过ifconfig修改

##### 6.1.4、in_ifaddr结构

+ **IP地址块**（存储主机的IP地址，子网掩码，广播地址），**这些配置属于主机，但又是配置到网络设备上**

#### 6.2、函数

##### 6.2.1、

##### 6.2.7、inet_ifa_byprefix()

##### 6.2.8、inet_abc_len()

#### 6.3、IP地址的设置

+ *我的疑问是，netlink和ioctl的区别是啥？*

##### 6.3.1、netlink接口

+ 0、
  + iproute2包，需要单独装，*至少有些版本上是*
+ 1、netlink消息结构
  + 对应[rfc3549](https://datatracker.ietf.org/doc/html/rfc3549)
+ 2、
+ 3、

##### 6.3.3、

#### 6.4、ioctl

#### 6.5、inetaddr_chain通知链

### chap7、接口层的输入

+ *我的问题是，怎么理解 接口层*（我大概想了一下，可能是那种5层结构的那种接口层，用于承上启下的）
+ 本章讨论网络设备通用接口、netpoll接口和netconsole驱动，涉及文件
  + include/linux/if.h，定义IPv4专用的接口层使用的结构、宏
  + include/linux/netdevice.h，定义网络设备结构、宏

#### 7.1、系统参数

+ dev_weight
+ netdev_budget
+ netdev_max_backlog
+ message_burst与message_cost

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

+ 0、
  + 自己看了RFC791，里面的**IP选项**的介绍能对得上12.1，但是**实现**不确定从哪来的
  + [rfc1122](https://datatracker.ietf.org/doc/html/rfc1122#page-35)，跟rfc791差不多，只不过人家实现了多种场景，（link layer，ip层、传输层）
+ 提到了2个rfc，rfc791，rfc1122

#### 12.1、IP选项

##### 12.1.1、选项列表的结束符，End of Option List

##### 12.1.9、

+ 我没在rfc791上看到，**作者让看rfc2113**

#### 12.2、ip_options结构

#### 12.3、在IP数据报中构建IP选项

+ `ip_options_build()`

#### 12.4、复制IP数据报中选项到指定的ip_options结构

#### 12.5、处理待发送IP分片中的选项

+ **复制标志**为0的给清理掉，*但我没找到相关字段？*

#### 12.6、解析IP选项

#### 12.7、还原在检验IP选项时修改的IP选项

#### 12.8、处理转发IP数据报中的IP选项

#### 12.9、处理IP数据报的源路由选项

#### 12.10、解析并处理IP首部中的IP选项

#### 12.11、路由警告选项的处理

#### 12.12、由控制信息生成IP选项信息块

### chap13、IP的分片与组装

+ **UDP很容易导致分片**

+ **分片**是**IP层和二层之间**的传输单元，分片对传输层是透明的

#### 13.1、系统参数

#### 13.2、分片

+ *两种分片在rfc791里没有提到，只是给了个分片过程，具体的实现，还是内核侧提供的，函数的话，还是看他们*

##### 13.2.1、快速分片

##### 13.2.2、慢速分片

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

chap16、

### chap17、邻居子系统（447/560）

#### 17.1、什么是邻居子系统

+ 涉及的文件
  + include/linux/inetdevice.h，定义IPv4专用的网络设备相关的结构、宏等
  + include/net/neighbour.h，定义邻居项等结构、宏和函数原型
  + net/core/neighour.c，邻居子系统的实现

#### 17.2

#### 17.3、邻居子系统的结构

##### 17.3.1、neigh_table结构

+ 存储**与邻居协议相关的参数、功能函数，以及邻居散列表**，一个实例对一个邻居协议，**所有的实例都链接在全局链表neigh_tables中**
  + *怎么链接呢？*
  + 对于ARP而言，neigh_table的实例就是`arp_tbl`

##### 17.3.2、neighbour结构

+ 存储**邻居的相关信息**，包括（状态、二层和三层协议地址、提供给三层协议的函数指针）**一个邻居，并不代表一个主机，而是一个三层协议地址**，对于多接口的主机，就有多三层协议地址
  + *难道真正的关联的是在这里面？*

##### 17.3.3、neigh_ops结构

+ 实现了L3与`dev_queue_xmit()`之间的调用桥梁
  + *理解不深*

##### 17.3.4、neigh_parms结构

+ 邻居协议参数配置块，**存储可调节的邻居协议参数**，**一个邻居协议对应一个参数配置块**，而每一个网络设备的IPv4的配置块中也存在一个存放默认值的邻居配置块

##### 17.3.5、pneigh_entry结构

+ **保存允许代理的条件**，只有和结构中的接收设备以及目标地址相匹配才能代理

##### 17.3.6、neigh_statistics结构

+ **存储统计信息**，一个该结构实例对应一个网络设备上的一种邻居协议

##### 17.3.7、hh_cache结构

+ **缓存二层首部**，`hardware_header`

#### 17.4、邻居表的初始化

#### 17.5、邻居项的状态机

#### 17.6、邻居项的添加与删除

#### 17.7、邻居项的创建与初始化

##### 17.7.1、neigh_alloc()

##### 17.7.2、neigh_create()

#### 17.8、邻居项散列表的扩容

+ `neigh_hash_grow`

#### 17.9、邻居项的查找

##### 17.9.1、neigh_lookup()

##### 17.9.2、neigh_lookup_nodev()

##### 17.9.3、__neigh_lookup()和neigh_lookup_errno()

#### 17.10、邻居项的更新

#### 17.11、垃圾回收

##### 17.11.1、同步回收

##### 17.11.2、异步回收

#### 17.12、外部事件

#### 17.13、邻居项状态处理定时器

#### 17.14、代理项

#### 17.15、输出函数（484/560）

##### 17.15.1、丢弃

##### 17.15.2、慢速发送

+ 1、neigh_resolve_output()

### chap18、ARP：地址解析协议（490/560）

### chap19、路由表  （513/560）

#### 19.1、什么是路由表

+ 路由子系统的核心是**转发信息库**（Forwarding Information Base）
+ 路由表是用来存储这样一些信息的
  + 一是用以确定输入数据报是应该上传给本机的上层协议还是继续转发的信息
  + 二是如果需要转发，正确转发数据报所需要的信息
  + 三是输出数据报应从哪个具体的网络设备输出的信息

+ 路由表项维护以及查找涉及的文件
  + include/net/ip_fib.h，定义路由表等结构、宏和函数原型
  + net/ipv4/fib_lookup.h，定义路由查找的相关函数原型
  + net/ipv4/fib_hash.c，实现路由表的查找和维护
  + net/ipv4/fib_frontend.c，实现操作路由表的接口函数和通知
  + net/ipv4/route.c，实现路由缓存项的操作函数

##### 19.1.1、路由的要素

+ 路由的要素
  + 1、路由表，是一个由**路由表项**组成的数据库
  + 2、metrics，是在**一条路由上配置的相关度量值**
  + 3、作用范围（scope），**IP地址和路由都有作用范围**
    + IP地址的作用范围：表示该IP地址距离本地主机有多远
    + 路由的作用范围：表示到目的网络的距离
  + 4、默认网关，指`0.0.0.0/0`路由

##### 19.1.2、特殊路由

+ 有2张特殊的路由表：
  + 一张表用于本地地址，存储了所有的本地地址
  + 一张表用于所有其他的路由

##### 19.1.3、路由缓存

+ 路由缓存分为2部分
  + 一部分是**与协议相关的缓存**（ipv4等三层协议），这是**缓存框架部分**
  + 另一部分是与**协议无关的缓存**，通常被称为DST，嵌套在缓存框架中，只存储与协议无关的信息

#### 19.2、系统参数

#### 19.3、路由表组成结构