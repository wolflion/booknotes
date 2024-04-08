### chap2、入门 9 19/213

#### 2.2、开发环境

##### 2.2.2、nvidia设备驱动程序

##### 2.2.3、cuda开发工具箱

##### 自己网上找的

+ [windows环境下CUDA安装及配置](https://blog.csdn.net/Junewang0614/article/details/127151979)
  + 1、安装驱动，找到nvidia控制面板，知道**GeForce MX230**，帮助里面的**系统信息**，再点开**组件**，发现是**CUDA12.3.101**
  + 2、从[链接](https://developer.nvidia.com/cuda-toolkit-archive)去下载12.3.1版本的，step by step
  + 3、验证，`nvcc -version`，设置path`set cuda`
  + 4、安装[cudnn](https://developer.nvidia.com/cudnn)，*不太清楚为啥要装这*
+ [Windows搭建CUDA开发环境 ](https://www.cnblogs.com/GeekPoplar/p/14950828.html)

##### 自己尝试安装的

+ [CUDA安装失败-Nsight compute安装失败-如何测试CUDA是否安装成功？Reason: VS** was not found](https://blog.csdn.net/PSpiritV/article/details/123435283)

### chap3、cuda C简介  16（26/213）

#### 3.2、第一个程序

##### 3.2.1、Hello World

##### 3.2.2、核函数调用

```c
#include <iostream>

//__global__ cuda为标准C增加的，表明这是在设备（device）上运行，而不是在主机上（host）
__global__ void kernel(void){}

int main(void){
    //标记为 设备代码（Device Code）
    kernel<<<1,1>>>();
    printf("hello, world!\n");
    return 0;
}
```



##### 3.2.3、传递参数

#### 3.3、查询设备

#### 3.4、设备属性的使用

### chap4、cuda C并行编程  26（36/213）

#### 4.2、CUDA并行编程

##### 4.2.1、矢量求和运算

###### 基于CPU的矢量求和

###### 基于GPU的矢量求和

##### 4.2.2、一个有趣的示例

### chap5、线程协作  42（52/213）

