## 《Linux内核源码剖析-TCP/IP实现》（下）

+ 2.6.20代码
+ https://lxr.missinglinkelectronics.com/
+ 下册（20-33章）

### chap20、路由缓存  1（10/529）

+ 缓存的主要工作是存储使路由子系统能够找到报文目的地的信息，并通过一组函数向更高层提供该信息。

+ 路由缓存涉及到的文件
  + include/net/route.h，定义目的路由缓存项
  + include/net/flow.h，定义查询路由缓存的条件组合结构、宏和函数原型等
  + include/net/dst.h，定义目的路由缓存项的部分结构、宏、函数等
  + net/ipv4/route.c，实现路由缓存项的操作函数

#### 20.1、系统参数

+ 系统参数（有11个）
  + flush，控制路由缓存的刷新

#### 20.2、路由缓存的组织结构

+ 路由缓存是用一张**散列表**来实现的，路由缓存散列表的类型是**rt_hash_bucket结构**，该结构只包含**指向缓存元素链表的一个指针**。缓存项的类型为rtable结构

##### 20.2.1、rtable结构

+ 查看`/proc/net/rt_cache`文件，或者通过`ip route list cache`和`route -C`来列出路由缓存的内容

##### 20.2.2、flowi结构

+ 根据诸如输入网络设备和输出网络设备、三层和四层协议报头中的参数等字段的组合对流量进行分类。

##### 20.2.3、dst_entry结构

+ dst_entry结构被用于存储缓存路由项中独立于协议的信息。

##### 20.2.4、dst_ops结构

+ dst_ops结构是使用路由缓存的三层协议与独立于协议的缓存之间的接口

#### 20.3、初始化

+ 由`ip_rt_init()`进行初始化的   11（20/529）

#### 20.4、创建路由缓存项

20.5、添加路由表项到缓存中

20.6、输入路由缓存查询

20.7、输出路由缓存查询

20.7.1、ip_route_output_key()

#### 20.8、垃圾回收

#### 20.9、刷新缓存

##### 1.3.1、TSO/GSO

##### 1.3.2、I/O AT

##### 1.4.1、slab分配器

##### 20.9.7、通过写/proc的flush文件

#### 20.10、ICMP重定向消息的处理

#### 20.11、ICMP目的不可达，需要分片

### chap21、路由策略

+ *感觉目录，是从上层往下层的，lionel*

2.1、引言

2.2、协议简介

2.3、网络架构

2.4、系统调用接口

2.5、协议无关接口

#### 21.6、路由策略的查找

### chap22、套接口层  56（65/529）

+ 套接口是啥
  + 1983年，4.2BSD引入
  + **通用的网络应用程序编程接口**
+ 套接口包含啥？
  + *本书好像没讲？*

+ 套接口层的作用
  + **为应用程序提供了一个访问网络和进程间通信的**通用接口
  + 位于**应用程序和协议栈**之间
  + 对应用程序屏蔽了与协议相关实现的具体细节，将应用程序发送的与协议无关的请求映射到与协议相关的实现
+ 套接口层怎么与其上下层衔接
  + 应用程序中调用库函数，而库函数通过系统调用进入套接口层
  + Linux套接口层实现提供了一组**专门的套接口系统调用**
  + **图22-1**（套接口层将一般的请求转换为指定的协议操作）
+ 套接口的I/O操作以及套接口选项（23，24两章介绍的）
+ 涉及文件
  + include/linux/net.h，定义套接口层相关的结构、宏和函数原型
  + include/net/sock.h，定义基本的传输控制块结构、宏和函数原型
  + net/socket.c，实现套接口层的调用
  + net/ipv4/af_inet.c，网络层和传输层接口

#### 22.1、socket结构

+ `struct socket{};`

#### 22.2、proto_ops结构

+ 是一组与套接口系统调用相对应的传输层函数指针，可以看作是**一张套接口系统调用到传输层函数的跳转表**
+ 完成的是**从与协议无关的套接口层到协议相关的传输层**的转接，proto结构又将**传输层映射到网络层**，那么可想而知**每个传输层的协议都需要定义一个特定的proto_ops结构实例和proto结构实例**。

#### 22.3、套接口文件系统

##### 22.3.1、套接口文件系统类型

+ sockfs文件系统类型`sock_fs_type`，通过`get_sb()`和`alloc_inode()`和`destroy_inode()`分配和释放与套接口文件相关的i结点
+ 通过`/proc/filesystem`文件查看操作系统支持的文件系统

##### 22.3.2、套接口文件系统超级块操作接口

##### 22.3.3、套接口文件的inode

+ `struct socket_alloc{};`由socket结构和inode结构两部分组成

##### 22.3.4、sock_alloc_inode()

##### 3.2.5、sock_destroy_inode()

#### 22.4、套接口文件

+ 创建套接口文件时，使file结构中的f_op指向了`socket_file_ops`，**通过socket_file_ops可以看到套接口文件支持哪些系统调用**

##### 22.4.1、套接口文件与套接口的绑定

+ 1、sock_map_fd()
+ 2、sock_attach_fd()

##### 22.4.2、根据文件描述符获取套接口

+ 1、sockfd_lookup_light()
+ 2、sock_from_file()

#### 22.5、进程、文件描述符和套接口

+ 在task_struct结构中，files指向file_struct结构，该结构的主要功能是管理fd_array指针数组指向的描述符，每个file结构描述一个打开的文件

#### 22.6、套接口层的系统初始化

+ `sock_init()`

#### 22.7、套接口系统调用

+ 理解proto_ops结构和proto结构的差异和调用层次

##### 22.7.1、套接口系统调用入口

+ **表22-5**，套接口系统调用

##### 22.7.2、socket系统调用

+ 1、sys_socket()
+ 2、

##### 22.7.3、bind系统调用

+ 1、sys_bind()

##### 22.7.4、listen系统调用

##### 22.7.5、accept系统调用

##### 22.7.6、connect系统调用

##### 22.7.7、shutdown系统调用

+ 1、sys_shutdown
+ 2、套接口层的实现

##### 22.7.8、close系统调用

+ 1、关闭套接口
+ 2、套接口层的实现

##### 22.7.9、select系统调用的实现

### chap23、套接口I/O  91（100/529）

+ 套接口I/O涉及的文件
  + include/linux/socket.h，定义套接口的结构、宏和函数原型
  + net/core/iovec.c，实现对I/O向量块的复制操作
  + net/socket.c，实现套接口层的调用

#### 23.1、输出/输入数据的组织

##### 23.1.1、msghdr结构

+ `struct msghdr{};`

##### 23.1.2、verify_iovec()

+ 发送和接收的数据的msghdr结构需要在用户态组织，**而在内核中是不信任用户态的数据的**，因此需要对用户态提供的msghdr结构进行校难。

##### 23.1.3、

23.1.4、

23.1.5、

##### 23.1.6、csum_partial_copy_fromiovecend()

#### 23.2、输出系统调用

##### 23.2.1、sock_sendmsg()

##### 23.2.2、sendto系统调用

##### 23.2.3、send系统调用

##### 23.2.4、sendmsg系统调用

#### 23.3、输入系统调用

+ 图23-3，recvmsg系统调用过程

#### 23.4、网络设备处理层初始化

### chap24、套接口选项  99（108/529）

+ 套接口选项的实现涉及的文件
  + net/socket.c，实现套接口层的调用
  + net/ipv4/af_inet.c，网络层和传输层接口

#### 24.1、setsockopt系统调用

#### 24.2、ioctl系统调用

##### 24.2.1、ioctl在文件系统内的调用过程

##### 24.2.2、套接口文件ioctl调用接口的实现

##### 24.2.3、套接口层的实现

#### 24.3、getsockname系统调用

#### 24.4、getpeername系统调用

### chap25、传输控制块  111（120/529）

+ 根据协议族和传输层协议的特点，分层次地定义了多个结构用来组成传输控制块
  + sock_common、**传输控制块信息的最小集合**，由sock和inet_timewait_sock结构前面相同部分单独构成
  + sock、比较通用的网络层描述块，构成传输控制块的基础，与具体的协议族无关
  + inet_sock、
  + inet_connection_sock、支持面向连接特性的描述块，
  + tcp_sock、request_sock、

+ 涉及的文件
  + include/net/sock.h，定义基本的传输控制块结构、宏和函数原型
  + include/net/inet_sock.h，定义IPv4专用的传输控制块
  + net/core/sock.c，实现传输层通用的函数
  + net/socket.c，实现套接口层的调用

#### 25.1、系统参数

+ optmem_max，每个传输控制块辅助缓冲区的上限，`sock_kmalloc()`
  + **辅助数据**包括进行设置选项、设置过滤时分配的内存和组播设置
+ rmem_default，传输控制块接收缓冲区大小的上限的默认值
+ rmem_max，传输控制块接收缓冲区大小的上限的最大值
+ vmem_default，传输控制块发送缓冲区大小的上限的默认值
+ vmem_max，传输控制块发送缓冲区大小的上限的最大值

#### 25.2、传输描述块结构

##### 25.2.1、sock_common结构

##### 25.2.2、sock结构

##### 25.2.3、inet_sock结构

+ **是IPv4协议专用的传输控制块，是对sock结构的扩展**，在传输控制块的基本属性已具备的基础上，进一步提供了IPv4协议专有的一些属性

#### 25.3、proto结构

+ proto结构为**网络接口层**，结构中的操作实现传输层的操作和从传输层到网络层调用的跳转，**在proto结构中某些成员跟proto_ops结构中的成员对应**

##### 25.3.1、ptoto实例组织结构

##### 25.3.2、proto_register()

+ **注册proto实例到proto_list链表中**

##### 25.3.3、proto_unregister()

#### 25.4、传输控制块的内存管理

##### 25.4.1、传输控制块的分配和释放

+ 1、sk_alloc()
  + 在创建套接口时，TCP、UDP和原始IP会分配一个传输控制块
+ 2、sk_clone()
+ 3、sk_free()
+ 4、sock_put()

##### 25.4.2、普通的发送缓存区的分配

+ 1、sock_alloc_send_skb()
+ 2、sock_alloc_send_pskb()
+ 3、sock_wait_for_wmem()

##### 25.4.3、发送缓存的分配与释放

+ 1、sock_wmalloc()，**分配发送缓存**
+ 2、skb_set_owner_w()
+ 3、sock_wfree()

##### 25.4.4、接收缓存的分配与释放

+ 1、sk_stream_set_owner_r()和sk_stream_rfree()
+ 2、

##### 25.4.5、辅助缓存的分配与释放

+ 1、sock_kmalloc()，**分配有关选项的缓存**
+ 2、sock_kfree_s()，**释放由sock_kmalloc()分配的缓存**

#### 25.5、异步IO机制

##### 25.5.1、sk_wake_async()

##### 25.5.8、sock_fasync()

+ **实现了对套接口的异步通知队列增加和删除的更新操作**

#### 25.6、网络设备的禁用

##### 25.6.1、socket_lock_t结构

+ 实现控制用户进程和下半部间同步锁和控制下半部间同步锁都是由`socket_lock_t`结构描述的。

##### 25.6.2、控制用户进程和下半部间同步锁

+ 传输控制块通常在两种执行体中执行，**进程上下文**和**软中断上下文**，对传输控制块的访问完全是异步的，因此**为了防止在访问传输控制块时产生冲突，加入了锁机制**。
+ 1、lock_sock()
+ 2、release_sock()
+ 3、sock_owned_by_user宏

##### 25.6.3、控制下半部间同步锁

### chap26、TCP：传输控制协议  149（158/529）

+ TCP的内容分为7章
+ 图26-1，TCP函数调用关系
+ TCP传输控制块的管理、套接口选项、ioctl、差错处理以及缓存管理的文件
  + include/linux/tcp.h
  + include/net/sock.h
  + include/net/inet_connection_sock.h
  + include/net/inet_hashtables.h，定义管理传输控制块的散列表
  + net/ipv4/af_inet.c，网络层和传输层接口
  + net/ipv4/tcp_ipv4.c，传输控制块与网络层之间的接口实现
  + net/ipv4/tcp.c，传输控制块与应用层之间的接口实现
  + net/core/strem.c，TCP中流内存管理的实现

#### 26.1、系统参数

+ 40、tcp_timestamps
+ 46、tcp_workaround_signed_windows

#### 26.2、TCP的inet_protosw实例

+ `struct inet_protosw inetsw_array[] ={.protocol = IPPROTO_TCP}`

#### 26.3、TCP的net_protosw实例

+ `struct net_protocol tcp_protocol ={.handler = tcp_v4_rcv,}`

#### 26.4、TCP传输控制块

+ 3种类型的TCP传输控制块
  + tcp_request_sock
  + tcp_sock，*本章讲这个*，**在连接建立之后终止之前使用，TCP状态为ESTABLISHED**
  + tcp_timewait_sock，**在终止连接过程中使用**

##### 26.4.1、inet_connection_sock结构

+ **所有面向连接传输控制块**，在**inet_sock结构的基础上**，增加了**连接、确认和重传**

##### 26.4.2、inet_connection_sock_af_ops结构

+ 封装了一组**与传输层相关的操作集**，TCP中的实例是**ipv4_specific**

##### 26.4.3、tcp_sock结构

+ **TCP协议的控制块**，是在**inet_connection_sock**基础上，扩展了**滑动窗口协议、拥塞控制算法**

##### 26.4.4、tcp_options_received结构

+ **保存接收到的TCP选项信息**，如时间戳、SACK等

##### 26.4.5、tcp_skb_cb结构

+ skb_buff结构的cb成员，**TCP利用这个字段存储了一个tcp_skb_cb结构**

#### 26.5、TCP的proto结构和proto_ops结构的实例

+ TCP的传输层接口为tcp_prot，另一个是`inet_stream_ops`，*是不是作者字写错了*

#### 26.6、TCP状态迁移图

+ *跟用户态是一样的*

#### 26.7、TCP首部

#### 26.8、TCP校验和

+ TCP的校验和**覆盖TCP首部及TCP数据**，IP首部中的校验和**只覆盖IP的首部**，不覆盖IP数据报中的任何数据
+ 校验和规则，**每16位字取反后相加**，**TCP段都包含一个12B的伪首部**

##### 26.8.1、输入TCP段的校验和检测

###### 1、tcp_v4_checksum_init()

###### 2、tcp_checksum_complete()和tcp_checksum_complete_user()

##### 26.8.2、输出TCP段校验和的计算

+ tcp_v4_send_check()

#### 26.9、TCP的初始化

+ tcp_init()由**IPv4协议族的初始化函数inet_init()调用**

#### 26.10、TCP传输控制块的管理

+ **TCP存在多个状态**

##### 26.10.1、inet_hashinfo结构

+ **tcp_hashinfo对所有的散列表进行集中管理**

##### 26.10.2、管理除LISTEN状态之外的TCP传输控制块

+ tcp_v4_hash()

##### 26.10.3、管理LISTEN状态的TCP传输控制块

#### 26.11、TCP层的套接口选项

+ tcp_setsockopt()

#### 26.12、TCP的ioctl

+ TCP中的几个ioctl命令
  + SIOCINQ
  + SIOCATMARK
  + SIOCOUTQ

#### 26.13、TCP传输控制块的初始化

+ tcp_prot中，将**init接口设置为tcp_v4_init_sock()**

#### 26.14、TCP的差错处理

+ tcp_v4_err()

#### 26.15、TCP传输控制块层的缓存管理

##### 26.15.1、缓存管理的算法

###### 3、

###### 4、等待可用的缓存

##### 26.15.2、发送缓存的管理

###### 1、分配SKB

###### 2、确认发送缓存是否可用

##### 26.15.3、接收缓存的管理

###### 1、确认接收缓存是否可用

###### 2、释放SKB

### chap27、TCP的定时器  196（205/529）

+ *我的问题*，【还是需要理论基础】，*以下都是网上搜集，不代表真实性*
  + **TCP的控制，就靠TCP的定时器**
+ *我归纳的本章介绍*
  + 7个定时器，**每个里面，都分2个小节，一是相关的处理函数，二是激活**
+ TCP为每条连接建立7个定时器，**连接建立、重传、延时ACK、持续、保活、FIN_WAIT_2、TIME_WAIT**
  + **一般内核只用4个就完成了7个功能**
+ 涉及的文件
  + net/ipv4/tcp_time.c，TCP的定时器
  + net/ipv4/inet_connection_sock.c，基于连接的传输控制块实现
  + net/ipv4/tcp_output.c，TCP的输出
  + net/ipv4/tcp_input.c，TCP的输入

#### 27.1、初始化

+ tcp_init_xmit_timers()

#### 27.2、连接建立定时器

##### 27.2.1、连接建立定时器处理函数

###### 1、

###### 3、inet_csk_reqsk_queue_prune()

##### 27.2.2、连接建立定时器的激活

#### 27.3、重传定时器

##### 27.3.1、重传定时器处理函数

###### 1、tcp_write_timer()

###### 3、tcp_write_timeout()

##### 27.3.2、重传定时器的激活

#### 27.4、延时ACK定时器

##### 27.4.1、延时ACK定时器处理函数

##### 27.4.2、延时ACK定时器的激活

#### 27.5、持续定时器

##### 27.5.1、持续定时器处理函数

##### 27.5.2、持续定时器的激活

#### 27.6、保活定时器

##### 27.6.1、保活定时器处理函数

##### 27.6.2、保活定时器的激活

#### 27.7、FIN_WAIT_2定时器

##### 27.7.1、FIN_WAIT_2定时器处理函数

##### 27.7.2、FIN_WAIT_2定时器的激活

#### 27.8、TIME_WAIT定时器

### chap28、TCP连接的建立   217（226/529）

+ 自己搜的
  + 主动，被动、同时
    + **主动**打开，client向server发送SYN
    + 被动打开，server等待client的答复
    + 同时打开，client和server同时发送SYN
  + TCP连接建立过程中有哪些系统调用
    + socket
    + bind
    + listen
    + accept
    + connect
  + **不管主动，被动，三次握手，都是client发SYN，server回SYN+ACK**
  + TCP接收队列、全连接、半连接
    + 接收队列，kernel中接收到但未被应用程序处理的TCP数据包的缓存区
    + **全连接**，已3次握手，双向传输
    + **半连接**，一方只完成了三次握手的前2步，**单向传输**，或者是网络中断，或者是对方拒绝连接

+ TCP连接建立的过程（侦听套接口的IP地址和端口的绑定，套接口的侦听和accept，客户端接口的IP地址和端口的绑定，以及被动打开、主动打开和同时打开的实现过程）
+ 涉及的文件
  + include/linux/tcp.h
  + include/net/request_sock.h
  + include/net/inet_sock.h
  + include/net/inet_connection_sock.h
  + net/ipv4/inet_connection_sock.c，基于连接的传输控制块实现
  + net/ipv4/af_inet.c，网络层和传输层接口

#### 28.1、服务端建立连接过程

+ TCP连接的过程
  + client发送一个SYN段，标识希望连接的服务器端口以及初始序号
  + server回复一个包含服务器初始序号以及对clientSYN段确认的SYN+ACK段作为应答
  + client发送确认序号为服务器初始序号加1的ACK段，对服务器SYN段进行确认

#### 28.2、网络输出软中断

##### 28.2.1、request_sock_queue结构

+ **存放三次握手的信息**

##### 28.2.2、listen_sock结构

+ 存储**连接请求块**，在**listen调用之后才会创建**，request_sock_queue结构中`listen_opt`成员

##### 28.2.3、tcp_request_sock结构

+ **保存双方的初始序号、双方的端口及IP地址、TCP选项，如是否支持窗口扩大因子、是否支持SACK等，并控制连接的建立**

###### 1、inet_request_sock结构

###### 2、request_sock结构

##### 28.2.4、request_sock_ops结构

+ TCP中指向的实例是`tcp_request_sock_ops`

#### 28.3、bind系统调用的实现

##### 28.3.1、bind端口散列表

##### 28.3.2、传输接口层的实现

+ `inet_csk_get_port()`

#### 28.4、listen系统调用的实现

##### 28.4.1、inet_listen()

##### 28.4.2、实现侦听：inet_csk_listen_start()

#### 28.5、accept系统调用的实现

##### 28.5.1、套接口层的实现：inet_accept()

##### 28.5.2、传输接口层的实现：inet_csk_accept()

#### 28.6、被动打开

##### 28.6.1、第一次握手：发送SYN段

##### 28.6.2、第二次握手：接收SYN+ACK段

##### 28.6.3、第三次握手：发送ACK段

#### 28.7、connect系统调用的实现

##### 28.7.1、套接口层的实现：inet_stream_connect

##### 28.7.2、传输套接口层的实现

#### 28.8、主动打开

##### 28.8.1、第一次握手：发送SYN段

##### 28.8.2、第二次握手：接收SYN+ACK段

##### 28.8.3、第三次握手：发送ACK段

#### 28.9、同时打开

##### 28.9.1、SYN_SENT状态接收SYN段

##### 28.9.2、SYN_RECV状态接受SYN+ACK段

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

### chap32、TCP连接的终止 442（451/529）

#### 32.1、连接终止过程

##### 32.1.1、正常关闭

##### 32.1.2、同时关闭

#### 33.2、shutdown传输接口层的实现

##### 33.2.1、tcp_shutdown()

+ 是TCP的shutdown系统调用的传输接口层实现，由套接口层的实现`inet_shutdown()`调用

##### 33.2.2、tcp_send_fin()

+ 过程：
  + 由于发送FIN无需占用额外的负载

#### 32.3、close传输接口层的实现：tcp_close()

#### 32.4、被动关闭：FIN段的接收处理

#### 32.5、主动关闭

##### 32.5.1、timewait控制块的数据结构

+ 1、inet_timewait_death_row结构
+ 2、inet_timewait_sock结构
+ 3、tcp_timewait_sock结构

##### 32.5.2、timewait控制块取代TCP传输控制块

##### 32.5.3、

##### 32.5.4、

##### 32.5.5、FIN_WAIT2和TIME_WAIT状态处理

+ 1、TCP输入入口
+ 2、FIN_WAIT2和TIME_WAIT状态的输入处理

##### 32.5.6、timewait控制块的2MSL超时处理

+ 1、2MSL等待超时时间较短的超时处理
+ 2、2MSL等待超时时间较长的超时处理
  + (1)、tw_timer定时器的例程
  + (2)、twkill_work工作队列例程

### chap33、UDP：用户数据报  473（482/529）

#### 33.1、引言

+ UDP不提供可靠性
+ 涉及的文件
  + include/net/udplite.h，定义轻量级UDP专用的函数
  + include/linux/udp.h，定义UDP传输控制块
  + net/ipv4/udp.c
  + net/ipv4/udplite.c
  + net/core/sock.c，实现传输层通用的函数
  + net/ipv4/datagram.c，实现UDP的connect调用
  + net/ipv4/af_inet.c，网络层和传输层接口

##### 33.1.1、UDP首部

+ 端口号用来标识发送和接收进程

##### 33.1.2、UDP的输入与输出

+ 发送，UDP层  udp_sendmsg->udp_push_pending_frame()
  + 另一个调用是 IP层的 `ip_append_data()`
+ 接收，IP层是，`ip_local_deliver()`
  + UDP层，udp_rcv()->`_udp4_lib_rcv()`->udp_queue_rcv_skb()->sock_queue_rcv_skb()->skb_queue_tail()->`sk->sk_data_ready()`
+ `struct sock`中的**sk_receive_queue**是接收队列

#### 33.2、UDP的inet_protosw结构

+ `struct inet_protosw inetsw_array[] = `
  + 传输层操作接口为`udp_prot`
  + 套接口层操作接口为`inet_dgram_ops`

#### 33.3、UDP的传输控制块

+ `struct udp_sock{}`是对`inet_sock`结构的扩展

#### 33.4、UDP的proto结构和proto_ops结构的实例

+ 传输层操作接口为`udp_prot`
+ 套接口层操作接口为`inet_dgram_ops`
+ *没有扩展*

#### 33.5、UDP的状态

+ UDP的传输是没有状态的

#### 33.6、UDP传输控制块的管理

+ UDP并不是在hash接口中将其传输控制块添加到**udp_hash散列表**中的，而是**在绑定端口后，才将其添加到散列表**，关键字为端口号与散列表大小取模后的值。
+ `udp_sock`，`socket结构`，`struct file结构`

#### 33.7、bind系统调用的实现

+ **UDP的绑定实际上完成两个功能**：
  + 根据选取的合适端口号将传输控制块添加到udp_hash散列表中
  + 并将端口号设置到传输控制块中
+ `inet_bind()`
  + 原始IP，调用bind接口进行绑定
  + 其他的包括TCP和UDP则调用`get_port()`进行绑定
+ 在UDP中实现传输接口层`get_port()`接口的函数是`udp_v4_get_port()`，**真正绑定的是`__udp_lib_get_port()`**

#### 33.8、UDP套接口的关闭

+ UDP的传输接口层的close接口为`udp_lib_close()`，通过对传输接口层的unhash接口的调用，把传输控制块从散列表中删除
  + `sk_common_release()`

#### 33.9、connect系统调用的实现

+ `inet_dgram_connect()`为connect在UDP套接口层的实现

##### 33.9.1、udp_disconnect()

+ 传输接口层UDP的disconnect接口的实现

##### 33.9.2、ipv4_datagram_connect()

+ 传输接口层UDP的connect接口的实现

#### 33.10、select系统调用的实现

+ `udp_poll()`

#### 33.11、UDP的ioctl

+ SIOCOUTQ，返回标识该传输控制块为发送而分配的所有数据区的总大小
+ SIOCINQ，获取在接收队列缓存中第一个未读数据报的长度

#### 33.12、UDP的套接口选项

+ UDP套接口选项入口为`udp_setsockopt()`
  + 如果是`SOL_UDP`和`SOL_UDPLITE`级别，调用`udp_lib_setsockopt()`
  + 否则通过IP接口调用`ip_setsockopt()`

#### 33.13、UDP校验和

##### 33.13.1、输入UDP数据报校验和的计算

+ 1、udp4_csum_init()
  + 用于UDP数据报接收校验的初始化
+ 2、udp_lib_checksum_complete()
  + 基于伪首部累加和，完成全包校验和的检测

##### 33.13.2、输出UDP数据报校验和的计算

+ 1、udp4_hwcsum_outgoing()
+ 2、csum_tcpudp_magic()和udp_csum_outgoing()
  + udp_csum_outgoing()用在硬件不支持完成校验和的情况下

#### 33.14、UDP的输出：sendmsg系统调用

##### 33.14.1、udp_sendmsg()

+ 实现了UDP数据报的组织和发送

##### 33.14.2、udp_push_pending_frames()

+ 将待发送数据打包成一个UDP数据报输出

#### 33.15、UDP的输入

##### 33.15.1、UDP接收的入口：udp_rcv()

+ UDP层的数据接收，对于套接口而言，就是**接收队列的入队操作**，在IP层，如果是发送到本地数据，则会交由`ip_local_deliver_finish()`处理

##### 33.15.2、UDP组播数据报输入：__udp4_lib_mcast_deliver()

##### 33.15.3、udp_queue_rcv_skb()

+ 将UDP数据报添加到所属传输控制块的接收队列中的功能由udp_queue_rcv_skb()来实现

#### 33.16、recvmsg系统调用的实现

+ 实现UDP的传输接口层的recvmsg接口函数为`udp_recvmsg()`

+ udp_recvmsg()实现了主动从传输控制块的接收队列中读取数据到用户空间的缓冲区中。

#### 33.17、UDP的差错处理：udp_err()

#### 33.18、轻量级UDP

+ 在UDP-Lite协议中，一个数据包到底需不需要对其负载进行校验，或者是校验多少位都是由用户控制的
+ `s = socket(PF_INET, SOCK_DGRAM, IPPROTO_UDPLITE);`