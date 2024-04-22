### chap12、网络

+ *书上是从sk_buff，到net_device，再到ip层，再到L4（传输层），然后应用层socket，【我看的时候呢，再好反了一下】，先了解一下sk_buff和net_device基础知识后，从应用层socket下手去看的*
+ 头文件在include/net中，而不是`include/linux`中，主要是太多了

#### 12.1、互联的计算机

#### 12.2、ISO/OSI 和 TCP/IP 参考模型

#### 12.3、通过套接字通信

+ 网卡的运作方式与普通的块设备和字符设备完全不同
  + 一个原因是（所有层次）使用了许多不同的通信协议，为建立连接需要指定许多选项，且无法在打开设备文件时完成这些任务。
+ 套接字现在用于定义和建立网络连接，以便可以用操作inode的普通方法（特别是读写操作）来访问网络。
+ 在创建套接字时，不仅要区分地址和协议族，还要区分基于流的通信和数据报的通信。（对面向流的套接字来说）同样重要的一点是，套接字是为客户端程序建立的，还是为服务器程序建立的。
  + *这是为何？*

##### 12.3.1、创建套接字

+ in.h中的`struct sockaddr_in{};`

##### 12.3.2、使用套接字

+ 1、echo客户端
+ 2、echo服务器

##### 12.3.3、数据报套接字（UDP）

+ UDP与TCP的不同：

#### 12.4、网络实现的分层模型

+ 图12-3，（613/1058），*这个图比较好理解*
  + 用户空间（应用程序，C标准库）
  + 内核应用层（struct socket, struct sock）
  + 传输层（struct proto）
  + 网络层（struct packet_type，特定于协议）
  + 主机到网络层（dev.c，struct net_device，driver.c特定于硬件）
+ 大量的**函数指针**
+ 传输的基本单位是（以太网）帧，网卡以帧为单位发送数据。
  + 以太网帧（mac首部）
    + IP（ip首部）
      + tcp（tcp首部）
        + http（http首部）
  + *这部分是移到sk_buff的指针结构来实现的*

#### 12.5、网络命名空间

+ *这有啥用啊*

#### 12.6、套接字缓冲区

+ skbuff.h中的`struct sk_buff {};`
+ *没细看呢*

##### 12.6.1、

#### 12.7、网络访问层

+ 该层主要负责在计算机之间传输信息，与网卡的设备驱动程序直接协作。
+ *接收和发送是重点，注册，反而看起来更简单些？*

##### 12.7.1、网络设备的表示

+ 1、数据结构
  + netdevice.h中的`struct net_device{};`
+ 2、注册网络设备

##### 12.7.2、接收分组

##### 12.7.3、发送分组

#### 12.8、网络层

+ 该层不仅负责发送和接收数据，还负责在彼此不直接连接的系统之间转发和路由分组。
  + 查找最佳路由并选择适当的网络设备来发送分组，也涉及对底层地址族的处理（如特定于硬件的MAC地址），这是该
    层至少要与网卡松散关联的原因。
  + **在网络层地址和网络访问层之间的指派是由这一层完成的**
+ IP是在1981年正式定义的，rfc791

##### 12.8.1、IPv4

+ **图12-15**，分组穿过互联网络层的路线   （634/1058）

##### 12.8.2、接收分组

##### 12.8.3、交付到本地传输层（ip_defrag、ip_local_deliver）

##### 12.8.4、分组转发（ip_forward）

##### 12.8.5、发送分组（ip_queue_xmit）

+ 1、转移到网络访问层（ip_output）
+ 2、分组分片（ip_fragment）
+ 3、路由
  + 每个接收到的分组都属于下列3类之一：
    + （1）其目标是本地主机
    + （2）其目标是当前主机直接连接的计算机
    + （3）其目标是远程计算机，只能经由中间系统到达
  + **路由的起始点是ip_route_input函数，它首先试图在路由缓存中查找路由**
  + **路由结果关联到一个套接字缓冲区，套接字缓冲区的dst成员指向一个dest_entry结构的实例**，include/net/dst.h中的`struct dst_entry`
  + **neighbour成员**存储了计算机在本地网络中的IP和硬件地址，这可以通过网络访问层直接到达。include/net/neighbour.h中的`struct neighbour{};`

##### 12.8.6、netfilter

##### 12.8.7、IPv6

#### 12.9、传输层

##### 12.9.1、UDP

+ ip_local_deliver负责分发IP分组传输的数据内容，net/ipv4/udp.c中的udp_rcv用于进一步处理UDP数据报
+ 在`__udp4_lib_rcv()`中调用`__udp4_lib_lookup()`，**查找目标套接字**

##### 12.9.2、TCP

+ 讨论TCP协议的3个主要部分
  + 连接建立
  + 连接终止
  + 数据流的按序传输
+ **TCP连接总处于某个明确定义的状态**

+ **TCP状态转换图**，*怎么去记住这个图，lionel*

###### 1、TCP首部

###### 2、接收TCP数据

+ **TCP数据是如何传递到传输层的**，tcp_v4_rcv是TCP层的入口
+ 系统中的每个TCP套接字都归入3个散列表之一，分别接受下列状态的套接字
  + 完全连接的套接字
  + 等待连接（监听状态）的套接字
  + 处于建立连接过程中（使用下文讨论的三次握手）的套接字

###### 3、三次握手

###### 4、被动连接建立（tcp_v4_rcv）

+ **在接收到一个连接请求的SYN分组后触发的**

###### 5、主动连接建立（tcp_v4_connect）

+ 主动连接建立发起时，通过用户空间调用open，发出socksetcall系统调用到内核函数tcp_v4_connect

###### 6、分组传输

###### 7、接收分组（tcp_v4_rcv）

+ tcp_v4_rcv()

###### 8、发送分组

+ tcp_sendmsg()

###### 9、连接终止

#### 12.10、应用层

+ 使用了socketcall系统调用，*是这样嘛，我没太用过啊*
+ 对程序使用的每个套接字来说，都对应于一个socket结构和sock结构的实例。二者分别充当向下（到内核）和向上的（到用户空间）接口。

##### 12.10.1 socket数据结构

+ net.h中的`struct socket{ ...; const struct proto_ops *ops; ...;}`，**socket的定义并未绑定到具体协议。这也说明了为什么需要用proto_ops指针指向一个数据结构，其中包含用于处理套接字的特定于协议的函数**
+ net.h中的`struct proto_ops{}`，**C库函数会通过socketcall系统调用导向上述的函数指针。**。结构中包含的sock指针，指向一个更为冗长的结构，包含了对内核有意义的附加的套接字管理数据。
+ include/net/sock.h中的`struct sock{}`，**内核自身将最重要的一些成员放置到sock_common结构中**，将该结构的一个实例嵌入到struct sock开始处。**系统的各个sock结构实例被组织到一个协议相关的散列表中。**
+ socket结构的ops成员类型为struct proto_ops，而sock的prot成员类型为struct proto
  + include/net/sock.h中的`struct proto`，*一个在include中的sock.h里，一个在net.h里*
  + sock的prot成员类型为struct proto，这里给出的操作用于（内核端）套接字层和传输层之间的通信
  + socket结构的ops成员所包含的各个函数指针则用于与系统调用通信。

##### 12.10.2、套接字和文件

+ fs.h中的`struct inode{}`
+ net/socket.c中的`struct file_operations socket_file_ops = `，**绑定到网络子系统的代码**
+ inode和套接字的关联，是通过下列辅助结构，将对应的两个结构实例分配到内存中的连续位置，include/net/sock.h中的`struct socket_alloc{};`

##### 12.10.3、socketcall系统调用

+ Linux提供了socketcall系统调用，它实现在sys_socketcall中，**17个套接字操作只对应到一个系统调用**
+ net/socket.c中的`sys_socketcall()`，通过里面的`int call`当关键字，用switch来选择
  + sys_bind
  + sys_listen

##### 12.10.4、创建套接字（sys_socket）

+ sys_socket
  + sock_create
    + `__sock_create`
      + sock_alloc
      + `net_families[family]->create`
  + sock_map_fd
+ net.h中的`struct net_proto_family{}`
+ net/socket.c中的`static struct net_proto_family *net_families[NPROTO]`，（sock_register用于向该数据库增加新数据项）
+ 在为套接字分配内存后，刚好调用函数create。inet_create用于因特网连接（TCP和UDP都使用该函数）。它创建一个内核内部的sock实例，尽可能初始化它，并将其插入到内核的数据结构。
+ map_sock_fd为套接字创建一个伪文件（文件操作通过socket_ops指定）。还要分配一个文件描述符，将其作为系统调用的结果返回。

##### 12.10.5、接收数据（sys_recvfrom）

+ sys_recvfrom
  + fget_light，**根据task_struct的描述符表，查找对应的file实例**
  + sock_from_file
  + sock_recvmsg
    + sock->ops->recvmsg
  + move_addr_to_user

##### 12.10.6、发送数据（sys_sendto）

+ sys_sendto
  + fget_light
  + sock_from_file
  + move_addr_to_kernel，**从用户空间复制到内核空间**
  + sock_sendmsg->`sock->ops->sendmsg`

#### 12.11、内核内部的网络通信

##### 12.11.1、通信函数

+ net.h中的kernel开头的函数，比如`kernel_sendmsg()`这种
+ net/socket.c中的kernel开头的函数实现，`kernel_connect(){}`

##### 12.11.2、netlink机制

+ 1、数据结构
  + 指定地址
  + netlink协议族
  + 消息格式
  + 跟踪netlink连接
  + 特定于协议的操作
+ 2、编程接口
  + net/netlink/af_netlink.c中的`netlink_kernel_create()`

#### 12.12、小结