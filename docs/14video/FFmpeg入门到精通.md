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

#### 2.3、ffplay常用命令

##### 2.3.1、ffplay常用参数

+ `ffplay --help`

##### 2.3.2、ffplay高级参数

##### 2.3.3、ffplay的数据可视化分析应用

#### 2.4、小结

### chap3、FFmpeg转封装（83/300）

#### 3.1、音视频文件转MP4格式

##### 3.1.1、MP4格式标准介绍

+ 1、

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

### chap7、FFmpeg采集设备

#### 7.1、FFmpeg中的Linux设备操作

##### 7.1.1、Linux下查看设备列表

+ `ffmpeg -hide_banner -devices`，*这倒是能输出，但怎么对应的呢*