## 《网络上C++类面试题整理》

### 程序知识

+ 1、C++从程序到可执行的过程
+ 2、编译原理（静态，动态）区别，分别啥时候用它们
+ 3、C++内存布局，堆、栈的区别，优缺点，栈空间的大小
  + 5个区（堆，栈，全局，常量存储 ，代码区）
+ 4、2G内存OS中，能分配4G的数组吗？
  + *我觉得可以啊，【虚拟内存】*

### C++类

#### 基础

+ 1、命名空间的作用？
+ 2、数组与链表的区别？
+ 3、const作用？
+ 4、C++与C的区别？
+ 5、面向对象与面向过程的区别？
+ 6、库函数是什么，为什么要引入？

#### OO

+ 1、3大特性，多态实现原理和应用场景，虚函数表
  + **多态**（静态多态，动态多态）【静态的话，就是**重载**，动态的话就要**虚函数**】
  + **多态要有3个条件**：继承，虚函数重写，父类指针（引用）指向子类
+ 2、重载与重写，重定义
  + 重载(overload)，类中同名
  + 重写（overriding），子类中存在与父类同名，在调用时优先调用子类
  + 重定义，子类改变基类的行为或属性
+ 3、为啥虚析构，不虚构造
+ 4、**多级继承**（C++多继承），一个子类，多个父类，**菱形继承**出现二义性，用**虚继承**解决，cnblogs.com/D_booker

#### STL

##### 容器类

+ 1、vector与list区别？vector与deque的区别？背后的结构？
+ 2、vector尾部添加元素，需要连续吗？

#### 11&14

+ 1、智能指针，shared_ptr实现底层，有啥问题？
  + *我只记得是计数的差异，引入了weak_ptr*
  + auto_ptr为啥会被弃用，*不清楚呢*，自己想的
+ 2、lambda捕获参数，有哪几种情况？

### DS类

#### 树

+ 1、map的底层原理；增加、查询、删除的时间复杂度；红黑树有什么特殊要求？

#### 排序

+ 1、STL的sort内部实现
  + *系统的话，看下 候捷的 《STL源码剖析》*
  + zhihu上yinnn
  + 数量小的时候用insert_sort()、大的时候用quick_sort()
+ 2、C++ sort为啥要设置几种排序？除了这种，还知道哪些排序，基数排序稳定性是怎么实现的？
+ 3、快排（分治，找一个基准数）与冒泡（交换）
+ 4、各个算法的时间复杂度
  + 冒泡，插入，选择都是$O(n^2)$
  + 快排，以及其它是$O(nlog(_n))$

### Network类

+ 1、TCP/IP的三次握手，*小林网络*应该可以的
+ 2、tcp和udp的区别？TCP是怎么保证可靠性？按序到达如何做到？tcp服务器挂了，客户端会怎么样？
+ 3、网络故障排查

### OS类

+ 1、OS中一个进程要删除正在被写入的文件，能不能删除它
  + *我觉得是可以的，至少在linux上*，按文件系统底层原理，inode对应的写，以及真正文件的映射，是2个过程？
+ 2、线程安全性？标准库线程安全吗？

#### 多线程

### Linux服务器类

#### 命令行类

+ 1、查看上一条命令、查询进程命令

#### API类

+ 1、5种I/O模型，展开讲讲
  + 理解与参考，来自《UNP》chap6
    + 以数据输入为例，一个输入操作，通常分为2个阶段：
      + 1、等待OS内核把数据准备好  **这时候，没准备好，到底是一直等？还是轮询问？**
      + 2、将数据从OS内核复制到用户进程空间
  + 具体的5种
    + 阻塞型I/O：一直等
    + 非阻塞型I/O：返回错误，一直问OS好了没
    + I/O复用（select和poll）*本章*：**阻塞在select或poll上，他们有多个文件描述符**
    + 信号驱动式I/O（SIGIO）*本章*：**就绪了会发送信号给用户进程**，这是何时开始一个I/O操作
    + 异步I/O：**OS通知我们该I/O操作何时完成**
  + ref
    + [答案参考](https://github.com/CyC2018/CS-Notes/blob/master/notes/Socket.md)，*讲了I/O模型、I/O复用*
      + [深入浅出Unix IO模型](https://zhuanlan.zhihu.com/p/375786746)，*感觉这个内容就来自UNP*
      + [Linux网络（三）—— 几种套接字I/O模型](https://blog.csdn.net/qq_41033011/article/details/107822048)，*这又进行了扩展*【同步、异步、阻塞、非阻塞、并发、迭代】
+ 2、I/O多路复用（见游双的[《Linux高性能服务器编程》](https://lionelshen.com/booknotes/10network/Linux%E9%AB%98%E6%80%A7%E8%83%BD%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%96%E7%A8%8B/) 中的chap9:I/O复用
  + select
  + poll
  + epoll
    + lt和et模式

### DP类

+ 1、设计模式是什么？

### DB类

+ 1、数据库索引？

### 其它

+ 与python相比，C++有什么优势

### 手斯代码类

+ 1、括号生成（n=1时，n=3时，生成的括号）

+ 2、最大岛屿数量（dfs？）

+ 3、二叉树的中后序遍历

+ 4、手写一个栈

+ 5、两个栈实现队列

+ 6、leetcode，/coin-lcci，面试题08.11硬币，（25，10，5，1，n=10时，输出几种方式）

  + ```cpp
    int count(int amout,vector<int>&coins){
        int coin_nums = coin.size();
        vector<int> dp(amount+1.0);
        int l = coin.size();
        int dp[0]=1;
        for(int j=0;j<coin_nums;j++){
            for(int i=coins[j];j<=amount;i++){
                dp[i]=dp[i-coins[j]];
            }
        }
        return dp[amount];
    }
    ```

  + 

+ 7、原地顺时针转一个方阵，空间复杂度为O(1)

### 最后

#### ref

+ [超全的C++开发工程师面经]()https://blog.csdn.net/haimianjie2012/article/details/121974652  *我没看这个*
+ [cvte C++面试凉经解答](https://www.nowcoder.com/discuss/479396435532746752?sourceSSR=users)，*这个里面，基本都差不多，但还可以再细化看一下*
+ [华为OD面经（C++）](https://blog.csdn.net/GBS20200720/article/details/124329501)，*整理完了*

#### 知识总结类

+ C++面试，https://gitee.com/MSX12321/interview