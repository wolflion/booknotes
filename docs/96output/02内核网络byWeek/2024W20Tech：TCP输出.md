## 缘起

+ 看《Linux内核源码剖析-tcpip实现》下，chap30输出
+ 调用`tcp_transmit_skb()`场景
  + 首次发送TCP段
  + 重传
  + 建立TCP连接时发送SYN段

## 内容

### 一、书中内容

#### 1.1、TCP中MSS概念

+ Max Segment Size相关的是网络设备接口的**MTU**

#### 1.1、sendmsg的调用过程

+ 套接口层，`inet_sendmsg()`
+ 传输接口层，`tcp_sendmsg()`

## 最后