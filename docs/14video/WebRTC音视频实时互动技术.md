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

#### 9.4、小结

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

### chap13、WebRTC源码分析

### 学习过程

+ 05-31，中午1个cubi