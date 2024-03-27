## 《精通Linux内核网络》

+ 《Linux kernel networking implementation and theory》，apprize.best/linux/kernel/index.html
  + 自己的问题是，*如何搭建一个内核网络栈，调试环境？*

### chap2、Netlink套接字 13（28/564）

+ *以前没怎么听过，chatgpt问了一下（2024-03-22），可以用于IPC，现在没有废弃，但用得场景不多，暂时不投了吧*

#### 2.1、Netlink簇

+ rfc 3549，Linux Netlink as an IP Services Protocol

### chap7、Linux邻居子系统   153（168/564）

+ 负责发现发现当前链路上的结点，并将L3地址转换为L2地址。

#### 7.1、邻居子系统的核心

+ include/net/neighbour.h中的`struct neighbour{};`
+ **内核将L3地址和L2地址之间的映射存储在了被称为接表的数据结构中**，也就是ARP表，include/net/neighbour.h中的`struct neigh_table{};`
  + v4是`arp_tbl`
  + v6是`nd_tbl`，都是上面的结构的实例

##### 7.1.1、创建和释放邻居

+ `_neigh_create()`

##### 7.1.2、用户空间和邻接子系统之间的交互

+ ip neigh命令
+ arp命令

##### 7.1.3、处理网络事件

+ ARP中，用`arp_netdev_event()`

#### 7.2、ARP协议（IPv4）

+ rfc826
+ ARP报头用结构`struct arphdr{};`

7.2.1、ARP：发送请求

##### 7.2.2、ARP：接收请求和应答

#### 7.3、NDISC协议（IPv6）

+ Neighbour Discovery
+ rfc2461

7.3.1、重复地址检测（DAD）

+ Duplicate Address Detection

7.3.2、NIDSC：发送请求

##### 7.3.3、NDISC：接收邻居请求和通知

+ `ndsic_rcv()`

#### 7.4、总结

#### 7.5、快速参考

+ 核心代码
  + net/core/neighbour.c/.h，以及include/uapi/linux/neighbour.h
+ ARP代码
  + net/ipv4/arp.c和.h，以及include/uapi/linux/if_arp.h
+ NDISC代码
  + net/ipv6/ndisc.c和.h

##### 7.5.1、方法

7.5.2、宏

7.5.3、结构neigh_statistics

7.5.4、表

### chap11、第4层协议

+ *把套接字，相关的放到了4层协议里了？*

#### 11.1、套接字

11.2、创建套接字

11.3、用户数据包协议（UDP）