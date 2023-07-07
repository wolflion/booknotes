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

#### 2.1、容器配置器的标准接口

2.2、具备次配能力（sub-allocation）的SGI空间配置器

2.2.9、重新充填free lists

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

#### 7.2、可配接（adaptable）的关键

##### 7.2.1、unary_function

##### 7.2.2、binary_function

#### 7.3、算术类（）仿函数

#### 7.4、关系运算类（）仿函数

#### 7.5、逻辑运算类（）仿函数

#### 7.6、证同（identity）、选择（select）、投射（project）

### chap8、配接器（adapters）

#### 8.1、配接器之概观与分类 

8.1.1、应用于容器，container adapters

8.1.2、应用于迭代器，iterator adapters

8.1.3、应用于仿函数，functor adapters

#### 8.2、container adapters

#### 8.3、iterator adapters

#### 8.4、functor adapters