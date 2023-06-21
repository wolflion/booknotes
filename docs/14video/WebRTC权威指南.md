## 《WebRTC权威指南》

### chap3、本地媒体

#### 3.1、WebRTC中的媒体

##### 3.1.1、轨道

+ MediaStreamTrack是WebRTC中的基本媒体单元。
+ **对源的一切控制都通过轨道实施**

##### 3.1.2、流

+ MediaStream，有2种方式可创建:
  + 一是通过从现有MediaStream中复制轨道来请求对本地媒体的访问
  + 二是使用对等连接来接收新的流
+ 请求和访问本地媒体只有一种方式，通过调用`getUserMedia()`

#### 3.2、捕获本地媒体

#### 3.3、媒体选择和控制

+ 可约束的属性分为两种类型：枚举属性或范围属性
+ **约束的工作原理**，约束结构是一个对象，它具有两个可选的属性: mandatory和optional.

#### 3.4、媒体流示例

#### 3.5、可运行的本地媒体代码示例

##### 3.5.1、Web服务器

+ 3.5.1.1、
+ 3.5.1.2、
+ 3.5.1.3、log.js

##### 3.5.2、客户端WebRTC应用程序

### chap4、信令

#### 4.1、信令的作用

+ （1）协调媒体功能和设置
+ （2）标识和验证会话参与者的身份
+ （3）控制媒体会话、指示进度、更改会话和终止会话
+ （4）当会话双方同时尝试建立或更改会话时，实施双占用分解（Glare Resolution）

##### 4.1.1、为何没有建立信令标准

+ 因为要让两个浏览器能够进行互操作，并不需要建立信令标准。

##### 4.1.2、媒体协商

+ **信令最重要的功能**在于，在参与对等连接的两个浏览器之间交换会话描述协议（Session Description Protocol, SDP）对象中包含的信息。

##### 4.1.3、标识和身份验证

+ 除信令外，还有两种渠道可用于确定身份:
  + 第一种渠道是参见Web应用程序提供的上下文
  + 第二咱渠道是查看URL中可能传递的标识，此URL可能包含随机令牌

##### 4.1.4、控制媒体会话

##### 4.1.5、双占用分解

#### 4.2、信令传输

##### 4.2.1、HTTP传输

+ 跨域资源共享（Cross-Origin Resources Sharing, CORS）

##### 4.2.2、WebSocket传输

##### 4.2.3、数据通道传输

#### 4.3、信令协议

##### 4.3.1、信令状态机

##### 4.3.2、信令标识

##### 4.3.3、HTTP轮询

##### 4.3.4、WebSocket代理

##### 4.3.5、Google应用程序引擎通道API

##### 4.3.6、WebSocket SIP

##### 4.3.7、WebSocket Jingle

##### 4.3.8、数据通道专有信令

##### 4.3.9、使用叠加网络的数据通道

#### 4.4、信令选项总结

#### 4.5、可运行的信令通道代码示例

##### 4.5.1、Web服务器

+ 4.5.1.1、
+ 4.5.1.2、

##### 4.5.2、信令通道

+ 4.5.2.1、

##### 4.5.3、客户端WebRTC应用程序

#### 4.6、参考资料

### chap5、对等媒体

#### 5.1、WebRTC媒体流

##### 5.1.1、不采用WebRTC的媒体流

+ **媒体流就与Web浏览通信**，走到一起了，【Web服务器，就要多处理，以及视频时高带宽】

##### 5.1.2、采用

+ **Internet 跃点**

#### 5.2、WebRTC和网络地址转换

+ NAT本质是**NAPT**（网络地址和端口转换器）
+ 端到端，以及**对等协议**时，就会遇到困难

##### 5.2.1、通过多个NAT的对等媒体流

+ **打洞技术**（见chap9）

##### 5.2.2、通过通用NAT的对等媒体流

##### 5.2.3、私有地址和公共地址

+ *得想一下，NAT的规则或作用呢，lionel*

#### 5.3、STUN服务器

+ **NAT会话遍历实用工具（Session Traversal Utilities for NAT）**
+ 每个浏览器通过发送STUN数据包来查询STUN服务器，本身不常见

#### 5.4、TRUN服务器

+ **中继型NAT遍历（Traversal Using Relay aroud NAT）**
+ 浏览器通过查询TURN服务器来获取媒体中继地址。
+ **媒体中继地址**：是一个公共IP地址，用于转发从浏览器收到的数据包，或者将收到的数据包转发给浏览器。

#### 5.5、候选项

+ 实现打洞的协议称为**交互式连接建立（Interactive Connectivity Establishment）ICE协议**

### chap6、对等连接和提议/应答协商

#### 6.1、对等连接

+ `RTCPeerConnection`是WebRTC技术的主要API，**在两个浏览器之间建立媒体和数据连接路径**
+ 是一组路径建立进程（ICE）以及一个可确定应建立哪些媒体和路径的协商器
+ `addStream()`，**不会产生任何媒体流。只是告诉浏览器，你希望浏览器与对等端就MediaStream发送问题进行协商**

#### 6.2、提议/应答协商

#### 6.3、JavaScript提议/应答控制

#### 6.4、可运行的代码示例：对等连接和提议/应答协商

### chap7、数据通道

#### 7.1、数据通道简介

+ 数据通道模型基于WebSocket建立，具有简单且可设置的send方法和onmessage处理程序。

#### 7.2、使用数据通道

#### 7.3、可运行的数据通道代码示例

### chap8、W3C文档

#### 8.1、WebRTC API参考

+ 0
  + **PC是peer Connection，gUM是getUserMedia**
+ WebRTC API接口摘要
  + MediaStreamTrack
+ WebRTC RTCPeerConnection API
+ WebRTC SDP处理 API
  + RTCSessionDescription
+ WebRTC ICE处理 API
+ WebRTC 数据通道 API
+ WebRTC DTMF处理API
  + RTCDTMFSender
+ WebRTC 统计数据处理API
  + RTCStatusReport
+ 8 WebRTC 身份处理API
+ 9 WebRTC 流处理 API
+ 10 WebRTC 轨道处理API
  + MediaStreamTrack
+ 11 WebRTC 约束和功能API
  + Constrainable

#### 8.2、WebRTC 建议

#### 8.3、WebRTC 草案

#### 8.4、相关工作

##### 8.4.1、MediaStream录制API规范

##### 8.4.2、图像捕获API

##### 8.4.3、future

##### 8.4.4、媒体隐私

##### 8.4.5、MediaStream的非活动状态

#### 8.5、参考资料

### chap9、NAT和防火墙穿透

### chap10、协议

### chap11、IETF文档

### chap12、与IETF相关的RFC文档

### chap13、安全与隐私

### chap14、实现和应用

#### 14.1、浏览器

#### 14.2、其他浏览器

#### 14.3、STUN和TURN服务器实现

+ rfc5766-turnserver是一个出色的基于标准的开源STUN和TURN服务器。

#### 14.4、参考资料