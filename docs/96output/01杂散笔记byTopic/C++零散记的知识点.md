## 2020年

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
  + C++锁、互斥锁（mutex），条件锁（条件变量，condition_variable）、自旋锁，递归锁，信号量
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

