## 《高性能CUDA应用设计与开发》

### chap1、cuda入门与编程思想

#### 1.2、一个用以区别CUDA与传统程序开发的示例

+ seqSerial.cpp
+ **注意编译选项**，cuda的话是用`nvcc seqSerial.cpp -o seqSerial`
+ 使用Thrust API 大规模并行的CUDA代码，seqCuda.cu

#### 1.3、选择合适的CUDA API

+ 数据并行 Thrust API
+ C或C++的 Runtime API
  + 有个例子，*没用名称*，例1.7（P6）
+ C或C++的Driver API

#### 1.4、CUDA的一些基本概念

+ 用于CUDA的GPU是安装于主机系统（Host）中的独立设备

  + GPCPU通过PCIe总线与主机系统相连
  + 每个GPCPU都是一个独立的设备（Device）
  + CUDA数据传输方式
    + cudaMemory()，使用Thrust API
    + 通过**页锁定内存的映射**进行隐式数据传输
    + *未写*
    + 在最底层，主机通过设备驱动程序的软件模块同GPGPU硬件进行交互
+ GPGPU运行在一个和主处理器相隔离的存储空间中
  + 除了一些低端设备外，所有的**GPGPU都拥有自己的物理内存（如RAM）**，比传统的主机内存拥有更大的带宽
  + **CUDA4.0提供了统一虚拟编址（UVA）**
+ CUDAKernel是可在主机代码中调用而在CUDA设备上运行的子程序
  + **kernel没有返回值，不算一个函数**
  + kernel通过`__global__`来定义
+ Kernel的调用是异步的，即主机仅仅把要执行的Kernel顺序提交给GPCPU，并不等待其执行完成，然后直接处理后面其他的任务。
  + CUDA提供了两种常用的同步方式：
    + 显式调用cudaThreadSynchronize()，该函数使主机进入阻塞状态，停止运行并等待所有已提交的Kernel执行完毕
    + 利用cudaMemory()实现阻塞式数据传输--在cudaMemory()内部调用了cudaThreadSynchronize()
+ GPU上的基本运行单位是线程
  + **线程级并行（TLP）**
  + **指令级并行（ILP）**
  + **执行配置**，不仅定义了执行Kernel所需的线程数量，还包含了1D，2D或3D计算网格中各维度的分配。**“<<<”和">>>"**括住，其后是由圆括号包含的参数列表
  + `fillKernel<<<nBlocks,nThreadsPerBlock>>>(d_a,n);`，fillKernel是核函数
+ GPU上最大可共享的内存区域称为全局内存

#### 1.5、理解首个Runtime Kernel

+ **线程会计算各自在网络中的线程ID（程序中为tid）**，通过三个Kernel固有变量计算得到，定义如下
  + blockIdx.x：线程块序号，该线程出现在程序员所指定的网络。因为是1D的，所以未使用y和z分量
  + blockDim.x：每个线程块内的或维度线程数量
  + threadIdx.x：用于定位该线程在线程块内的编号
  + *这个计算逻辑，代码方面，还不是完全懂，lionel*

#### 1.6、GPGPU编程的三条法则

##### 1.6.1、将数据放入并始终存储于GPGPU

##### 1.6.2、交给GPGPU足够多的任务

##### 1.6.3、注重GPGPU上的数据重用，以避免带宽限制

#### 1.7、大O记号的思想与数据传输

  #### 1.8、CUDA和Amdahl定律

  #### 1.9、数据并行与任务并行

+ 数据并行或**循环级并行**，即for循环内数据操作的并行
+ 任务并行，**通过将多个任务并发执行来减少串行执行时间**
+ 图1.4（顺序执行），图1.5（异步执行）

#### 1.10、混合执行：同时使用CPU和GPU资源

+ 代码1.11

#### 1.11、回归测试与正确性

#### 1.12、静默错误

+ 代码1.14，实现双精度归约的Thrust API程序，seqBig.cu

  #### 1.13、调试简介

  #### 1.14、UNIX调试方法

  ##### 1.14.1、NVIDIA cuda-gdb调试器

  + `-g -G`参数
  + 例1.17，cuda-gdb启动界面
      + `cuda-gdb SeqRuntime`
+ 还是用`l,bt,r,p,quit`这些命令

  ##### 1.14.2、CUDA内存检查器

+ 例1.23，改进的Kernel导致内存越界访问

  ##### 1.14.3、通过UNIX ddd界面使用cuda-gdb

  + **GNU ddd（数据可视调试器）**

#### 1.15、使用Parallel Nsight进行Windows调试

#### 1.16、本章小结

### chap2、CUDA在机器学习与优化中的应用

### chap10、在云计算和集群环境中使用CUDA

#### 10.1、消息传递接口（MPI）

##### 10.1.1、MPI编程模型

+ `MPI_Send()`
+ MPI定义了并行计算拓扑结构，用来将一组进程组织成一个MPI会话。**MPI会话的尺寸在整个程序执行生命周期中是固定不变的**。

##### 10.1.2、MPI通信器

+ **通信器**是一个分布式对象，用于完成集中式和点对点的通信
  + 集中式，指与指定通信组中的所有处理器有关的MPI函数
  + 点对点，用于单个MPI进程之间的进行传递
+ 调用`MPI_INIT()`后，**MPI会立即创建`MPI_COMM_WORLD`通信器，MPI_COMM_WORLD包含了程序中所有的MPI进程**
+ 一个MPI程序可以创建多个相互独立的通信器，用来独立处理一组任务相关的消息，或从一组进程到另一组相关进程之间进行通信

##### 10.1.3、MPI进程号

+ **进程号也称为“任务ID”**，MPI程序**通常选用进程号为0的进程作为主进程**，通过主进程控制其余进程号大于0的从进程
+ 例10.1，MPI基本程序

##### 10.1.4、主从模式

```cpp
MPI_Comm_size(MPI_COMM_WOLRD,&numtasks);
MPI_Comm_rank(MPI_COMM_WOLRD,&rank);
if(rank == 0){
    //此处放置主进程代码
}else{
    //此处放置从进程代码
}
```



##### 10.1.5、点对点模式基础

#### 10.2、MPI通信机制

+ 在HPC中，**InfiniBand（IB）是一种常用的高速低延迟的通信链路**