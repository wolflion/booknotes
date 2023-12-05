## 《深入理解Linux网络》

### chap1、绪论

#### 1.1、我在工作中的困惑

##### 1.1.5、访问127.0.0.1过网卡吗？

##### 1.1.6、软中断和硬中断

##### 1.1.7、零拷贝到底是怎么回事

##### 1.1.8、DPDK

#### 1.2、本书内容结构

#### 1.3、一些约定

+ 内核3.10版本

#### 1.4、一些术语

+ hi：CPU开销中**硬中断消耗的部分**
+ si：CPU开销中**软中断消耗的部分**
+ NAPI：2.5内核引入的一种高效网卡数据处理的技术

### chap2、内核是如何接收网络包的

#### 2.1、相关实际问题

+ 1、RingBuffer是什么，RingBuffer为什么会丢包？
+ 2、网络相关的硬中断、软中断都是什么？
+ 5、tcpdump是如何工作的？
+ 6、iptable/netfilter是在哪一层实现的？
+ 8、网络接收过程中的CPU开销如何查看？

#### 2.2、数据是如何从网卡到协议栈的

+ `read()`的系统调用

##### 2.2.1、Linux网络收包总览

+ 网络分层
+ linux内核中，网络设备驱动在`driver/net/ethernet`，目前下有个intel，表明**intel系列网卡**，协议栈模块位于**kernel和net目录下**
+ 内核和网络设备驱动是**通过中断的方式**来处理的，给CPU的相关引脚触发一个电压变化，中断函数分为**上半部分，下半部分**（上半部分，只进行最简单的工作，**快速处理然后释放CPU**，下半部，慢慢处理），**2.4以后，下半部实现方式是软中断ksoftirqd内核线程**（软中断是通过**给内存中的一个变量赋予二进制值以标记有软中断发生**）

+ 图2.2**内核收包路径**
  + 1、
  + 2、网卡把帧DMA到内存
  + 3、硬中断通知CPU
  + 4、
  + 5、
  + 6、
  + 7、协议层开始处理网络帧，处理完后的数据data被放到socket的接收队列中
  + 8、内核唤醒用户进程

##### 2.2.2、Linux启动

###### 创建ksoftirqd内核线程

+ **线程数量，取决于你机器的核数**
+ 系统初始化的时候在kernel/smpboot.c中调用了**smpboot_register_percpu_thread**，该函数进一步执行**spawn_ksoftirqd**（kernel/softirq.c）来创建出softirqd线程
+ ksoftirqd被创建出来后，它就会进入自己的线程循环函数**ksoftirqd_should_run**和**run_ksoftirqd**了
+ include/linux/interrupt.h中**定义了软中断**（不仅仅只有网络）

###### 网络子系统初始化

+ `subsys_initcall()`来初始化各个子系统
+ 为每个CPU都申请一个softnet_data数据结构，这个数据结构里的poll_list用于等待驱动程序将其poll函数注册进来
+ **open_softirq为每一种软中断都注册一个处理函数**。
  + kernel/softirq.c

###### 协议栈注册

+ 对应的实现函数分别是ip_rcv()，tcp_v4_rcv()和udp_rcv()。
+ **内核是通过注册的方式来实现的**。
+ fs_initcall调用inet_init后开始网络协议栈注册，通过inet_init，将这些函数注册到inet_protos和ptype_base数据结构中。
+ **inet_protos记录着UDP、TCP的处理函数地址，ptype_base存储着ip_rcv()函数的处理地址**

###### 网卡驱动初始化

+ 每一个驱动程序，会使用`module_init()`向内核注册一个初始化函数，**当驱动程序被加载时，内核会调用这个函数**。
  + igb网卡的驱动程序代码位于drivers/net/ethernet/intel/igb/igb_main.c中
+ 驱动的pci_register_driver调用完成后，Linux内核就知道了该驱动的相关信息，**比如igb网卡驱动的igb_driver_name和igb_rpobe函数地址，等等**
+ 驱动的probe方法执行的目的**就是让设备处于ready状态**

###### 启动网卡

+ `struct net_device_ops`变量
+ 当启用一个网卡时（ifconfig eth0 up），**net_device_ops变量中定义的ndo_open方法会被调用**。
+ 创建队列

##### 2.2.3、迎接数据的到来

###### 硬中断处理

###### ksoftirqd内核线程处理中断

###### 网络协议栈处理

###### IP层处理

+ 主入口是**ip_rcv**，net/ipv4/ip_input.c，里面的**NF_HOOK()**，其**最后一个参数是ip_rcv_finish()**
  + *想深入研究netfilter可以从搜索NF_HOOK的这些引用处入手，lionel，作者的建议*

##### 2.2.4、收包小结

+ 收包之前的工作
  + 1、创建ksoftirqd线程
  + 2、协议栈注册，每个协议都会将自己的处理函数注册一下
  + 3、网卡驱动初始化，把自己的DMA准备好，把NAPI的poll函数地址告诉内核
  + 4、启动网卡，分配RX，TX队列，注册中断对应的处理函数
+ 数据到来的工作
  + 1、网卡将数据帧DMA到内存的RingBuffer中，然后向CPU发起中断通知

#### 2.3、本章小结

### chap3、内核是如何与用户进程协作的

#### 3.1、相关实际问题

+ 1、阻塞到底是怎么一回事？
+ 2、同步阻塞IO都需要哪些开销？
+ 3、多路复用epoll为什么就能提高网络性能？
+ 4、epoll也是阻塞的？
+ 5、为什么Redis的网络性能很突出？

#### 3.2、socket的直接创建

+ socket是如何在内核中表示的。
  + socket.c中的`sock_create()`，又调用af_inet.c中的`inet_create()`
  + net/core/sock.c，*lionel，这个干啥的？一般都是sock结构体啥的啊，*，这里调用了`sock_init_data()`
+ **当软中断上收到数据包时会通过调用sk_data_ready函数指针（实际被设置成了sock_def_readable()）来唤醒在sock上等待的进程**。

#### 3.3、内核和用户进程协作之阻塞方式

+ **图3.4，同步阻塞工作流程**
  + 1、创建socket（用户进程）
  + 2、等待接收（等待队列）
  + 3、数据包抵达网卡
  + 4、网卡把帧DMA到内存
  + 5、硬中断通知CPU
  + 9、唤醒等待队列上的进程（ksoftirqd内核线程）

##### 3.3.1、等待接收消息

+ recv依赖的底层实现，**strace命令跟踪**，clib库函数recv会执行recvfrom系统调用
+ **图3-5，recvfrom系统调用**

##### 3.3.2、软中断模块

+ **图3.9**
+ net/ipv4/tcp_ipv4.c中的`tcp_v4_rcv()`
+ kernel/sched/core.c

##### 3.3.3、同步阻塞总结

+ 同步阻塞方式接收网络包的整个过程分两步：
  + 1、我们自己代码的代码所在的进程，调用`socket()`会进入内核态创建必要的内核对象
  + 2、硬中断、软中断上下文（系统线程ksoftirqd）

#### 3.4、内核和用户进程协作之epoll

##### 3.4.1、epoll内核对象的创建

+ fs/eventpoll.c中的`epoll_create`，里面有`struct eventpoll{}`的定义，`ep_alloc()`会进行初始化

##### 3.4.2、为epoll添加socket

+ fs/eventpoll.c中的`epoll_ctl`，

##### 3.4.3、epoll_wait之等待接收

+ fs/eventpoll.c中的`epoll_wait`，

##### 3.4.4、数据来了

+ net/ipv4/tcp_ipv4.c中的`tcp_v4_rcv()`
+ net/ipv4/tcp_input.c中的`tcp_rcv_established()`
+ net/core/sock.c中的`sock_def_readable()`

##### 3.4.5、小结

+ **图3.26，epoll原理汇总**

#### 3.5、本章总结

### chap4、内核是如何发送网络包的

4.1、相关实际问题

4.2、网络包发送过程总览

#### 4.3、网卡启动准备

+ drivers/net/ethernet/intel/igb/igb_main.c中的`__igb_open()`
  + **真正的RingBuffer构造过程是在`igb_setup_tx_resources()`中完成的**

#### 4.4、数据从用户进程到网卡的详细过程

##### 4.4.1、send系统调用实现

+ sendto系统调用
+ 主要干了2件事情
  + 1、在内核中把真正的socket找出来，这个里面记录了各种协议栈的函数地址
  + 2、构造一个struct msghdr对象，把用户传入的数据，都装进去

##### 4.4.2、传输层处理

+ 传输层拷贝
+ 传输层发送

##### 4.4.3、网络层发送处理

+ net/ipv4/ip_output.c中的`ip_queue_xmit()`

##### 4.4.4、邻居子系统

##### 4.4.5、网络设备子系统

##### 4.4.6、软中断调度

##### 4.4.7、igb网卡驱动发送

4.5、RingBuffer内存回收

4.6、本章总结

### chap5、深度理解本机网络IO

#### 5.2、跨机网络通信过程

##### 5.2.1、跨机数据发送

##### 5.2.2、跨机数据接收

##### 5.2.3、跨机网络通信汇总

+ send
  + 系统调用
  + 协议栈（传输层、网络层）
  + 邻居子系统
  + 网络设备子系统
  + 驱动程序
  + 网卡
+ recvfrom
  + 进程调度
  + 协议栈（传输层、网络层）
  + 网络设备子系统
  + 驱动程序
  + 软中断
  + 硬中断
  + 网卡

#### 5.3、本机发送过程

##### 5.3.1、网络层路由

+ 网络层入口函数是`ip_queue_xmit()`，net/ipv4/ip_output.c

##### 5.3.2、本机IP路由

+ **选用哪个设备是路由相关函数`__ip_route_output_key()`确定的**
+ **设置本机IP的时候，调用`fib_inetaddr_event()`函数完成设置的**

##### 5.3.3、网络设备子系统

+ 网络设备子系统的入口函数是`dev_queue_xmit()`，net/core/dev.c

##### 5.3.4、“驱动”程序

+ 对于真实的igb网卡来说，它的驱动代码都在drivers/net/ethernet/intel/igb/igb_main.c文件中
+ drivers/net/loopback.c
  + `loopback_xmit()`
+ net/core/dev.c
  + `__napi_schedule()`

#### 5.4、本机接收过程

+ 软中断处理函数`net_rx_action()`

#### 5.5、本章总结

### chap6、深度理解TCP连接建立过程

#### 6.2、深入理解listen

##### 6.2.1、listen系统调用

+ net/sock.c中的`listen`
+ `err = sock->ops->listen(sock, backlog);`

##### 6.2.2、协议栈listen

+ net/ipv4/af_inet.c中的`inet_listen()`
+ **服务端的全连接队列长度**
+ net/ipv4/inet_connection_sock.c中的`inet_csk_listen_start()`

##### 6.2.3、接收队列定义

+ include/net/inet_connection_sock.h
+ include/net/request_sock.h中的`struct request_sock_queue{};`

##### 6.2.4、接收队列申请和初始化

+ net/ipv4/inet_connection_sock.c中的`inet_csk_listen_start()`

##### 6.2.5、半连接队列长度计算

+ net/core/request_sock.c

##### 6.2.6、listen过程小结

+ listen最主要的工作，**申请和初始化接收队列，包括全连接队列和半连接队列**
  + 全连接队列，是个**链表**
  + 半连接队列，是个**哈希表**

#### 6.3、深入理解connect

+ **图6.4 socket数据结构**

##### 6.3.1、connect调用链展开

+ net/sock.c中的`connect`
+ ipv4/af_inet.c中的`inet_stream_connect()`
+ net/ipv4/tcp_ipv4.c中的`tcp_v4_connect()`

##### 6.3.2、选择可用端口

+ net/ipv4/inet_hashtables.c中的`inet_hash_connect()`

##### 6.3.3、端口被使用过怎么办

+ 

##### 6.3.4、发起syn请求

+ net/ipv4/tcp_output.c中的`tcp_connect()`，根据入参sk中的信息，构建一个完美的syn报文，并将它发送出去

##### 6.3.5、connect小结

+ **客户端在执行connect函数的时候，把本地socket状态设置成了TCP_SYN_SENT，选了一个可用的端口，接着发出SYN握手请求并启动重传定时器**。

#### 6.4、完整TCP连接建立过程

##### 6.4.1、客户端connect

##### 6.4.2、服务端响应SYN

##### 6.4.3、客户端响应SYNACK

##### 6.4.4、服务端响应ACK

##### 6.4.5、服务端accept

+ net/ipv4/inet_connection_sock.c中的`inet_csk_accept()`

##### 6.4.6、连接建立过程总结

#### 6.5、异常TCP连接建立情况

##### 6.5.1、connect系统调用耗时失控

##### 6.5.2、第一次握手丢包

##### 6.5.3、第三次握手丢包

##### 6.5.4、握手异常总结

#### 6.6、如何查看是否有连接队列溢出发生

##### 6.6.1、全连接队列溢出判断

##### 6.6.2、半连接队列溢出判断

#### 6.7、本章总结

### chap7、一条TCP连接消耗多大内存

#### 7.1、相关实际问题

7.2、Linux内核如何管理内存

7.3、TCP连接相关内核对象

#### 7.4、实测TCP内核对象开销

#### 7.5、本章小结

### chap8、一台机器最多能支持多少条TCP连接

#### 8.2、理解Linux最大文件描述符

#### 8.3、一台服务端机器最多可以支撑多少条TCP连接

#### 8.4、一台客户端机器最多只能发起65535条连接吗

#### 8.5、单机百万并发连接的动手实验

### chap9、网络性能优化建议

### chap10、容器网络虚拟化

#### 10.2、veth设备对

