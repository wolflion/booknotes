## 《FFmpeg入门到精通》

+ 第1部分:（1-7）
+ 第2部分:FFmpeg的API使用篇（8-10）

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
+ 13、解析edts容器

##### 3.1.2、MP4分析工具

+ 1、Elecard StreamEye

##### 3.1.3、MP4在FFmpeg中的Demuxer

+ `ffmpeg -h demuxer=mp4`，查看MP4文件的Demuxer信息

##### 3.1.4、MP4在FFmpeg中的Muxer

+ 1、faststart参数使用案例
+ 2、dash参数使用案例
+ 3、isml参数使用案例

3.2、视频文件转FLV

3.3、视频文件转M3U8

3.4、视频文件切片

#### 3.5、音视频文件音视频流抽取

##### 3.5.1、FFmpeg抽取音频文件中的AAC音频流

+ `ffmpeg -i input.mp4 -vn -acodec copy output.aac`

3.6、系统资源使用情况

3.7、小结

### chap4、FFmpeg转码

#### 4.1、FFmpeg软编码H.264与H.265

4.1.1、x264编码参数简介

4.1.2、H.264编码举例

#### 4.2、FFmpeg硬编解码

4.2.1、Nvidia GPU硬编解码

4.2.2、Intel QSV

4.2.3、树莓派

4.2.4、OS X系统

#### 4.3、FFmpeg输出MP3

4.3.1、MP3编码参数介绍

4.3.2、MP3的编码质量设置

4.3.3、平均码率编码参数ABR

#### 4.4、FFmpeg输出AAC

4.4.1、FFmpeg中的AAC编码器使用

4.4.2、FDK AAC第三方的AAC编解码Codec库

4.4.3、高质量AAC设置

4.4.4、AAC音频质量对比

#### 4.5、系统资源使用情况

#### 4.6、小结

### chap5、FFmpeg流媒体

#### 5.1、FFmpeg发布和录制RTMP流

##### 5.1.1、RTMP参数说明

+ rtmp_app，参数，*怎么拉出来的？*
+ **通过wireshark抓包来分析的**

##### 5.1.2、RTMP参数举例

+ `ffmpeg -rtmp_app live -i rtmp://publish.chinaffmpeg.com -c copy -f flv out.flv` 推流，*录制推流不行*
+ `ffmpeg -re input.mp4 -c copy -f flv -rtmp_app live rtmp://publish.chinaffmpeg.com` 发布流

#### 5.2、FFmpeg录制RTSP流

+ *基本不太用了*

##### 5.2.2、RTSP参数使用举例

+ `./ffmpeg -rtsp_transport tcp -i rtsp://47.90.47.25/test.ts -c copy -f mp4 output.mp4`

#### 5.3、FFmpeg录制HTTP流

##### 5.3.2、HTTP参数使用举例

+ 1、seekable参数举例
  + `ffmpeg -ss 300 -seekable 0 -i http://bbs.chinaffmpeg.com/test.ts -c copy output.mp4`

+ 2、headers参数举例

#### 5.4、FFmpeg录制和发布UDP/TCP流

#### 5.5、FFmpeg推多路流

#### 5.6、FFmpeg生成HDS流

#### 5.7、FFmpeg生成DASH流

#### 5.8、小结

### chap6、FFmpeg滤镜使用

#### 6.1、FFmpeg滤镜filter描述格式

#### 6.2、FFmpeg为视频加水印

##### 6.2.1、文字水印

6.7、FFmpeg为视频加字幕

6.13、小结

### chap7、FFmpeg采集设备

#### 7.1、FFmpeg中的Linux设备操作

##### 7.1.1、Linux下查看设备列表

+ `ffmpeg -hide_banner -devices`，*这倒是能输出，但怎么对应的呢*