## 《嵌入式Linux网络体系结构设计与TCPIP协议栈》

### chap2、Linux 网络包传输的关键数据结构——Socket Buffer

+ 也就是`sk_buff`

#### 2.1 Socket Buffer 设计概述

#### 2.2 Socket Buffer 的构成

2.3 sk_buff 数据域的设计和含义

2.4 操作 sk_buff 的函数

2.5 数据分片和分段

2.6 本章总结

### 第 3 章 网络设备在内核中的抽象——struct net_device 数据结构

3.1 协议栈与网络设备

3.2 struct net_device 数据结构

3.3 struct net_device 数据结构中数据域的功能分类

3.4 函数指针

3.5 本章总结

### chap7、网络层传送--【就是我理解的IP层】

### chap10、套接字层实现

+ 网络应用程序与 TCP/IP 协议栈之间的接口——套接字（Socket）
+ 传输层以上执行的网络功能都是在用户地址空间完成的。

#### 10.1、套接字概述

+ 应用程序如何使用 netlink 机制来控制 TCP/IP 内部协议的操作？*这个不太会*

##### 10.1.1、什么是套接字

+ 1、套接字接口的基本功能
  + 传输数据
  + 为 TCP 管理连接
  + 控制或调节 TCP/IP 协议栈的操作
+ 2、套接字层 API 构成
  + 一部分由网络的功能组成（由 struct prot 数据结构描述），*这个结构，有印象，不太了解*
  + 另一部分包含了一种方式，将网络的操作映射到 Linux 常规 I/O 操作上，这样应用程序人员就可以如调用普通文件 I/O 操作一样使用 TCP/IP 来传送和接收数据（struct proto_ops数据结构描述）

##### 10.1.2、套接字与管理套接字的数据结构

+ 管理套接字传送数据的结构是 Socket Buffer，struct sk_buff 数据结构中存放了套接字接收/发送的数据。
+ 每个程序使用的套接字都有一个 struct socket 数据结构与 struct sock 数据结构的实例。*两者的异同*
  + 1、struct socket 数据结构，net.h中，
  + 2、struct sock 数据结构，sock.h中，
+ **图10-2**，Linux套接字的继承与包含关系，*这个图还是比较重要的*

#### 10.1.3、套接字与文件

+ 跟VFS有关了

#### 10.2、套接字层的初始化

+ 1、套接字层初始化完成的基本任务
+ 2、套接字层初始化代码的实现
  + net/sock.c中的`static int __init sock_init(void){}`

#### 10.3 地址族的值和协议交换表

##### 10.3.1 协议交换表的数据结构

+ include/net/protocol.h中的`struct inet_protosw`
  + 定义了一个数据结构来管理和描述套接字层对应系统调用套接字操作函数块 struct proto_ops，与内核协议相关套接字操作函数块 struct proto 之间的对应关系。

##### 10.3.2、套接字支持多协议栈的实现

+ **图10-4**， AF_INET 协议族的协议交换表

#### 10.4 IPv4 中协议成员注册和初始化

+ 1、套接字层的操作函数块
  + 由函数 inet_register_protosw 注册到 inetsw_array[]协议交换表中的，由函数 inet_unregister_protosw将结构块从协议交换表中移出。
+ 2、协议实例的网络功能函数块
+ 3、协议交换表初始化

#### 10.5 套接字 API 系统调用的实现

##### 10.5.1 系统调用简述

##### 10.5.2 套接字 API 系统调用的实现

+ 1、
+ 2、
+ 3、由内核系统调用函数到协议实例函数
+ 4、内核套接字系统调用函数说明

#### 10.6、创建套接字

##### 10.6.1、sock_create 函数创建套接字

+ __sock_create 函数的任务是为套接字预留需要的内存空间，由sock_alloc函数完成这项功能。
+ 套接字数据结构 struct socket 实际上也是 inode 文件系统节点数据结构的一部分，它是由 sock_alloc 函数调用 new_inode 创建的。

##### 10.6.2、协议族套接字创建函数的管理

+ 内核中所有传输层协议的套接字创建函数结构块组织在 static struct net_proto_family *net_families[NPROTO]数组中
+ 1、协议族套接字创建函数的数据结构
+ 2、协议族套接字创建函数的数组
+ 3、协议族套接字创建函数的实例化
+ 4、将协议族套接字创建函数注册到 net_families 数组中

##### 10.6.3、AF_INET套接字的创建

+ AF_INET 协议族套接字的创建由 inet_create 函数实现，该函数定义在 net/ipv4/af_inet.c 文件中。
+ inet_create 函数定义为静态函数 static，是因为该函数不是直接调用，它是通过 struct net_proto_family 数据结构的 create 函数指针来调用的。

#### 10.7、I/O系统调用和套接字

+ open 系统调用不能用于创建套接字，创建套接字必须使用 socket 库函数调用。
+ struct inode 数据结构由 sock_alloc（定义在 net/socket.c 文件中）函数创建和初始化。
  + sock_alloc函数，由sock_create函数调用
+  SOCKET_I（定义在 include/linux/socket.h文件中）将 inode 映射到 socket，SOCK_INODE 将 socket 映射到 inode。
+ socket_file_ops 的数据域初始化为套接字的 I/O 系统调用函数

#### 10.8、本章总结