## 缘起

+ 之前写了一篇《并行计算之CUDA和MPI入门》是从并行计算的角度来说的，从wiki上找了个例子，大概说一下并行计算部分是在GPU上运行的，有个大概的概念
+ 这一篇算是《高性能CUDA应用设计与开发》chap1CUDA入门的读书笔记，自己又从网上找了些资料和代码例子，不太熟的地方，问了问AI，算是初步自洽吧

## 内容

### 一些黑话

+ device侧（gpu）
+ host侧（cpu）
+ kernel（GPU上运行的函数）
+ CUDA（Compute Unified Device Architecture）

### 硬件与软件

+ SP（streaming processor）

+ SM（streaming multiprocessor），也称为GPU大核

+ **一个GPU，有多个SM**

+ 摘抄的，*目前还不太能完全理解*，先记一下

  + ```cpp
    是的，网格（grid）与SM（Streaming Multiprocessor）之间存在关系。
    
    在CUDA架构的GPU中，SM是负责执行CUDA核函数的处理单元。一个GPU通常包含多个SM，每个SM都是一个独立的处理单元，具有自己的寄存器文件、共享内存和调度器等资源。
    
    当一个CUDA程序在GPU上执行时，网格中的线程块会被分配到不同的SM上进行并行执行。每个SM负责执行一个或多个线程块，其中的线程以SIMD（Single Instruction, Multiple Data）方式进行并行操作。
    
    网格的大小和线程块的分配是由GPU的调度器决定的。调度器将线程块动态地分配给可用的SM，以充分利用GPU的计算资源。
    
    因此，网格的大小和线程块的分配会影响到SM的利用情况。较大的网格可以使多个SM同时工作，从而充分发挥GPU的并行计算能力。同时，合理的网格大小和线程块的数量可以避免资源竞争和调度延迟，提高并行计算的效率。
    
    总结来说，网格的大小和线程块的分配与SM紧密相关，通过合理配置网格和线程块，可以充分利用GPU上的SM资源，以实现高性能的并行计算。
    ```

### 线程块的表示

+ **在CUDA编程中，通常使用三维的网格来组织线程块**。（x,y,z）

#### thread,block,grid【这三个是通用名词】

+ *grid代表是个啥概念，跟sm，又是个啥关系？ lionel，这个不太明白*

+ thread
  + 线程是CUDA中最基本的执行单元，它代表了一个独立的计算任务或数据处理任务。
+ block
  + 线程块是由一组线程组成的逻辑单元，这些线程在执行过程中可以协同工作和共享数据。
  + 线程块内的线程可以通过共享内存进行通信和同步。
  + *不同block之间的线程不能通信，lionel*
+ grid
  + 网格是由一组线程块组成的逻辑单元，它表示整个并行计算任务的范围和规模。
  + 网格中的线程块可以独立地执行，并且可以通过全局内存进行通信和同步。

+ 一个网格（Grid）由多个线程块（Block）组成。
+ 每个线程块（Block）内部包含多个线程（Thread）。
+ 线程块（Block）和线程（Thread）可以通过特殊变量来访问其在网格（Grid）和线程块（Block）中的索引。

> thread, block 和 grid其实都可以算作 kernel内的并行层次, 而流(stream)是kernel外的并行层次.
>
> https://developer.aliyun.com/article/994308

#### threadidx，blockDim，blockidx，gridDim【是特殊变量】

+ `blockIdx`是一个特殊变量，用于表示当前线程块在整个网格中的位置。它也是一个`dim3`类型的变量，通常使用三维的线程块索引来标识当前线程块的位置。
+ `blockDim`是一个特殊变量，用于表示线程块中包含的线程数量。
+ `gridDim`是一个特殊变量，用于表示整个网格中包含的线程块数量。
+ `threadIdx`，它是一个特殊变量，用于表示当前线程在其所属线程块中的位置。

### kernel中<<<>>>中的语法

+ `kernelFunction<<<gridDim, blockDim>>>(arguments);`
  + `gridDim`是一个`dim3`类型的变量，用于指定网格的大小，即网格中的线程块数量
  + `blockDim`是一个`dim3`类型的变量，用于指定线程块的大小，即线程块中的线程数量。
+ **kernel中<<<>>>里面可以放多少个参数**，只能放3个
  + `kernelFunction<<<gridDim, blockDim, sharedMemSize>>>(arguments);`
    + 可选的动态共享内存大小（shared memory size）
+ 多个参数的代码，**其实，差异是在，是否绑定默认的流上**
  + ref，https://www.cnblogs.com/devilmaycry812839668/p/16865535.html

```cpp
const int N = 1 << 20;

__global__ void kernel(float *x, int n)
{
    int tid = threadIdx.x + blockIdx.x * blockDim.x;
    for (int i = tid; i < n; i += blockDim.x * gridDim.x) {
        x[i] = sqrt(pow(3.14159,i));
    }
}

int main()
{
    const int num_streams = 8;

    cudaStream_t streams[num_streams];
    float *data[num_streams];

    for (int i = 0; i < num_streams; i++) {
        cudaStreamCreate(&streams[i]);
 
        cudaMalloc(&data[i], N * sizeof(float));
        
        // launch one worker kernel per stream
        kernel<<<1, 64, 0, streams[i]>>>(data[i], N);

        // launch a dummy kernel on the default stream
        kernel<<<1, 1>>>(0, 0);
    }

    cudaDeviceReset();

    return 0;
}
```



### 流stream，代码

+ 流(stream)分为**默认流(或者叫做NULL流)** 和 **非默认流**.

+ 流是CUDA中的一种执行模型，它可以将代码划分为不同的流，并同时执行这些流。
+ **流同步**是指等待流中的所有操作完成。这可以通过调用cudaStreamSynchronize()函数来实现。
  + 流同步可以用于确保在流中的所有操作都已完成之前，不会执行后续的操作。例如，如果我们在一个流中执行了一些计算，然后在另一个流中使用计算结果，那么我们需要在第二个流中执行cudaStreamSynchronize()函数，以确保第一个流中的计算已经完成。
+ Stream是一系列CUDA指令的序列，它们在CUDA设备上按照指定的顺序执行。
+ 使用Stream可以实现CUDA中的并发执行，将多个任务在不同的Stream中同时执行，从而提高性能。
+ 不同的Stream之间可以是并行的，同一个Stream内的指令是串行执行的。

#### API

```cpp
cudaStreamCreate()：创建一个流。
cudaStreamDestroy()：销毁一个流。
cudaStreamSynchronize()：等待一个流中的所有操作完成。
cudaStreamQuery()：查询一个流的状态。
```

#### code

```cpp
/*
创建了一个CUDA Stream stream，然后在kernel函数中启动CUDA内核。通过在内核调用中传递stream参数，我们将内核启动到指定的Stream上。接下来，使用cudaMemcpyAsync()函数将设备端的数据异步复制到主机端，并在调用cudaStreamSynchronize()函数时等待Stream中的操作完成。最后，打印主机端的数据。
*/

#include <stdio.h>
#include <cuda_runtime.h>

__global__ void kernel(int* A) {
    int idx = threadIdx.x + blockIdx.x * blockDim.x;
    A[idx] = idx;
}

int main() {
    const int N = 1024;
    const int threadsPerBlock = 256;

    int* hostA = new int[N];
    int* deviceA;

    cudaMalloc((void**)&deviceA, N * sizeof(int));

    cudaStream_t stream;
    cudaStreamCreate(&stream);

    //这个语法，https://www.cnblogs.com/devilmaycry812839668/p/16865535.html
    //第4个参数stream，并不是 <<<>>> 语法的一部分，而是在调用内核函数时传递给 CUDA 运行时的流（stream）对象。
    kernel<<<(N + threadsPerBlock - 1) / threadsPerBlock, threadsPerBlock, 0, stream>>>(deviceA);

    cudaMemcpyAsync(hostA, deviceA, N * sizeof(int), cudaMemcpyDeviceToHost, stream);

    cudaStreamSynchronize(stream);

    for (int i = 0; i < N; i++) {
        printf("%d ", hostA[i]);
    }
    printf("\n");

    cudaFree(deviceA);
    cudaStreamDestroy(stream);
    delete[] hostA;

    return 0;
}
```



### 事件event，代码

+ 事件是CUDA中的一种同步机制，它可以用来记录流中的时间点。
+ Event是CUDA中的同步机制，用于在CUDA设备上进行时间点的标记和同步操作。
+ 可以将Event看作是一个时间戳，可以在CUDA流的执行过程中插入事件，以标记某个特定的时间点。
+ Event可以用于等待某个事件的发生，从而实现**流间的同步**操作。例如，在一个CUDA流中插入一个Event，然后在另一个流中通过cudaEventSynchronize()函数等待该Event，以确保前一个流的操作完成。
+ Event也可以用于测量CUDA流中的时间间隔，通过cudaEventRecord()和cudaEventElapsedTime()等函数可以测量两个事件之间的时间差。
+ *lionel，流间同步，是个啥，得想一下*

#### API

```cpp
cudaEventCreate()：创建一个事件。
cudaEventDestroy()：销毁一个事件。
cudaEventRecord()：在流中记录一个事件。
cudaEventSynchronize()：等待一个事件完成。
cudaEventElapsedTime()：获取两个事件之间的时间间隔。
```

#### code

```cpp
/*
创建了两个CUDA事件 startEvent 和 endEvent。然后使用 cudaEventRecord() 记录起始事件和结束事件。在执行CUDA计算任务之后，我们使用 cudaEventSynchronize() 等待结束事件发生，以确保任务执行完毕。最后，使用 cudaEventElapsedTime() 计算起始事件和结束事件之间的时间间隔，并打印出来。
*/
#include <stdio.h>
#include <cuda_runtime.h>

int main() {
    cudaEvent_t startEvent, endEvent;
    cudaEventCreate(&startEvent);
    cudaEventCreate(&endEvent);

    cudaEventRecord(startEvent, 0);  // 记录起始事件

    // 在此处执行一些CUDA计算任务

    cudaEventRecord(endEvent, 0);  // 记录结束事件
    cudaEventSynchronize(endEvent);  // 等待结束事件

    float elapsedTime;
    cudaEventElapsedTime(&elapsedTime, startEvent, endEvent);  // 计算时间间隔

    printf("Elapsed time: %.2f ms\n", elapsedTime);

    cudaEventDestroy(startEvent);
    cudaEventDestroy(endEvent);

    return 0;
}
```



### event和stream一起使用

+ 在CUDA中，Event（事件）和Stream（流）是两个重要的概念，用于实现并发执行和异步操作。

+ 事件和流可以一起使用来测量代码的执行时间。例如，我们可以创建一个事件来记录流的开始时间，然后创建一个事件来记录流的结束时间。通过计算这两个事件之间的时间间隔，我们可以得到代码的执行时间。

```cpp
/*
我们首先创建了两个CUDA流 streamA 和 streamB。然后，在 kernelA 中启动了一个CUDA内核，将其放在 streamA 上执行。接着，在 kernelB 中启动了另一个CUDA内核，将其放在 streamB 上执行。

然后，我们创建了两个CUDA事件 eventA 和 eventB。在 streamA 中使用 cudaEventRecord() 记录了 eventA 的发生时间点，在 streamB 中使用 cudaEventRecord() 记录了 eventB 的发生时间点。

接着，我们使用 cudaEventSynchronize() 在主机端等待事件 eventA 和 eventB 的发生。通过这种方式，我们能够确保在主机端检查 streamA 的操作是否已经完成，并打印相应的消息，然后再检查 streamB 的操作是否已经完成，并打印相应的消息。

最后，我们销毁事件和流，并释放相关的内存。

通过结合使用Event和Stream，我们可以实现不同流中的操作的并发执行，并能够在主机端进行流间的同步和检查操作完成的时间点。这种方式可以提高CUDA程序的性能和效率。
*/
#include <stdio.h>
#include <cuda_runtime.h>

__global__ void kernelA(int* A) {
    int idx = threadIdx.x + blockIdx.x * blockDim.x;
    A[idx] = idx;
}

__global__ void kernelB(int* A) {
    int idx = threadIdx.x + blockIdx.x * blockDim.x;
    A[idx] *= 2;
}

int main() {
    const int N = 1024;
    const int threadsPerBlock = 256;

    int* hostA = new int[N];
    int* deviceA;

    cudaMalloc((void**)&deviceA, N * sizeof(int));

    cudaStream_t streamA, streamB;
    cudaStreamCreate(&streamA);
    cudaStreamCreate(&streamB);

    // 在 streamA 上执行 kernelA
    kernelA<<<(N + threadsPerBlock - 1) / threadsPerBlock, threadsPerBlock, 0, streamA>>>(deviceA);

    // 在 streamB 上执行 kernelB
    kernelB<<<(N + threadsPerBlock - 1) / threadsPerBlock, threadsPerBlock, 0, streamB>>>(deviceA);

    // 创建两个事件，用于标记两个流中的操作完成时间点
    cudaEvent_t eventA, eventB;
    cudaEventCreate(&eventA);
    cudaEventCreate(&eventB);

    // 在 streamA 中记录事件 eventA
    cudaEventRecord(eventA, streamA);

    // 在 streamB 中记录事件 eventB
    cudaEventRecord(eventB, streamB);

    // 主机端等待事件 eventA 的发生
    cudaEventSynchronize(eventA);

    // 在主机端检查 streamA 中的操作是否完成
    printf("kernelA completed.\n");

    // 主机端等待事件 eventB 的发生
    cudaEventSynchronize(eventB);

    // 在主机端检查 streamB 中的操作是否完成
    printf("kernelB completed.\n");

    cudaEventDestroy(eventA);
    cudaEventDestroy(eventB);
    cudaStreamDestroy(streamA);
    cudaStreamDestroy(streamB);
    cudaFree(deviceA);
    delete[] hostA;

    return 0;
}
```



## 最后

### 进阶

+ 内存模型
+ 流stream，事件event
  + https://no5-aaron-wu.github.io/2021/11/28/CUDA-3-StreamAndEvent/
+ CPU&GPU
  + GPU（计算密集、数据并行）

### ref

+ [推荐几个不错的CUDA入门教程（非广告）](https://godweiyang.com/2021/01/25/cuda-reading/#!)
+ [CUDA相关](https://face2ai.com/categories/CUDA/)，作者github上写的[例子](https://github.com/Tony-Tan/CUDA_Freshman)，工程名是CUDA_Freshman