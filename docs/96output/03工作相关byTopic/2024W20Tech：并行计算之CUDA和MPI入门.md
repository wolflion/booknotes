## 缘起

+ 以输出，让自己输入，正好本周技术栈上投的跟这个相关，*本周就没怎么投tcp_output.c相关了*
+ 自己根据问题，然后输出的啊，没找人评审过，只是自己记录下，*如有错误，我更后面更正*
+ 用时39min，2024-05-25的11:35-12:14

## 内容

+ 学多线程的一点经验，**不用先去学一些API，先找到一个场景，然后再想着怎么用API，先逻辑通了，API就可以看manual的事**，否则API也没着，也不太会解决问题
  + 多线程就用**生产者-消费者**
  + 并行就用**矩阵加法，乘法**

### 1.1、CUDA和MPI区别，是否可以共用

#### 1.1.1、啥是并行计算

+ 相比对串行来说的，可以把活分解到gpu或者分布式cpu上

#### 1.1.2、CUDA

+ 就是把一些累加工作，放到了gpu上，*这个例子，可以看wiki上，那个经典的矩阵相加*

#### 1.1.3、MPI

+ 分布式吧，没有特殊的gpu设备，*需要用户把矩阵相加，自己分拆，利用主从模式，比如0号进程只负责收，其它进程负责算*

#### 1.1.4、两者可以共用

### 1.2、CUDA-runtime的API例子

#### 1.2.1、以矩阵相为例

+ [链接](https://zh.wikipedia.org/wiki/CUDA#CUDA_Runtime_API)
+ `nvcc vector_add.cu -arch=native -o vector_add.exe`

#### 1.2.2、host和device是啥

+ host可以理解为，进入CPU执行的部分
+ device可以理解为，进入GPU执行的部分

#### 1.2.3、代码

```cu
#include <stdio.h>
#include <stdlib.h>

#define N 1024

// Device code: 送入GPU执行的部分

__global__ void VecAdd(float* A, float* B, float* C)
{
    //线程索引是一个三元组(x, y, z)，其中x、y和z分别表示线程在x、y和z方向上的索引。
    // threadId、blockid、blockdim，这3个名词要理解一下
    int i = threadIdx.x; // thread 的 x 坐标
    if (i < N){
        C[i] = A[i] + B[i];
    }
}
            
// Host code: 送入CPU执行的部分

int main()
{
	size_t size = N * sizeof(float);
	
	int i;

	float* h_A = (float*)malloc(size);
	float* h_B = (float*)malloc(size);
	float* h_C = (float*)malloc(size);

	for(i = 0; i < N; i++){
		h_A[i] = (float)rand() / (float)RAND_MAX;
		h_B[i] = (float)rand() / (float)RAND_MAX;
	}

	float* d_A;
	cudaMalloc(&d_A, size); // cudaError_t cudaMalloc ( void** devPtr, size_t size )
	float* d_B;
	cudaMalloc(&d_B, size);
	float* d_C;
	cudaMalloc(&d_C, size);

	//h2d
	cudaMemcpy(d_A, h_A, size, cudaMemcpyHostToDevice);
	cudaMemcpy(d_B, h_B, size, cudaMemcpyHostToDevice);
	
	// 调用上面写的，device code送入GPU并执行，执行时一个gird只有一个block，一个block有n个thread
	VecAdd<<<1, N>>>(d_A, d_B, d_C); //1个线程块中，包含有N个线程

	cudaMemcpy(h_C, d_C, size, cudaMemcpyDeviceToHost);
	
	for(i = 0; i < N; i++){
        printf("%f ", h_C[i]);
	}

	cudaFree(d_A);
	cudaFree(d_B);
	cudaFree(d_C);
 
    free(h_A);
	free(h_B);
	free(h_C);
}
```



### 1.3、MPI的简单例子

+ 矩阵相加（用了一下 主从模式），*如果啥也不用，看不出rank的区别*
+ 演示了如何使用MPI实现矩阵相加的主从模式。在主从模式中，进程0作为主进程，负责将矩阵A和B发送给其他进程，并接收其他进程的计算结果。其他进程作为从进程，负责接收矩阵A和B，并计算矩阵C。最后，进程0将所有结果组合成最终的矩阵C。
+ *来自于AI*，书上的代码不想敲了

```c
#include <mpi.h>
#include <iostream>

using namespace std;

int main(int argc, char *argv[]) {
  // 初始化MPI环境
  MPI_Init(&argc, &argv);

  // 获取当前进程的秩
  int rank;
  MPI_Comm_rank(MPI_COMM_WORLD, &rank);

  // 获取进程总数
  int size;
  MPI_Comm_size(MPI_COMM_WORLD, &size);

  // 定义矩阵A和B
  int A[size][size];
  int B[size][size];

  // 初始化矩阵A和B
  if (rank == 0) {
    for (int i = 0; i < size; i++) {
      for (int j = 0; j < size; j++) {
        A[i][j] = i + j;
        B[i][j] = i - j;
      }
    }
  }

  // 进程0将矩阵A和B发送给其他进程
  if (rank == 0) {
    for (int i = 1; i < size; i++) {
      MPI_Send(&A[i][0], size, MPI_INT, i, 0, MPI_COMM_WORLD);
      MPI_Send(&B[i][0], size, MPI_INT, i, 0, MPI_COMM_WORLD);
    }
  } else {
    // 其他进程接收矩阵A和B
    MPI_Recv(&A[rank][0], size, MPI_INT, 0, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
    MPI_Recv(&B[rank][0], size, MPI_INT, 0, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
  }

  // 计算矩阵C
  int C[size][size];
  for (int i = 0; i < size; i++) {
    for (int j = 0; j < size; j++) {
      C[i][j] = A[i][j] + B[i][j];
    }
  }

  // 进程0接收其他进程的计算结果
  if (rank == 0) {
    for (int i = 1; i < size; i++) {
      MPI_Recv(&C[i][0], size, MPI_INT, i, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
    }
  } else {
    // 其他进程将计算结果发送给进程0
    MPI_Send(&C[rank][0], size, MPI_INT, 0, 0, MPI_COMM_WORLD);
  }

  // 打印矩阵C
  if (rank == 0) {
    for (int i = 0; i < size; i++) {
      for (int j = 0; j < size; j++) {
        cout << C[i][j] << " ";
      }
      cout << endl;
    }
  }

  // 结束MPI环境
  MPI_Finalize();

  return 0;
}
```



## 最后

### next

+ 1、mpi的进阶用法，比如怎么不死锁？效率最高，*这是自己看的过程中，想到的，但自己没有啥实际经验*
  + 当然，mpi_recv和mpi_send，用得不多，只是简单了解
+ 2、mpi的组通信，**集合通信相关**，自己了解一些集合通信概念，但怎么跟MPI关联，以及集合通信的底层，目前不太完全知道
+ 3、cuda后面怎么投，不太确定，真正用gpu写.cu代码的场景还不太多，*虽然自己有个gpu的开发环境*，这个再具体想下怎么整，遇到了.cu代码后，再去读代码，现在先把调用关系知道一下
+ 4、.cu代码，要再进一步的了解一下基本概念

### ref

+ 《高性能CUDA应用设计与开发》chap10
+ 《高性能计算之并行编程技术-MPI并行程序设计》前12章