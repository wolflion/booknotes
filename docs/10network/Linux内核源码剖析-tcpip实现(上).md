## 《Linux内核源码剖析-TCP/IP实现》

+ 2.6.20代码
+ https://lxr.missinglinkelectronics.com/
+ 上册（1-19章）

### ReadMe

#### 0.2、各章的内容

+ 问题1、include/linux下面的，与include/net下面的区别？

##### 18、ARP（net/ipv4/arp.c）

+ 头文件嘛，include/linux/if_arp.h
+ [【计算机网络】详解网络层（二）ARP和RARP](https://blog.csdn.net/wenqian1991/article/details/44133039)
+ [【Linux 内核网络协议栈源码剖析】ARP地址解析协议](https://www.cnblogs.com/ztguang/p/12645483.html)

##### 19、路由表（net/ipv4/fib_开头的.c，route.c）

+ 头文件的一些结构体定义，主要在`include/net/ip_fib.h`

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

### chap9、流量控制（184/560）

+ *iptables与流量控制 有啥关系？*

#### 9.1、通过流量控制后输出

+ 在链路层，每个数据包通过邻居子系统之后，都由dev_queue_xmit()来进行输出，然后根据输出网络设备的排队规程来确定是否通过QoS之后发送，还是直接调用网卡驱动注册的发送函数把数据包发送出去。
+ 流量控制在接口层输出的位置
  + 邻居子系统，调用dev_queue_xmit()
  + 接口层dev_queue_xmit()，入QoS队列激活数据包发送软中断
  + 数据包发送软中断，数据包发送软中断被激活 出QoS队列调用dev_hard_start_xmit()
  + 网络设备，`dev->hard_start_xmit()`
+ 涉及的文件
  + include/net/pkt_sched.h，定义了操作排队规则的宏等
  + include/net/sch_generic.h，定义排队规则、类和过滤器的结构、宏等
  + net/sched/sch_api.c，操作排队规则的接口函数
  + net/sched/sch_generic.c，基本的数据包调度函数
  + net/core/dev.c，网络设备注册、输入和输出等接口

##### 9.1.1、dev_queue_xmit()

##### 9.1.2、qdisc_restart()

+ 从排队规则中获取一个可以输出的报文，然后将其输出到网络设备

#### 9.2、构成流量控制的三种元素

+ 在启用了流量控制的情况下，每个网络设备至少会配置一个排队规则。

##### 9.2.1、排队规则

+ 1、Qdisc结构
+ 2、Qdisc_ops结构
  + 描述队列操作的接口，每个排队规则都必须要实现该接口
+ 3、排队规则相关函数
  + dev_init_scheduler()，初始化排队规则的相关数据，在注册网络设备的register_netdevice()中被调用
  + pktsched_int()，
  + register_qdisc()，

##### 9.2.2、类

+ 1、xxx_class结构
+ 2、Qdisc_class_ops结构，用于类操作的接口，排队规则实现了分类就必须实现该接口
+ 3、类相关函数
  + qdisc_graft()
  + dev_graft_qdisc()

##### 9.2.3、过滤器

+ **当报文被送到一个具有多个类的排队规则中时，排队规则将调用tc_classify()**
  + 检查过滤器是否接受`skb->protocol`所指定的协议
  + 然后，调用过滤器的`classify()`做出决定是否接受或分到哪个类中

+ 1、tcf_proto结构
+ 2、tcp_proto_ops结构
  + **描述过滤器的结构**，如果排队规则实现了分类，则必须实现过滤器用来分类
+ 3、过滤器相关函数
  + tc_filter_init()
  + register_tcf_proto_ops()
  + unregister_tcf_proto_ops()
  + tcf_proto_lookup_ops()

#### 9.3、默认的FIFO排队规则

##### 9.3.1、pfifo_fast_init()

+ FIFO排队规则的初始化函数，用来初始化三个优先级队列

##### 9.3.2、pfifo_fast_reset()

##### 9.3.3、pfifo_fast_enqueue()

##### 9.3.4、pfifo_fast_dequeue()

##### 9.3.5、pfifo_fast_requeue()

#### 9.4、netlink的tc接口

+ **tc就是通过netlink对流量控制进行配置的命令**

#### 9.5、排队规则的创建接口

+ 当用tc工具创建排队规则时，通过netlink后最终由`tc_modify_qdisc()`来处理

##### 9.5.1、类的创建接口

+ 使用tc工具配置排队规则类，通过netlink后最终由`tc_ctl_tclass()`来处理

##### 9.5.2、过滤器的创建接口

+ 使用tc工具配置类的过滤器，通过netlink后最终由`tc_ctl_tfilter()`来处理

#### 其它

+ *我之前以为过滤器是独立的，其实也属于IP层*

### chap10、Internet协议族  （219/560）

+ 支持32种协议族，`net_families[NPROTO];`，**每个协议族用一个net_proto_family结构实例来表示**，调用**sock_register()将结构注册到全局数组net_families中**
  + *有哪些具体的协议没有啊*，PF_INET，PF_LOCAL，PF_UNIX应该都算
+ 涉及的文件
  + include/linux/net.h，套接口层的结构、宏和函数原型
  + include/linux/protocol.h，注册传输层协议的结构、宏和函数原型
  + net/ipv4/af_inet.c，网络层和传输层接口
  + net/ipv4/ip_input.c，IP数据报的输入

#### 10.1、net_proto_family结构

+ net_proto_family屏蔽了（不同的协议族，传输层结构和实现的差异，各自套接口创建函数也有差异）差异，统一用`sock_register()`注册到net_families数组中，**提供了协议族到套接口创建**之前的接口

#### 10.2、inet_protosw结构

+ **只在套接口层起作用**
+ inetsw_array数组包含了三个inet_protosw结构的实例：**TCP，UDP和原始套接口**
+ inet_register_protosw()将inetsw_array数组中的inet_protosw结构实例，以其type值为key组织到散列表inetsw中，**各协议族中的type值相同而protocol值不同的inet_protosw结构实例，在inetsw散列表中以type为关键字连接成链表**，通过inetsw散列表可以找到所有协议族的inet_protosw结构实例

#### 10.3、net_protocol结构

+ 定义了协议族中支持的传输层协议以及传输层的报文接收例程。是**网络层和传输层之间的桥梁**。
  + 这里“传输层”，包含了ICMP和IGMP
  + **当网络数据报从网络层流向传输层时，会调用此结构中的传输层协议数据报接收处理函数**

#### 10.4、Internet协议族的初始化

+ 初始化函数是`inet_init()`，通过fs_initcall(inet_init)，将inet_init加到内核的初始化列表中，保证了此函数会在系统启动时被调用

### chap11、IP：网际协议 （227/560）

#### 11.1、引言

+ 涉及的文件
  + include/net/ip.h，定义IP层相关的结构、宏和函数原型
  + include/linux/inetdevice.h，定义IPv4专用的网络设备相关的结构、宏等
  + net/ipv4/ip_output.c，IP数据报的输出
  + net/ipv4/ip_sockglue.c，IP层套接口选项
  + net/ipv4/ip_input.c，IP数据报的输入
  + net/ipv4/ip_forward.c，IP数据报的转发
  + net/ipv4/inetpeer.c，对端信息块的管理
  + net/ipv4/af_inet.c，网络层和传输层接口

##### 11.1.1、IP首部

+ 计算一份数据报IP检验和的方法
  + 首先把检验和字段置为0
  + 然后将整个首部看成由一串16位字组成，对其中的每个16位进行二进制反码求和，结果存在检验和字段中

##### 11.1.2、IP数据报的输入与输出

+ 图11-3，**IP层主要函数调用关系**

#### 11.2、IP的私有信息控制块

+ IP层在SKB中有个信息控制块`inet_skb_parm`结构，存储在skb_buff结构的cb成员中。

#### 11.3、系统参数

+ ip_default_ttl，
+ ip_dynaddr，
+ ip_local_port_range，
+ ip_no_pmtu_disc

#### 11.4、初始化

#### 11.5、IP层套接口选项

+ IP层套接口选项
  + IP_OPTIONS
  + IP_PKTINFO

#### 11.6、ipv4_devconf结构

+ 是**网络设备接口的IPv4系统配置**，是**系统全局变量**，该配置对所有接口有效。

#### 11.7、套接口的错误队列

+ 在传输控制块中有一个用于保存错误信息的队列`sk_error_queue`

##### 11.7.1、添加ICMP差错信息

##### 11.7.2、添加由本地产生的差错信息

##### 11.7.3、读取错误信息

+ recvmsg()用来接收远端发送到所在套接口的数据的，但也可通过设置flags为`MSG_ERRQUEUE`来读取传输控制块错误队列上的错误信息。

#### 11.8、报文控制信息

##### 11.8.1、IP控制信息块

+ 由`ipcm_cookie{}`结构描述，存储有关输出的控制信息

##### 11.8.2、报文控制信息的输出

##### 11.8.3、报文控制信息的输入

+ `ip_cmsg_recv()`用于获取报文控制信息

#### 11.9、对端信息块

+ 由`inet_peer`结构描述，用来保存对端的一些信息
+ 对端信息块以`v4addr`为关键字

##### 11.9.1、系统参数

+ inet_peer_gc_maxtime

##### 11.9.2、对端信息块的创建和查找

+ 通过`inet_getpeer()`来实现的，**由参数create来区分是创建还是查找**

##### 11.9.3、对端信息块的删除

##### 11.9.4、垃圾回收

+ 1、对端信息块的释放
+ 2、同步清理
+ 3、异步清理

#### 11.10、IP数据报的输入处理  (254/560)

+ IP数据报定义，`struct packet_type ip_packet_type{}`
+ 1、`ip_rcv()`
  + ip_rcv()处理完成并经`PRE-ROUTING`点netfilter处理后，再由`ip_rcv_finish()`处理
  + ip_rcv_finish()中，根据数据报的路由信息，决定**数据报是转发还是输入到本机**
    + 输入到本机由`ip_local_deliver()`处理
    + 转发由`ip_forward()`处理
+ 2、`ip_rcv_finish()`

##### 11.10.1、IP数据报输入到本地

+ 1、ip_local_deliver()
  + **先判断接收到的数据报是不是分片**，若是分片，则需将分片重组
+ 2、ip_local_deliver_finish()

##### 11.10.2、IP数据报的转发

+ 转发由`ip_forward()`

#### 11.11、IP数据报的输出处理

##### 11.11.1、IP数据报输出到设备

+ 经过路由之后都要输出到网络设备，**输出到网络设备的接口就是`ip_output()`**
+ dst_output0()
+ ip_output()
+ ip_finish_output()
+ ip_finish_output2()

##### 11.11.2、TCP输出的接口

+ **ip_build_and_send_pkt()和ip_send_reply()只有在发送特定段时才会被调用**

+ 1、ip_queue_xmit()
+ 2、ip_build_and_send_pkt()
+ 3、ip_send_reply()
  + **用于构成并输出RST和ACK段，在tcp_v4_send_reset()和tcp_v4_send_ack()中被调用**

##### 11.11.3、UDP输出的接口

+ 1、ip_append_data()
+ 2、ip_ufo_append_data()
+ 3、ip_push_pending_frames()

#### 11.12、IP层对GSO的支持

##### 11.12.1、inet_gso_segment()

+ IP层gso_segment接口的实现函数

##### 11.12.2、inet_gso_send_check()

+ IP层gso_send_check接口的实现函数，**在分段之前对伪首部进行校验和计算**

### chap12、IP选项处理（288/560）

+ 思考、
  + 自己看了RFC791，里面的**IP选项**的介绍能对得上12.1，但是**实现**不确定从哪来的
  + [rfc1122](https://datatracker.ietf.org/doc/html/rfc1122#page-35)，跟rfc791差不多，只不过人家实现了多种场景，（link layer，ip层、传输层）
+ 提到了2个rfc，rfc791，rfc1122

#### 12.1、IP选项

##### 12.1.1、选项列表的结束符，End of Option List

##### 12.1.2、空操作

##### 12.1.3、安全选项

##### 12.1.4、严格源路由选项（SSRR）

##### 12.1.9、

+ 我没在rfc791上看到，**作者让看rfc2113**

#### 12.2、ip_options结构

+ IP选项信息块

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

### chap13、IP的分片与组装（313/560）

+ **UDP很容易导致分片**

+ **分片**是**IP层和二层之间**的传输单元，分片对传输层是透明的

#### 13.1、系统参数

+ ipfrag_high_thresh
+ ipfrag_low_thresh
+ ipfrag_max_dist，允许接收来自同一个源IP地址的IP分片数量的上限值，**用于防御Dos攻击**，默认值64
+ ipfrag_secret_interval
+ ipfrag_time

#### 13.2、分片

+ **ip_finish_output()**将数据报发送出去之前，会检测数据报长度，超出MTU，调用**ip_fragment()对数据报分片**，否则直接调用ip_finish_output2()输出到数据链路层。

+ *两种分片在rfc791里没有提到，只是给了个分片过程，具体的实现，还是内核侧提供的，函数的话，还是看他们*
+ *快速分片，慢速分片，两者区别*

##### 13.2.1、快速分片

+ 当传输层已将数据分块，并将这些块链接在`skb_shinfo(skb)->frag_list`，此时则可以通过快速分片进行处理
+ **有4种情况不能快速分片：**
  + 有分片长度大于MTU
  + 除最后一个分片外还有分片长度未8字节对齐
  + IP首部中的MF或片偏移不为0，说明SKB不是一个完整的IP数据报
  + 此SKB被克隆

##### 13.2.2、慢速分片

#### 13.3、组装

+ **每一个将被重新组合的IP数据报都用一个ipq结构实例来表示**

##### 13.3.1、ipq结构

+ 用于保存分片的数据结构必须做到以下几点：
  + 1、快速定位属于某一数据报的一组分片
  + 2、在属于某一数据报的一组分片中快速插入新的分片
  + 3、有效地判断一个数据报的所有分片是否已经全部接收
  + 4、具有组装超时机制，如果在重组完成之前定时器溢出，则删除该数据报的所有内容

##### 13.3.2、ipq散列表和链表的维护

+ ipq_kill()，
+ ipq_put()，释放irq及分片

##### 13.3.3、ipq散列表的重组

+ **所有的分片重组都是通过ipq散列表进行的**
+ ipfrag_init()
+ ipfrag_secret_rebuild()

##### 13.3.4、超时IP分片的清除

+ ip_frag_create()
+ ip_expire()

##### 13.3.5、垃圾收集

+ ip_evictor()主要对ipq中分片进行条件性的清理

##### 13.3.6、相关分片组装函数

+ ip_find()
+ ip_frag_create()
+ ip_frag_intern()
+ ip_frag_queue()
+ ip_frag_reasm()

##### 13.3.7、分片组装

+ **图13-5**
+ IP分片组装过程三步：
  + 1、首先判断接收到的IP数据报是否是一个分片
  + 2、以(addr，saddr，protocol，id，ipfrag_hash_rnd)计算键值，在ipq散列表中找到分片所属的ipq，并按序插入到该ipq的分片链表中
  + 3、如果此时该ipq的所有分片已经全部到达，则将这些分片组装成一个完整的IP数据报

### chap14、ICMP：Internet控制报文协议（340/560）

+ ICMP是网络层的，**可以看作IP协议的附属协议**，因为它主要被IP用来与其他主机或路由器交换错误报文及其他需注意的信息。
+ 涉及文件
  + net/ipv4/icmp.c，ICMP协议的处理
  + net/ipv4/af_inet.c，网络层和传输层接口

#### 14.1、ICMP报文结构

#### 14.2、注册ICMP报文类型

+ `struct net_protocol icmp_protocol={.handler = icmp_rcv,};`
+ 接收ICMP报文例程是`icmp_rcv()`

#### 14.3、系统参数

+ icmp_echo_ignore_all
+ icmp_echo_ignore_broadcasts

#### 14.4、ICMP的初始化

+ icmp_init，在`inet_init()`中调用，主要功能是**为每个CPU创建一个基于原始流、IPPROTO_ICMP协议类型的套接口供内核使用**

#### 14.5、输入处理icmp_rcv

+ ICMP报文到达时，IP层通过inet_protos[IPPROTO_ICMP]找到该函数进行输入处理

##### 14.5.1、差错控制

+ icmp_unreach()，处理**目的不可达、源端被关闭、超时、参数错误**这四种类型的差错ICMP报文

##### 14.5.2、重定向处理

+ icmp_redirect()

##### 14.5.3、请求回显

+ `struct icmp_bxm{};`

##### 14.5.4、时间戳请求

+ icmp_timestamp()

##### 14.5.5、地址掩码请求和应答

#### 14.6、输出处理icmp_send

##### 14.6.1、发送ICMP报文

+ 输出各种指定类型和编码的ICMP报文，**不能应答目的地址为组播或广播类型的硬件地址或IP地址的报文**

##### 14.6.2、发送回显应答和时间戳应答报文

+ icmp_reply()，**回显应答和时间戳应答**
+ icmp_push_reply()
  + **用来创建待发送ICMP报文，然后将其添加到传输控制块的发送缓冲队列中**，完成后，如果套接口的输出队列上还有未输出的报文，则计算ICMP报文的校验和并将其输出

### chap15、IP组播（363/560）

+ **只有通过组播路由协议守护进程（mrouted），依靠路由协议（静态路由、ospf、rip）来生成转发缓存**，才能真正实现组播功能。
+ 涉及文件
  + include/linux/mroute.h，定义IP组播相关的虚拟接口结构、组播转发缓存结构等
  + net/ipv4/ipmr.c，IP组播的输入和转发
  + net/ipv4/ip_output.c，IP数据报的输出，包括IP组播的输出

#### 15.1、初始化

+ 为IP组播建立环境，包括**创建组播转发缓存池，为临时路由转发缓存建立定时器**
+ `ip_mr_init()`在`inet_init()`中被调用来初始化IP组播

#### 15.2、虚拟接口

+ 组播报文可以通过两条途径来收发：
  + 一是**直接通过LAN网络设备收发**
  + 二是**打包成二级单播IP数据报，然后通过隧道传输**
+ 由**vif_device结构来描述**，**标志flags**来区分虚拟接口当前描述的是**物理网络设备还是IP-IP隧道**

##### 15.2.1、虚拟接口的添加

+ `struct vifctl{};`

##### 15.2.2、虚拟接口的删除：vif_delete()

+ 以MRT_DEL_VIF为optname参数值调用ip_mroute_setsockopt()时被激活。**参数vifi是待删除虚拟接口的索引**

##### 15.2.3、查找虚拟接口：ipmr_find_vif()

+ 根据参数给出的网络设备**遍历vif_table数组**查找对应的虚拟接口，该函数通常在输入或输发组播报文时被调用。

#### 15.3、组播转发缓存（Multicast Forwarding Cache，MFC）

+ 用来存储组播转发所需的全部信息

##### 15.3.1、组播转发缓存的创建

+ ipmr_mfc_add()实现

##### 15.3.2、组播转发缓存的删除

+ ipmr_mfc_delete()

##### 15.3.3、组播转发缓存的查找

##### 15.3.4、向组播路由守护进程发送报告

+ IGMPMSG_NOCACHE和IGMPMSG_WRONGVIF类型报告的首部为IGMP协议，负载为igmpmsg结构；
+ IGMPMSG_WHOLEPKT类型报告的首部为igmpmsg结构，负载部分为完整的IP数据报文，这可能是PIM报文也可能是组播报文
+ `struct igmpmsg{};`

#### 15.4、临时组播转发缓存

+ IGMPv3协议，最重要的特征是**增加了对组播源的过滤**

##### 15.4.1、临时组播转发缓存队列

+ **临时组播转发缓存项**都存储在**单项链表mfc_unres_queue中**，而cache_resolve_queue_len则是该链表的当前长度

##### 15.4.2、创建临时组播转发缓存

+ ipmr_cache_unresolved()完成四件事
  + 1、创建临时组播转发缓存项
  + 2、向组播路由守护进程发送IGMPMSG_NOCACHE类型报告
  + 3、启动定时器
  + 4、缓存该组播报文到对应的临时组播转发缓存项中，但一个临时组播转发缓存项最多存3个报文

##### 15.4.3、用于超时而删除临时组播转发缓存的定时器

##### 15.4.4、释放临时组播缓存项中保存的临时组播报文

#### 15.5、外部事件

+ 当一个网络设备的状态发生变化时，**IP组播模块通过注册到通知链中的ip_mr_notifer收到通知，然后调用ipmr_device_event()来处理该事件**。
+ ipmr_device_event()只关心**NETDEV_UNREGISTER**事件，遍历vif_tables数组，删除与该网络设备相关的虚拟接口

#### 15.6、组播套接口选项

##### 15.6.1、IP_MULTICAST_TTL

##### 15.6.2、IP_MULTICAST_LOOP

+ 开启和禁用**组播回环**，0为禁止，1为允许

##### 15.6.3、IP_MULTICAST_IF

+ **为发送组播设置默认的网络设备接口**

##### 15.6.13、MCAST_MSFILTER

#### 15.7、组播选路套接口选项  378（389/560）

##### 15.7.1、MRT_INIT

##### 15.7.2、MRT_DONE

#### 15.8、组播的ioctl

##### 15.8.1、SIOCGETVIFCNT

+ **获取通过虚拟接口组播包的统计信息**，通过`sioc_vif_req`结构返回

#### 15.9、组播报文的输入 ip_mr_input()

#### 15.10、组播报文的转发

##### 15.10.1、ip_mr_forward()

##### 15.10.2、ipmr_queue_xmit()

+ 转发组播报文，在ip_mr_forward()中被调用

#### 15.11、组播报文的输出ip_mc_output()

### chap16、IGMP：Internet组管理协议  390（400/560）

+ 涉及的文件
  + include/linux/inetdevice.h，定义IPv4专用的网络设备相关的结构、宏
  + include/linux/igmp.h，定义IGMP报文等结构、宏和函数原型
  + net/ipv4/igmp.c，IGMP协议的实现

#### 16.1、in_device结构中的组播参数

+ **网络设备层和IPv4相关的配置**都存放在in_device结构中，包括有关组播的配置信息

#### 16.2、ip_mc_list结构

+ 描述**网络设备加入组播组的配置块**，用来维护网络设备组播的状态，包括组播地址、过滤模式、源地址等

#### 16.3、系统参数

#### 16.4、IGMP的版本与协议结构

##### 16.4.1、IGMP的版本

16.4.4、

#### 16.5、IGMP报文的输入igmp_rcv

+ 在inet_init()中注册的igmp_protocol结构定义的

#### 16.6、函数

##### 16.6.1、ip_mc_find_dev()

##### 16.6.2、ip_check_mc()

#### 16.7、成员关系查询

#### 16.8、成员关系报告

##### 16.8.1、最近离开组播组列表的维护

##### 16.8.2、is_in()

+ **判断指定组播组的特定组播源是否属于特定的组记录类型**

##### 16.8.3、add_grec()

+ 组成或发送各种类型的V3报告报文

##### 16.8.4、普通查询的报告

+ igmp_gq_start_timer()

##### 16.8.5、V1和V2的报告以及V3的当前状态记录报告

+ igmp_start_timer()

##### 16.8.6、主动发送组关系报告

+ igmp_ifc_event()

#### 16.9、维护套接口组播状态

#### 16.10、维护网络设备组播状态

#### 16.11、ip_mc_source()

+ **阻塞或开通组播源**

#### 16.12、ip_mc_msfilter()

+ 设置组播源过滤列表

#### 16.13、网络设备组播硬件地址的管理

+ ip_mc_filter_add()实现，**在完成套接口和网络设备配置之后被调用**

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

+ 涉及文件
  + net/ipv4/arp.c
  + include/linux/if_arp.h

#### 18.1、ARP报文格式

18.2、系统参数

#### 18.3、注册ARP报文类型

#### 18.4、ARP初始化`arp_init()`

+ `arp_init()`，[**net/ipv4/arp.c**](https://elixir.bootlin.com/linux/latest/source/net/ipv4/arp.c#L1463)

#### 18.5、ARP的邻居项函数指针表，`struct neigh_ops arp_generic_ops`

+ 还是在`arp.c`中

#### 18.6、ARP表，`struct neigh_table arp_tbl`

+ `struct neigh_table arp_tbl`

#### 18.7、函数

18.7.1、arp_error_report()

18.7.2、arp_solicit()

18.7.3、

18.7.4、arp_filter()

#### 18.8、IPv4中邻居项的初始化，`arp_constructor`

+ [arp_constructor](https://elixir.bootlin.com/linux/latest/source/net/ipv4/arp.c#L130)

#### 18.9、ARP报文的创建，`arp_create()`

#### 18.10、ARP的输出，`arp_send()`

#### 18.11、ARP的输入，`arp_rcv()`

18.11.1、arp_rcv()

18.11.2、arp_process()

#### 18.12、ARP代理

+ *没太懂，啥是ARP代理*

##### 18.12.1、apr_process()

18.12.2、arp_fwd_proxy()

18.12.3、parp_redo()

#### 18.13、ARP的ioctl

#### 18.14、外部事件，`arp_netdev_event()`

+ `arp_netdev_event()`，[**net/ipv4/arp.c**](https://elixir.bootlin.com/linux/latest/source/net/ipv4/arp.c#L1272)

#### 18.15、路由表项与邻居项的绑定，`arp_bind_neighbour()`

+ `arp_bind_neighbour()`，*没找到*

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

+ *本节的结构体定义，主要在`include/net/ip_fib.h`中*

+ 2.6.20版本支持两种查找路由表项的算法
  + FIB_HASH，系统默认算法
  + FIB_TRIE，**在超大路由表的情况下可以提高查找效率**
+ 路由表是由fib_table结构来描述的，所有的fib_table结构链接在全局散列表fib_table_hash中

##### 19.3.5、fib_info结构

##### 19.3.6、fib_nb结构

#### 19.4、路由表的初始化`fib_hash_init`

+ `fib_hash_init()`，*这个竟然没找到*

#### 19.5、netlink接口

##### 19.5.1、netlink路由表项消息结构

##### 19.5.2、inet_rtm_newroute()

+ [**net/ipv4/fib_frontend.c**,](https://elixir.bootlin.com/linux/latest/source/net/ipv4/fib_frontend.c#L885)

##### 19.5.3、

#### 19.6、获取指定的路由表`fib_new_table()`

+ `fib_new_table()`，[**include/net/ip_fib.h**](https://elixir.bootlin.com/linux/latest/source/include/net/ip_fib.h#L305)，跟这里面的static是对得上的
+ fib_frontend.c中的有点差异，*这个可能是另一种调用场景*

#### 19.7、路由表项的添加`fib_hash_insert()`

+ *我感觉是不是hash的方式，V6的版本不用了*

19.8、路由表的删除

#### 19.9、外部事件

##### 19.9.1、网络设备状态变化事件`fib_netdev_event()`

+ `fib_netdev_event()`，[**net/ipv4/fib_frontend.c**](https://elixir.bootlin.com/linux/latest/source/net/ipv4/fib_frontend.c#L1463)

##### 19.9.6、fib_magic()

#### 19.10、选路

+ *从代码命名的角度来猜测，选路，一般得是route.c了*

##### 19.10.1、输入选路：ip_route_input_slow()

+ **路由时，一般用`ip_route_input()`选路，当缓存中没有时**，调用`ip_route_input_slow()`在路由表中查找，找到就**加入缓存中**，[**net/ipv4/route.c**](https://elixir.bootlin.com/linux/latest/source/net/ipv4/route.c#L2224)

##### 19.10.4、fib_lookup()

##### 19.10.5、fn_hash_lookup()

+ *这个没找到*

#### 19.11、ICMP重定向消息的发送`ip_rt_send_redirect()`

+ `ip_rt_send_redirect()`，[**net/ipv4/route.c**](https://elixir.bootlin.com/linux/latest/source/net/ipv4/route.c#L874)