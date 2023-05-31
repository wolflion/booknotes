## 《WebRTC音视频实时互动技术》

### chap7、WebRTC中的SDP

+ SDP（Session Description Protocol）是WebRTC的灵魂

#### 7.1、SDP标准规范

7.2、WebRTC中的SDP的整体结构

7.3、媒体信息

7.4、PlanB与UnifiedPlan

7.5、WebRTC如何保证数据安全

7.6、RTP扩展头

7.7、服务质量

7.8、SDP详解

7.9、ORTC

7.10、小结

### chap8、各端的互联互通

#### 8.1、WebRTC Native的核心

+ web端的核心是**RTCPeerConnection对象**
+ native的核心对象是**PeerConnectionFactory**，不能传输数据，**传输数据**是由**PeerConnection对象**
+ 223/476，中的图

#### 8.2、Android端的实现

#### 8.3、iOS端的实现

#### 8.4、PC端与Mac端的实现

+ Native方案，底层API
+ Electron，基于Chrome浏览器内核
  + javascript
+ Flutter方案
  + google推出，目标是开发一套代码让所有类型的终端都可运行

#### 8.5、小结

+ 实现一对一实时通信的逻辑：
  + 申请访问音视频设备权限
  + 引入WebRTC库（Web端不需要）
  + 采集音视频数据，并创建音视频数据通道
  + 采集音视频并展示本地预览
  + 与服务器建立信令通道，并通过信令驱动程序的运转
  + 进行媒体协商
  + 渲染远端视频

### chap9、网络传输协议RTP与RTCP

#### 9.1、如何选择UDP与TCP

+ 乱序（抖动）
+ **TCP的丢包重传机制**
+ 基于UDP，通过**NACK、FEC、Jitter Buffer以及NetEQ技术**解决丢包和抖动问题，又不产生影响服务质量的时延

#### 9.2、RTP

+ UDP**传输一些有前后逻辑关系的数据时**就不太行，而音视频就是这类数据，所以在**UDP之上增加一个协议，RTP**，RTP与HTTP一样，都是应用层
+ 问题1：RTP是如何传输有前后关系的音视频数据的？*lionel问题*

##### 9.2.1、RTP的协议头

+ 发送的数据包都打上了一个编号，对应的字段，叫做**Sequence Number字段**
+ 接收端从同一个端口中通过**PayLoadType字段**，来区分不同类型的数据
  + VP8的PT一般是96
  + Opus的PT一般是111
+ **SSRC字段**区分啥呢？**多个不同源（参与人）通过同一个端口发送**
  + 每个源的SSRC必须唯一
  + 每个SSRC所代表的数据流的Sequence Number都是单独计数的
+ *276/476的还有些要扩展的，lionel*

##### 9.2.2、RTP的使用

+ 创建/解析RTP包
  + `RtpPacket`类，module/rtp_rtcp_source目录中的rtp_packet.h/cc
+ 根据RTP包进行逻辑处理
  + **通过缓冲队列JitterBuffer**解决抖动问题
  + 279/476再好好看看，*lionel*

##### 9.2.3、RTP的扩展头

##### 9.2.4、RTP中的填充数据

#### 9.3、RTCP

+ 与RTP处于同一层级，可以做**丢包控制**，接收端可以使用RTCP的RR报文向发送端发送接收报告，记录着从上次报告到本次报告之间丢失了多少包，丢包率是多少，延时是多少。

##### 9.3.1、RTCP报文分类

+ 293/476

+ SR
+ RR（Receiver Report packet）

##### 9.3.2、RTP协议头

+ RTP中的PT值是在SDP中定义的，而RTCP的PT值来自（SR,RR）这些
+ RFC3550

##### 9.3.3、WebRTC的反馈报文

+ PF为205（RTPFB报文，反馈RTP相关的信息）和206（PSFB报文，反馈负载相关的信息）的报文类型属于**反馈报文**
+ 297/476

#### 9.4、小结

+ RTP特别适用于音视频数据传输

### chap10、WebRTC拥塞控制

#### 10.1、WebRTC的拥塞控制算法

##### 10.1.1、Goog-REMB

##### 10.1.2、Transport-CC

##### 10.1.3、基于丢包的拥塞评估算法原理

##### 10.1.4、WebRTC拥塞控制流程

#### 10.2、拥塞控制算法比较

#### 10.3、小结

### chap11、WebRTC源码分析入门

+ **读开源项目的源码是开发人员的一门必修课**

#### 11.1、WebRTC源码的选择

11.2、WebRTC开发环境的搭建

#### 11.4、WebRTC目录结构

##### 11.4.1、WebRTC主目录

+ 接口层，api目录
+ 业务处理层
  + pc目录（PerrConnection）
  + call目录（与session业务类似）
  + media目录（媒体引擎）
+ 音频处理层
  + audio
  + video
  + common_audio，**音频的基本算法**
  + common_videosdk，**视频算法工具**
+ 基础支撑层
  + sdk目录，Android端的视频采集
  + p2p目录，端到端的网络连接，dtls，stun协议实现
  + stats目录，统计相关
  + rt_base目录
  + rt_tools目录
  + tools_webrtc目录
  + system_wrapper目录

##### 11.4.2、modules目录

+ 音频处理类
  + audio_coding
  + audio_device
+ 视频处理类
+ 网络及流控类
  + rtp_rtcp
  + bitrate_controller
  + congestion_controller
  + remote_bitrate_estimator
  + pcing

#### 11.5、小结

### chap12、分析WebRTC源码的必经之路

#### 12.1、信令服务器实现分析

##### 12.1.1、信令服务器的组成

##### 12.1.2、信令服务器的工作流程

#### 12.2、PeerConnection客户端分析

##### 12.2.1、运行peerconnection_client

##### 12.2.2、peerconnection_client的组成

##### 12.2.3、界面的展示

##### 12.2.4、视频的渲染

##### 12.2.5、WebRTC的使用

+ 与chap8对应起来看

##### 12.2.6、信令的处理

#### 12.3、小结

+ 剖析了WebRTC中的peerconnection程序的实现细节
+ **环境搭建好**

### chap13、WebRTC源码分析

#### 13.1、WebRTC的数据流

+ 408/476，图13.1，WebRTC数据流
+ 带宽的评估，有两种方法
  + 基于丢包的评估方法
  + 基于延迟的评估方法
    + 基于接收端，Goog-REMB，**已被淘汰**
    + 基于发送端，Transport-CC
+ WebRTC对于丢包有2种处理方法：
  + 通过RTCP的NACK消息：**让发送端将丢失的包重新补发**
  + 使用FEC算法，**发送一些冗余数据来修复丢失的包**，以避免通过NACK重发包带来的延迟

#### 13.2、WebRTC线程模型

+ **学习和掌握任何一个系统或开源库，了解并掌握其线程模型是至关重要的**
+ 411/476，WebRTC线程模型

##### 13.2.1、WebRTC线程的创建与使用

+ 从创建的角度看，WebRTC线程被划分为两层
  + 第一层: 网络线程、工作线程、信令线程
  + 第二层：音视频相关线程（没有开启相关业务，就没有相关线程）
+ 从运行的角度，WebRTC线程被划分为三层
  + 第一层: 网络线程、工作线程、信令线程
  + 第二层:音视频编解码层
  + 第三层:硬件访问层，用于进行音视频的采集与渲染

##### 13.2.2、线程切换

###### 1、利用Post()和PostTask()方法切换线程

+ rtc_base目录中的thread.h中的`Post()`

###### 2、利用Send()和Invoke()方法切换线程

###### 3、外部接口的线程切换

#### 13.3、网络传输

##### 13.3.1、网络接收与分发模块类关系图

+ 435/476中图13.7，8大模块
  + 1、
  + 7、定义了创建Socket的工厂类
  + 8、机制的封装

##### 13.3.2、网络连接的建立

+ WebRTC中，有三种建立网络连接的方法
  + 1、在收集Candidate并返回给应用层的过程中，如果已经收到Remote Candidate，则会创建Connection对象
  + 2、

#### 13.4、音视频数据采集

##### 13.4.1、音频采集与播放

+ 445/476，音频处理流程

##### 13.4.2、视频采集与渲染

+ 452/476，视频数据流转

#### 13.5、音视频编解码

##### 13.5.1、音频编码

+ AudioCodingModuleImpl类中的`encoder_stack_`成员指定的

##### 13.5.2、音频解码

+ 460/476，音频编码与发送

##### 13.5.3、视频编码

##### 13.5.4、视频解码

#### 13.6、小结

### 学习过程

+ 05-31，中午1个cubi，晚上1个cubi，把书上chap7-13给初步看了一下，**chap13确实是精华**