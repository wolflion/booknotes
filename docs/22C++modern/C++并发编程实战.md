### chap1、你好，C++并发世界

#### 1.1、什么是并发

#### 1.2、为什么使用并发

#### 1.3、在C++中使用并发和多线程

#### 1.4、开始入门

```c++
#include <iostream>
#include <thread>
using namespace std;
void hello() {
    cout<<"Hello Concurrent World\n";
}

int main() {
    thread t(hello);
    t.join();
}
```



### chap2、管理线程

#### 2.1、基本线程管理

```cpp
// 2.1 当线程仍然访问局部变量时返回的函数
struct func {
    int &i;
    func(int &i_) : i(i_){}
    
    void operator() {
        for(unsigned j = 0; j < 1000000; ++j) {
            do_something(i); //1.对悬空引用可能的访问
        }
    }
};

void oops() {
    int some_local_state = 0;
    func my_func(some_local_state);
    std::thread my_thread(my_func);
    my_thread.detach(); //2.不等待线程完成
} //3.新的线程可能仍在运行
```



#### 2.2、传递参数给线程函数

+ `std::thread t(f,3,std::string(buffer));//不要太依赖于thread构造函数的隐式转换，**直接用`std::string()`**转好`
+ `std::thread t(update_data_for_widget, w, std::ref(data)); //update_data_for_widget被传入引用`
+ `std::bind`
+ `std::ref`
+ `std::unique_ptr`
+ **std::thread的实例是可移动的（movable），即使他们不是可复制的（copyable）。**，所以第三个参数可以用`std::move()`

#### 2.3、转移线程的所有权

```c++
// 2.5 从函数中返回std::thread
std::thread f()
{
    void some_function();
    return std::thread(some_function);
}

std::thread g()
{
    void some_other_function(int);
    std::thread t(some_other_function, 42);
    return t;
}



// 2.6 scoped_thread和示例用法

// 2.7 生成一批线程并等待它们完成
void do_work(unsigned id);
void f()
{
    std::vector<std::thread> threads;
    for(unsigned i=0;i<20;i++){
        threads.push_back(std::thread(do_work,i)); //生成线程
    }
    std::for_each(threads.begin(), threads.end(),std::mem_fn(&std::thread::join)); //轮流在每个线程上调用join()
}
```



#### 2.4、在运行时选择线程数量

+ `std::thread::headware_currency()`

#### 2.5、标识线程

+ `std::this_thread::get_id();`
+ `std::thread::id master_thread;//thread::id是成员类型`

#### 2.6、小结

### chap3、在线程间共享数据

#### 3.1、线程之间共享数据的问题

+ 3.1.1、竞争条件
+ 3.1.2、避免有问题的竞争条件

#### 3.2、用互斥元保护共享数据

+ 3.2.1、使用C++中的互斥元
+ 3.2.2、为保护共享数据精心组织代码
+ 3.2.3、
+ 3.2.4、死锁：问题和解决方案
+ 3.2.5、避免死锁
+ 3.2.6、用std::unique_lock灵活锁定
+ 3.2.7、
+ 3.2.8、

#### 3.3、用于共享数据保护的替代工具

+ 3.3.1、在初始化时保护共享数据
+ 3.3.2、
+ 3.3.3、递归锁

#### 3.4、小结

### chap4、同步并发操作

#### 4.1、等待事件或其它条件

+ 4.1.1、用条件变量等待条件
+ 4.1.2、使用条件变量建立一个线程安全队列

#### 4.2、使用future等待一次性事件

+ 4.2.1、从后台任务中返回值

```c++
#include <future>
#include <iostream>

int find_the_answer_to_ltuase();
void do_other_stuff();
int main()
{
    std::future<int> the_answer = std::async(find_the_answer_to_ltuase);
    do_other_stuff();
    std::cout<<"The answer is "<<the_answe.get()<<std::endl;
}
```



+ 4.2.2、
+ 4.2.3、
+ 4.2.4、
+ 4.2.5、

#### 4.3、

#### 4.4、使用操作同步来简化代码

#### 4.5、小结

### chap5、C++内存模型和原子类型上操作

#### 5.1、内存模型基础

##### 5.1.1、

+ 四个原则：
	+ 1、每个变量都是一个对象，包括作为其成员变量的对象
	+ 2、每个对象至少占有一个内存位置
	+ 3、基本类型都有确定的内存位置（无论类型大小如何，即使它们是相邻的，或是数组的一部分）
	+ 4、相邻位域是相同内存中的一部分

##### 5.1.2、

+ **当有线程对内存位置上的数据进行修改，那就有可能会产生条件竞争**。
+ 为了避免条件竞争，两个线程就需要一定的执行顺序。
	+ 第一种，使用**互斥量来确定访问的顺序**
	+ 第二种，使用**原子操作**，决定两个线程的访问顺序。
+ **如果不规定两个不同线程对同一内存地址访问的顺序，那么访问就不是原子的**。

##### 5.1.3、

+ **每个C++程序中的对象，都有（由程序中的所有线程对象）确定好的修改顺序，且在初始化开始阶段确定**。

#### 5.2、C++中的原子操作及类型

+ 标准原子类型
+ std::atomic_flag上的操作

```c++
// 使用std::atomic_flag的自旋锁互斥实现
class spinlock_mutex{
    std::atomic_flag flag;
public:
    spinlock_mutex():flag(ATOMIC_FLAG_INIT){}
    void lock() {
        while(falg.test_and_set(std::memory_order_acquire));
    }
    void unlock() {
        flag.clear(std::memory_order_release);
    }
};
```



+ std::atomic<bool>的操作
+ std::atomic<T*>上的操作：指针算术运算
+ 标准原子整型的操作
+ std::atomic<>初级类模板
+ 原子操作的自由函数

#### 5.3、同步操作和强制顺序

+ 0

```c++
// 5.2、从不同的线程中读取和写入变量
#include <vector>
#include <atomic>
#include <iostream>

std::vector<int> data;
std::atomic<bool> data_ready(false);

void reader_thread() {
    while(!data_ready.load()) {//1
        std::this_thread::sleep(std::milliseconds(1));
    }
    std::cout << "The answer=" << data[0] << "\n";//2
}

void writer_thread() {
    data.push_back(42);//3
    data_ready = true;//4
}
```



+ 5.3.1、synchronizes-with关系
+ 5.3.2、happens-before关系

```c++
// 5.3、一个函数调用的参数的估计顺序是未指定的
#include <iostream>
void foo(int a, int b) {
    std::cout<<a<<","<<b<<std::endl;
}

int get_num() {
    static int i = 0;
    return ++i;
}

int main() {
    foo(get_num(),get_num());
}
```



+ 5.3.3、原子操作的内存顺序
  + 顺序一致顺序
  + 7、使用获取-释放顺序和MEMORY_ORDER_CONSUME的数据依赖
+ 5.3.4、释放序列和synchronizes-with
+ 5.3.5、屏障
+ 5.3.6、用原子操作排序非原子操作

#### 5.4、小结



### chap6、基于锁

#### 6.1、

+ **序列化（serialzation）**：线程轮流访问被保护的数据。**这是对数据进行串行的访问，而非并发**。

  ##### 6.1.1、

+ **设计并发数据结构时**，需要考量两方面：
	+ 一是确保访问安全
	+ 二是真正并发访问

	  #### 6.2、基于锁
	
+ *用了栈和队列？*



##### 7.1.2、无锁数据结构

+ **无锁结构就意味着线程可以并发的访问数据结构**

##### 7.1.4、无锁数据结构的利与弊

+ 使用无锁结构的主要原因：
	+ 1、将并发最大化。**使用基于锁的容器，会让线程阻塞或等待；互斥锁削弱了结构的并发性**。
	+ 2、鲁棒性。**当一个线程在获取锁时被终止，那么数据结构将会被永久性的破坏**。

#### 7.2、

+ **无锁结构依赖于原子操作和内存序，以确保多线程以正确的顺序访问数据结构**。所有原子操作默认使用的是`memory_seq_cst`内存序。

#### 7.3、

##### 7.3.3、指导建议：小心ABA问题