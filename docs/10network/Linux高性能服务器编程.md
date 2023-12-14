## 《Linux高性能服务器编程》

+ Part01、TCP/IP协议详解（1-4）
+ Part02、深入解析高性能服务器编程（4-15）
  + 网络编程API（5-7）
    + 6高级IO函数（pipe、dup、dup2、readv、writev、sendfile、mmap、munmap、splice、tee、fcntl）
  + 高性能程序的一般框架（8）
  + 服务器程序的IO单元（9-12）：IO事件、信号、定时、libevent
  + 服务器程序的逻辑单元（13-15）：多进程、多线程
+ Part03、高性能服务器优化与监测（16-17）

### chap1、TCP/IP协议族

#### 1.1、TCP/IP协议族体系结构以及主要协议
##### 0
+ 四层，**主要是`socket()`上面的是应用层，（传输层及以下，是内核层）**，*这是之前没有注意到的,lionel*
##### 1.1.1、数据链路层
+ ARP,RARP
##### 1.1.2、网络层
+ 数据包的选路和转发。
+ **IP协议使用逐跳（hop by hop）的方式确定通信路径**
+ 两类IP、ICMP
+ ICMP分为两种：差错报文；查询报文
##### 1.1.3、传输层
+ 为两台主机上的应用程序提供端到端（end to end）的通信
+ TCP、UDP、SCTP
##### 1.1.4、应用层
+ 处理应用程序的逻辑
+ telent是协议，ospf，dns
+ ping只是命令，不是协议，用的是ICMP协议。
#### 1.2、封装

#### 1.3、分用

#### 1.4、测试网络

#### 1.5、arp协议工作原理
##### 1.5.1、以太网arp请求/应答报文详解
##### 1.5.2、arp高速缓存的查看和修改
+ `arp -a`  【arp -d删除，-s添加】
##### 1.5.3、使用tcpdump观察arp通信过程
+ `tcpdump -i eth0 -ent 'dst 192.168.1.109 and src 192.168.1.108'`
+ **tcpdump抓取的数据包本质上是以太网帧**

#### 1.6、dns工作原理
##### 1.6.1、dns查询和应答报文详解
##### 1.6.2、linux下访问dns服务
+ `more /etc/resolv.conf`
+ `host -t A www.baidu.com`
##### 1.6.3、使用tcpdump观察dns通信过程
+ `tcpdump -i eth0 -nt -s 500 port domain`

#### 1.7、socket与tcp/ip协议族的关系
+ **数据链路层、网络层、传输层协议都是在内核中实现的**，OS就需要实现一组系统调用，使得应用程序能够访问这些协议提供的服务。
+ socket的这组API有两点功能：
	+ 1、将应用程序数据从用户缓冲区中复制到TCP/UDP内核缓冲区，以交付内核来发送数据（或读取数据）
	+ 2、应用程序可以通过它们来修改内核中各层协议的某些头部信息或其他数据结构，从而精细地控制底层通信的行为


### chap2、ip协议详解

#### 0、
+ 从2方面探讨：
	+ IP头部信息
	+ IP数据报的路由和转发
#### 2.1、IP服务的特点
+ 为上层协议提供**无状态、无连接、不可靠的服务**
#### 2.2、IPv4头部结构
##### 2.2.1、IPv4头部结构（20字节*8=160位），IP头部最长60字节

+ 4位版本号
+ 4位头部长度
+ 8位服务类型
+ 16位总长度
+ 16位标识
+ 3位标志
+ 13位片偏移
+ 8位生存时间
+ 8位协议
+ 16位头部校验和
+ 32位源端IP地址
+ 32位目的端IP地址
+ 选项，最多40字节

##### 2.2.2、使用tcpdump观察ipv4头部结构
+ `tcpdump -ntx -i lo` 抓取本地回路上的数据包
#### 2.3、IP分片
+ **当IP数据报的长度超过帧的MTU时，它将被分片传输**（分片可能发生在发送端、可能在中转路由器上，也可能在传输过程中）
+ IP头部中的三个字段给IP分片和重组提供了足够的信息：**数据报标识、标志和位偏移**，每个分片都有自己的头部，**但具有不同的位偏移**，除了最后一个分片外，其它分片都得将**设置MF标志**
+ **以太网帧的MTU是1500字节（减去IP头部20字节，最多1480字节）** 【`ifconfig`看了下,`lo`是65535字节，其它是1500，`netstat -i`也可以看到】
#### 2.4、IP路由
##### 2.4.1、ip模块工作流程

+ **[图2-3](https://blog.csdn.net/Big_Tree712/article/details/130324057)**，*相当于把这本书抄下来了*

##### 2.4.2、路由机制
+ `route`或`netstat`的命令
+ Flags字段
  + U，该路由项是活动的
  + H，该路由项的目标是一台主机
  + G，该路由项的目标是一台网关
  + D，该路由项是由重定向生成的
  + M，该路由项被重定向修改过
##### 2.4.3、路由表更新

+ 可以通过命令 静态更新
  + route add
  + route del
+ 动态更新
  + BGP，Border Gateway Protocol（边际网关协议）
  + RIP，Routing Information Protocol（路由信息协议）
  + OSPF

#### 2.5、IP转发
+ 要找开一下`/proc/sys/net/ipv4/ip_forward`内核参数默认被设置为0
+ 对于允许IP转发的系统（主机或路由器），数据报转发子模块将对期望转发的数据报执行如下操作：
  + 检查数据报头部的TTL值。如果TTL值已经是0，则丢弃该数据报
  + 查看数据报头部的严格源路由选择选项。被设置，就检测数据报的目标IP地址是否是本机的某个IP地址。如果不是，则发送一个ICMP源站选路失败报文给发送端
  + 如果有必要，则给源端发送一个ICMP重定向报文，以告诉它一个更合理的下一跳路由器
  + 将TTL值减1
  + 处理IP头部选项
  + 如果有必要，则执行IP分片操作
#### 2.6、重定向
##### 2.6.1、icmp重定向报文
+ `/proc/sys/net/ipv4/conf/all/send_redirects`，内核参数指定是否允许发送ICMP重定向报文
+ `/proc/sys/net/ipv4/conf/all/accept_redirects`，接收ICMP重定向报文
##### 2.6.2、主机重定向实例
#### 2.7、IPv6头部结构
##### 0、
+ RFC 2460
##### 2.7.1、ipv6固定头部结构（40字节大小）
+ 4位版本号
+ 8位通信类型
+ 20位流标签
+ 16位净荷长度
+ 8位下一个包头
+ 8位跳数限制
+ 128位源端IP地址
+ 128位目的端IP地址
##### 2.7.2、ipv6扩展头部

### chap3、tcp协议详解

#### 0
+ 以下4个方面讨论TCP协议
	+ TCP头部信息（管理TCP连接，控制两个方向的数据流）
	+ TCP状态转移过程（TCP连接的任意一端都是一个状态机）
	+ TCP数据流（交互数据流、成块数据流）
	+ TCP数据流的控制（超时重传、拥塞控制）
#### 3.1、tcp服务的特点
+ TCP连接是全双工的，**即双方的数据读写可以通过一个连接进行**。
+ TCP协议是一对一的，所以广播和多播不能使用TCP服务
+ 发送端执行的写操作次数和接收端执行的读操作次数之间没有任何数量关系，**这就是字节流的概念**
+ [图3-1，TCP字节流服务，图3-2，UDP数据报服务](https://www.cnblogs.com/zhongqifeng/p/14837644.html)，*通过这2个图来理解一下，lionel*
#### 3.2、[tcp头部结构](https://blog.csdn.net/Big_Tree712/article/details/130345727)
##### 3.2.1、tcp固定头部结构
##### 3.2.2、tcp头部选项
+ 头部选项最多40个字节（头部最长60字节，减去 20字节的固定部分）
+ 7个选项，kind的取值
	+ 0
	+ 1
	+ 2
	+ 3
	+ 4
	+ 5
	+ 8
##### 3.2.3、使用tcpdump观察tcp头部信息
#### 3.3、tcp连接的建立和关闭
##### 3.3.1、使用tcpdump观察tcp连接的建立和关闭
##### 3.3.2、半关闭状态
##### 3.3.3、连接超时
+ `/proc/sys/net/ipv4/tcp_syn_retries`中定义了重连次数
#### 3.4、tcp状态转移
##### 3.4.1、tcp状态转移总图
##### 3.4.2、time_wait状态
#### 3.5、复位报文段
##### 3.5.1、访问不存在的端口
##### 3.5.2、异常终止连接
+ `socket`选项中的`SO_LINGER`来发送复位报文段，以异常终止连接
##### 3.5.3、处理半打开连接
#### 3.6、tcp交互数据流
+ **仅包含很少的字节**，对实时性要求高
#### 3.7、tcp成块数据流
+ `tcpdump -nt -i eth0 port 20` #vsftpd服务器程序使用端口号20
#### 3.8、tcp带外数据
+ **用于迅速通告对方本端发生的重要事件**，比普通数据（也叫带内数据）有更高的优先级
+ 利用的是TCP头部中的紧急指针标志和紧急指针两个字段
#### 3.9、tcp超时重传
+ iperf是一个测量网络状况的工具
#### 3.10、拥塞控制
##### 3.10.1、拥塞控制概述
+ rfc5681
+ 拥塞控制的四个部分
  + 慢启动（slow start）
  + 拥塞避免（congestion avoidance）
  + 快速重传（fast retransmit）
  + 快速恢复（fast recovery）
+ 拥塞控制的最终受控变量是发送端向网络一次连续写入（收到其中每一个数据的确认之前）的数据量，我们称为SWND（Send Windows，发送窗口）
##### 3.10.2、慢启动和拥塞避免

+ 发送端判断拥塞发生的依据有如下两个：
  + 传输超时，或者说TCP重传定时器溢出
  + 接收到重复的确认报文段

##### 3.10.3、快速重传和快速恢复	

### chap4、tcp/ip通信案例：访问internet上的Web服务器

#### 4.1、实例总图
#### 4.2、部署代理服务器
##### 4.2.1、http代理服务器的工作原理
##### 4.2.2、部署squid代理服务器
#### 4.3、使用tcpdup抓取传输数据包
#### 4.4、访问dns服务器
#### 4.5、本地名称查询
#### 4.6、HTTP通信
##### 4.6.1、http请求
##### 4.6.2、http应答
#### 4.7、实例总结

### chap5、Linux网络编程基础API

#### 0、

+ 3个方面讨论Linux网络API
  + socket地址API
  + socket基础API
  + 网络信息API

#### 5.1、socket地址api

##### 5.1.1、

+ 5-1：判断机器字节序

```c
#include <stdio.h>
void byteorder() {
union{
short value;
char union_bytes[sizeof(short)];
} test;
test.value = 0x0102;
if((test.union_bytes[0] == 1) && (test.union_bytes[1] == 2)){
printf("big endian\n");
} else if((test.union_bytes[0] == 2) && (test.union_bytes[1] == 1)){
printf("little endian\n");
}else{
printf("unknown...\n");
}
}
```


##### 5.1.2、通用

##### 5.1.3、专用

##### 5.1.4、IP地址转换函数

#### 5.8、数据读写

##### 5.8.1、tcp数据读写

+ 5-6：发送带外数据

```c
#include<sys/socket.h>
#include<netinet/in.h>
#include<arpa/inet.h>
#include<asser.h>
#include<stdio.h>
#include<unistd.h>
#include<string.h>
#include<stdlib.h>

int main(int argc, char *argv[]){
    if(argc <= 2){
        printf("usage: %s ip_address port_number\n", basename(argv[0]));
        return 1;
    }
    const char* ip = argv[1];
    int port = atoi(argv[2]);
	struct sockaddr_in server_address;
	bzero(&server_address, sizeof(server_address));
	server_address.sin_family = AF_INET;
	inet_pton(AF_INET, ip, &server_address.sin_addr);
}
```

+ 5-7：接收带外数据

##### 5.8.2、udp数据读写

##### 5.8.3、通用数据读写函数

#### 5.9、带外标记

+ Linux内核检测到TCP紧急标志时，将通知应用程序有带外数据需要接收。
+ 内核通知应用程序带外数据到达的两种常见方式是：
  + I/O复用产生的异常事件
  + SIGURG信号

#### 5.11、socket选项

#### 5.12、网络信息api

##### 5.12.3、getaddrinfo

### chap6、高级I/O函数

#### 6.1、pipe函数

#### 6.2、dup函数和dup2函数

#### 6.3、readv函数和writev函数

+ readv函数将数据从文件描述符读到分散的内存块中，即**分散读**
+ writev函数，**集中写**

#### 6.4、sendfile函数

+ **在两个文件描述符之间直接传递数据（完全在内核中操作）**，从而避免了内核缓冲区和用户缓冲区之间的数据拷贝，效率很高，称为**零拷贝**。
+ `#include <sys/sendfile.h>  ssize_t sendfile(int out_fd, int in_fd, off_t *offset, size_t count);`

#### 6.5、mmap函数和munmap函数

+ **mmap函数用于申请一段内存空间**。我们可以将这段内存作为进程间通信的共享内存，也可以将文件直接映射到其中。

#### 6.6、splice函数

+ **splice函数用于在两个文件描述符之间移动数据，也是零拷贝操作**
+ `#include<fcntl.h>  ssize_t splice(int fd_in, loff_t *off_in, int fd_out, loff_t* off_out, size_t len, unsigned int flags);`

#### 6.7、tee函数

+ **tee函数在两个管道文件描述符之间复制数据，也是零拷贝操作**。它不消耗数据，因此源文件描述符上的数据仍然可以用于后续的读操作。

#### 6.8、fcntl函数

+ `#include <fcntl.h>  int fcntl(int fd, int cmd, ...);`

### chap7、Linux服务器程序规范
#### 7.1、日志

##### 7.1.1、linux系统日志

+ rsyslogd守护进程既能接收用户进程输出的日志，又能接收内核日志。
+ 终端输出内核信息`dmesg`
+ 内核`printk()`->内核环状缓存 ->`/proc/kmsg`
+ 用户过程`syslog()`->`/dev/log`->`syslogd`->配置文件`/var/log/*`
7.1.2、syslog日志
+ **`syslog()`函数与rsyslogd守护进程通信**
+ **`openlog()`函数改变syslog的默认输出方式，进一步结构化日志内容**

#### 7.2、用户信息

##### 7.2.1、UID、EUID、GID和EGID

##### 7.2.2、切换用户

+ 代码7-2：切换用户
```c
static bool switch_to_user(uid_t user_id, gid_t gp_id){
//先确保目标用户不是root
if((user_id == 0) && (gp_id == 0)) {
	return false;
}
//确保当前用户是合法用户：root或者目标用户
gid_t gid = getgid();
uid_t uid = getuid();
if(((gid != 0) || (uid != 0)) && ((gid != gp_id) || (uid != user_id))){
return false;
}
//如果不是root，则已经是目标用户
if(uid != 0){
return true;
}
//切换到目标用户
if((setgid(gp_id) < 0) || (setuid(user_id) < 0)){
return false;
}
}
```
#### 7.3、进程间关系

##### 7.3.1、进程组

##### 7.3.2、会话

##### 7.3.3、用ps命令查看进程关系

+ `ps -o pid,ppid,pgid,sid,comm | less`
	+ bash（fork出来2个，ps和less 命令）
7.4、系统资源限制
```c
#include<sys/resource.h>
int getrlimit(int resource, struct rlimit *rlim);
int setrlimit(int resource, const struct rlimit *rlim);

struct rlimit{
rlim_t rlim_cur;  //指定资源的软限制
rlim_t rlim_max;  //指定资源的硬限制
}
```
7.5、改变工作目录和根目录

+ chdir()
```c
#include <unistd.h>
char* getcwd(char *buf, size_t size);
int chdir(const char* path);
```

#### 7.6、服务器程序后台化

+ daemon
+ 代码清单7-3：将服务器程序以守护进程的方式运行
```c
bool daemonize() {
//创建子进程，关闭父进程，这样可以使程序在后台运行
pid_t pid = fork();
if(pid < 0) { return false;}
else if(pid > 0) {exit(0);}
//设置文件权限掩码。当进程创建文件（使用open()）时，文件的权限将是mode&07777
umask(0);
//创建新的会话，设置本进程为进程组的首领
pid_t sid = setsid();
if(sid < 0) {return false;}
//切换工作目录
if((chdir("/")) < 0) {return false;}
//关闭标准输入设备、输出设备、错误输出设备
close(STDIN_FILENO);close(STDOUT_FILENO);close(STDERR_FILENO);
//关闭其它已经打开的文件描述符，代码省略

//将标准输入、标准输出和标准错误输出都定向到 /dev/null文件
open("/dev/null", O_RDONLY);
open("/dev/null", O_RDWR);
open("/dev/null", O_RDWR);
return true;
}
```

### chap8、高性能服务器程序框架

+ 将服务器解构为三个主要模块：
  + I/O处理单元：4种I/O模型+2种高效事件处理模式
  + 逻辑单元：2种高效并发模式，**有限状态机**
  + 存储单元：

#### 8.1、服务器模型

+ C/S模型
  + **TCP/IP协议在设计和实现上并没有客户端和服务器的概念，在通信过程中所有机器都是对等的**。
  + 由于客户连接请求是**随机到达的异步事件**，服务器需要使用某种I/O模型来监听这一事件（**`select`系统调用**）  
  + 图8-2  （142/363）
+ P2P模型
  + 缺点：
    + 1、当用户之间传输的请求过多时，网络的负载将加重
    + 2、**主机之间很难互相发现**，需要带有一个**专门的发现服务器**

#### 8.2、服务器编程框架

+ I/O处理单元是**服务器管理客户连接的模块**。
+ 一个逻辑单元是**一个进程或线程**
+ 网络存储单元
+ 请求队列是**各单元之间的通信方式的抽象**

#### 8.3、I/O模型

+ ref（解释了一些名词）
  + [高性能服务器程序框架 总结](http://www.ninesnow.cn/2020/08/%e9%ab%98%e6%80%a7%e8%83%bd%e6%9c%8d%e5%8a%a1%e5%99%a8%e7%a8%8b%e5%ba%8f%e6%a1%86%e6%9e%b6-%e6%80%bb%e7%bb%93/)
  + 阻塞与非阻塞的重点在于状态（**当前线程是否挂起**），同步和异步的重点在于消息通讯机制（异步**无结果也返回，真有结果时通过一些方式通知调用者**）

+ 阻塞与非阻塞
+ 阻塞I/O：
+ **非阻塞I/O通常要和其他I/O通知机制一起使用，比如I/O复用和SIGIO信号**。
+ I/O复用：**通过I/O复用函数向内核注册一组事件，内核通过I/O复用函数把其中就绪的事件通知给应用程序**
  + `select()、poll()、epoll_wait()`
+ 信号驱动I/O：**靠`SIGIO`信号**
+ **从理论上来说，阻塞I/O、I/O复用和信号驱动I/O都是同步I/O模型**。
+ **异步模型**：`aio.h`

#### 8.4、两种高效的事件处理模式

+ 服务器程序通常需要处理三类事件：**I/O事件、信号及定时事件**。

##### 8.4.1、Reactor模式（同步I/O模型）

+ **Reactor模式**：它要求主线程（I/O处理单元）只负责监听文件描述符上是否有事件发生，有的话就立即将该事件通知工作线程（逻辑单元）。除此之外，**主线程不做任何其他实质性工作**。
+ 图8-5（147/363）
+ 使用同步I/O模型（epoll_wait为例）实现的Reactor模式的工作流程是：
  + 1）主线程往epoll内核事件表中注册socket上的读就绪事件
  + 2）主线程调用epoll_wait等待socket上有数据可读
  + 3）当socket上有数据可读时，epoll_wait通知主线程。主线程则将socket可读事件放入请求队列
  + 4）睡眠在请求队列上的某个工作线程被唤醒，它从socket读取数据，并处理客户请求，然后往epoll内核事件表中注册该socket上的写就绪事件
  + 5）主线程调用epoll_wait等待socket可写
  + 6）当socket可写时，epoll_wait通知主线程。主线程将socket可写事件放入请求队列。
  + 7）睡眠在请求队列上的某个工作线程被唤醒，它往socket上写入服务器处理客户请求的结果

##### 8.4.2、Proactor模式

+ **Proactor模式**：将所有I/O操作都交给主线程和内核来处理，工作线程仅仅负责业务逻辑。
+ 图8-6（147/363）
+ 使用异步I/O模型（以aio_read和aio_write为例）实现的Proactor模式的工作流程：
  + 1）主线程调用aio_read函数向内核注册socket上的读完成事件，并告诉内核用户读缓冲区的位置，以及读操作完成时如何通知应用程序（这里以信号为例，详情参考sigevent的man手册）
  + 2）主线程继续处理其他逻辑
  + 3）当socket上的数据被读入用户缓冲区后，内核将向应用程序发送一个信号，以通知应用程序数据已经可用
  + 4）应用程序预先定义好的信号处理函数选择一个工作线程来处理客户请求。工作线程处理完客户请求之后，调用aio_write函数向内核注册socket上的写完成事件，并告诉内核用户写缓冲区的位置，以及写操作完成时如何通知应用程序（仍然以信号为例）
  + 5）主线路继续处理其他逻辑
  + 6）当用户缓冲区的数据被写入socket之后，内核将向应用程序发送一个信号，以通知应用程序数据已经发送完毕
  + 7）应用程序预先定义好的信号处理函数选择一个工作线程来做善后处理，比如决定是否关闭socket

##### 8.4.3、同步的方式模拟出Proactor模式

#### 8.5、两种高效的并发模式

##### 8.5.1、半同步/半异步模式

##### 8.5.2、领导者/追随者模式

#### 8.6、有限状态机

#### 8.7、提高服务器性能的其它建议

##### 8.7.1、池

+ **池**：是一组资源的集合，这组资源在服务器启动之初就被完全创建好并初始化，这称为**静态资源分配**。
  + 内存池
  + 进程池
  + 线程池
  + 连接池

##### 8.7.2、数据复制

+ 当两个工作进程之间要传递大量的数据时，我们就应该考虑使用共享内存来在它们之间直接共享这些数据，而不是使用管道或者消息队列来传递。

##### 8.7.3、上下文切换和锁

### chap9、I/O复用

+ **I/O复用使得程序能同时监听多个文件描述符**
+ 网络程序需要使用I/O复用技术
  + 客户端程序要同时处理多个socket，**非阻塞connect技术**
  + 客户端程序要同时处理用户输入和网络连接
  + TCP服务器要同时处理监听socket和连接socket
  + 服务器要同时处理TCP请求和UDP请求
  + 服务器要同时监听多个端口，或者处理多种服务
+ **I/O复用虽然能同时监听多个文件描述符，但它本身是阻塞的**。当多个文件描述符同时就绪时，如果不采取额外的措施，程序就只能按照顺序依次处理其中的每一个文件描述符，**使得服务器程序看起来像是串行工作的**。如果要实现并发，只能使用多进程或多线程

#### 9.1、select系统调用

+ select系统调用的用途是：**在一段时间内，监听用户感兴趣的文件描述符上的可读、可写和异常等事件**。

##### 9.1.1、select api

```c
#include <sys/select.h>
int select(int nfds, fd_set *readfs, fd_set *writefds, fd_set* exceptfds,
		  struct timeval* timeout);
```

+ nfds，指定被监听的文件描述符的总数
+ readfs，writefs，exceptfs，可读，可写，异常事件对应的文件描述符集合
+ **fd_set结构体**，容纳的文件描述符数量由FD_SETSIZE指定，这就限制了select能同时处理的文件描述符的总量
+ timeout，是超时时间

##### 9.1.2、文件描述符就绪文件

+ 下列socket可读
  + socket内核接收缓存区中的字节数大于或等于其低水位标记SO_RCVLOWAT
  + socket通信的对方关闭连接
  + 监听socket上有新的连接请求
  + socket上有未处理的错误
+ 下列socket可写
  + socket内核发送缓存中的可用字节数大于或等于其低水位标记SO_SNDLOWAT
  + socket的写操作被关闭
  + socket使用非阻塞connect连接成功或失败（超时）之后
  + socket上有未处理的错误
+ **网络程序中，select能处理的异常情况只有一种：socket上接收到带外数据**。

##### 9.1.3、处理带外数据（out-of band OOB)

+ *带外数据，会让socket的返回值异常状态？*

#### 9.2、poll系统调用

+ 与select类似，**指定时间内轮询一定数量的文件描述符**，以测试其中是否有就绪者

```c
#include <poll.h>
int poll(struct pollfd* fds, nfds_t nfds, int timeout);
```

+ fds是**pollfd结构类型的数组**，指定所有我们感兴趣的文件描述符上发生的可读、可写和异常事件

```c
struct pollfd{
    int fd;//文件描述符
    short events;//注册的事件 【监听的事件】
    short revents;//实际发生的事件，由内核填充
};
```

+ nfds，指定被监听事件集合fds的大小
+ timeout，poll的超时值

#### 9.3、epoll系列系统调用

##### 9.3.1、内核事件表

+ epoll是Linux特有的I/O复用函数。它在实现和使用上与select和poll有很大差异。
  + epoll使用一组函数来完成任务，而不是单个函数。
  + epoll把用户关心的文件描述符上的事件放在内核里的一个事件表中，从而无须像select和poll那样每次调用都要重复传入文件描述符集或事件集。
  + **epoll需要使用一个额外的文件描述符，来唯一标识内核中的这个事件表**。
+ 这个额外的文件描述符用`epoll_create()`创建

```c
#include<sys/epoll.h>
int epoll_create(int size);
```

+ 操作epoll的内核表的函数是`epoll_ctl()`

```c
#include<sys/epoll.h>
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
```

+ epfd，要操作的文件描述符
+ op，指定操作类型
  + EPOLL_CTL_ADD，往事件表中注册fd上的事件
  + EPOLL_CTL_MOD，修改fd上的注册事件
  + EPOLL_CTL_DEL，删除fd上的注册事件

##### 9.3.2、epoll_wait函数

+ 作用是：**在一段超时时间内等待一组文件描述符上的事件**。

```c
#include <sys/epoll.h>
int epoll_wait(int epfd, struct epoll_event *events, int maxevents,
			  int timeout)
```

##### 9.3.3、lt和et模式

+ epoll对文件描述符的操作有两种模式：
  + LT（Level Trigger 电平触发）：默认的工作方式，这种模式下相当于一个效率较高的poll。
  + ET（Edge Trigger 边沿触发）：当epoll_wait检测到其上有事件发生并将此事件通知应用程序后，应用程序必须立即处理该事件，因为后续的epoll_wait调用将不再向应用程序通知这一事件。

##### 9.3.4、epolloneshot事件

#### 9.4、三组I/O复用函数的比较

#### 9.5、I/O复用的高级应用一：非阻塞connect

#### 9.6、I/O复用的高级应用二：聊天室程序

##### 9.6.1、客户端

##### 9.6.2、服务端

#### 9.7、I/O复用的高级应用三：同时处理tcp和udp服务

#### 9.8、超级服务xinetd

##### 9.8.1、xinetd配置文件

+ 采用`/etc/xinetd.conf`主配置文件和`/etc/xinetd.d`目录下的子配置文件

##### 9.8.2、xinetd工作流程

+ 有时间日期服务daytime
+ `cat /var/run/xinetd.pid`
+ 图9-1（195/363）

chap10、信号

chap11、定时器

chap12、高性能I/O框架库Libevent

### chap13、多进程编程

#### 0

+ 复制进程映像的fork系统调用和替换进程映像的exec系列系统调用
+ 僵尸进程以及如何避免僵尸进程

#### 13.1、fork系统调用

+ 子进程的代码与父进程完全相同，同时它还会复制父进程的数据（堆数据、栈数据和静态数据）。
+ 数据的复制采用的是所谓的写时复制（copy on write），即只有在任一进程（父进程或子进程）对数据执行了写操作时，复制才会发生（先是缺页中断，然后操作系统给子进程分配内存半复制父进程的数据）

#### 13.2、exec系列系统调用

#### 13.3、处理僵尸进程

#### 13.4、管道

+ 管道能在父、子进程间传递数据，利用的是fork调用之后两个管道文件描述符（fd[0]和fd[1]）都保持打开。

#### 13.5、信号量

13.5.1、信号量原语

+ P（passeren，传递，进入临界区）和V（vrijgeven，释放，退出临界区）

+ sys/sem.h

  13.5.2、semget系统调用

+ **创建信号量集**
  13.5.3、semop系统调用
  13.5.4、semctl系统调用
  
+ **允许调用者对信号量进行直接控制**
  13.5.5、特殊键值ipc_private

#### 13.6、共享内存

##### 0、

+ sys/shm.h

13.6.1、shmget系统调用

13.6.2、shmat和shmdt系统调用

13.6.3、shmctl系统调用

13.6.4、共享内存的posix方法

13.6.5、共享内存实例

#### 13.7、消息队列

13.7.1、msgget系统调用
13.7.2、msgsnd系统调用
13.7.3、msgrcv系统调用
13.7.4、msgctl系统调用

#### 13.8、ipc命令

+ `ipcs`
+ `ipcrm`命令来删除遗留在系统中的共享资源

#### 13.9、在进程间传递文件描述符

+ **传递一个文件描述符并不是传递一个文件描述符的值，而是要在接收进程中创建一个新的文件描述符，并且该文件描述符和发送进程中被传递的文件描述符指向内核中相同的文件表项**。
+ 代码13-5：在进程间传递文件描述符

```c
int main()
{
	int pipefd[2];
	int fd_to_pass = 0;
	//创建父、子进程间的管道，文件描述符pipefd[0]和pipefd[1]都是UNIX域socket
	int ret = socketpair(PF_UNIX, SOCK_DGRAM, 0, pipefd);
	assert(ret != -1);
	pid_t pid = fork();
	assert(pid >= 0);
	if(pid == 0){
		close(pipefd[0]);
		fd_to_pass = open("test.txt",O_RDWR,0666);
		//子进程通过管道将文件描述符发送到父进程。如果文件test.txt打开失败，则子进程将标准输入文件描述符发送到父进程
		send_fd(pipefd[1], (fd_to_pass > 0)?fd_to_pass:0);
		close(fd_to_pass);
		exit(0);
	}
	close(pipefd[1]);
	fd_to_pass = recv_fd(pipefd[0]); //父进程从管道接收目标文件描述符
	char buf[1024];
	memset(buf,'\0',1024);
	read(fd_to_pass, buf, 1024); //读目标文件描述符，以验证其有效性
	printf("I got fd %d and data %s\n", fd_to_pass, buf);
	close(fd_to_pass);
}
```

### chap14、多线程编程

+ POSIX线程同步方式：**POSIX信号量、互斥锁和条件变量**

#### 14.1、linux线程概述

##### 14.1.1、线程模型

+ 一个进程可以拥有M个内核线程和N个用户线程，其中M<=N。
+ 并且在一个系统的所有进程中，M和N的比值都是固定的。
+ 按照M:N的取值，线程的实现方式可分为三种模式：
  + 完全在用户空间实现
  + 完全由内核调度
  + 双层调度（two level scheduler）

##### 14.1.2、linux线程库

#### 14.2、创建线程和结束线程

+ pthread_create
+ pthread_exit
+ pthread_join
+ pthread_cancel

```c
#include <pthread.h>
int pthread_cancel(pthread_t thread);
```

#### 14.3、线程属性

#### 14.4、posix信号量

+ **`pthread_join`可以看作一种简单的线程同步方式**，它无法高效地实现复杂的同步需求，比如控制对共享资源的独占式访问，又抑或是在某个条件满足之后唤醒一个线程。
+ POSIX信号量函数的名字是以`sem_`开头，常用的是下面5个

```c
#include <semaphore.h>
int sem_init(sem_t *sem, int pshard, unsigned int value);
int sem_wait(sem_t *sem);
```

#### 14.5、互斥锁

##### 14.5.1、互斥锁基础api

##### 14.5.2、互斥锁属性

##### 14.5.3、死锁举例

#### 14.6、条件变量

+ 如果说互斥锁是用于同步线程对共享数据的访问的话，那么条件变量则是**用于在线程之间同步共享数据的值**。
+ 条件变量提供了一种**线程间的通知机制**：当某个共享数据达到某个值的时候，唤醒等待这个共享数据的线程。
+ 5个函数

#### 14.7、线程同步机制包装类

#### 14.8、多线程环境

##### 14.8.1、可重入函数

+ 如果一个函数能被多个线程同时调用且不发生竞态条件，则我们称它是**线程安全的（thread safe）**，或者说它是**可重入函数**。

##### 14.8.2、线程和进程

+ **子进程只拥有一个执行线程**

##### 14.8.3、线程和信号

+ **每个线程都可以独立地设置信号掩码**。`sigprocmask()`

### chap15、进程池和线程池

15.1、进程池和线程池概述

#### 15.2、处理多客户

+ 监听socket和连接socket是否都由主进程统一管理？
+ **半同步/半反应堆模式**是由主进程统一管理这两种socket的
+ **半同步/半异步**（领导者/追随者）模式，则由主进程管理所有监听socket，而各个子进程分别管理属于自己的连接socket。

#### 15.3、半同步/半异步进程池实现

+ 代码清单15-1

15.4、用进程池实现的简单cgi服务器

15.5、半同步/半反应堆线程池实现

15.6、用线程池实现的简单web服务器

### chap16、

### chap17、系统监测工具

#### 17.1、tcpdump

+ tcpdump**支持表达式**进一步过滤数据包
  + 类型
  + 方向：`tcpdump dst port 13579`
  + 协议：`tcpdump icmp`

17.2、lsof

17.3、nc

17.4、strace

17.5、netstat

17.6、vmstat

17.7、ifstat

17.8、mpstat