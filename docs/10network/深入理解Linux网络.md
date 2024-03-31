## 《深入理解Linux网络》

+ 核心4.4，**从用户进程到网卡的详细过程**

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
  
  + **内核在内部创建了一系列socket相关的内核对象**
  + net/socket.c中的`sock_create()`，又调用af_inet.c中的`inet_create()`
    + 中间先是调用`__sock_create()`，然后先分配`sock_alloc()`，调用创建`create()`，**对于AF_INET协议族来说，调用的是`inet_create`**，其它协议族来说，不一定就这个了
  + inet_create中，**根据类型SOCK_STREAM**查找到对于TCP定义的操作方法实现集合inet_stream_ops和tcp_prot，并把它们分别**设置到socket->ops和sock->sk_prot上**
  
  + net/core/sock.c，`sock_init_data()`，**将sock中的sk_data_ready函数指针进行了初始化，设置为默认sock_def_readable**
  
+ **当软中断上收到数据包时会通过调用sk_data_ready函数指针（实际被设置成了sock_def_readable()）来唤醒在sock上等待的进程**。

#### 3.3、内核和用户进程协作之阻塞方式

+ **图3.4，同步阻塞工作流程**
  + 1、创建socket（用户进程）
  + 2、等待接收（等待队列）
  + 3、数据包抵达网卡
  + 4、网卡把帧DMA到内存
  + 5、硬中断通知CPU
  + 6、处理完发出软中断
  + 7、从RingBuffer上摘下数据包
  + 8、收到socket的接收队列中
  + 9、唤醒等待队列上的进程（ksoftirqd内核线程）

##### 3.3.1、等待接收消息

+ recv依赖的底层实现

+ **strace命令跟踪**，clib库函数recv会执行recvfrom系统调用
+ **图3-5，recvfrom系统调用**
  + 1、系统调用recvfrom
  + 2、inet_stream_ops
  + 3、tcp_prot
  + 4、接收队列里数据为空
  + 5、修改当前进程状态
  + 6、主动让出CPU（Linux将调度下一个进程）
+ **recvfrom最后是怎么把自己的进程阻塞掉的**
  + net/socket.c中的`sock_recvmsg()->__sock_recvmsg()->__sock_recvmsg_nosec()`
  + net/ipv4/af_inet.c
  + net/ipv4/tcp.c
  + net/core/sock.c
+ **sk_wait_data是怎样把当前进程给阻塞掉的**
  + 1、定义等待队列并关联current，include/linux/wait.h
  + 2、获取socket等待队列头
  + 3、插入
  + 4、修改当前进程状态
  + 首先在宏
  + 紧接着，
  + 接着调用prepare_to_wait来把新定义的等待队列wait插入sock对象的等待队列，kernel/wait.c
  + 后面当内核收完数据产生就绪事件的时候，就可以查找socket等待队列上的等待项，进而可以找到回调函数和在等待该socket就绪事件的进程了
  + 最后调用sk_wait_event让出CPU，进程将进入睡眠状态，**这会导致一次进程上下文的开销**，而这个开销是昂贵的，大约需要消耗几个微秒的CPU时间

##### 3.3.2、软中断模块

+ **负责接收和处理数据包**的软中断
  + 网络包到网卡后怎么被网卡接收，最后再交由**软中断处理的**
  + **tcp协议的接收函数`tcp_v4_rcv()`**

+ **图3.9**软中断接收数据过程
  + 1、
  + 2、
  + 3、唤醒等待队列上的进程
+ kernel/sched/core.c
+ 软中断（**也就是linux中的ksoftirqd线程**）里收到数据包以后，发现是TCP包就会执行tcp_v4_rcv函数。接着往下，如果是ESTABLISH状态下的数据包，则最终会把数据拆出来放到对应socket的接收队列中，然后调用sk_data_ready来唤醒用户进程。
  + net/ipv4/tcp_ipv4.c中的`tcp_v4_rcv()`，根据收到的网络包的header里的source和dest信息在本机上查询对应的socket。找到了后，直接进入接收的主体函数`tcp_v4_do_rcv()`，**假设是处理ESTABLISH状态下的包**，就又进入`tcp_rcv_established()`，再通过调用`tcp_queue_rcv()`，完成了将接收到的数据放到socket的接收队列上
  + net/ipv4/tcp_input.c，里的了`tcp_queue_rcv()`
  + **调用tcp_queue_rcv接收完成之后，接着调用sk_data_ready来唤醒在socket上等待的用户进程**。
  + lionel，没写呢
  + **在socket上等待而被阻塞的进程就被推入可运行队列里了，这又将产生一次进程上下文切换的开销**

##### 3.3.3、同步阻塞总结

+ 同步阻塞方式接收网络包的整个过程分两步：
  + 1、我们自己代码的代码所在的进程，调用`socket()`会进入内核态创建必要的内核对象
  + 2、硬中断、软中断上下文（系统线程ksoftirqd）
+ **进程3.12**，同步阻塞流程汇总

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

+ *自己想复习的是*
  + 用户进程到网卡的过程，*差不多在chap5，又复习了一下了*

4.1、相关实际问题

#### 4.2、网络包发送过程总览

+ server端口的（read,  send）调用，**调用send之后内核是怎样把数据包发送出去的**
+ Linux源码最重要是要有整体上的把握，**而不是一开始就陷入各种细节**

+ 图4-1，**网络发送过程概览**
  + 1、调用系统调用发送，send
  + 2、内存拷贝
  + 3、协议处理
  + 4、进入驱动RingBuffer
  + 5、实际发送
  + 6、中断通知发送完成
  + 7、清理RingBuffer
+ 图4-2，**网络发送过程**
+ 图4-3，**发送完毕清理**
+ **硬中断最终触发的软中断却是NET_RX_SOFTIRQ，而不是NET_TX_SOFTIRQ**

#### 4.3、网卡启动准备

+ **网卡支持多队列，第一个队列都是由一个RingBuffer表示的**
+ 网卡在启动时最重要的任务之一就是**分配和初始化RingBuffer**

+ drivers/net/ethernet/intel/igb/igb_main.c中的`__igb_open()`
  + **真正的RingBuffer构造过程是在`igb_setup_tx_resources()`中完成的**
+ 一个传输RingBuffer的内部也不仅仅是一个环形队列数组：
  + igb_tx_buffer数组，这个数组是内核使用的，通过vzalloc申请
  + e1000_adv_tx_desc数组，这个数组是网卡硬件使用的，通过dma_alloc_coherent分配

#### 4.4、数据从用户进程到网卡的详细过程

##### 4.4.1、send系统调用实现

+ sendto系统调用，net/socket.c中，**内部真正使用的是sendto系统调用**
+ 主要干了2件事情
  + 1、在内核中把真正的socket找出来，这个里面记录了各种协议栈的函数地址
  + 2、构造一个struct msghdr对象，把用户传入的数据，都装进去
+ 到了，协议栈的`inet_sendmsg()`

##### 4.4.2、传输层处理

###### 传输层拷贝

+ 对于tcp协议来说，那就是net/ipv4/af_inet.c中的`tcp_sendmsg()`
+ **只有满足forced_push(tp)或者skb==tcp_send_head(sk)成立时**，内核才会真正启动发送数据包
  + **如果条件不满足，这次用户要发送的数据只是拷贝到内核就算完事了**

###### 传输层发送

+ net/ipv4/tcp_output.c中的`tcp_write_xmit()`
+ 图4-10，**传输层发送流程**
  + 1、循环获取待发送skb
  + 2、滑动窗口管理
  + 3、克隆skb
  + 4、设置TCP头

##### 4.4.3、网络层发送处理

+ **内核层的发送的实现们于net/ipv4/ip_output.c**

+ net/ipv4/ip_output.c中的`ip_queue_xmit()`
+ **在网络层主要处理路由项查找、IP头设置、netfilter过滤、skb切分（大于MTU的话）**

##### 4.4.4、邻居子系统

+ 位于**网络层和数据链路层中间**，作用是**为网络层提供一个下层的封装，让网络层不必关心下层的地址信息，让下层来决定发送到哪个MAC地址**。
+ 代码位于**net/core/neighbour.c**
+ 主要是**查找或者创建邻居项**
  + **创建邻居项**的时候，有可能会发出实际的arp请求
+ 图4-16，**邻居子系统**

##### 4.4.5、网络设备子系统

+ 图4.17，**网络设备子系统**
  + 1、选择发送队列（根据XPS或自动选择）
  + 2、获取排队规则

##### 4.4.6、软中断调度

+ **软中断**是由内核进程来运行的，该进程会进入net_tx_action函数，在该函数中能获取发送队列，并也最终调用到驱动程序里的入口函数`dev_hard_start_xmit()`
+ net/core/dev.c

##### 4.4.7、igb网卡驱动发送

+ **网络设备子系统中的dev_hard_start_xmit()**，会调用到驱动里的发送函数`igb_xmit_frame()`

#### 4.5、RingBuffer内存回收

+ `igb_poll()`，调用`igb_clean_tx_irq()`
  + 主要干了啥呢，**清理了skb，解除DMA映射**，只有等对方收到ACK后**才会删除skb**

#### 4.6、本章总结

+ Q5、为什么Kafka的网络性能很突出？
  + **采用的是sendfile系统调用来发送网络数据包**（所谓的零拷贝），减少了内核态和用户态之间的频繁数据拷贝

### chap5、深度理解本机网络IO

+ *换成更通俗的问法*：【本机网络IO，**无非就是的收发都在本机的情况**】
  + 1、复习一下跨机网络通信
  + 2、本机发送
  + 3、本机接收
+ *RingBuffer不是很熟*

#### 5.1、相关实际问题

+ LNMP中的nginx和php-fpm进程，就是通过本机来通信的
+ 微服务中的sidecar模式，也是本机网络IO
+ 问题
  + 1、127.0.0.1本机网络IO需要经过网卡吗？ **不需要网卡**【但为啥呢？走的是**虚拟的环回设备lo**】
  + 2、数据包在内核中是什么走向，和外网发送相比流程上有什么差别？  **节约了驱动上的一些开销**
  + 3、访问本机服务时，使用127.0.0.1能比使用本机IP（如192.168.1.1）更快吗？**一样的**【但为啥呢？走的是**虚拟的环回设备lo**】

#### 5.2、跨机网络通信过程

##### 5.2.1、跨机数据发送（chap4介绍了）

+ 数据发送流程
  + 一、用户态（send，1、调用系统调用发送）
  + 二、内核态（send系统调用，2、内存拷贝）
    + skb，3、协议处理  4、进入驱动RingBuffer
    + **传输队列**
    + 7、收到中断后，清理RingBuffer
  + 三、PCI总线，5、实际发送，**6、中断通知CPU，发送完成** 
+ **5-2数据发送源码**
  + 一、应用层，main中的send()
  + 二、系统调用，net/socket.c中的`SYSCALL_DEFINE6(sendto)`，`__sock_sendmsg_nosec(){}`中的`sock_sendmsg{sock->ops->sendmsg()}`
  + 三、协议栈，net/ipv4/af_inet.c中的`inet_sendmsg(){sk->sk_port->sendmsg()}`
    + 协议栈，传输层，net/ipv4/tcp.c中的`tcp_sendmsg()`
    + 协议栈，传输层，net/ipv4/tcp_output.c中的`tcp_transmit_skb()`中的`icsk->icsk_af_ops->queue_xmit()`
    + 协议栈，网络层，net/ipv4/ip_output.c中的`ip_queue_xmit()`
    + 协议栈，网络层，net/ipv4/ip_output.c中的`ip_finish_output2()`
  + 三、邻居子系统，include/net/dst.h中的`dst_neigh_outpu()`
    + 邻居子系统，include/net/neighbour.h中的`neigh_hh_output()`
  + 四、网络设备子系统，net/core/dev.c中的`dev_queue_xmit()`
    + 网络设备子系统，net/core/dev.c中的`dev_hard_start_xmit()`
  + 五、驱动程序，drivers/net/ethernet/intel/igb/igb_main.c中的`igb_xmit_frame()`，以及`igb_xmit_frame_ring()`
  + 六、硬件
+ **图5.3，RingBuffer清理**
  + 一、硬件
  + 二、硬中断，drivers/net/ethernet/intel/igb/igb_main.c中的`igb_msix_ring()`
    + `__napi_schedule()`再调用
  + 三、软中断，net/core/dev.c中的`net_rx_action(){work = n->poll()}`
  + 四、驱动，drivers/net/ethernet/intel/igb/igb_main.c中的`igb_poll()`
    + 驱动，`igb_clean_tx_irq()`

##### 5.2.2、跨机数据接收

+ 接收过程
  + 一、网卡，1、数据帧从外部网络到达网卡
    + 3、**给CPU一个中断**，硬中断通知给CPU
    + 4、CPU响应中断，简单处理后，发出软中断
    + 5、**ksoftirqd线程处理软中断**，调用网卡驱动注册的poll函数开始收包，**进入内核态**
  + 二、PCIe总线，2、网卡把帧DMA到内存
  + 三、内核态，6、帧被从RingBuffer上摘下来保存为一个skb
  + 四、用户态，7、协议层开始处理网络帧，处理后的数据放到socket的接收队列中
  + 8、**内核唤醒用户进程**（*这个流程上，在图上没看懂啊*，跟CPU有啥关系？）
+ 数据接收源码
  + 一、硬件
  + 二、软中断，drivers/net/ethernet/intel/igb/igb_main.c中的
    + net/core/dev.c中的`__napi_schedule()`
  + 三、软中断，net/core/dev.c中的`net_rx_action()`
    + 软中断，驱动，drivers/net/ethernet/intel/igb/igb_main.c中的`igb_poll()`
    + 软中断，驱动，net/core/dev.c中的`deliver_skb()`
  + 四、协议栈，IP层，ipv4/ip_input.c中的`ip_rcv`
    + 协议栈，传输层，net/ipv4/tcp_ipv4.c中的`tcp_v4_rcv()`
    + 协议栈，传输层，net/ipv4/tcp_ipv4.c中的`tcp_rcv_established(){tcp_queue_rcv(); sk->sk_data_ready()}`
  + 五、用户进程，main中的`recvfrom(fd,buff,BUFFSIZE)`

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
+ **对于本机网络IO来说，在local路由表中就能找到路由项，对应的设备都将使用loopback网卡，也就是lo设备**
+ 一、网络层工作流程
  + 1、ip_queue_xmit
    + ip_route_output_ports，查找并设置路由项
    + 设置IP头
  + 2、ip_local_out
    + netfilter过滤统计工作
  + 3、ip_finish_output
    + 大于MTU的话要分片
+ 二、邻居子系统，`dst_neigh_output()`

##### 5.3.2、本机IP路由

+ **选用哪个设备是路由相关函数`__ip_route_output_key()`确定的**
+ **设置本机IP的时候，调用`fib_inetaddr_event()`函数完成设置的**

##### 5.3.3、网络设备子系统

+ 网络设备子系统的入口函数是`dev_queue_xmit()`，net/core/dev.c
+ 一、邻居子系统
+ 二、网络设备子系统
  + dev_queue_xmit()，**选择队列skb入队出队并发送**
  + dev_hard_start_xmit()
+ 三、驱动程序
  + igb_xmit_frame
    + 1、获取可用缓存区，并关联skb
    + 2、dma_map_single构造内存映射

##### 5.3.4、“驱动”程序

+ 对于真实的igb网卡来说，它的驱动代码都在drivers/net/ethernet/intel/igb/igb_main.c文件中
+ drivers/net/loopback.c
  + `loopback_xmit()`
    + 1、剥离skb
    + 2、加入input_pkt_queue
    + 3、触发软中断
+ net/core/dev.c
  + `__napi_schedule()`

#### 5.4、本机接收过程

+ 软中断处理函数`net_rx_action()`
+ 一、软中断，net_rx_action，**调用驱动提供的poll函数**
+ 二、驱动程序，process_backlog，**使用新链表**
  + process_queue，`skb`的队列

#### 5.5、本章总结

+ *自己想的*
  + 细节是**中断**，这个怎么理解呢？
    + 网卡收到数据了，**给CPU一个硬中断**，然后**CPU简单处理后，发出软中断**
    + *这里的，软中断、硬中断*，怎么理解？

### chap6、深度理解TCP连接建立过程

#### 6.2、深入理解listen

##### 6.2.1、listen系统调用

+ net/sock.c中的`listen`
+ **用户态socket_fd只是一个整数而已，内核没有办法直接用**，要先查找socket内核对象`sock = sockfd_lookup_light(fd,&err,&fput_needed);`
+ *入参的backlog是啥意思？*
+ `err = sock->ops->listen(sock, backlog);`

##### 6.2.2、协议栈listen

+ net/ipv4/af_inet.c中的`inet_listen()`
+ **服务端的全连接队列长度**，执行listen函数时传入的backlog和net.core.somaxconn之间较小的那个值。
  + 啥叫*全连接？相对于谁的全？*
+ net/ipv4/inet_connection_sock.c中的`inet_csk_listen_start()`

##### 6.2.3、接收队列定义

+ include/net/inet_connection_sock.h
+ include/net/request_sock.h中的`struct request_sock_queue{};`
+ **全连接队列**，不需要进行复杂的查找工作，accept处理时只要先进先出即可，**以链表的方式管理**
+ **半连接队列**，`listen_opt`，**需要在第三次握手时快速地查找出第一次握手时留存的request_sock对象**，所以需要用**哈希表管理**

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

