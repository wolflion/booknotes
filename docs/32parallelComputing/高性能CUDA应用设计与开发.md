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
  + CUDAKernel是可在主机代码中调用而在CUDA设备上运行的子程序
    + **kernel没有返回值，不算一个函数**
    + kernel通过`__global__`来定义
  + Kernel的调用是异步的，即主机仅仅把要执行的Kernel顺序提交给GPCPU，并不等待其执行完成，然后直接处理后面其他的任务。
  + GPU上的基本运行单位是线程
  + GPU上最大可共享的内存区域称为全局内存

#### 1.5、理解首个Runtime Kernel

#### 1.6、GPGPU编程的三条法则

  + 1、将数据放入并始终存储于GPGPU

  #### 1.8、CUDA和Amdahl定律

  #### 1.9、数据并行与任务并行

  #### 1.13、调试简介

  #### 1.14、UNIX调试方法

  ##### 1.14.1、NVIDIA cuda-gdb调试器

  + `-g -G`参数

  ##### 1.14.2、CUDA内存检查器

  ##### 1.14.3、通过UNIX ddd界面使用cuda-gdb

  + **GNU ddd（数据可视调试器）**

#### 1.15、使用Parallel Nsight进行Windows调试

#### 1.16、本章小结

### chap2、CUDA在机器学习与优化中的应用