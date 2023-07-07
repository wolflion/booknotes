## 《STL源码剖析》

### chap1、STL概论与版本简介

#### 1.9、可能令你困惑的C++语法

##### 1.9.1、stl_config.h中的各种组态

##### 1.9.2、临时对象的产生与运用

##### 1.9.3、静态常量整数成员在class内部直接初始化

##### 1.9.4、increment/decrement/dereference操作符

##### 1.9.5、前闭后开区间表示法`[)`

##### 1.9.6、function call操作符`operator()`

### chap2、空间配置器（allocator）

+ 隐藏在容器背后，*这个可以听一下，侯捷在b站的视频（关于内存的使用）*

#### 2.1、容器配置器的标准接口

+ allactor的各种使用

##### 2.1.1、设计一个简单的空间配置器，JJ::allocator

+ P44，2jjalloc.h
+ P46，2jjalloc.cpp

#### 2.2、具备次配能力（sub-allocation）的SGI空间配置器

##### 2.2.1、SGI标准的空间配置器，std::allocator

##### 2.2.9、重新充填free lists

##### 2.2.10、内存池（memory pool）

#### 2.3、内存基本处理工具

##### 2.3.1、uninitialzed_copy

##### 2.3.1、uninitialzed_fill

##### 2.3.1、uninitialzed_fill_n

### chap3、迭代器（iterator）概念与traits编程技法

3.1、迭代器设计思维-STL关键所在

3.2、迭代器（iterator）是一种smart pointer

3.3、迭代器相应型别（associated types）

3.4、Traits编程技法-STL源代码门钥

3.5、std::iterator的保证

3.6、iterator源代码完整重列

3.7、

### chap4、序列式容器（sequence containers）

#### 4.1、容器的概观与分类

### chap5、关联式容器

### chap6、算法（algorithms）

6.1、算法概观

6.2、算法的泛化过程

6.3、数值算法<stl_numeric.h>

6.7、其它算法

### chap7、仿函数（functors，另名  函数对象 function objects）

#### 7.1、仿函数（functor）概观

+ 开始叫仿函数，后来规范叫**函数对象**（一种具有函数特质的对象）
+ 函数指针不能满足STL对抽象性的要求，无法和STL其它组件搭配，产生更灵活的变化

```cpp
//lionel，这两个场景的差异性，第1种，我确实还有点懵的
greater<int> ig;
cout<<boolalpha<<ig(4,6);//false  【ig调用其operator()，并赋予两个参数4，6】
cout<<greater<int>()(6,4);//true  【greater<int>()产生了一个无名（临时）对象，(6,4)是指定两个参数】
```

+ 以操作数分
  + 一元仿函数
  + 二元仿函数
+ 以功能分
  + 算术运算（Arithmetic）
  + 关系运算（Rational）
  + 逻辑运算（Logical）

#### 7.2、可配接（adaptable）的关键

##### 7.2.1、unary_function

+ 一元必须继承它

##### 7.2.2、binary_function

+ 二元必须继承它

#### 7.3、算术类（Arithmetic）仿函数

+ 内建的“算术类仿函数”
+ P419，7functor-arithmetic.cpp
+ 证同元素（identity element）

#### 7.4、关系运算类（Rational）仿函数

+ 内建的“关系运算类仿函数”
+ P421，7functor-rational.cpp

#### 7.5、逻辑运算类（Logical）仿函数

+ 内建的“逻辑运算类仿函数”
+ P421，7functor-logical.cpp

#### 7.6、证同（identity）、选择（select）、投射（project）

+ *这一节没太懂啥意思，lionel*

### chap8、配接器（adapters）

+ Adapter定义：**将一个class的接口转换为另一个class的接口，使原本因接口不兼容而不能合作的classes，可以一起运作**。

#### 8.1、配接器之概观与分类 

##### 8.1.1、应用于容器，container adapters

+ queue和stack，是配接器，修饰deque

##### 8.1.2、应用于迭代器，iterator adapters

+ *没怎么看会，lionel，*

##### 8.1.3、应用于仿函数，functor adapters

+ *稍微看了一下，没有贯穿会*
+ P431，8functor-adapter.cpp

#### 8.2、container adapters

##### 8.2.1、stack

+ `class Sequence = deque<T>`

##### 8.2.2、queue

+ `class Sequence = deque<T>`

#### 8.3、iterator adapters

##### 8.3.1、insert iterators

##### 8.3.2、reverse iterators

##### 8.3.3、stream iterators

#### 8.4、functor adapters

##### 8.4.1、对返回值进行逻辑否定：not1，not2

##### 8.4.2、对参数进行绑定：bind1st，bind2nd

##### 8.4.3、用于函数合成：compose1，compose2

##### 8.4.4、用于函数指针：ptr_fun

##### 8.4.5、用于成员函数指针：mem_fun，mem_fun_ref