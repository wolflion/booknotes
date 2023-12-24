## 《Unix网络编程卷2-IPC》

### 0

#### 目录

+ part1、简介（1-3）
+ Part2、消息传递(4-6)
+ Part3、同步（7-11）
+ Part4、共享内存区（12-14）
+ Part5、远程过程调用（15-16）

### chap7、互斥锁与条件变量

#### 7.1、概述

+ **同步**是为了允许在线程或进程间共享数据

#### 7.2、互斥锁：上锁与解锁

+ 任何时刻只有一个线程在执行其临界区的指令
+ pthread_mutex_t类型
  + 静态分配，`PTHREAD_MUTEX_INITIALIZER`
  + 动态分配，`pthread_mutex_init()`
+ 虽然互斥锁保护的是临界区，**实际上保护的是在临界中被操纵的数据**
+ 互斥锁是**协作锁**（cooperative）

7.3、生产者-消费者问题

+ mutex/prodcons2.c

7.4、对比上锁与等待

+ mutex/prodcons3.c

#### 7.5、条件变量：等待与信息发送

+ 互斥锁用于**上锁**，**条件变量，则用于等待**，**每个条件变量总是有一个互斥锁与之关联**
+ `pthread_cond_t`变量
+ `pthread_cond_wait()`
+ `pthread_cond_signal()`，*这里的signal是啥？*

+ mutex/prodcons6.c

7.6、条件变量：定时等待与广播

7.7、互斥锁和条件变量的属性

#### 7.8、小结

### chap8、读写锁