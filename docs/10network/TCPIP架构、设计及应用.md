## 《TCP/IP架构、设计及应用》

### chap3、KERNEL IMPLEMENTATION OF SOCKETS  125/783

+ *协议族*（PF_INET）与*socket类型*（SOCK_STREAM），这些是啥关系？
+ 每一个协议族都需要注册？
+ `socket()`需要family/type/protocol的原因是 *内核*需要
+ *以PF_INET*为例，这是IPv4相关的？

#### 3.1、SOCKET LAYER

+ sock结构体，关联了协议族和协议类型，`struct inet_protosw`

#### 3.2、VFS AND SOCKET

+ 图3.2，socket通过进程文件表访问

#### 3.3 PROTOCOL SOCKET REGISTRATION

+  list head array inetsw[SOCK_MAX] 

#### 3.4 struct inet _ protosw

#### 3.5、SOCKET ORGANIZATION IN THE KERNEL

+ 图3-4，
  + 第一层，用户空间，Application（bind这样的）
  + 第二层，内核空间
    + socket layer(protocol family)，`sock->ops`，比如inet_bind这样的
    + socket layer(IP protocol specific)，`sk->prot`，比如tcp_sendmsg这样的

#### 3.6、SOCKET

+ net/ipv4/af_inet.c，`static struct inet_protosw inetsw_array[]{};`，指定了各种SOCK_TYPE，*这里的prot好理解，但ops怎么理解呢，比如对应的inet_dgram_ops，怎么去找到函数*
  + SOCK_STREAM
  + SOCK_DGRAM
  + SOCK_RAW
+ include/linux/net.h中的`struct socket{};`

#### 3.7、inet _ create (see cs 3.4 )

+ net/ipv4/af_inet.c中的`static int inet_create()`

##### 3.7.1 Sock

#### 3.8 FLOW DIAGRAM FOR SOCKET CALL

#### 3.9、SUMMARY

+ net_families is a global array to indexed on net family number. 
+ ` inet_register_protosw() `支持INET family

### chap4、KERNEL IMPLEMENTATION OF TCP CONNECTION SETUP （144/783）