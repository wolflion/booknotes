### chap1、理解网络编程和套接字

#### 1.1、理解网络编程和套接字

+ **接受连接请求**【即服务端】的套接字创建过程
  + 1、调用`socket()`创建套接字
  + 2、调用`bind()`分配IP地址和端口号
  + 3、调用`listen()`转为可接收请求状态
  + 4、调用`accept()`受理连接请求
+ **请求连接**【即客户端】
  + 1、调用`socket()`
  + 2、调用`connect()`

#### 1.2、基于Linux的文件操作

+ **socket会被认为是文件的一种**，直接用I/O操作即可，`open()、close()、write()`
+ **书上提到了`_t`后缀的数据类型**（一般都是typedef声明引起的）

##### lionel注：

+ TLPI的chap56，提到了**流socket**和**数据报sockect**，本章用到的例子是前者。

### chap2、套接字类型与协议设置

#### 2.1、套接字协议及其数据传输特性

+ 关于协议
  + **对话中使用的通信规则**
+ 协议族（Protocol Family）
  + socket的第1个参数
  + `sys/socket.h`
  + `PF_INET`，ipv4
  + `PF_PACKET`，底层套接字
+ 套接字类型（type）
  + **套接字的数据传输方式**，socket的第2个参数
  + 1、面向连接的套接字（SOCK_STREAM）
    + 传输过程中数据不会消失
    + 按序传输数据
    + 传输的数据不存在数据边界-*lionel，这个其实没太理解*
  + 2、面向消息的套接字（SOCK_DGRAM）
    + 强调快速传输而非传输顺序
    + 39/421
+ 协议的最终选择
  + socket的第3个参数
  + 一般为0，**但 数据传输方式相同，协议不同，需要第3个参数指定**。【同一协议族中存在多个数据传输方式相同的协议】

##### lionel注：

+ `socket()`的3个参数讲解，知识自己盲区，到时试下`man socket`看看

### chap3、地址族与数据序列

#### 3.1、

#### 3.2、地址信息的表示

#### 3.3、网络字节序与地址变换

#### 3.4、网络地址的初始化与分配

##### lionel注：

+ 没具体看，知识盲区

### chap4-5、基于TCP的服务端/客户端

#### 4.1、理解TCP和UDP

#### 4.2、实现基于TCP的服务器端/客户端

+ 就是**本章1.1+1.2**，即连上后进行**读写+关闭**

#### 4.3、实现迭代服务器端/客户端

+ *这个要想一下原理，lionel*

#### 5.1、回声客户端的完美表现

#### 5.2、TCP原理

##### lionel注：

+ 要让我**实现迭代服务器端/客户端**，我也不会

### chap6、基于UDP的服务端/客户端

### chap7、优雅的断开套接字连接

#### 7.1、基于TCP的半关闭

+ 单方面断开连接带来的问题
  + `close()`是完全关闭，**不仅无法传输数据，而且也不能接收数据**。
+ 套接字和流（stream）
+ 针对优雅断开的shutdown函数
  + 第2个参数决定断开连接的方式
    + SHUT_RD：断开输入流
    + SHUT_WR：断开输出流
    + SHUT_RDWR：同时断开I/O流

```c
#include<sys/socket.h>
int shutdown(int sock, int howto);
```



+ 为何需要半关闭
+ 基于半关闭的文件传输程序
  + file_server.c
  + file_client.c

### chap8、域名及网络地址

#### 8.1、域名系统

+ 什么是域名
+ DNS服务器

#### 8.2、IP地址和域名之间的转换

+ 程序中有必要使用域名吗？
  + **所有学习都要在开始前认识到其必要性**。
+ 利用域名获取IP地址
  + gethostbyname.c

```c
#include <netdb.h>
struct hostent* gethostbyname(const char* hostname);
struct hostent{
    char* h_name; //官方域名
    char** h_aliases; //其它域名
    int h_addrtype;//ipv4还是ipv6
    int h_length;//IP地址长度
    char** h_addr_list;//以整数形式保存域名对应的IP地址
}
```



+ 利用IP地址获取域名
  + gethostbyaddr.c

```c
#include <netdb.h>
struct hostent* gethostbyaddr(const char *addr, socklen_t len, int family);
```



### chap9、套接字的多种可选项

#### 9.1、套接字可选项和I/O缓冲大小

+ 套接字多种可选项
  + 协议层
    + SOL_SOCKET
    + IPPROTO_IP
    + IPPROTO_TCP
+ getsockopt&setsockopt
  + sock_type.c

```c
#include<sys/socket.h>
int getsockopt(int sock, int optname, void *optval, socklen_t *optlen);

int setsockopt(int sock, int level, int optname, const void *optval, socklen_t *optlen);
```



+ SO_SNDBUF&SO_RCVBUF
  + SO_RCVBUF：输入缓冲大小相关可选项
  + SO_SNDBUF：输出缓冲大小相关可选项
  + **这2个选项既可以读取当前I/O缓冲大小，也可以进行更改**。
  + get_buf.c
  + set_buf.c

#### 9.2、SO_REUSEADDR

+ 0
  + **学习SO_REUSEADDR可选项之前，应理解好Time-wait状态**。

+ 发生地址分配错误（Binding Error）
  + reuseaddr_eserver.c
+ Time-wait状态
  + **先断开连接的套接字必然会经过Time-wait状态**
+ 地址再分配

#### 9.3、TCP_NODELAY

+ Nagle算法
+ 禁用Nagle算法
  + **将套接字可选项TCP_NODELAY改为1（真）**

### chap10、多进程服务器端

#### 10.1、进程概念及应用

+ 并发服务器端的实现方法
  + 多进程服务器：通过创建多个进程提供服务
  + 多路复路服务器：通过捆绑并统一管理I/O对象提供服务
  + 多线程服务器：通过生成与客户端等量的线程提供服务
+ 理解进程
  + **占用内存空间的正在运行的程序**
+ 通过调用fork函数创建进程

#### 10.2、进程和僵尸进程

+ 僵尸（Zombie）进程
  + 完成工作没有被销毁，继续占用系统中的重要资源
+ 产生僵尸进程的原因
  + 父进程没有管子进程
+ 销毁僵尸进程1：利用wait函数
+ 销毁僵尸进程2：利用waitpid函数

#### 10.3、信号处理

+ 向操作系统求助
+ 关于JAVA的题外话：保持开放思维
+ 信号与signal函数
+ 利用sigaction函数进行信号处理

#### 10.4、基于多任务的并发服务器

+ 基于进程的并发服务器模型
  + 图10-2：并发服务器模型  （183/421）
+ 实现并发服务器
+ 通过fork函数复制文件描述符

#### 10.5、分割TCP的I/O程序

+ 分割I/O程序的优点
+ 回声客户端的I/O程序分割

### chap11、进程间通信

#### 11.1、进程间通信的基本概念

+ 对进程间通信的基本概念
+ 通过管道实现进程间通信

#### 11.2、运用进程间通信

+ 保存消息的回声服务器端
+ 如果想构建更大型的程序

### chap12、I/O复用

#### 12.1、基于I/O复用的服务器端

+ 多进程服务器端的缺点和解决方法
+ 理解复用
  + 定义：
    + 1、在1个通信频道中传递多个数据（信号）的技术
    + 2、为了提高物理设备的效率，用最少的物理要素传递最多数据时使用的技术
  + 复用技术的优点：
    + 减少连线长度
    + 减少纸杯个数
+ 复用技术在服务器端的应用

#### 12.2、理解select函数并实现服务器端

+ select函数的功能和调用顺序
+ 设置文件描述符
+ 设置检查（监视）范围及超时
+ 调用select函数后查看结果

### chap13、多种I/O函数

#### 13.1、send&recv函数

+ Linux中的send&recv

```c
#include<sys/socket.h>
ssize_t send(int sockfd, const void* buf, size_t nbytes, int flags);

ssize_t recv(int sockfd, void *buf, size_t nbytes, int flags);
```



+ MSG_OOB：发送紧急消息
  + oob_send.c
  + oob_recv.c
+ 紧急模式工作原理
+ 检查输入缓冲
  + peek_send.c
  + peek_recv.c

#### 13.2、readv&writev函数

+ 使用readv&writev函数
  + writev.c
  + readv.c

```c
#include<sys/uio.h>
ssize_t writev(int filedes, const struct iovec* iov, int iovcnt);

struct iovec{
    void* iov_base; //缓冲地址
    size_t iov_len; //缓冲大小
}

ssize_t readv(int filedes, const struct iovec *iov, int iovcnt);
```



+ 合理使用readv&writev函数

### chap14、多播与广播

#### 14.1、多播

+ 多播的数据传输方式及流量方面的优点
+ 路由（Routing）和TTL（Time to Live：生存时间），以及加入组的方法
+ 实现多播Sender和Receiver
  + news_sender.c
  + news_receiver.c

#### 14.2、广播

+ 广播的理解及实现方法
  + **广播与多播相同，也是基于UDP完成的**
+ 实现广播数据的Sender和Receiver
  + news_sender_brd.c
  + news_receiver_brd.c