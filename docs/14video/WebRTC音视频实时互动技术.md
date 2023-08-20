## 《WebRTC音视频实时互动技术》

### chap1、音视频直播的前世今生

#### 1.1、音视频的历史

#### 1.2、移动互联网

#### 1.3、音视频直播的两条技术路线

+ 2条技术路线
  + 以音视频会议为代表的实时互动直播
    + WebEx
    + 时延500ms
  + 以娱乐直播为代表的流媒体分发
    + 时延3s
+ 不同的直播技术
  + WebRTC
  + RTMP
  + HTTP-FLV
  + HLS
  + DASH

#### 1.4、音视频直播的现状

#### 1.5、音视频直播的未来

+ 有了AI后，可以对视频进行二次处理

#### 1.6、小结

### chap2、为什么要使用WebRTC

#### 2.1、自研直播客户端架构

+ 29/476

##### 2.1.1、拆分音视频模块

##### 2.1.2、跨平台

##### 2.1.3、插件化管理

##### 2.1.4、其他

+ 音视频不同步
+ 回音
+ 音视频的实时性
+ **网络拥塞，丢包，延时，抖动，混音**

#### 2.2、WebRTC客户端架构

+ 42/476

#### 2.3、小结

### chap3、音视频实时通信的本质

+ 线上与线下的区别
  + 一是实时性不够
  + 二是业务数据有损失
    + 1、摄像头采集的角度过于狭小
    + 2、设备的质量无法保障
    + 3、现场的气氛是无法通过摄像头和麦克风采集到的

#### 3.1、两种指标

##### 3.1.1、实时通信延迟指标

##### 3.1.2、音视频服务质量指标

#### 3.2、实时通信的主要矛盾

##### 3.2.1、增加带宽

##### 3.2.2、减少数据量

##### 3.2.3、适当增加时延

##### 3.2.4、提高网络质量

##### 3.2.5、快速准确地评估带宽

#### 3.3、小结

### chap4、构建WebRTC一对一信令服务器

#### 4.1、WebRTC一对一架构

#### 4.2、细化架构

#### 4.3、信令

##### 4.3.1、信令定义

##### 4.3.2、信令时序

##### 4.3.3、信令传输协议的选择

#### 4.4、构建信令服务器

##### 4.4.1、信令服务器的实现方案

##### 4.4.2、信令服务器的业务逻辑

##### 4.4.3、信令服务器的实现

##### 4.4.4、信令服务器的安装与部署

##### 4.4.5、信令服务器的完整代码

#### 4.5、小结

### chap5、WebRTC实现一对一通信

#### 5.1、浏览器对WebRTC的支持

#### 5.2、遍历音视频设备 

+ `enumerateDevices()`
+ `MediaDeviceInfo`结构体

#### 5.3、采集音视频数据

+ `getUserMedia()`
+ `MediaStreamConstrains`结构体

#### 5.4、MediaStream与MediaStreamTrack

#### 5.5、本地视频预览

#### 5.6、信令状态机

#### 5.7、RTCPeerConnection

##### 5.7.1、创建RTCPeerConnection对象

+ `new RTCPeerConnection(configuration)`

##### 5.7.2、RTCPeerConnection与本地音视频数据

##### 5.7.3、媒体协商

##### 5.7.4、ICE

###### 1、什么是Candidate

###### 2、收集Candidate

###### 3、交换Candidate

###### 4、尝试连接

##### 5.7.5、SDP与Candidate消息的交换

##### 5.7.6、远端音视频渲染

##### 5.7.7、客户端完整例子

#### 5.8、小结

### chap6、WebRTC中的ICE实现

#### 6.1、Candidate种类与优先级

#### 6.2、ICE策略

#### 6.3、P2P连接

##### 6.3.5、NAT类型检测

#### 6.4、网络中继

#### 6.5、小结

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

+ WebRTC是通过增加带宽、减少数据量、提高音视频质量、适当增加时延以及更准确的带宽评估等方法来提升音视频服务质量的。

#### 10.1、WebRTC的拥塞控制算法

+ GCC（基于发送端的拥塞控制算法Transport-CC，基于接收端的拥塞控制算法Goog-REMB，**后者已淘汰**）、BBR、PCC
+ GCC（google Congestion Control）
+ BBR（Bottleneck Bandwidth and Round-trip propagation time），瓶颈带宽和往返传播时间
+ PCC（Perfromance-oriented Congestion Control）
+ TCC（Transport-wide Congestion Control），传输带宽拥塞控制
+ Goog-REMB（Google Receiver Estimated Maximum Bitrace），google接收端评估的最大码流
+ QUICK（Quick UDP Internet Connection），使用UDP的快速网络连接协议，即HTTP/3
+ SCTP（Stream Control Transmission Protocol），流控传输协议

##### 10.1.1、Goog-REMB

+ 302/476
+ **左侧发送端模块**（Encoder，Pacer，Loss-Based Controller，Delay-Based Controller）
+ 主要是**右边接收端的**
+ RemoteBitrate Estimator
  + **总负责人**，
  + 一方面，要与外面的模块打交道，从网络收/发模块获取RTP包的传输信息用于拥塞评估，或将内部评估出的下一时刻的发送码流（大小）输出给网络收/发模块，让其通知发送端进行流控
  + 另一方面，组织内部的模块
+ Inter Arrival
  + 将数据包按帧进行分组，然后对相邻的两组数据包进行单向梯度计算
    + 每组数据包的发送时长，T
    + 每组数据包的接收时长，t
    + 两组数据包大小之差，G
+ OverUse Estimator
  + 利用Inter Arrival模块的计算结果，通过**卡尔曼滤波器**估算出下一时刻发送队列的增长趋势
+ OverUse Detector
  + 用于检测当前网络的拥塞状态
+ AIMD Rate Controller（**Additive-Increase/Multiplication-Decrease**，慢加快减机制）
  + 用于计算发送码流大小

##### 10.1.2、Transport-CC

+ 与上面算法的2个区别
  + 一是将算法从接收端到了发送端，使得评估和控制合为一体，代码的处理更简洁方便
  + 二是从卡尔曼滤波器找成**TrendLine滤波器**（最小二乘法滤波器、通过斜率增大或减小来判断当前网络的拥塞情况）
+ GoogCcNetworkController
+ SendSideBandwidthEstimation
+ DelayBaseBwe
+ Trendline

##### 10.1.3、基于丢包的拥塞评估算法原理

+ `SendSideBandwidthEstimation::UpdateEstimate`
  + 丢包低于2%，码率可以增加，一般增加8%
  + 丢包在2%-10%，码率不变
  + 丢包在10%，码率要降低（1-0.5*丢包率）乘 当前码率

##### 10.1.4、WebRTC拥塞控制流程

+ 发送端启动时设置一个初始带宽
+ 发送模块将音视频数据发送给远端
+ 远端收到后，定期向发送端反馈RTCP报文
+ 当发送端的接收模块收到RTCP反馈报文后，将数据交由RTCP解析模块进行解析，解析后的数据再交由**拥塞评估模块**（Transport-CC）计算
+ 有了评估带宽，一方面通知编码器，让编码器控制其码流大小，另一方面告之Pacer模块，控制码流的发送速度
+ 两方面的限制，最终发送码流不会超过带宽的估算值

#### 10.2、拥塞控制算法比较

+ RMCAT（RTP Media Congestion Avoidance Techniques），
+ NADA（Network Assisted Dynamic Adaptation），思科
+ SCReAM（Self Clocked Rate Adaptation for Multimedia），爱立信

##### 10.2.1、拥塞控制的准确性

##### 10.2.2、与TCP连接并存时的公平性

##### 10.2.3、同种类型连接的公平性

##### 10.2.4、拥塞控制算法在丢包情况下的表现

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

+ 05-31，中午1个cubi，晚上1个cubi，把书上chap7-13给初步看了一下，**chap13确实是精华**；跑完步回来，又花了1个cubi，把1-6快速给过了一下，（4-6没太好好读，时间有限），更偏向于客户端吧，目前精力不在这。