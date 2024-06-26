## 《Linux高性能服务器编程》

+ Part01、TCP/IP协议详解（1-4）
+ Part02、深入解析高性能服务器编程（5-15）
  + 网络编程API（5-7）
    + 6高级IO函数（pipe、dup、dup2、readv、writev、sendfile、mmap、munmap、splice、tee、fcntl）
  + 高性能程序的一般框架（8）
    + 服务器程序解构为**IO单元、逻辑单元、存储单元**
  + 服务器程序的IO单元（9-12）：IO事件、信号、定时、libevent
    + chap9(IO复用)，chap10(信号)，chap11(定时器)，chap12(libevent)
    + **Linux服务器程序必须处理的三类事件：IO事件、信号事件、定时事件**
      + IO事件，*是啥呢*
      + 信号事件，*就是触发的信号*
      + 定时事件，*定期检测客户连接的状态*
    + 优秀的开源IO框架库--libevent
  + 服务器程序的逻辑单元（13-15）：多进程、多线程
+ Part03、高性能服务器优化与监测（16-17）
+ *自己想写一个libevent的使用例子，2024-05-27*

### 前言

#### 为什么要写这本书

+ 国内书的问题**内容宽泛而空洞**
  + 囊括了最新技术，**但一个最基本的技术细节，也无法解释清楚**
  + 有些书呢，都是从网络上摘抄，不仅没有自己的观点，**连自己的总结，也没有**
+ 大师们的经典，**只专注于一个问题，对每个技术细节都精雕细琢**

#### 读者对象

+ 有一定的Linux系统编程和C++编程基础

#### 本书特色

+ 作者写过完整的负载均衡服务器程序springsnail
+ pjhq87@gmail.com

### chap1、TCP/IP协议族

#### 1.1、TCP/IP协议族体系结构以及主要协议
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
#### 1.2、封装（encapsulation）

+ 封装，是**在上层数据的基础上加上自己的头部信息**，以实现该层的功能。
+ 经过TCP封装后的数据称为**TCP报文段**（TCP message segment）

#### 1.3、分用（demultiplexing）

+ 分用是**依靠头部信息中的类型字段实现的**，rfc1700
+ IP，ARP用的是帧，**以太网帧类型字段值是`0x800`**，`0x806`是ARP
+ ICMP，TCP，UDP都靠IP协议
+ TCP和UDP靠的是**端口**，所有知名应用层协议使用的端口号都可在`/etc/services`文件中找到

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

+ 从2方面探讨
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

+ 以下4个方面讨论TCP协议
  + TCP头部信息（管理TCP连接，控制两个方向的数据流）
  + TCP状态转移过程（TCP连接的任意一端都是一个状态机）
  + TCP数据流（交互数据流、成块数据流）
  + TCP数据流的控制（超时重传、拥塞控制）
#### 3.1、tcp服务的特点

+ 传输层协议就TCP和UDP，**TCP相对UDP，面向连接、字节流和可靠传输**。

+ **使用TCP协议通信的双方必须先建立连接，然后才能开始数据的读写**。TCP连接是全双工的，**即双方的数据读写可以通过一个连接进行**。
+ TCP协议是一对一的，所以广播和多播不能使用TCP服务
+ 发送端执行的写操作次数和接收端执行的读操作次数之间没有任何数量关系，**这就是字节流的概念**
+ TCP字节流，UDP数据报，两者差异
  + **TCP有个接收缓冲区**
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

+ `sudo tcpdump -i eth0 -nt '(src 192.168.1.109 and dst 192.168.1.108) or (src 192.168.1.108 and dst 192.168.1.109)'`
+ 然后再`telnet 192.168.1.109 80`
+ **图3-6，TCP连接的建立和关闭时序图**

##### 3.3.2、半关闭状态
##### 3.3.3、连接超时

+ iptables命令用于过滤数据包

```shell
sudo iptables -F
sudo iptables -I INPUT -p tcp --syn -i eth0 -j DROP

sudo tcpdump -n -i eth0 port 23
date; telnet 192.168.1.108; date
```



+ `/proc/sys/net/ipv4/tcp_syn_retries`中定义了重连次数
#### 3.4、tcp状态转移
##### 3.4.1、tcp状态转移总图

+ 服务器通过listen系统调用进入LISTEN状态，被动等待客户端连接，然后**向客户端发送SYN标志的确认报文段**，进入SYN_RCVD状态，服务器成功收到客户端回的确认字段，连接转移到**ESTABLISHED状态**。
+ 客户端主动关闭连接（通过close或者shutdown），服务器通过返回确认报文字段进入**CLOSE_WAIT状态**，也就是等待服务器应用程序关闭连接，服务器检测到客户端关闭连接后，给客户端发送一个结束报文段来关闭连接，进入**LAST_ACK状态**，等待客户端确认
+ 客户端通过connect系统调用主动与服务器建立连接。**首先给服务器发送一个同步报文段，使连接转移到SYN_SENT状态**，connect可能会因为如下两个原因失败返回：**失败后，返回到初始的CLOSED状态**。
  + 如果connect连接的目标端口不存在，或者端口处于TIME_WAIT状态，服务器给客户端发送一个复位报文段
  + 如果目标端口存在，但connect在超时时间内未收到服务器的确认报文段
+ 客户执行主动关闭时，它将向服务器发送一个结束报文段，**进入FIN_WAIT_1状态**，客户端收到服务器确认后，**进入FIN_WAIT_2状态**，这时**服务端的状态是CLOSE_WAIT**，如果服务器也关闭，就进入**TIME_WAIT状态**

##### 3.4.2、time_wait状态

+ TIME_WAIT状态存在的原因有两点：
  + 可靠地终止TCP连接
  + 保证让迟来的TCP报文段有足够的时间被识别并丢弃

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

+ **Out Of Band，OOB**带外数据，**用于迅速通告对方本端发生的重要事件**，比普通数据（也叫带内数据）有更高的优先级

+ UDP没有实现带外数据传输，TCP也没有真正的带外数据，**利用的是TCP头部中的紧急指针标志和紧急指针两个字段**
+ TCP发送带外数据的过程：
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
+ 拥塞控制的算法（reno，vegas，cubic），`/proc/sys/net/ipv4/tcp_congestion_control`指示机器当前所使用的拥塞控制算法
+ 拥塞控制的最终受控变量是发送端向网络一次连续写入（收到其中每一个数据的确认之前）的数据量，我们称为SWND（Send Windows，发送窗口）
##### 3.10.2、慢启动和拥塞避免

+ RFC5681中提到2个种实现方式，**使CWND按照线性方式增加，从而减缓扩大**
  + 1、每个RTT时间按`CWND+=min(N,SMSS)`计算新的CWND，而不论该RTT时间内发送端收到多少个确认
  + 2、每收到一个对新数据的确认报文段，按照`CWND+=SMSS*SMSS/CWND`来更新CWND

+ 发送端判断拥塞发生的依据有如下两个：
  + 传输超时，或者说TCP重传定时器溢出
  + 接收到重复的确认报文段

##### 3.10.3、快速重传和快速恢复

+ 发送端如果收到了3个重复的确认字段，就认为拥塞发生了，启用快速重传和快速恢复算法，过程
  + 1、当收到第3个重复报文段时，`ssthresh=max(FlightSize/2,2*SMSS)`，得出ssthresh，立即重传丢失的报文段，设置CWND，`CWND=ssthresh+3*SMSS`
  + 2、每次收到1个重复的确认时，设置`CWND=CWND+SMSS`。
  + 3、当收到新数据的确认时，设置`CWND=ssthresh`，**ssthresh是新的慢启动门限值**

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

#### 5.2、创建socket

#### 5.3、命令socket

#### 5.4、监听socket

#### 5.5、接受连接

#### 5.6、发起连接

#### 5.7、关闭连接

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

##### 5.11.1、so_reusedaddr选项

##### 5.11.2、so_rcvbuf和so_sndbuf选项

##### 5.11.3、so_rcvlowat和so_sndlowat选项

##### 5.11.4、so_linger选项

#### 5.12、网络信息api

##### 5.12.1、gethostbyname和gethostbyaddr

##### 5.12.2、getservbyname和getservbyport

##### 5.12.3、getaddrinfo

##### 5.12.4、getnameinfo

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

+ 代码8-1，状态独立的有限状态机
+ 代码8-2，带状态转移的有限状态机
+ 代码8-3，http请求的读取和分析

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

#### 0、我自己想和解答的【问AI】

+ 1、什么是I/O复用（问的AI）
  + 复用：多个相似的操作，合并为一个
  + I/O复用：多个I/O合并到一个I/O上，*这些都是正确的废话，还是不好理解*
  + 可以理解的解释的是，**一个进程同时监听了多个I/O事件，不用为每个I/O事件创建进程**
  + 举例：web服务器中，传统的情况，需要为每个客户端创建一个进程或线程

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
+ 代码9-1，同时接收普通数据和带外数据

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

+ 代码9-2，

##### 9.3.3、lt和et模式

+ epoll对文件描述符的操作有两种模式：
  + LT（Level Trigger 电平触发）：默认的工作方式，这种模式下相当于一个效率较高的poll。
  + ET（Edge Trigger 边沿触发）：当epoll_wait检测到其上有事件发生并将此事件通知应用程序后，应用程序必须立即处理该事件，因为后续的epoll_wait调用将不再向应用程序通知这一事件。
+ 代码9-3，

##### 9.3.4、epolloneshot事件

+ 代码9-4，

#### 9.4、三组I/O复用函数的比较

+ **表9-2，select、poll和epoll的区别**
  + 事件集合
  + 应用程序索引就绪文件描述符的时间复杂度
  + 最大支持文件描述符数
  + 工作模式
  + 内核实现和工作效率

#### 9.5、I/O复用的高级应用一：非阻塞connect

+ 代码9-5，
+ *非阻塞的一些问题*

#### 9.6、I/O复用的高级应用二：聊天室程序

+ 以poll为例实现一个简单的聊天室程序
  + 客户端功能：
    + 从标准输入终端读入用户数据，并发送至服务器
    + 往标准输出终端打印服务器发送它的数据
  + 服务器功能：
    + 接收客户数据，并把客户数据发送给每一个登录到该服务器上的客户端

##### 9.6.1、客户端

+ 客户端，使用poll同时监听用户输入和网络连接，并利用spilice函数将用户输入内容直接定向到网络连接上以发送之，从而实现数据零拷贝，提高了程序执行效率

+ 代码9-6，

##### 9.6.2、服务端

+ 代码9-7，聊天室服务器

#### 9.7、I/O复用的高级应用三：同时处理tcp和udp服务

+ 代码9-8，

#### 9.8、超级服务xinetd

+ **同时管理着多个子服务，即监听多个端口**

##### 9.8.1、xinetd配置文件

+ 采用`/etc/xinetd.conf`主配置文件和`/etc/xinetd.d`目录下的子配置文件

##### 9.8.2、xinetd工作流程

+ 有时间日期服务daytime
+ `cat /var/run/xinetd.pid`
+ 图9-1（195/363）

### chap10、信号

+ **信号**，是由用户、系统或者进程发送给目标进程的信息，以通知目标进程某个状态的改变或系统异常。
+ Linux信号可由如下条件产生：
  + 对于前台进程，用户可以通过输入特殊的终端字符来给它发送信号，`Ctrl+C`
  + 系统异常，非法内存段
  + 系统状态变化，alarm定时器到期引起SIGALRM信号
  + 运行kill命令或者调用kill函数
+ **服务器程序必须处理（或至少忽略）一些常见的信号**

#### 10.1、Linux信号概述

##### 10.1.1、发送信号

+ `kill(pid_t pid, int sig)`，**信号值都大于0，如果sig为0，则kill函数不发送任何信号**。

##### 10.1.2、信号处理方式

+ `bits/signum.h`，
+ `typedef void (*_sighandler_t)(int)`
+ `signal.h`，`sighandler_t`，中的`SIG_IGN`表示忽略信号，和`SIG_DFL`信号的默认处理方式
+ 信号的默认处理方式
  + 结束进程，Term
  + 忽略信号，Ign
  + 结束进程并生成核心转储文件，Core
  + 暂停进程，Stop
  + 继续进程，Cont

##### 10.1.3、Linux信号

+ bits/signum.h中定义了标准信号
+ **与网络编程关系紧密的几个信号：SIGHUP、SIGPIPE、SIGURG**，后面还有**SIGALRM、SIGCHLD**

##### 10.1.4、中断系统调用

+ **POSIX没有规定这种行为，是Linux独有的**。
+ 处于阻塞状态的系统调用时接收到信号，并且也为该信号设置了信号处理函数，**则默认情况下系统调用被中断**，并且errno被设置为EINTR。

+ `sigaction()`为信号标置`SA_RESTART`

#### 10.2、信号函数

##### 10.2.1、signal系统调用

+ `_sighandler_t signal(int sig, _sighandler_t _handler)`，返回一个函数指针

##### 10.2.2、sigaction系统调用

+ `int sigaction(int sig, const struct sigaction* act, struct sigaction* oact);`
  + sig指出要捕获的信号类型
  + act是新的信号处理方式
  + oact是先前的处理方式
+ **struct sigaction结构体**中的**sa_hander成员指定信号处理函数**，sa_mask，sa_flags成员用于设置程序收到信号时的行为

#### 10.3、信号集

##### 10.3.1、信号集函数

+ **一组信号**用`sigset_t`结构表示

+ `sigset_t`，长整型数组，数组的每个元素的每个位表示一个信号
+ 具体的操作
  + sigemptyset()
  + sigfillset()
  + sigaddset()
  + sigdelset()

##### 10.3.2、进程信号掩码

+ *设置掩码，有什么用，lionel*  【有些信号，不能被进程接收】

+ 利用`sigaction`结构体的`sa_mask`成员来设置进程的信号掩码

+ `sigprocmask()`

##### 10.3.3、被挂起的信号

+ **设置进程信号掩码后，被屏蔽的信号将不能被进程接收**

+ `sigpending()`

#### 10.4、统一事件源

+ **信号是一种异步事件**：信号处理函数和程序的主循环是两条不同的执行路线。
+ 解决方案是：
  + **把信号的主要处理逻辑放到程序的主循环中，当信号处理函数被触发时，它只是简单地通知主循环程序接收到信号，并把信号值传递给主循环**
+ IO框架库和后台服务器程序都统一处理信号和IO事件，比如LibeventIO框架库和xinetd超级服务
+ 代码清单10-1，统一事件源

#### 10.5、网络编程相关信号

##### 10.5.1、SIGHUP

+ **当挂起进程的控制终端时，SIGHUP信号将被触发**，如果没有控制终端，那就**强制服务器重读配置文件**，典型的例子是，xinetd
+ 分析xinetd处理SIGHUP信号的流程，*这个暂时先不看，lionel*
  + 先通过`ps`看下，xinetd创建的子进程
  + `strace`跟踪一程序执行时调用的系统调用和接收到的信号
+ 代码10-2，用strace命令查看xinetd处理SIGHUP的流程

##### 10.5.2、SIGPIPE

+ **默认情况下，往一个读端关闭的管道或socket连接中写数据将引发SIGPIPE信号**
+ 可以用`send()`反馈的errno的值来判断管道或者socket连接的读端是否关闭
+ 也可以用IO复用系统调用

##### 10.5.3、SIGURG

+ Linux环境下，内核通知应用程序带外数据到达主要有两种方法：
  + 1种是chap9的，IO复用技术，**select在接收到带外数据时返回，并向应用程序报告socket上的异常事件**
  + 另一种是使用SIGURG信号
+ 代码10-3，用SIGURG检测带外数据是否到达

### chap11、定时器

+ **网络程序需要处理的第三类事件是定时事件**，比如定期检测一个客户连接的活动状态
+ **两种高效的管理定时器的容器：时间轮、时间堆**
+ **定时**，指在一段时间之后触发某段代码的机制，我们可以在这段代码中依次处理所有到期的定时器。**定时机制是定时器得以被处理的原动力**。
+ Linux提供了三种定时方法
  + 1、socket选项SO_RCVTIMEO和SO_SNDTIMEO
  + 2、SIGALRM信号
  + 3、I/O复用系统调用的超时参数

#### 11.1、socket选项SO_RCVTIMEO和SO_SNDTIMEO

+ 设置socket接收数据超时时间和发送数据超时时间。**仅对数据接收和发送相关的socket专用系统调用有效**
  + send、sendmsg、recv、recvmsg、accept、connect
+ 代码清单11-1，设置connect超时时间，*代码没看呢*

#### 11.2、SIGALRM信号

+ 由alarm和settimer函数设置的实时闹钟一旦超时，将触发SIGALRM信号
+ **一般而言，SIGALRM信号按照固定的频率生成**，即由alarm或settimer函数设置的定时周期T保持不变

##### 11.2.1、基于升序链表的定时器

+ **定时器通常至少要包含两个成员：一个超时时间（相对时间或者绝对时间）和一个任务回调函数**。

+ 代码清单11-2，升序定时器链表

##### 11.2.2、处理非活动连接

+ 代码清单11-3，关闭非活动连接

#### 11.3、I/O复用系统调用的超时参数

+ 代码清单11-4，利用I/O复用系统调用定时

#### 11.4、高性能定时器

##### 11.4.1、时间轮

+ 代码清单11-5，

##### 11.4.2、时间堆

+ 代码清单11-6，

### chap12、高性能I/O框架库Libevent

+ 处理三类事件（chap9,10,11）需要考虑如下三个问题
  + 1、统一事件源
  + 2、可移植性
  + 3、对并发编程的支持

#### 12.1、I/O框架库概述

+ **Reactor模式**的I/O框架库包含如下几个组件：
  + 1、句柄
  + 2、事件多路分发器
  + 3、事件处理器和具体的事件处理器
  + 4、Reactor

#### 12.2、libevent源码分析

##### 12.2.1、一个实例

##### 12.2.2、源代码组织结构

##### 12.2.3、event结构体

##### 12.2.4、往注册事件队列中添加事件处理器

##### 12.2.5、往事件多路分发器中注册事件

##### 12.2.6、eventop结构体

##### 12.2.7、event_base结构体

##### 12.2.8、事件循环

### chap13、多进程编程

+ 包含的内容
  + 复制进程映像的fork系统调用和替换进程映像的exec系列系统调用
  + 僵尸进程以及如何避免僵尸进程
  + 进程间通信最简单的方式：管道
  + 3种System V进程间通信方式：**信号量、消息队列和共享内存**，因为是 AT&T System V2版本引入的，所以称为System V
  + 进程间传递文件描述符的通用方法：**通过UNIX本地域socket传递特殊的辅助数据**
    + 辅助数据见5.8.3小节，**msg_control和msg_controllen成员**

#### 13.1、fork系统调用

+ 子进程的代码与父进程完全相同，同时它还会复制父进程的数据（堆数据、栈数据和静态数据）。
+ 数据的复制采用的是所谓的写时复制（copy on write），即只有在任一进程（父进程或子进程）对数据执行了写操作时，复制才会发生（先是缺页中断，然后操作系统给子进程分配内存半复制父进程的数据）

#### 13.2、exec系列系统调用

#### 13.3、处理僵尸进程

+ `wait()和waitpid()`

#### 13.4、管道

+ 管道能在父、子进程间传递数据，利用的是fork调用之后两个管道文件描述符（fd[0]和fd[1]）都保持打开。

#### 13.5、信号量

##### 13.5.1、信号量原语

+ P（passeren，传递，进入临界区）和V（vrijgeven，释放，退出临界区）

+ sys/sem.h

##### 13.5.2、semget系统调用

+ **创建信号量集**
##### 13.5.3、semop系统调用

##### 13.5.4、semctl系统调用

+ **允许调用者对信号量进行直接控制**
  
##### 13.5.5、特殊键值ipc_private

+ 代码13-3，使用IPC_PRIVATE信号量

#### 13.6、共享内存

+ 最高效的IPC机制，**因为它不涉及进程之间的任何数据传输**，带来的问题是
  + **必须用其它辅助手段来同步进程对共享内存的访问**，否则会产生竞态条件
  + **共享内存通常和其他进程间通信房事一起使用**
+ sys/shm.h

##### 13.6.1、shmget系统调用

##### 13.6.2、shmat和shmdt系统调用

##### 13.6.3、shmctl系统调用

##### 13.6.4、共享内存的posix方法

##### 13.6.5、共享内存实例

+ 代码13-4，使用共享内存的聊天室服务器程序

#### 13.7、消息队列

+ 两个进程之间传递二进制块数据，**可以不一定像管道或命名管道一样，必须先进先出的方式接收数据**

+ *这个例子，我没太写过，要尝试写下，再用man看下，怎么使用，lionel*

+ sys/msg.h中，以下4个系统调用

##### 13.7.1、msgget系统调用

+ 创建或获取一个消息队列，`int msgget(key_t key, int msgflg)`，key是一个键值，**标识一个全局唯一的消息队列**
+ `struct msqid_ds{}`

##### 13.7.2、msgsnd系统调用

##### 13.7.3、msgrcv系统调用

##### 13.7.4、msgctl系统调用

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

+ LinuxThreads，**不能充分利用多处理器系统的优势**
+ NPTL的优势
  + 内核线程不再是一个进程

#### 14.2、创建线程和结束线程

##### 1、pthread_create

##### 2、pthread_exit

##### 3、pthread_join，**回收线程**【**等待其他线程结束**】

+ 类似于wait和waitpid
+ **`pthread_join()`函数只能等待一个指定的线程**
+ 使用场景：
  + 等待指定线程结束，阻塞当前线程，直到指定线程退出
  + 线程同步，当主线程需要等待其他线程完成任务后再继续执行，可以使用`pthread_join()`等待其他线程的结束
  + 获取线程的返回值
  + 资源回收

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

void* thread_function(void* arg) {
    int thread_id = *(int*)arg;
    printf("Thread %d is running.\n", thread_id);
    // 模拟线程执行一些任务
    for (int i = 0; i < 5; i++) {
        printf("Thread %d: %d\n", thread_id, i);
    }
    int* result = malloc(sizeof(int));
    *result = thread_id * 100;
    pthread_exit(result);
}

int main() {
    pthread_t thread;
    int thread_id = 1;
    int* result;

    // 创建线程，使用thread_function函数，并把 thread_id 当作参数传递过去
    if (pthread_create(&thread, NULL, thread_function, &thread_id) != 0) {
        fprintf(stderr, "Failed to create thread.\n");
        return 1;
    }

    printf("Main thread is waiting for the thread to finish.\n");

    // 等待线程结束并获取返回值result
    if (pthread_join(thread, (void**)&result) != 0) {
        fprintf(stderr, "Failed to join thread.\n");
        return 1;
    }

    printf("Thread returned: %d\n", *result);

    free(result);

    return 0;
}
```



##### 4、pthread_cancel

```c
#include <pthread.h>  //记的这个头文件
int pthread_cancel(pthread_t thread);
```

#### 14.3、线程属性

+ `pthread_attr_t`结构体

#### 14.4、posix信号量

+ **`pthread_join`可以看作一种简单的线程同步方式**，它无法高效地实现复杂的同步需求，比如控制对共享资源的独占式访问，又抑或是在某个条件满足之后唤醒一个线程。
  + POSIX信号量
  + 互斥量
  + 条件变量
+ POSIX信号量函数的名字是以`sem_`开头，常用的是下面5个

```c
#include <semaphore.h>
int sem_init(sem_t *sem, int pshard, unsigned int value);
int sem_wait(sem_t *sem);
```

#### 14.5、互斥锁

+ *这些自己用得都比较少*
+ **确保其独占式的访问**，P，V操作

##### 14.5.1、互斥锁基础api（5个）

```c
int pthread_mutex_init();
int pthread_mutex_destroy();
int pthread_mutex_lock();
```



##### 14.5.2、互斥锁属性

```c
int pthread_mutexattr_init();
int pthread_mutexattr_destroy();
```



##### 14.5.3、死锁举例

+ **死锁**使得一个或多个线程被挂起而无法继续执行，而且这种情况还不容易被发现

+ 代码14-1，按不同顺序访问互斥锁导致死锁

#### 14.6、条件变量

+ 如果说互斥锁是用于同步线程对共享数据的访问的话，那么条件变量则是**用于在线程之间同步共享数据的值**。
+ 条件变量提供了一种**线程间的通知机制**：当某个共享数据达到某个值的时候，唤醒等待这个共享数据的线程。
+ 5个函数

```c
int pthread_cond_init();
int pthread_cond_destroy();
```

##### 自己提问及搜的

+ 条件变量（Condition Variable）是线程同步的一种机制，用于解决多线程间的协调和通信问题。它的引入主要是为了解决以下问题：
  + 1、避免忙等待：当线程需要等待某个条件满足时，常见的做法是通过轮询或忙等待来检查条件是否满足。这种方式会消耗CPU资源，并降低系统的性能。条件变量提供了一种更高效的等待机制，使得线程可以进入睡眠状态，等待条件满足时被唤醒，从而避免了忙等待。
  + 2、线程间的通信：多个线程之间可能需要进行通信，例如一个线程产生了某个结果，其他线程需要等待该结果可用后才能继续执行。条件变量提供了一种线程间的通信机制，使得一个线程可以通知其他线程某个条件的变化，从而实现线程间的协调和同步。
  + 3、避免竞争条件：多线程环境下，可能会存在共享资源的竞争条件（Race Condition），即多个线程同时访问和修改共享资源，而没有合适的同步机制。条件变量结合互斥锁（Mutex）可以解决竞争条件问题。线程可以在条件变量上等待某个条件的满足，而在修改共享资源之前使用互斥锁进行保护，从而确保线程之间的安全访问。

#### 14.7、线程同步机制包装类

+ 代码14-2，locker.h

#### 14.8、多线程环境

##### 14.8.1、可重入函数

+ 如果一个函数能被多个线程同时调用且不发生竞态条件，则我们称它是**线程安全的（thread safe）**，或者说它是**可重入函数**。
+ 有小部分是不可重入的，对应的函数版本后面要加上`_r`

##### 14.8.2、线程和进程

+ 思考一个问题
  + 如果一个多线程程序的某个线程调用了fork函数，那么新创建的子进程是否将自动创建和父进程相同数量的线程呢？
  + **答案是“否“**

+ 代码14-3

+ **子进程只拥有一个执行线程**
+ 代码14-4

##### 14.8.3、线程和信号

+ **每个线程都可以独立地设置信号掩码**。`sigprocmask()`
+ 代码14-5

### chap15、进程池和线程池

+ 动态创建子进程（子线程）来实现并发服务器，有以下缺点
  + 动态创建进程（线程）比较耗时，将导致较慢的客户响应
  + 动态创建的子进程（子线程）只为一个客户服务，导致系统上产生大量的细微进程（或线程）。进程（或线程）间的切换将消耗大量CPU时间

#### 15.1、进程池和线程池概述

+ **进程池中进程数量，一般跟CPU数量差不多**
  + 进程池中的所有子进程都运行着相同的代码，并具有相同的属性，比如优先级、PGID等
+ 主进程选择哪个子进程来为新任务服务，有两种方式
  + 主进程使用某种算法来主动选择子进程（随机算法、Round Robin轮流选取算法）
  + 主进程和所有子进程通过一个共享的工作队列来同步，子进程都睡眠在该工作队列上
+ **选择好子进程后，主进程还需要使用某种通知机制来告诉目标子进程有新任务需要处理，并传递必要的数据**

+ **图15-1，进程池模型**

#### 15.2、处理多客户

+ **处理多客户任务时**，需要问，监听socket和连接socket是否都由主进程统一管理？
+ 第8章的并发模式
  + **半同步/半反应堆模式**是由主进程统一管理这两种socket的
    + 主进程接受新的连接以得到连接socket，再把它传递给子进程（线程池，进程池不同传递方法）
  + **半同步/半异步**（领导者/追随者）模式，则由主进程管理所有监听socket，而各个子进程分别管理属于自己的连接socket。
    + 子进程可以自己调用accept来接受新的连接，
    + 父进程无须向子进程传递socket，只需要通知一声，“我检测到新的连接，你来接受它”
+ 图15-2
  + 客户任务是无状态的，可以考虑使用不同的子进程来为该客户的不同请求服务
  + **如果客户任务是存在上下文关系的，则最好一直用同一个子进程来为之服务**

#### 15.3、半同步/半异步进程池实现

+ 代码清单15-1（**基于图8-11所示的半同步/半异步并发模式的进程池**）
+ 避免在父、子进程之间传递文件描述符，**接受新连接的操作放到子进程中**
+ 对这种模式而言，**一个客户连接上的所有任务始终是由一个子进程来处理的**

#### 15.4、用进程池实现的简单cgi服务器

+ 代码15-2（重新写了一下6.2节的功能）

#### 15.5、半同步/半反应堆线程池实现

+ **用一个工作队列完全解除了主线程和工作线程的耦合关系**
  + 主线程往工作队列中插入任务
  + 工作线程通过竞争来取得任务并执行它

+ 15-4代码

#### 15.6、用线程池实现的简单web服务器

##### 15.6.1、http_conn类

+ 线程池的模板参数类，用以封装对逻辑任务的处理

##### 15.6.2、main函数

+ 15-6的代码

### chap16、服务器调制、调试和测试

#### 16.1、最大文件描述符数

+ **用户级限制**，目标用户运行的所有进程总共能打开的文件描述符数
+ **系统级限制**，所有用户总共能打开的文件描述符数

+ `ulimit -n`，查看用户级
+ `ulimit -SHn max-file-number`，设定用户级
+ 以上都是Session级别的，**永久修改**的话，在`/etc/security/limits.conf`
+ 修改系统级`sysctl -w fs.file-max=max-file-number`，这是临时有效
  + **永久的话**，需要在`/etc/sysctl.conf`中添加`fs.file-max=max-file-number`，执行`sysctl -p`使更改生效

#### 16.2、调整内核参数

+ 几乎所有的内核模块，包括内核核心模块和驱动程序，**都在`/proc/sys`文件系统下提供了某些配置文件以供用户调整模块的属性和行为**。通常一个配置文件对应一个内核参数，文件名就是参数的名字，文件的内容是参数的值。

+ `sysctl -a`查看这些内核参数

##### 16.2.1、/proc/sys/fs目录下的部分文件

+ **文件系统**
+ file-max
+ epoll/max_user_watches

##### 16.2.2、/proc/sys/net目录下的部分文件

+ **网络模块**
+ core/somaxconn
+ ipv4/tcp_max_syn_backlog
+ ipv4/tcp_wmem
+ ipv4/tcp_syncookies

#### 16.3、gdb调试

##### 16.3.1、用gdb调试多进程程序

###### 1、单独调试子进程

+ `gdb`进去后，**直接`attach 子进程号`**，设置子进程中的断点

###### 2、使用调试器选项follow-fork-mode

+ `set follow-fork-mode parent`或`set follow-fork-mode child`，设置为父或子进程

##### 16.3.2、用gdb调试多线程程序

+ info threads，**ID前面有"`*`"号的线程是当前被调试的线程**
+ thread ID，调试目标ID指定的线程
+ set scheduler-locking [off|on|setp]，**off不锁任何线程，这是默认值**
  + on表示只有当前被调试的线程会继续执行
  + step表示在单步执行的时候，只有当前线程会执行

#### 16.4、压力测试

+ 16-4的代码

### chap17、系统监测工具

#### 17.1、tcpdump

+ tcpdump**支持表达式**进一步过滤数据包
  + 类型
  + 方向：`tcpdump dst port 13579`
  + 协议：`tcpdump icmp`

#### 17.2、lsof（list open file）

+ *还是到时用man吧*

#### 17.3、nc（netcat）

+ 快速构建网络连接

#### 17.4、strace

+ 测试服务器性能，**跟踪程序运行过程中执行的系统调用和接收到的信号**，并将系统调用名、参数、返回值及信号名输出到标准输出或者指定的文件

#### 17.5、netstat

+ **网络信息统计工具**，打印本地网卡接口上的全部连接、路由表信息、网卡接口信息等

#### 17.6、vmstat（virtual memory statistics）

+ **实时输出系统的各种资源的使用情况**，比如进程信息、内存使用、CPU使用率以及I/O使用情况
+ `iostat`获得磁盘使用情况的更多信息
+ `mpstat`获得CPU使用情况的更多信息

#### 17.7、ifstat（interface statiscis）

+ **简单的网络流量监测工具**

#### 17.8、mpstat（multi-processor statistics）

+ **实时监测多处理器系统上每个CPU的使用情况**
+ mpstat和iostat都**集成在包sysstat中**

### 最后

#### 履历

+ 2024-05-06
  + 把chap10信号，复习了一下，*剩下，就是代码没读和运行了*
  + 之前chap11定时器，没怎么看，*现在看了一遍，也没怎么看出啥内容来，代码也没看或调试*