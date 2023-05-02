## 《操作系统：设计与实现》

+ 32讲，[视频链接](https://space.bilibili.com/202224425/channel/collectiondetail?sid=192498)
+ http://jyywiki.cn/OS/2022/  课程视频

### 并发1

+ 对应25、26、27三章

#### 多处理器编程：从入门到放弃

##### 上一级复习

+ 状态机
+ **只能改变内存的状态**

##### Three Easy Pieces：并发

+ 为什么这门课（先）讲并发？

##### 并发的基本单位：线程

+ 共享内存的多个执行流
  + 执行流拥有独立的堆栈/寄存器
  + 共享全部的内存（指针可以互相引用）

+ 用状态机的视角来理解

##### 入门：thread.h简化的线程API

+ create(fn)
+ join()
+ 编译时需要增加 -lpthread

##### 入门（cont‘d）

+ Hello, Multi-threaded World!

```c
#include "thread.h"
void Ta() { while (1) { printf("a");
void Tb() { while (1) { printf("b");
int main() {
create(Ta);
create(Tb);
```

##### 入门（cont‘d）

+ 如何证明线程确实共享内存？
  + shm-test.c
+ 如何证明线程具有独立堆栈（以及确定它们的范围？）
  + stack-probe.c
  + `| -sort nk`，**线程大概是8M**
  + `strace`查看系统调用
+ 更多的习题：
  + 创建线程使用的是哪个系统调用
  + 能不能用gdb调试？
  + 基本原则：**有需求，就能做到（RTFM）**

##### thread.h背后：POSIX Threads

+ 设置更大的线程栈

##### 第2课  放弃（1）：原子性

##### 例子：山寨多线程支付宝

##### 例子：Microsoft Diablo 复制物品

##### 例子：求和

```c
#define N 100000000
long sum = 0;
void Tsum() { for (int i = 0; i < N;
int main() {
create(Tsum);
create(Tsum);
join();
printf("sum = %ld\n", sum);
}
```

##### 原子性的丧失

+ 单处理器多线程
+ 多处理器多线程
  + 线程根本就是并发执行的

##### 原子性的丧失：有没有感到后怕？

+ printf还能在多线程程序里调用吗？

##### 实现原子性

+ **互斥和原子性是本学期的重要主题**
  + lock(&lk)
  + unlock(&加)

##### 放弃（2）：顺序

##### 例子：求和（再次出现）

##### 顺序的丧失

+ **编译器对内存访问"eventually consistent"的处理导致共享内存作为线程同步工具的失效**

##### 实现源代码的按顺序翻译

##### 放弃（3）：可见性

##### 例子

```c
int x = 0, y = 0;
void T1() {
x = 1;
asm volatile("" : : "memory"); //
printf("y = %d\n", y);
}
void T2() {
y = 1;
asm volatile("" : : "memory"); //
printf("x = %d\n", x);
}
```

##### 现代处理器：处理器也是（动态）编译器

##### 多处理器间即时可见性的丧失

##### 宽松内存模型（Relaxed/Weak Memory Model）

##### 实现顺序一致性

##### 总结

##### 总结

+ 本次课回答的问题
+ Q：如何理解多处理器系统？
  + Take-away message
+ 多处理器入门
  + 多处理器程序=状态机（共享内存；非确定选择线程执行）
  + thread.h=create+join
+ 多处理器编程：**放弃你对“程序”的旧理解**
  + 不原子、能乱序、不立即可见

#### 理解并发程序执行