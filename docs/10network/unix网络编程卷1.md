## 《Unix网络编程》

+ 网络编程，在哪个层次分的tcp和udp，是什么函数分的？

### 0

+ [代码链接](https://github.com/unpbook/unpv13e)，官方的
  + *为啥没有unp.h*，难道要执行后才有？
+ Part1（1-2）
+ Part2：基本套接字编程（3-）

### chap1、简介

#### 1.1、概述

+ **程序相互通信所用的协议（protocol）**
  + *有个疑问是，如果选了一个应用层协议，我为什么使用socket()时要手动指定传输层的协议呢?*

#### 1.2、一个简单的时间获取客户程序

+ *这个是最简单的，没有server*，"unp.h"从哪下载？

#### 1.3、协议无关性

+ `sockaddr_in`是IPv4，来了个IPv6就要改代码，有个新的**套结字地址结构**（`sockaddr_storage`）

#### 1.4、错误处理：包裹函数

#### 1.5、一个简单的时间获取服务器程序

#### 1.6、本书中C/S程序示例索引表

#### 1.7、OSI模型

#### 1.8、BSD网络支持历史

#### 1.9、测试用网络及主机

#### 1.10、Unix标准

##### 1.10.1、POSIX的背景

1.10.2、开放团体的背景

1.10.3、标准的统一

##### 1.10.4、因特网工程任务攻坚组

#### 1.11、64位体系结构

#### 1.12、小结

+ 引入简单却完整的TCP客户和服务程序

#### 习题

+ 1.1、找出你自己的网络拓扑信息?

### chap2、传输层：TCP、UDP和SCTP

#### 2.1、概述

#### 2.2、总图

#### 2.6、TCP连接的建立和终止

##### 2.6.1、三路握手

##### 2.6.4、TCP状态转换图

+ **TCP一个连接定义了11种状态**

#### 2.9、端口号

#### 2.10、TCPduankou号与并发服务器

#### 2.11、缓冲区大小

#### 2.14、小结

### chap3、套接字编程简介

#### 3.1、概述

+ 先了解一下**套接字结构**
  + 从内核到进程方向传递的是**值-结果参数**（也就是指针，C里没有引用）
+ **地址转换函数**，inet_addr()，inet_nota()，inet_pton()，inet_ntop()
  + 存在一个问题是**与所转换的地址类型协议相关**（IPv4还是IPv6）
  + `sock_`开头的函数，**以协议无关的方式使用套接字地址结构**

#### 3.2、套接字地址结构

+ 以`sockaddr_`开头

##### 3.2.1、IPv4套接字地址结构

+ `sockaddr_in`，在`<netinet/in.h>`中

```c
struct in_addr{
    in_addr_t s_addr;//32位IPv4地址
};

struct sockaddr_in{
    uint8_t sin_len;  //结构体的长度
    sa_family_t sin_family;  //AF_INET
    in_port_t  sin_port;   //16-bit TCP or UDP port number  network byte ordered
    struct in_addr sin_addr;  //32-bit IPv4 address network bytes ordered
    char  sin_zero[8];  //unused
};
```

##### 3.2.2、通用套接字地址结构

+ `<sys/socket.h>`

##### 3.2.3、IPv6套接字地址结构

+ `<netinet/in.h>`

```c
struct in6_addr{
    unit8_t s6_addr;
};

struct sockaddr_in6{
    uint8_t sin6_len;  //结构体的长度
    sa_family_t sin6_family;  //AF_INET6
    in_port_t  sin6_port;   //16-bit TCP or UDP port number  network byte ordered
    uint32_t   sin6_flowinfo;
    struct in6_addr sin6_addr;  //32-bit IPv4 address network bytes ordered
    char  sin6_scope_id;  //unused
};
```



##### 3.2.4、新的通用套接字地址结构

+ `sockaddr_storage`结构，在`<netinet/in.h>`

```c
struct sockaddr_storage{
    uint8_t ss_len;
    sa_family_t  ss_family;
};
```

##### 3.2.5、套接字地址结构的比较

#### 3.3、值-结果参数

+ 1、从进程到内核传递套接字地址结构的函数有3个：bind，connect和sendto
+ 2、从内核到进程传递套接字地址结构的函数有4个：accept，recvfrom，getsockname，getsockname

#### 3.4、字节排序函数

+ 大端
+ 小端

#### 3.5、字节操纵函数

+ b开头表示字节（bcopy，bzero）
+ mem开头表示内存（memset）
+ str开头表示字符串

#### 3.6、inet_aton、inet_addr和inet_ntoa函数

#### 3.7、inet_pton和inet_ntop函数

+ **p（presentation）表达，n（numeric）数值**

#### 3.8、sock_ntop和相关函数

#### 3.9、readn、writen和readline函数

#### 3.10、小结

+ 套接字地址结构

#### 习题

### chap4、基本TCPtao接字编程

#### 4.1、概述

+ 完整的TCP C/S 程序所xu要的基本tao接字函数。
+ 以此例子，加以改进，**并发服务器**，（多客户连接时，要fork新的进程），再改进到多线程
+ TCP C/S 的**典型时间表**，图4-1

#### 4.2、socket函数

+ 原型是`int socket()`
  + 问题是，怎么去记呢？
  + family，AF（**地址**）
    + AF_INET，AF_INET6这种
  + type（类型，怎么分的）
  + protocol（0是什么意思）
    + 指定了tcp，udp
+ `AF_X`与`PF_X`关系与区别？

#### 4.3、connect函数  [client]

+ TCP协议的话，在此阶段，**触发三次握手**

#### 4.4、bind函数

＋ 本地协议地址 ，怎么理解？

#### 4.5、listen函数（server）

+ 主动socket和被动socket？
+ tcp状态转换图
+ 有使用顺序的，「一般在哪中间」

#### 4.6、accept函数（server）

#### 4.7、fork和exec函数

+ APUE里的

#### 4.8、并发服务器

+ 什么叫 迭代服务器（iterative server），服务器分几种？

#### 4.9、close函数

#### 4.10、getsockname和getpeername函数

+ 本地和外地 怎么 理解

#### 4.11、小结

+ 服务器开发的话，「服务器模型」是不是要了解一下呢
+ sock_addr结构体

#### 习题

+ 4.1、4.4节中，头 中定义的`INADDR_`常值是主机字节序的。我们该如何bian别？

### chap5、TCP客户/服务器程序示例

#### 5.6、正常启动

#### 5.19、小结

+ *对比简单例子*，这个有点fuza了，增加了进程开发、传输的数据格式

### chap6、I/O利用：select和poll函数

#### 6.1、概述

+ *为何要引入I/O利用*，这是解决什么问题？c还是s端？
  + 客户处理2个输入？标准输入和tcp socket？

#### 6.2、模型

#### 6.12、小结

+ 5种I/O模型，moren是阻塞型

#### 习题

+ 6.2、

### chap7、接字选项

#### 7.1、概述

+ *为何要有这个选项*

#### 7.11、fcntl函数

#### 7.2、小结

+ rfc设计的时候，为何有这个值？

### chap8、基本UDPtao接字编程

#### 8.1、概述

+ socket()函数里指定了协议，那么  用户态 怎么 知道不同流程呢？

#### 8.15、使用select函数的TCP和UDP回射服务器程序

#### 8.16、小结

+ 如何给UDP应用程序增加可靠性

### chap11、名字与地址转换

#### 11.22、小结

+ fei除了一些函数`getservbyname()`，现在在用吗？

### chap26、线程

### chap28、原始tao接字

### chap29、数据链路访问

### chap30、C/S程序设计范式

#### 30.1、

30.2、