### chap1、绪论

#### 1.4、视频压缩编码技术综述

##### 1.4.1、基本结构

+ 分析->量化->编码->传输

+ 根据**信源模型**（分析），视频编码分为两部分:
  + 基于波形的编码
  + 基于内容的编码

##### 1.4.2、基于波形的编码

##### 1.4.3、基于内容的编码

##### 1.4.4、立体（三维）视频编码

### chap2、数字视频

### chap3、视频压缩编码的基本原理

#### 3.1、预测编码

##### 3.1.1、预测编码的基本概念

+ 经过压缩编码后传输的是**像素的预测值和实际值之差**，利用了**同一幅图像的邻近元素之间有着相关性**
+ **差分脉冲编码（DPCM）**

##### 3.1.2、帧内预测编码

+ 1、一维最佳预测
+ 2、二维最佳预测
+ 3、预测编码增益
+ 4、预测编码的量化器
+ 5、二维预测编码器

##### 3.1.3、帧间预测编码

+ 1、单向预测
  + （1）预测原理
  + （2）基于块匹配算法的运动矢量估计
  + （3）搜索方法
+ 2、双向预测
+ 3、基于块运动补偿OBMC

##### 3.1.4、运动估计

#### 3.2、变换编码

##### 3.2.1、变换编码的基本概念

+ **平坦区域和内容缓慢变化区域**占大部分（**即，图像中直流和低频区占大部分**），空间域的图像变换到频域或所谓的变换域，会产生相关性很小的一些变换系数，并可对其进行压缩编码，即**所谓的变换编码**
+ 正交变换

##### 3.2.2、K-L变换

+ Hotelling变换

##### 3.2.3、离散余弦变换（DCT）

+ 一维DCT变换
+ 二维DCT变换

##### 3.2.4、锯齿型扫描和游程编码

+ 1、锯齿型扫描
+ 2、游程编码

#### 3.3、变换编码与预测编码的比较

#### 3.4、熵编码

##### 3.4.1、变长编码

##### 3.4.2、算术编码

### chap4、视频编码标准简介（64/300）

4.1、视频编码发展简史

4.2、H.261标准

4.3、H.263标准

4.4、MPEG-1标准

4.5、MPEG-2标准

4.6、MPEG-4标准

4.7、JPEG标准

4.8、JPEG2000标准

4.9、AVS标准

### chap5、H.264/AVC编码器原理（89/300）

#### 5.1、概述

+ AVC

#### 5.2、H.264/AVC编解码器

##### 5.2.1、H.264编解码器特点

##### 5.2.2、H.264编码器

##### 5.2.3、H.264解码器

#### 5.3、H.264/AVC的结构

##### 5.3.1、名词解释

+ 1、场和帧
  + 视频的一场或一帧可用来产生一个编码图像
  + 视频帧可分成两种类型：连续或隔行视频帧
+ 2、宏块、片
  + 一个编码图像通常划分成若干宏块组成
  + I宏块
  + P宏块
  + B宏块

##### 5.3.2、档次和级

+ 基本档次
+ 主要档次
+ 扩展档次

##### 5.3.3、编码数据格式

+ 1、H.264的视频格式
+ 2、H.264的编码格式
  + （1）得到高的视频压缩比
  + （2）具有良好的网络亲和性
+ H.264的功能分为两层:视频编码层（VCL，Video Coding Layer）和网络提取层（NAL，Network Abstraction Layer）VCL数据，即编码处理的输出，它表示被压缩编码后的视频数据序列

##### 5.3.4、参数图像

##### 5.3.5、片和片组

+ 1、片
+ 2、片组

#### 5.4、帧内预测

+ I_PCM用于以下目的:
  + (1) 允许编码器精确地表示像素值

##### 5.4.1、4*4亮度预测模式

##### 5.4.2、16*16亮度预测模式

##### 5.4.3、8*8色度块预测模式

##### 5.4.4、帧内预测模式的选择和编码

#### 5.5、帧间预测

#### 5.6、H.264的SP/SI帧技术

#### 5.7、整数变换与量化

#### 5.8、CAVLC

#### 5.9、CABAC

#### 5.10、码率控制

#### 5.11、去方块滤波

#### 5.12、其余特征