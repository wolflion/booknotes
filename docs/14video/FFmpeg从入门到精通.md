## 《FFmpeg入门到精通》

+ 第1部分:FFmpeg的命令行使用篇（1-7）
+ 第2部分:FFmpeg的API使用篇（8-10）

### chap1、FFmpeg简介

#### 1.1、FFmpeg的定义

+ **FF**表示（Fast Forward），**mpeg**（Moving Picture Experts Group，动态图像专家组）

#### 1.2、FFmpeg的历史

+ 法国Fabrice Bellard，2000年初版，找到了**Michael Niedermayer**维护

#### 1.3、FFmpeg的基本组成

+ 封装模块AVFormat
  + MP4，FLV，KV，TS的文件的封装和解封装
  + RTMP，RTSP，MMS，HLS的网络协议的封装
+ 编解码模块AVCodec
  + MPEG4，AAC，MJPEG，H.264（AVC）使用x264解码器，H.265（HEVC）使用x265解码器，MP3（mp3lame）编码，libmp3lame编码器
+ 滤镜模块AVFilter
  + crop滤镜
  + vflip滤镜
  + *lionel，这部分不熟，要好好再看一下*

#### 1.4、FFmpeg的编解码工具ffmpeg

+ 主要工作流程4步
  + 解封装（Demuxing）
  + 解码（Decoding）
  + 编码（Encoding）
  + 封装（Muxing）
+ 共有6个步骤：（*就是多了前后的输入、输出*）
  + 读取输入源
  + 进行音视频的解封装
  + 解码每一帧音视频数据
  + 编码每一帧音视频数据
  + 进行音视频的重新封装
  + 输出到目标

#### 1.5、FFmpeg的播放器ffplay

+ 使用FFmpeg的avformat与avcodec
+ **想要使用ffplay，就需要有SDL来进行ffplay的基础支撑**

#### 1.6、FFmpeg的多媒体分析器ffprobe

+ `./ffprobe -show_streams output.mp4`

#### 1.7、FFmpeg编译

##### 1.7.1、FFmpeg之Win平台编译

+ 使用MinGW-64（Minimalist GNU for Windows），再配合MSYS（Minimal SYStem）

##### 1.7.2、Linux

##### 1.7.3、OS X

#### 1.8、FFmpeg编码支持与定制

+ *lionel，没看*

##### 1.8.1、FFmpeg的编码器支持

##### 1.8.2、FFmpeg的解码器支持

##### 1.8.3、FFmpeg的封装支持

##### 1.8.4、FFmpeg的解封装支持

##### 1.8.5、FFmpeg的通信协议支持

+ `./configure --list-protocols`

#### 1.9、小结

### chap2、FFmpeg工具使用基础(52/300)

+ FFmpeg中常用的工具主要是ffmpeg、ffprobe、ffplay，它们分别用作多媒体的编解码工具、内容分析工具、播放器

#### 2.1、ffmpeg常用命令

+ *lionel，这部分内容可以*

##### 2.1.1、ffmpeg的封装格式

+ AVFormat模块中，libavformat库
+ `ffmpeg --help full`找到`AVFormatContext`参数

##### 2.1.2、ffmpeg的转码参数

+ AVCodec模块，libavcodec
+ `ffmpeg --help full`找到`AVCodecContext`参数

##### 2.1.3、ffmpeg的基本转码原理

+ `./ffmpeg -i  input.rmvb -vcodec mpeg4 -b:v 200k -r 15 -an ouput.mp4`
  + 码率变成200k
  + 帧率变成15
  + 不包含音频（`-an参数`）

#### 2.2、ffprobe高级参数

+ 用于**查看多媒体文件的信息**，`ffprobe --help`
+ `ffprobe -show_packets input.flv`，**show_packets**查看的多媒体数据包信息使用PACKET标签括起来
+ `-show_data`
+ `-show_format`，分析多媒体的封装格式
+ `-show_frames`，查看第一帧的信息
+ `-show_streams`，查看多媒体文件中的流信息，*lionel，这里面的流，怎么理解？*
+ 可以把文件格式转换成**json，csv以及其它**，只需要`ffprobe -of json或csv`
+ **select_stream**，只看音频（a）、视频（v）、字幕（s），`ffprobe -show_frames -select_streams v -of xml input.mp4`

#### 2.3、ffplay常用命令

+ 需要SDL-1.2，才能有效生成ffplay。*lionel，我不知道这么理解，正不正确*

##### 2.3.1、ffplay常用参数

+ `ffplay --help`
+ `ffplay -ss 30 -t 10 inpu.mp4`，从视频的第30秒开始，播放10秒
+ `ffplay -window_title "播放测试" rtmp://up.v.test.com/live/stream`，播放器的窗口显示标题是“播放测试”，但打开网络直播流

##### 2.3.2、ffplay高级参数

+ 用time，查看命令运行时长

##### 2.3.3、ffplay的数据可视化分析应用

+ `ffplay -vismv pf output.mp4`，查看B帧预测与P帧预测信息，希望将信息在窗口中显示出来

#### 2.4、小结

+ ffmpeg主要用于音视频编解码
+ ffprobe主要用于音视频内容分析
+ ffplay主要用于音视频播放、可视化分析

### chap3、FFmpeg转封装（83/300）

#### 3.1、音视频文件转MP4格式

##### 3.1.1、MP4格式标准介绍

+ 0、了解MP4前要知道的几个概念
  + MP4文件由许多个Box与FullBox组成
  + 每个Box由Header和Data两部分组成
  + FullBox是Box的扩展，其在Box结构的基础上，在Header中增加8位version标志和24位的flags标志
  + Header包含了整个Box的长度的大小（size）和类型（type），size为0时，代表这个Box是文件的最后一个Box。
  + Data为Box的实际数据，可以是纯数据，也可以是更多的子Box
  + 当一个Box中的Data是一系列的子Box时，这个Box又可以称为Container Box
  + **MP4常用参考标准排列方式**
    + 一到六级？
+ 1、moov容器
  + 定义了一个MP4文件中的数据信息，类型是moov，是一个容器Atom，至少包含以下的一种
    + mvhd标签
    + cmov标签
    + rmra标签
+ 2、
+ 3、
+ 4、
+ 5、
+ 6、
+ 7、解析hdlr容器
  + 描述了媒体流的播放过程
+ 8、解析minf容器
  + 包含了很多重要的子容器，**容器中的信息将作为音视频数据的映射存在**
    + 视频信息头
    + 音频信息头
    + 数据信息
    + 采样表
+ 9、解析vmhd容器
+ 10、解析smbd容器
+ 11、解析dinf容器
+ 12、解析stbl容器
  + **采样参数列表的容器（Sample Table Atom）**，包含转化媒体时间到实际的sample的信息。包含下列子容器
    + 采样描述容器（Sample Description Atom），stsd
    + 采样时间容器（Time To Sample Atom），stts
    + 
+ 13、解析edts容器
  + 定义了**创建Movice媒体文件中一个track的一部分媒体**，所有的edts数据都在一个表里，包括每一部分的时间偏移量和长度。
+ *lionel，作者分析的官方文档在哪？*

##### 3.1.2、MP4分析工具

+ 1、Elecard StreamEye
+ 2、mp4box
+ 3、mp4info

##### 3.1.3、MP4在FFmpeg中的Demuxer

+ `ffmpeg -h demuxer=mp4`，查看MP4文件的Demuxer信息

##### 3.1.4、MP4在FFmpeg中的Muxer

+ 1、faststart参数使用案例
  + `./ffmpeg -i input.flv -c copy -f mp4 -movflags faststart output.mp4`与`./ffmpeg -i input.flv -c copy -f mp4 output.mp4`区别，就是少了`-movflags faststart`。**将moov移动到mdat前面**
+ 2、dash参数使用案例
  + `./ffmpeg -i input.flv -c copy -f mp4 -movflags dash output.mp4`
+ 3、isml参数使用案例
  + `./ffmpeg -re -i input.flv -c copy -movflags imsl+frag_keyframe -f ismv Stream`

#### 3.2、视频文件转FLV

+ Adobe发布的，**均以FLVTAG的形式存在**

##### 3.2.1、FLV格式标准介绍

+ 0
  + 分2部分：一部分是FLV文件头，另一部分是FLV文件内容
+ 1、FLV文件头解析
  + *lionel，应该也有手册*
+ 2、FLV文件内容格式解析
+ 3、FLVTAG格式解析
+ 4、VideoTag数据解析
+ 5、AudioTage数据解析
+ 6、ScriptData格式解析

##### 3.2.2、FFmpeg转FLV参数

##### 3.2.3、FFmpeg文件转FLV举例

##### 3.2.4、FFmpeg生成带关键索引的FLV

+ `ffmpeg -i input.mp4 -c copy -f flv -flvflags add_keyframe_index output.flv`

##### 3.2.5、FLV文件格式分析工具

+ flvparse
+ FlvAnalyzer
+ `ffprobe -v trace -i output.flv`

#### 3.3、视频文件转M3U8

+ 常见的流媒体格式，**以文件列表的形式存在**

##### 3.3.1、M3U8格式标准介绍

+ EXTM3U
  + **必须包含的标签，并且必须在文件的第一行**
+ EXT-X-VERSION

##### 3.3.2、FFmpeg转HLS参数

##### 3.3.3、FFmpeg转HLS举例

+ `./ffmpeg -re -i input.mp4 -c copy -f hls -bsf:v h264_mp4toannexb output.m3u8`
+ 1、start_number参数
+ 2、hls_time参数
+ 3、
+ 4、
+ 5、
+ 6、
+ 7、
+ 8、
+ 9、

#### 3.4、视频文件切片

+ 与HLS基本类似，但HLS切片在标准中只支持TS格式的切片

##### 3.4.1、FFmpeg切片segment参数

##### 3.4.2、FFmpeg切片segment举例

+ 1、segment_format指定切片文件的格式
+ 2、
+ 3、
+ 4、segment_format按照时间点剪切

##### 3.4.3、FFmpeg使用ss和t参数进行切片

+ 1、
+ 2、
+ 3、

#### 3.5、音视频文件音视频流抽取

##### 3.5.1、FFmpeg抽取音频文件中的AAC音频流

+ `ffmpeg -i input.mp4 -vn -acodec copy output.aac`

##### 3.5.2、FFmpeg抽取音视频文件中的H.264视频流

+ `./ffmpeg -i input.mp4 -vcodec copy -an output.h264`

##### 3.5.3、FFmpeg抽取音视频文件中的H.265数据

#### 3.6、系统资源使用情况

+ `./ffmpeg -re -i input.mp4 -c copy -f mpegts output.ts`

#### 3.7、小结

+ 音视频文件转MP4，FLV，HLS以及视频文件切片处理等相关知识
+ 分析了常用的MP4，FLV，HLS的标准格式并给出相应的FFmpeg应用举例

### chap4、FFmpeg转码

#### 4.1、FFmpeg软编码H.264与H.265

+ FFmpeg本身并不支持H.264，而是通过第三方模块，如X264和OpenH264
+ `ffmpeg -h encoder=libx264`查看支持的像素格式，yuv420p，yuvj420p

##### 4.1.1、x264编码参数简介

+ *lionel，不知道怎么记*

##### 4.1.2、H.264编码举例

+ 1、编码器预译参数设置preset
+ 2、
+ 3、
+ 4、
+ 5、
+ 6、

#### 4.2、FFmpeg硬编解码

##### 4.2.1、Nvidia GPU硬编解码

+ 1、Nvidia硬编码参数
  + `ffmpeg -h encoder=h264_nvenc`
+ 2、举例

##### 4.2.2、Intel QSV

+ 0
+ 1、Intel QSV H.264参数说明
+ 2、举例
+ 3、Intel QSV H.265参数说明
+ 4、举例

##### 4.2.3、树莓派

+ 1、h264_omx参数说明
+ 2、举例

##### 4.2.4、OS X系统

+ 1、OS X硬编解码参数
+ 2、示例

#### 4.3、FFmpeg输出MP3

##### 4.3.1、MP3编码参数介绍

+ `ffmpeg -h encoder=libmp3lame`

##### 4.3.2、MP3的编码质量设置

+ `./ffmpeg -i INPUT -acodec libmp3lame OUTPUT.mp3`

##### 4.3.3、平均码率编码参数ABR

#### 4.4、FFmpeg输出AAC

+ aac：ffmpeg本身的AAC编码实现
+ libfaac：第三方的
+ libfdk_aac：第三方的

##### 4.4.1、FFmpeg中的AAC编码器使用

##### 4.4.2、FDK AAC第三方的AAC编解码Codec库

+ 1、
+ 2、

##### 4.4.3、高质量AAC设置

+ 1、HE-AAC音频编码设置
  + `./ffmpeg -i input.wav -c:a libfdk_aac -profile:a aac_he -b:a 64k output.m4a`
+ 2、

##### 4.4.4、AAC音频质量对比

#### 4.5、系统资源使用情况

+ 执行完`ffmpeg`相关命令后，用`top`看下

#### 4.6、小结

+ 转码需要占用大量的计算资源，所以需要优化

### chap5、FFmpeg流媒体

#### 5.1、FFmpeg发布和录制RTMP流

##### 5.1.1、RTMP参数说明

+ rtmp_app，参数，*怎么拉出来的？*
+ **通过wireshark抓包来分析的**

##### 5.1.2、RTMP参数举例

+ 1、
  + `ffmpeg -rtmp_app live -i rtmp://publish.chinaffmpeg.com -c copy -f flv out.flv` 推流，*录制推流不行*
  + `ffmpeg -re input.mp4 -c copy -f flv -rtmp_app live rtmp://publish.chinaffmpeg.com` 发布流
+ 2、rtmp_playpath参数
+ 3、

#### 5.2、FFmpeg录制RTSP流

+ *基本不太用了*，互联网转向了RTMP，

##### 5.2.1、RTSP参数说明

##### 5.2.2、RTSP参数使用举例

+ 1、
  + `./ffmpeg -rtsp_transport tcp -i rtsp://47.90.47.25/test.ts -c copy -f mp4 output.mp4`
+ 2、

#### 5.3、FFmpeg录制HTTP流

##### 5.3.1、

##### 5.3.2、HTTP参数使用举例

+ 1、seekable参数举例
  + `ffmpeg -ss 300 -seekable 0 -i http://bbs.chinaffmpeg.com/test.ts -c copy output.mp4`
+ 2、headers参数举例
+ 3、

##### 5.3.3、HTTP拉流录制

##### 5.3.4、拉取HTTP中的流录制FLV

#### 5.4、FFmpeg录制和发布UDP/TCP流

##### 5.4.1、TCP和UDP参数说明

##### 5.4.2、TCP参数举例

+ 1、TCP监听接收流
  + `./ffmpeg -listen 1 -f flv -i tcp://127.0.0.1:1234/live/stream -c copy -f flv output.flv`
+ 2、TCP请求发布流
+ 3、
+ 4、TCP拉流超时参数timeout
+ 5、
+ 6、

##### 5.4.3、TCP/UDP使用小结

#### 5.5、FFmpeg推多路流

5.5.1、管道方式输出多路流

5.5.2、tee封装格式输出多路流

5.5.3、tee协议输出多路流

#### 5.6、FFmpeg生成HDS流

##### 5.6.1、HDS参数说明

##### 5.6.2、HDS使用举例

+ 1、window_size参数控制文件列表大小
  + `./ffmpeg -i input -c copy -f hds -window_size 4 output`
+ 2、
+ 3、其它参数

#### 5.7、FFmpeg生成DASH流

##### 5.7.1、DASH参数说明

##### 5.7.2、DASH参数使用举例

+ 1、window_size与extra_window_size参数举例
+ 2、single_file参数举例

#### 5.8、小结

+ 对流媒体的支持，主要是RTMP，RTSP，TCP，UDP，HLS，HDS，DASH

### chap6、FFmpeg滤镜使用

#### 6.1、FFmpeg滤镜filter描述格式

6.1.1、FFmpeg滤镜Filter的参数排列方式

6.1.2、FFmpeg滤镜Filter时间 内置变量

#### 6.2、FFmpeg为视频加水印

##### 6.2.1、文字水印

6.2.2、图片水印

#### 6.3、FFmpeg生成画中画

#### 6.4、FFmpeg视频多宫格处理

#### 6.5、FFmpeg音频流滤镜操作

6.5.1、双声道合并单声道

6.5.2、双声道提取

6.5.3、双声道转双音频流

6.5.4、单声道转双声道

6.5.5、

6.5.6、

#### 6.6、FFmpeg音频音量探测

##### 6.6.1、音频音量获得

##### 6.6.2、绘制音频波形

#### 6.7、FFmpeg为视频加字幕

##### 6.7.1、ASS字幕流写入视频流

6.7.2、ASS字幕流写入封装容器

#### 6.8、FFmpeg视频抠图合并

#### 6.9、FFmpeg 3D视频处理

6.9.1、stereo3d处理3D视频

6.9.2、3D图像转换举例

#### 6.10、FFmpeg定时视频截图

6.10.1、vframe参数截取一张图片

6.10.2、fps滤镜定时获得图片

#### 6.11、FFmpeg生成测试元数据

6.11.1、FFmpeg生成音频测试流

6.11.2、

#### 6.12、FFmpeg对音视频倍速处理

6.12.1、atempo音频倍速处理

+ 半速处理
+ 2倍速处理

##### 6.12.2、setpts视频倍速处理

+ 半速处理
+ 2倍速处理

#### 6.13、小结

+ 滤镜处理avfilter

### chap7、FFmpeg采集设备

#### 7.1、FFmpeg中的Linux设备操作

##### 7.1.1、Linux下查看设备列表

+ `ffmpeg -hide_banner -devices`，*这倒是能输出，但怎么对应的呢*

7.1.2、

7.1.3、

7.1.4、

7.1.5、

7.1.6、

7.1.7、

#### 7.2、FFmpeg中OS X设备操作

7.2.1、

7.2.2、

#### 7.3、FFmpeg中Windows设备操作

7.3.1、

7.3.2、

##### 7.3.3、FFmpeg使用gdigrab采集窗口

+ （1）使用gdigrab采集整个桌面
  + `./ffmpeg.exe -f gdigrab -framerate 6 -i desktop out.mp4`
+ （2）使用gdigrab采集某个窗口
+ （3）使用gdigrab录制带偏移量的视频

#### 7.4、小结

+ Linux，OS X和Win上的设备采集方式，内容涉及到fbdev，v412，x11grab，avfoundataion，dshow，vfwcap，gdigrab

### chap8、FFmpeg接口libavformat的使用（254/300）

#### 8.1、音视频流封装

+ 流程如图
  + （1）API注册，`av_register_all`
  + （2）申请AVFormatContext，`avformat_alloc_output_context2`
  + （3）
  + （4）
  + （5）
  + （6）写容器尾信息

#### 8.2、音视频文件解封装

+ 流程如图
  + （1）API注册，`av_register_all`
  + （2）
  + （3）
  + （4）
  + （5）

#### 8.3、音视频文件转封装

+ 流程如图：
  + （1）API注册，`av_register_all`
  + （2）申请AVFormatContext，`avformat_alloc_output_context2`
  + （3）
  + （4）
  + （5）
  + （6）
  + （7）
  + （8）
  + （9）
  + （10）

#### 8.4、视频截取

+ 流程如图

#### 8.5、avio内存数据操作

+ 流程如图：
  + （1）API注册，`av_register_all`
  + （2）
  + （3）申请AVFormatContext，`avformat_alloc_context`
  + （4）
  + （5）
  + （6）
  + （7）读取帧，`av_read_frame`

#### 8.6、小结

+ 使用API对文件进行了封装（Mux）和解封装（Demux），总结了使用API的具体流程，介绍了**avio相关的知识点**，avio在自定义数据源方面特别有用。

### chap9、FFmpeg接口libavcodec的使用（267/300）

#### 9.1、FFmpeg旧接口的使用

9.1.1、

9.1.2、

9.1.3、

9.1.4、

#### 9.2、FFmpeg新接口的使用

##### 9.2.1、FFmpeg新接口音频编码

9.2.2、

9.2.3、

##### 9.2.4、FFmpeg新接口视频解码

+ 图9-8，视频解码API调用流程

#### 9.3、小结

### chap10、FFmpeg接口libavfilter的使用（284/300）

#### 10.1、filtergraph和filter简述

10.2、FFmpeg中预留的滤镜

10.2.1、音频滤镜

10.2.2、视频滤镜

10.3、avfilter流程图

+ 图10-1

#### 10.4、使用滤镜加LOGO操作

+ 操作步骤
  + （1）API注册，`av_register_all`
  + （2）
  + （3）
  + （4）
  + （5）
  + （6）
  + （7）
  + （8）
+ 参考代码，ffmpeg.org/doxygen/trunk/filtering_vedio_8c-example.html

#### 10.5、小结

### 最后