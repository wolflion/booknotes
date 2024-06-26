## 整体



## 我自己想的

### 服务器开发相关

+ 基础知识



### 我的疑惑

+ 1、为何引入I/O模型

  + AI说的

    > 引入I/O模型是为了有效地处理输入和输出操作，尤其是在涉及网络编程和并发处理的情况下。I/O（输入/输出）操作是计算机与外部世界进行交互的重要方式，例如从网络接收数据、读取文件、向屏幕输出等。
    >
    > 引入I/O模型的目的是提供一种结构化和可管理的方法来处理这些I/O操作，并解决以下几个关键问题：
    >
    > 1. 阻塞与非阻塞：阻塞I/O会导致应用程序在执行I/O操作时停止，并等待操作完成。这会造成资源的浪费，因为在等待I/O操作完成的同时，应用程序无法执行其他有用的任务。非阻塞I/O模型允许应用程序继续执行其他任务而不必等待I/O操作完成。通过引入I/O模型，可以提供非阻塞I/O的抽象和机制，以实现更高效的资源利用和响应性。
    > 2. 同步与异步：同步I/O模型要求应用程序主动等待I/O操作完成，而异步I/O模型允许应用程序提交I/O操作并继续执行其他任务，而不必等待操作完成。引入I/O模型可以提供异步I/O的机制，使得应用程序可以更高效地处理多个并发的I/O操作。
    > 3. 多路复用：多路复用是指同时监视多个I/O事件并选择就绪的事件来进行处理。通过引入I/O模型，可以提供多路复用的机制，以有效地管理多个并发的I/O操作，减少资源消耗和系统开销。
    > 4. 缓冲和流控制：I/O模型还提供了缓冲和流控制的机制，以优化数据的传输和处理过程。缓冲可以减少对底层资源的频繁访问，提高效率。流控制可以调整数据的传输速率，以适应接收方的处理能力，避免数据的丢失或过载。
    >
    > 总之，引入I/O模型可以提供一种抽象和机制，使得应用程序能够高效地处理输入和输出操作，同时解决阻塞、同步、多路复用和流控制等问题，提高系统的性能、可扩展性和响应性。不同的I/O模型适用于不同的场景和需求，可以根据具体情况选择合适的模型。

+ 2、在简单demo后，为何引入**信号**

  + 在《UNP卷1》想到的

  >在简单的网络编程示例中，引入信号机制可以用于处理以下场景：
  >
  >优雅关闭服务器：当服务器需要关闭时，而仍然有客户端连接或正在处理请求时，可以通过捕获终止信号（例如SIGINT或SIGTERM）来触发服务器的优雅关闭流程。在信号处理程序中，可以逐个关闭连接、保存状态、执行清理操作，然后正常退出服务器。
  >
  >处理超时：如果服务器需要对连接或请求设置超时时间，并在超时发生时采取相应的操作，可以使用定时器信号来实现。在设置超时时，可以使用定时器信号（例如SIGALRM）来触发超时处理程序，然后执行相应的操作，如关闭连接或发送超时响应。
  >
  >动态配置重新加载：如果服务器的配置是可动态修改的，而不希望重启服务器来应用新的配置，可以使用特定信号（例如SIGHUP）来触发重新加载配置文件的操作。在信号处理程序中，可以重新加载配置并对服务器进行必要的更新，从而使新配置生效。
  >
  >进程间通信：在复杂的网络编程场景中，可能涉及多个进程或线程之间的通信和协作。信号可以用作一种通信机制，其中一个进程可以向另一个进程发送信号来通知某个事件的发生，从而触发相应的处理逻辑。
  >
  >这些场景只是示例，在实际应用中可能还会有其他需要引入信号机制的情况。通过使用信号处理机制，可以增加程序的灵活性和可控性，并对特定事件或条件做出适当的响应。

+ 3、

### 我的感想

+ 网络编程的用户态看了一点，单独看API确实没用，**从rfc，内核，manual，应用场景，的角度再去看**



ref

+ 《Linux高性能服务器编程》
+ 《Unix网络编程卷1》



## 杂记

+ 01-04
  + 网络编程的简单例子
    + tcp,udp的c/s实现
    + 多进程的server
    + 多线程的server
    + tcp并发服务器实现
    + 用udp实现聊天室
    + 客户端下载服务器目录文件
  + 通过iperf对tcp,udp灌包
  + web服务器（C/S），分布式（IPC）
+ 01-08
  + zhihu，linux下多线程与并发服务器设计方案及常见问题
+ 01-10
  + 当困惑的时候，看看陈硕的那篇“谈谈网络编程学习经验”，仿hjie的**三境界**
    + 会用
    + 知道原理
    + 自己实现