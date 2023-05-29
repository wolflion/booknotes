## 《FFmpeg入门到精通》

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

3.1、音视频文件转MP4格式

3.2、视频文件转FLV

3.3、视频文件转M3U8

3.4、视频文件切片

3.5、音视频文件音视频流抽取

3.6、系统资源使用情况

3.7、小结

### chap4、FFmpeg转码

### chap5、FFmpeg流媒体

#### 5.1、FFmpeg发布和录制RTMP流

##### 5.1.1、RTMP参数说明

+ rtmp_app，参数，*怎么拉出来的？*
+ **通过wireshark抓包来分析的**

##### 5.1.2、RTMP参数举例

#### 5.2、FFmpeg录制RTSP流

#### 5.3、FFmpeg录制HTTP流

#### 5.4、FFmpeg录制和发布UDP/TCP流

#### 5.5、FFmpeg推多路流

#### 5.6、FFmpeg生成HDS流

#### 5.7、FFmpeg生成DASH流

#### 5.8、小结

