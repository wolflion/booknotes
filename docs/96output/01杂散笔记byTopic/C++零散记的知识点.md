## 2019年

### 53W（26-31）

#### 12.28

+ exit()和_exit()区别？有啥会导致什么问题的问题单

## 2020年

### W2（1.2-1.8）

#### 01-06

+ `long long mid = (left+right)>>1;` **按位运算**比除法快
+ `try{...}then`，std::invalid_argument、std::out_of_range，使用方法，没怎么用过
+ vi `~/.bashrc`，配置环境变量，PATH相关，修改后，source一下

#### 01-07

+ 返回string类型的空，`return "";`，这个对不？

### W3（1.9-1.15）

#### 01-13

+ int与string互转，string与char*互转
+ C++map排序的**仿函数**，linux中的cgroup知识

#### 01-14

+ 容器的各种用法，`pair<>`是在utility头文件，

#### 01-15

+ *2023年才开始做的，所谓刷题中的 自定义排序*

+ map中的sort问题（小于号重载、仿函数的应用），**sort默认小于号排序**
+ 2层map

### w4（1.16-1.22）

#### 01-19

+ `std::lock_guard<>`是不是只能保存一行，而不是一个语句块？

#### 01-20

+ queue的遍历？如何实现？

#### 12.08

+ 顺序容器：由插入顺序决定，添加到容器里
  + vector，动态数组，快速访问，尾部之外插入，删除慢
  + string，char*的标准类，与vector类似
  + deque，双向队列，快速访问，头尾插入元素，速度快
  + list，双向链表，任何位置插入，快
+ string与char*好处：
  + 不用担心内存是否足够，长度如何
  + `char *p = str.c_str();`**含有空字符**，`char *p = str.data();`，**不含有空字符**
+ stringstream使用
  + clear()，**并不清除stringstream内容**
  + `str("")`，**才清除stringstream内容**
+ 动态数组、堆，最好将结构或类的指针放入vector中，而不是结构或类本身，可避免移动时的构造与析构
+ 通过erase/insert的返回值来更新迭代器
+ 1780、统计连续字母最大出现次数，string
+ 1487，字符排序，vector
+ 关联容器：set,map为关键字
  + 红黑树，没有频繁插入/删除，主要查找
  + 非严格意义上的**平衡二叉树**
  + **思考**，默认升序，怎么改为降序？《effective stl》的item21
+ 3780、实现一个可存储若干个单词的字典 set
+ 1927、输入任意个整数，重新排序后输出，map
+ 拷贝构造函数
+ 重载/重写

#### 12.23

+ ```cpp
  //recursive_mutex是啥？map_是啥？
  std::lock_guard<std::recursive_mutex> lock(map_); //先锁住，再find
  
  std::shared_ptr<void> dataPtr = std::static_pointer_cast<void>(element);
  
  typedef std::function<void(const std::string &channel) call_t;
  
  (*iter).joinable();  //迭代器的
  (*iter).join();  // lionel，这2个有啥区别
  
  stringstream sin;
  sin.str.c_str();
  ```

+ map中的clear()与eraser()方法异同

+ std::move()的用法

## 2021

#### 01.05

+ 线程栈
+ Linux虚拟地址空间布局以及进程栈、线程栈总结
+ NPU的芯片不一样，1910，1980

### 51W（12.12-12.18）

#### 12.14

+ 回调函数
+ function、bind的用法
+ new、delete、`unique_ptr<T[]>`的用法，C++ Primer的12.2节里的，allocator类

#### 12.15

+ 可变数组，**零长度数组**，`struct Packet{int state; char cData[0];};`

#### 12.16

+ transfrom，emplace_back的区别？ *这应该是算法里的吧*



### 52W（12.19-12.25）

#### 12.20

+ 匿名space，相当于static？*是这样吗？*
+ `__unique_Name__`
+ savage noble，魔幻龙

#### 12.21

+ git blame -L 行数，文件名，
+ vi显示行号，`:set number`

#### 12.22

+ `std::any_of`，test if any element in range full fills condition

#### 12.23

+ `operator float()`，用于将本类对象转换为float
+ pytorch是啥？

#### 12.25

+ 继承类的构造函数初始化
+ std::move()的使用
+ `make_shared()，make_unique()`



### 53（12.26-2022.1.1）

#### 12.28

+ exit()和`_exit()`区别？有啥会导致什么问题的问题单

## 2022年

#### 04.15

+ 1、std::tuple，赋值，初始化，元素数量
+ 2、std::allactor
+ 3、new和delete
+ 4、右值引用，std::move()/emplace_back()
+ 5、虚函数，加不加const的区别
+ 6、auto推导
+ 7、using
+ 8、RAII和智能指针
+ 9、强制转换（**向上转换永远是安全的**）
+ 10、switch
+ 11、enum，**强类型枚举**，enum后面加上class
+ 12、lambda函数
+ 13、容器操作
+ 14、类
  + 静态成员函数，没有一般成员函数隐含的this指针
  + 头文件中extern，声明全局变量
+ 其它
  + 子类和继承里第一个基类的指针地址相同
  + gsl::not_null
  + sizeof，const，extern，const修饰指针、const修饰成员函数
  + thread C++11，join()，detach()
  + C++锁、互斥锁（mutex），条件锁（条件变量，condition_variable）、自旋锁，递归锁，信号量、读写锁
  + 大小端
  + std::set_intersetcion
  + 函数对象functor
  + malloc、realloc、alloc
  + 数组指针，(*p)[1]

#### 05.06

+ 对象移动
  + 右值引用：绑定到一个将要销毁的对象
  + 左值：对象的身份，持久对象
  + 右值：对象的值，短暂临时对象，字面常量
+ std::move()
+ noexcept，不抛异常
+ 移动构造函数，移动赋值运算符
+ 右值引用函数，`Foo sorted()&&;`
+ 多重继承，虚继承
  + 多重继承，构造顺序
  + 多重继承指针相同性问题
  + 隐式类型提升
  + 虚继承，解决二义性问题
    + 子类中只保留一个该虚基类的备份
  + 虚继承在子类中要优先构造
  + 虚析构的构造与析构顺序
+ 函数重载/运算符重载

