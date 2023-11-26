## 《POSIX多线程程序设计》

### chap1、概述

#### 1.1、舀水的程序员

+ 程序员：能独立活动的实体
+ 桶和桨：**一次只能由一人拥有的令牌（token）**，也就是**共享数据**或**同步对象**
+ 轻推或喊叫：与同步对象相关的通信机制，**条件变量**，通过信号或广播来指示共享数据的状态变化

#### 1.2、术语定义

##### 1.2.1、异步

+ **异步（asynchronous）**：表明事情相互独立地发生，除非有强加的依赖性。
  + 异步带来的最大复杂性就是：如果你没有同时执行多个活动，那么异步就没有什么优势。

##### 1.2.2、并发

+ **并发（concurrency）**：指事情同时发生。
+ 本书中，并发是指实际上可能串行发生的事情好像同时发生一样。
+ **并发描述了单处理系统中线程或进程的行为特点**。

##### 1.2.3、单处理器和多处理器

+ 单处理器：一台计算机只有一个编程人员可见的执行单元（处理器）。
+ 多处理器：一台计算机拥有多个处理器，它们共享同一个指令集和相同的物理内存。

##### 1.2.4、并行

+ **并行（parallelism）**：并发序列同时执行。**并行的补充含义是事情在相同的方向上独立进行（没有交错）**。

##### 1.2.5、线程安全和可重入

+ **线程安全**：指代码能够被多个线程调用而不会产生灾难性结果（**使用互斥量、条件变量和线程私有数据，实现线程的安全**）
+ 不需要保存永久状态的函数，可以通过整个函数调用的串行化来实现线程安全。
+ **将互斥量和数据流相关联，保护数据而不是代码**。
+ **可重入的函数应该避免依赖任何静态数据，最好避免依赖线程间任何形式的同步**。
+ **可重入**，是指“有效的线程安全”？采用比将函数或库转换成一系列区域更加复杂的方式代码成为线程安全的。

##### 1.2.6、并发控制功能

+ 任何并发系统都应该具有的基本功能：
  + 1、“执行环境”是并发实体的状态
  + 2、“调度”决定在某个给定时刻该执行哪个环境（或环境组），并在不同的环境中切换。
  + 3、“同步”为并发执行的环境提供了协调访问共享资源的一种机制
+ 同步机制
  + 1、互斥量、条件变量、信号量、事件
  + 2、消息传递机制（管道、socket、消息队列）

#### 1.3、异步编程是直观的......

##### 1.3.1、因为UNIX是异步的

+ 时间就是同步机制
+ UNIX管道和文件可以是同步机制
+ 线程比进程简单

##### 1.3.2、因为世界是异步的

+ **如果你认为你已经锁住会话互斥量，别人不应该再中断你，此时你就开始真正理解线程编程了，这会帮你更容易地设计更好的线程代码**。

#### 1.4、关于本书的实例

+ 代码在www.awl.com/cseng/titles/0-201-63392-2/
+ `CFLAGS=-pthread -std1 -wl`
+ sample.c

```c
#include<pthread.h>
#include<errors.h>  //这个在1.9中实现了
```



#### 1.5、异步编程举例

##### 1.5.1、基本的同步版本

+ [chap0105-1alarm.c](https://gitee.com/fewolflion/BookNote/blob/master/01lioneloutput/23multithread/13POSIX%E5%A4%9A%E7%BA%BF%E7%A8%8B%E7%A8%8B%E5%BA%8F%E8%AE%BE%E8%AE%A1/chap01%E6%A6%82%E8%BF%B0/chap0105-1alarm.c)

```c
#include "errors.h"

int main(int argc, char* argv[])
{
	int seconds;
	char line[128];
	char message[64];
	
	while(1)
	{
		printf("Alarm>");
		if(fgets(line,sizeof(line),stdin) == NULL) 
			exit(0);
		if(strlen(line)<=1)
			continue;
		
		if(sscanf(line,"%d %64[^\n]",&seconds,message)<2)
		{
			fprintf(stderr,"Bad command\n");
		}
		else
		{
			sleep(seconds);
			printf("(%d) %s\n",seconds,message);
		}
	}
}
```



##### 1.5.2、多进程版本

+ [chap0105-2alarm_fork.c](https://gitee.com/fewolflion/BookNote/blob/master/01lioneloutput/23multithread/13POSIX%E5%A4%9A%E7%BA%BF%E7%A8%8B%E7%A8%8B%E5%BA%8F%E8%AE%BE%E8%AE%A1/chap01%E6%A6%82%E8%BF%B0/chap0105-2alarm_fork.c)

##### 1.5.3、多线程版本

+ [chap0105-3alarm_thread.c](https://gitee.com/fewolflion/BookNote/blob/master/01lioneloutput/23multithread/13POSIX%E5%A4%9A%E7%BA%BF%E7%A8%8B%E7%A8%8B%E5%BA%8F%E8%AE%BE%E8%AE%A1/chap01%E6%A6%82%E8%BF%B0/chap0105-3alarm_thread.c)

##### 1.5.4、总结

+ **比较闹钟程序的两个异步版本是理解线程编程的很好途径**。
+ fork版本中
  + 每个闹铃有一个从主进程拷贝的独立地址空间，这意味着可以将闹铃时间和显示文本保存在局部变量中，一旦建立子进程（fork调用返回），父进程就可以改变这些变量而不会影响闹铃子进程。

#### 1.6、线程的好处

##### 0

+ 多线程编程模的优点：
  + 1、
  + 2、
  + 3、一种模块化编程模型，能清晰地表达程序中独立事件间的相互关系

##### 1.6.1、并行

+ 在多处理器系统中，线程模式可以让一个进程同时执行多个独立运算。

##### 1.6.2、并发

+ 线程编程模式允许程序在等待像I/O之类的阻塞操作时继续其他计算，这对于网络服务器和客户端是有用的
+ 另一种编写异步应用的方法是**将每个活动视为事件**。**事件**由隐藏的后台进程排队，串行分派并被程序处理--通常使用向分派器注册过的回调函数。
+ **事件机制大大减少了使用信号和异步I/O的复杂度**。

##### 1.6.3、编程模型
#### 1.7、线程的代价

##### 1.7.1、计算负荷

+ 线程代码中的负荷代价包括由于线程间同步所导致的直接影响
+ 在几乎任何线程代码中你都需要使用某种同步机制

##### 1.7.2、编程规则

+ 你需要明白同步协议和程序中的不变量（invariant），你不得不避免死锁、竞争和优先级倒置。
+ **一个进程中的所有线程共享地址空间，线程间没有保护界限**。

##### 1.7.3、更难以调试

#### 1.8、选择线程还是不用线程

+ 最适用使用线程的实现以下功能的应用：
  + 计算密集型应用：**计算分解到多个线程中实现**
  + I/O密集型应用：为提高性能，将I/O操作重叠。**很多线程可以同时等待不同的I/O操作**。分布式服务器应用就是很好的实例，它们必须响应多个客户的请求，必须为通过慢速的网络连接主动提供I/O做好准备。

#### 1.9、POSIX线程概念

+ *互斥量来保护共享数据，条件变量来通信*

##### 1.9.1、结构概览

+ 线程系统的三个基本要素：**执行环境、调度和同步**。
+ 基本的Pthreads同步模型使用互斥量来保护共享数据、使用条件变量来通信，还可以使用其他的同步机制，如信号量、管道和消息队列。
+ 互斥量允许线程在访问共享数据时锁定它，以避免其他线程的干扰。
+ 条件变量允许线程等待共享数据到达某个期望的状态（例如队列非空或者资源可用）

##### 1.9.2、类型和接口

##### 1.9.3、错误检查

+ [chap0109-1thread_error.c](https://gitee.com/fewolflion/BookNote/blob/master/01lioneloutput/23multithread/13POSIX%E5%A4%9A%E7%BA%BF%E7%A8%8B%E7%A8%8B%E5%BA%8F%E8%AE%BE%E8%AE%A1/chap01%E6%A6%82%E8%BF%B0/chap0109-1thread_error.c)
+ errors.h
  + `pthread_getspecific()`，**仅返回线程私有数据--共享“Key”值**。

```c
#ifndef __errors_h
#define __errors_h

#include <unistd.h>
#include <errno.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#ifdef DEBUG
#define DPRINTF(arg) printf arg
#else
#define DPRINTF(arg)
#endif

#define err_abort(code,text) do { \
	fprintf(stderr, "%s at \"%s\":%d; %s\n",\
	text, __FILE__,__LINE__,strerror(code));\
	abort();
}while(0)

#define errno_abort(text) do { \
	fprintf(stderr, "%s at \"%s\":%d; %s\n",\
	text, __FILE__,__LINE__,strerror(errno));\
	abort();
}while(0)

#endif
```



### chap2、线程

#### 2.1、建立和使用线程

+ [chap0201-1lifecycle.c]

```c
#include <pthread.h>
#include "errors.h"

/*Thread start routine.*/
void *thread_routine(void *arg)
{
	return arg;
}

int main(int argc, char *argv[])
{
	pthread_t thread_id;
	void *thread_result;
	int status;
	
	status = pthread_create(
	&thread_id,NULL,thread_routine,NULL);
	
	if(status != 0)
		err_abort(status,"Create thread");
	
	status = pthread_join(thread_id,&thread_result); // 以阻塞的方式等待thread指定的线程结束
	if(status != 0)
		err_abort(status,"Join thread");
	
	if(thread_result == NULL)
		return 0;
	else
		return 1;
}
```



#### 2.2、线程的生命周期

+ 线程状态（**状态转换图，比较重要**）
	+ 就绪keady
	+ 运行kuning
	+ 阻塞blocked
	+ 终止terminated
	
+ **线程状态转换**
##### 2.2.1、创建线程
+ `pthread_create()`
##### 2.2.2、线程启动
##### 2.2.3、运行和阻塞
##### 2.2.4、终止
##### 2.2.5、回收

### chap3、同步

#### 3.1、不变量、临界区和谓词
+ **不变量（invariant）**：是由程序作出的假设，**特别是有关变量组间关系的假设**，比如，数据之间的关系
+ **临界区**：影响共享数据的代码段。**临界区总能够对应到一个数据不变量，反之亦然**。
+ 不可变量可能会被破坏，而且会经常被独立的代码段破坏。窍门是要**确保在”不可预见“的线程访问被破坏的不变量之前将其修复**。
+ **谓词（Predicate）** 是描述代码所需不变量的状态的语句。在英语中，谓词可以是”队列为空“
  + 布尔变量、函数返回值、指针为空
#### 3.2、互斥量
+ **同步不仅仅在修改数据时重要，当线程需要读取其他线程写入的数据时，而且数据写入的顺序也有影响时，同样需要同步**。
##### 3.2.1、创建和销毁互斥量
+ chap0302-1mutex_static.c
```c
#include <pthread.h>
#include "errors.h"

/*
Declare a structure, with a mutex, statically initialized. This is 
the same as using pthread_mutex_init, with the default attributes.
*/

typedef struct my_struct_tag{
	pthread_mutex_t mutex;  // protects access to value
	int             value;  // Access protected by mutex
}my_struct_t;

my_struct_t data = { PTHREAD_MUTEX_INITALIZER, 0 };

int main(int argc, char* argv[])
{
	return 0;
}
```

+ chap0302-2mutex_dynamic.c

```c
#include<pthread.c>
#include"errors.h"

typedef struct my_struct_tag{
    pthread_mutex_t mutex;
    int value;
}my_struct_t;

int main(int argc, char *argv[]){
    my_struct_t *data;
    int status;
    data = malloc(sizeof(my_struct_t));
    if(data == NULL)
        errno_abort("Allocate structure");
    status = pthread_mutex_init(&data->mutex, NULL);
    if(status != 0)
        err_abort(status, "Init mutex");
    status = pthread_mutex_destroy(&data->mutex);
    if(status != 0)
        err_abort(status, "Destroy mutex");
    (void)free(data);
    return status;
}
```



##### 3.2.2、加锁和解锁互斥量
+ chap0302-3alarm_mutex.c



+ 3.2.2.1、非阻塞式互斥量锁  chap0302-4trylock.c
##### 3.2.3、使用互斥量实现原子操作
##### 3.2.4、调整互斥量满足工作
##### 3.2.5、使用多个互斥量
+ 3.2.5.1、加锁层次 backoff.c
+ 3.2.5.2、链锁
#### 3.3、条件变量
+ **条件变量是用来通知共享数据状态信息的**。使用条件变量来通知队列已空
+ **等待条件变量总是返回锁住的互斥量**
+ **条件变量的作用是发信号，而不是互斥**
##### 3.3.1、创建和释放条件变量
+ cond_static.c
##### 3.3.2、等待条件变量
+ cond.c
##### 3.3.3、唤醒条件变量等待线程
+ 发信号或广播
##### 3.3.4、闹铃实例的最终版本
+ alarm_cond.c
#### 3.4、线程间的内存可视性
+ Pthreads提供了一些有关内存可视性的基本规则
	+ 1、当线程调用pthread_create时，它所能看到的内存值也是它建立的线程能够看到的。
	+ 2、当线程解锁互斥量时看到的内存中的数据，同样也能被后来直接锁住（或通过等待条件变量锁住）相同互斥量的线程看到。
	+ 3、线程终止（或者通过取消操作，或者从启动函数中返回，或者调用pthread_exit）时看到的内存数据，同样能够被连接线程的其他线程（通过调用pthread_join）看到。
	+ 4、线程发信号或广播条件变量时看到的内存数据，同样可以被唤醒的其它线程看到。



### chap4、使用线程的几种方式

+ 线程编程模型：
	+ 流水线
	+ 工作组
	+ 客户端/服务器

#### 4.1、流水线
+ 图
+ pipe.c
#### 4.2、工作组
+ 图
+ crew.c
#### 4.3、客户端/服务器
+ 图
+ server.c

### chap5、线程高级编程
#### 5.1、一次性初始化
+ **使用`pthread_once`的主要原因是原来不能静态地初始化一个互斥量**。
+ once.c
#### 5.2、属性
##### 5.2.1、互斥量属性
+ mutex_attr.c
##### 5.2.2、条件变量
+ cond_attr.c
##### 5.2.3、线程属性
+ thread_attr.c
#### 5.3、取消
+ 取消一个线程就像告诉一个人停止他正在做的工作一样。
+ **取消允许你告诉一个线程关掉自己**。
+ cancel.c
##### 5.3.1、推迟取消
+ cancel_disable.c
##### 5.3.2、异步取消
+ **避免异步的取消**（问题是你能异步取消过程中做事是有限的）
+ cancel_async.c
+ cancel_cleanup.c
+ cancel_subcontract.c
#### 5.4、线程私有数据
+ 0
	+ 静态变量（static）、外部变量（extern）或堆变量的值，将是上次任何线程改写的值。
	+ 一个线程真正拥有的唯一私有存储是处理器寄存器
##### 5.4.1、建立线程私有数据
+ tsd_once.c
##### 5.4.2、使用线程私有数据
##### 5.4.3、使用destructor函数
+ tsd_destructor.c
#### 5.5、实时调度
##### 5.5.1、POSIX实时选项
##### 5.5.2、调度策略和优先级
+ sched_attr.c
+ sched_thread.c
##### 5.5.3、竞争范围和分配域
##### 5.5.4、实时调度的问题
##### 5.5.5、优先级相关互斥量
+ 5.5.5.1、优先级ceiling互斥量
+ 5.5.5.2、优先级继承互斥量
#### 5.6、线程和核实体
##### 5.6.1、多对一（用户级）
##### 5.6.2、一对一（内核级）
##### 5.6.3、多对少（两级）


### chap6、
#### 6.1、fork
+ **除非你打算很快地exec一个新程序，否则应避免（如果你能的话）在一个多线程的程序中使用fork**。
##### 6.1.1、fork处理器
+ atfork.c
#### 6.2、exec
+ **exec函数的功能是消除当前程序的环境并且用一个新程序代替它**。
#### 6.3、进程结束
#### 6.4、stdio
##### 6.4.1、flockfile和funlockfile
+ flock.c
##### 6.4.2、getchar_unlocked和putchar_unlocked
+ putchar.c
#### 6.5、线程安全的函数
##### 6.5.1、用户和终端ID
+ getlogin.c
##### 6.5.2、目录搜索
##### 6.5.3、字符串token
##### 6.5.4、时间表示
##### 6.5.5、随机数产生
##### 6.5.6、组和用户数据库
#### 6.6、信号
##### 6.6.1、信号行为
##### 6.6.2、信号掩码
##### 6.6.3、pthread_kill
+ susp.c
##### 6.6.4、sigwait和sigwaitinfo
+ sigwait.c
##### 6.6.5、SIGEV_THREAD
+ sigv_thread.c
##### 6.6.6、信号灯：与信号捕获函数同步
+ semaphore_singal.c

### chap7、Real code
#### 7.1、扩展同步
##### 7.1.1、barriers

+ barrier.h
+ barrier.c
+ barrier_main.c

  ##### 7.1.2、读写锁
+ rwlock.h
+ rwlock.c
+ rwlock_main.c
#### 7.2、工作队列管理器
+ workq.h
+ workq.c
+ workq_main.c
#### 7.3、对现存库的处理
##### 7.3.1、将函数库修改为线程安全的

##### 7.3.2、合适遗留库

### chap8、避免调试的提示
#### 8.1、避免不正确的代码
##### 8.1.1、避免依赖“线程惯量”

+ inertia.c

  ##### 8.1.2、别将你的赌押在线程竞争上

  ##### 8.1.3、合作避免僵局

  ##### 8.1.4、小心优先级倒置

  ##### 8.1.5、绝不要在谓词之间共享条件变量

  ##### 8.1.6、共享堆栈和相应的内存破坏
#### 8.2、避免性能问题
##### 8.2.1、了解并发串行化

##### 8.2.2、使用正确数目的互斥量

##### 8.2.3、绝不要与缓存作对

### 最后

#### 履历

+ 2021-9-19读了本书的一大部分，并手写了笔记，2023-03月左右又把这本书的笔记给电子化了一下【*之前最大的问题，还是在于只是抄写了书的笔记，而没有真正的理解代码和应用场景*】