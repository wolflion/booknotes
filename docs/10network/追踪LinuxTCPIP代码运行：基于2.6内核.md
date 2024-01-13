## 《追踪Linux TCP/IP代码运行：基于2.6内核》

+ 2.6.26内核

### chap1、本书的计划

#### 1.1、基本路线和要求

##### 1.1.3、分析路线

+ **通过socket程序全面掌握整个网络的工作过程**

+ 服务端执行路线
  + socket，bind，listen【与connect】，accept，read【对应write】，write【对应read】，close
  + *就是socket后，bind啥？，listen【监听有哪些连接】，把它们接收过来accept，再read/wirte后，关闭close*
+ 客户端执行路线
  + socket，connect【与listen】，write【与read】，read【与wirte】，close
  + *简单的说，就是socket，然后connnect，再write/read，最后close*
+ 代码1.1
  + *最后要close掉2个fd，也就是有多少个fd，就需要close掉多少？*

1.2、TCP/IP协议层的划分与基本知识

#### 1.3、函数到系统调用的过程

#### 1.4、网络文件系统

+ net/socket.c中的`sock_fs_type`
  + 此处是**虚拟文件系统**
  + *虚拟文件系统，与，file_system_type的意义，自己可以搜一下，本处不是重点*

```c
static struct file_system_type sock_fs_type = {
    .name = "sockfs";
    .get_sb = sockfs_get_sb;
    .kill_sb = kill_anon_super,
};
```



+ 启动时，init/main.c中的`kernel_init()`，继续调用`do_basic_setup()`，从而调用`do_initcalls()`来执行所有的init初始化函数
+ **do_one_initcall()机制**，*这部分，没怎么看*
+ 驱动程序中常用的module_init也是**通过initcall来实现的**
+ `sock_init()`注册成功后，内核初始化会执行它，来登记网络文件系统

```c
register_filesystem(&sock_fs_type);
sock_mnt = kern_mount(&sock_fs_type);//安装网络文件系统
```

+ **调用数据结构（sock_fs_type）中的钩子函数`get_sb()`**
+ `get_sb_pseudo()`·调用**超级块操作函数`sockfs_ops`的**

```c
static struct super_operations sockfs_ops = {
    .alloc_inode = sock_alloc_inode,
    .destroy_inode = sock_destroy_inode,
    .statfs = simple_statfs,
};
```



### chap2、socket的创建

#### 0、自己疑问

+ 2.3的协议族，在api用时，表示啥意思，在kernel层，比较难看得懂
+ 2.4能理解，**分配**是sk_alloc()，**初始化**是`sock_init_data()`

#### 2.1、本章几个重要数据结构

+ 1、socket结构的定义
  + include/linux/net.h中的`struct socket{};`
+ 2、sock结构的定义
  + 也是net.h的吗？ `struct sock{};`
+ **公用是socket结构，通用是sock结构，专用是具体协议族**
+ 3、sk_buff结构的定义
  + 也是net.h的吗？`struct sk_buff{};`
+ 4、tcp_sock结构
+ **在“用中理解”的方式**
  + 先编写函数，还是先编写结构
  + **逆向推理结构体是因何产生、因何而用**，这种方式能提高了理解、阅读代码的水平，更能增强逻辑思维的能力

#### 2.2、分配并初始化socket结构

+ net/socket.c中的`sys_socketcall()->sys_socket()->sock_create()->__sock_create()`
  + `sock_alloc()`
    + `SOCKET_I()`
    + `new_inode()`
      + `alloc_inode()->sock_alloc_inode()`

#### 2.3、使用协议族的函数表初始化socket

+ **AF_INET**，net/ipv4/af_inet.c中

#### 2.4、分配并初始化sock结构

+ 分配sock结构是代码，`sk = sk_alloc(net,PF_INET,GFP_KERNEL,answer_prot);`

+ `__sock_create()->inet_create()->sk_alloc()`
  + `sk_prot_alloc()`
  + `sock_lock_init()`
+ `__sock_create()->inet_create()->sock_init_data()`

#### 2.5、TCP协议对sock结构初始化

+ `if(sk->sk_prot->init){ err = sk->sk_prot->init(sk);}`，**这里调用了运输层的钩子函数`init()`**，这里需要结合**结构体tcp_prot**来看

#### 2.6、socket与文件系统的关联

+ 用`retval = sock_map_fd(sock)`来完成收尾工作

+ `sys_socketcall()->sys_socket()->sock_map_fd()`
  + 48（59/596），tcp_sock结构的关联图
  + `sock_alloc_fd()`
  + `sock_attach_fd()`
+ **socket_file_ops结构体**

### chap3、socket地址设置

#### 3.1、地址设置接口

+ *为什么server端要调用bind()*

#### 3.2、地址结构定义

+ *struct sockaddr_in与struct sockaddr能相互转换不？*

#### 3.3、地址类型

#### 3.4、设置地址和端口

#### 3.5、网络空间总管init_net

+ `net_ns_init()`

### chap4、路由

#### 4.1、路由函数表结构及关系图

4.2、路由函数表的初始化

4.3、通过路由函数表查找路由信息

4.4、路由的设置及相关结构的初始化

4.5、基于输出方向的路由表查找与创建

4.6、基于输入方向的路由表查找与创建

### chap5、通知链

+ **内核为了及时响应某些到来的事件，采取了通知链的方式来执行指定函数**

```cpp
struct notifier_block{
    //notifier_call函数指针，表示通知链节点要运行的函数
    int(*notifier_call)(struct notifier_block *, unsigned long, void *);//通知调用的函数
    struct notifier_block *next;//指向下一个通知节点，从而组成链队
    int priority;//优先级
};
```



#### 5.1、设备通知链节点的挂入

+ 利用`devinet_init()`，调用`register_netdevice_notifier()`将节点结构`ip_netdev_notifier`挂入内核的设备通知链netdev_chain队列中

#### 5.2、地址通知链节点的挂入

+ `ip_fib_init()`调用`register_inetaddr_notifier()`通知链节点`fib_inetaddr_notifier`

#### 5.3、通知链的调用和执行

+ **内核中有一个专门调用通知链函数`notifier_call_chain()`**

### chap6、netlink链

6.1、netlink的创建

6.2、注册路由的netlink

6.3、通过netlink通信 

### chap7、监听连接请求

7.1、内核的监听函数

7.2、内核的监听队列

### chap9、准备连接请求

9.1、内核的连接函数

9.2、分配数据包结构和数据块空间

9.3、构建、发送TCP数据包

9.4、进化成IP数据包

9.5、进化成以太网数据包

9.6、发送以太网数据包

### chap10、邻居子系统

+ 邻居是**另一台电脑，还得是同一个局域网**
+ 邻居子系统**为IP网络层和网络设备的链路层提供了地址转换有关系**，也就是说它指明了IP地址向MAC地址的转换关系，这其中当然离不开ARP协议，它就是一种**邻居协议**
+ `struct neigh_table{};`，**描述了邻居协议的参数和函数表**，内核中的arp_tbl就是ARP的邻居表结构

#### 10.1、邻居子系统的初始化

+ `inet_init()`里面调用了`arp_init()`

#### 10.2、查找邻居结构

+ /net/core/neighbour.c中的`neigh_lookup()`

#### 10.3、邻居子系统的发送事件

+ `neigh_resolve_output()`

#### 10.4、邻居子系统的接收处理

+ 先是对ARP头部进行检查，确定无误后最后调用`arp_process()`

### chap11、流量控制

#### 11.1、排队规则的初始化

+ Qdisc（Queueing discipline）
+ `struct Qdisc{};`结构体定义
+ 网卡，drivers/net目录下的dm9000.c为例 【*pci设备先不管*（r8169.c）】
  + 初始化函数`dm9000_probe()`，再注册`register_netdev(ndev);`
+ qdisc的“安装”过程函数，net/sched/sch_generic.c中`dev_init_scheduler()`
  + `dev->qdisc = &noop_qdisc;//初始设置排隐规则结构`，**这是个结构体**，里面挂入了**出队、入队函数**
+ `ifconfig eth0 up`启动过程
  + `ioctl()->sock_ioctl()->dev_ioctl()->SIOCSIFFLAGS->dev_ifsioc()->dev_change_flags()->IFF_UP->dev_open()->dev_activate()`
    + 调用`qdisc_create_dflt()`
      + 再调用`qdisc_alloc()`
      + 再调用`pfifo_fast_init()`

#### 11.2、排队规则的入队和发送

+ 入队，`pfifo_fast_enqueue()`
  + `qdisc_run()`
+ 发送，`qdisc_restart()`，**循环调用**
  + `struct Qdisc *q = dev->qdisc;//取得设备的排队规则`

### 最后

#### 履历

+ 2023-12-08开始看第1个cubi，*主要是觉得用了tc命令后，看到了chpa11*想串一下，同时跟《深入理解linux网络》也有知识点交集的地方。