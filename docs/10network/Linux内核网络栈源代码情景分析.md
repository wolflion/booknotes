## 《Linux内核网络栈源代码情景分析》

+ part3、网络栈实现分析
  + chap2、BSD socket层
    + net/socket.c中的**`sys_socketcall()`是总入口函数**，socket.c基本没啥处理，都直接调用af_inet.c中的
  + chap3、INET socket层实现分析
    + **为啥叫INET层**，（函数以inet字符开头，**INET域表示层的操作接口**）

### chap0、网络栈总体架构分析

#### 0.1、网络栈本质及其分层架构

#### 0.2、系统调用接口到内核的请求传递

##### 0.2.1、第一层入口：accept.S

##### 0.2.2、第二层入口：socket.S

##### 0.2.3、第三层入口：entry.S

### chap1、网络协议头文件分析

+ 基本都在include/linux目录下面

#### 1.1、etherdevice.h

#### 1.2、icmp.h

### chap2、BSD socket层实现分析

+ 总体还是按代码中的顺序逐行来解读的

#### 2.2、socket.c文件

+ net/socket.c

+ 网络栈的最上层实现函数，`INT $0x80`进入内核执行函数，**传输层才可以处理具体的请求，所以传输层之上通常都是检查字段或标志位**
+ 传输层L4，应用层L5，**socket.c中的定义的文件是L6**，是系统调用层与网络栈函数集合的接口层。

##### 2.2.1、头文件声明、全局变量定义、相关函数声明

+ *记下，file_operations*里的各种定义，知道各种差异？
+ **sockets_in_use**，Statistics counters of the socket lists，*不太清楚，写得啥意思*
+ 跟2.6.18已经对不上了

##### 2.2.2、move_addr_to_kernel和move_addr_to_user函数

+ 调用`copy_from_user()`和`copy_to_user()`

##### 2.2.4、sockfd_lookup

+ `socki_lookup()`应该已经改成了`sock_from_file()`，还有个`SOCKET_I()`

##### 2.2.5、sock_alloc

+ 诉求就是，分配空间，然后再初始化，*只不过实现的api不一样了*

##### 2.2.6、sock_release和sock_close

+ *重点是实现*

##### 2.2.7、网络套接字普通文件接口函数

+ *没具体看*

##### 2.2.15、sock_getsockname

+ 改成了`sys_getsockname()`

##### 2.2.21、sys_socketcall

+ **总入口函数**

##### 2.2.22、sock_register和sock_unregister

##### 2.2.24、sock_init

##### 2.2.26、socket.c文件小结

+ net/socket.c与其下层INET网络而言，**对应inet/af_inet.c文件中定义的函数，对应关系几乎是一一对应的**
+ socket.c直接调用af_inet.c中

#### chap3、INET socket层实现分析

#### 3.1、af_inet.c文件

+ net/inet/af_inet.c（**在net/ipv4目录中**）
+ **作为L5（表示层）的实现**
+ socket结构主要使用在L6层，sock结构贯穿L2-L5层（net/sock.h）

##### 3.1.1、头文件声明、相关宏及变量定义

##### 3.1.8、inet_autobind

##### 3.1.9、inet_listen

##### 3.1.10、inet_create

##### 3.1.11、inet_release

##### 3.1.12、inet_bind

##### 3.1.17、inet_accept

##### 3.1.18、inet_getname

##### 3.1.21、inet_shutdown

##### 3.1.22、inet_ioctl

##### 3.1.24、INET层操作函数集定义

+ `struct proto_ops inet_stream_ops`，*2.6里定义的比较多，还搞不清楚具体哪个变量，对应哪一个呢*

##### 3.1.26、af_inet.c文件小结

+ **INET层只是作为请求的过渡层，并不进行请求的实际处理**

### chap4、传输层实现分析

#### 4.1、tcp.c

4.2、tcp.h

4.3、udp.c

4.4、udp.h

4.5、sock.h

4.6、sock.c

4.7、datagram.c

4.8、icmp.c

4.9、icmp.h

4.10、igmp.c

4.11、snmp.h

4.12、protocol.h

4.13、protocol.c

4.14、proc.c

### chap5、网络层实现分析

#### 5.1、route.h

5.2、route.c

#### 5.3、ip.h

#### 5.4、ip.c

##### 5.4.1、头文件声明、变量定义

##### 5.4.2、ip_ioctl

5.5、ip_fw.c

#### 5.6、raw.c

##### 5.6.1、raw_err

##### 5.6.2、raw_rcv

#### 5.7、raw.h

#### 5.8、packet.c

##### 5.8.1、packet_recv

### chap6、链路层实现分析

#### 6.1、dev_mcast.c

##### 6.1.1、dev_mc_add

6.2、p8022.h

6.3、p8022call.h

6.4、datalink.h

6.5、p8022.c

6.6、psnap.h

6.7、psnapcall.h

6.8、psnap.c

6.9、eth.c

6.10、eth.h

6.11、p8023.c

6.12、arp.c

6.13、arp.h

6.14、devinit.c

#### 6.15、dev.c

##### 6.15.1、头文件声明、系统变量定义

##### 6.15.15、dev.c文件小结

##### 网络栈实现小结

### chap7、网络设备驱动程序分析

+ drivers/net目录

#### 7.1、关键变量、函数定义及网络设备驱动初始化

+ drivers/net/Space.c
+ drivers/net/net_init.c

#### 7.2、网络设备驱动程序结构

+ 系统调用我们注册的初始化函数，在初始化函数中完成**device结构的创建**，内核提供了`ether_setup()`
+ 为了与网络栈衔接，驱动程序定义发送、接收、中断，**总之围绕device结构而展开**，剩下就是对具体寄存器的操作

#### 7.3、本章小结

+ 从device结构变成了net_device结构

### chap8、系统网络栈初始化

#### 8.1、网络栈初始化流程

+ net/socket.c中的`sock_init()`
+ net/socket.c中的`proto_init()`
+ net/inet/af_inet.c中的`inet_proto_init`

#### 8.2、数据包传送通道解析

+ 数据包接收通道
  + 1、硬件坚挺物理传输介质，当完全接收后，产生中断
  + 2、系统调用驱动程序程序注册的中断处理程序，数据包从驱动层传递给链路层
  + 3、数据包队列
  + 4、网络下半部分执行函数`net_bh`
  + 5、假设使用IP协议
  + 6、假设是TCP传输
  + 7、用户，根据文件描述符得到对应的节点，由节点得到socket结构，从sock结构receive_queue指向的队列中取数据包，将数据包中数据拷贝到用户缓冲区，从而完成数据的读取
+ 数据包发送通道

#### 8.3、本章小结