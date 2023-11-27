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

+ iterator模式，**提供一种方法，使之能够依次巡访某个聚合物（容器）所含的各个元素，而又无需暴露该聚合物的内部表述方式**。
+ *如何设计迭代器*

#### 3.1、迭代器设计思维-STL关键所在

+ 以find为例，代码

#### 3.2、迭代器（iterator）是一种smart pointer

+ 指针最重要的作用是**内容提领（dereference）和成员访问（member access）**
  + **要重载`*`和`->`**
+ auto_ptr虽然已经废弃，但用法差不多`auto_ptr<string> ps(new string("jjhou"));`

#### 3.3、迭代器相应型别（associated types）

+ 只能获得**型别名称**，不能拿来做变量声明之用
  + 解决方法，**利用function template的参数推导（argument deducation）机制**
+ 相应型别有5种

#### 3.4、Traits编程技法-STL源代码门钥

+ *type_name不是太懂*

##### value_type

##### difference_type

##### reference_type

##### pointer_type

##### iterator_type

#### 3.5、std::iterator的保证

#### 3.6、iterator源代码完整重列

#### 3.7、SGI STL的私房菜:`__type_traits`

### chap4、序列式容器（sequence containers）

#### 4.1、容器的概观与分类

### chap5、关联式容器

+ multiset和multimap，**底层机制均以RB-tree（红黑树）**
+ *lionel，其它的C++11引入了吗？*
+ *图5-1的那个描述，是啥，我没懂*
+ 关联式容器每笔数据（每个元素）**都有一个键值（key）和一个实值（value）**，按**键值**，以某种特定规则将这个元素放置于适当位置，没有所谓头尾（**只有最大和最小元素**），内部结构是**一个balanced binary tree（平衡二叉树）**，但有许多类型
  + AVL-tree
  + RB-tree
  + AA-tree

#### 5.1、树的导览

+ *比较不错的思路，用树状图表示来记录相关术语*，图5-2

##### 5.1.1、二叉搜索树

+ 先是二叉树
+ 二叉搜索树
  + **对数时间**的元素插入与访问
  + **节点放置规则**，任何节点的键值一定大于其左子树中的每一个节点的键值，并小于其右子树中的每一个节点的键值。
  + 找最大和最小值是极简单的
  + 删除操作，*虽然看起来复杂，但是不是可以用递归，lionel*
  + 插入操作

##### 5.1.2、平衡二叉搜索树

+ **平衡**，没有任何一个节点过深（深度过大）

##### 5.1.3、AVL tree（Adelson-Velskii-Landis）

##### 5.1.4、单旋转

##### 5.1.5、双旋转

#### 5.2、RB-tree（红黑树）

+ 满足规则
  + 1、每个节点不是红就是黑
  + 2、根节点为黑（**根黑**）
  + 3、如果节点为红，其子节点必须为黑
  + 4、任一节点至NULL（树尾端）的任何路径，所含之黑节点数必须相同

##### 5.2.1、插入节点

##### 5.2.2、一个由上而下的程序

##### 5.2.3、RB-tree的节点设计

##### 5.2.4、RB-tree的迭代器

##### 5.2.5、RB-tree的数据结构

##### 5.2.6、RB-tree的构造与内存管理

##### 5.2.7、RB-tree的元素操作

#### 5.3、set

+ set，所有元素都会根据元素的**键值**自动排序，**set元素的键值就是实值，实值就是键值**，不允许两个元素有相同的键值
+ `set<T>::iterarator`是`const_iterator`

#### 5.4、map

+ map，也是排序，**所有元素都是`pair`**
+ 键值不能改，**实值可以修改**
+ insert()函数
+ subscript（下标）操作符

#### 5.5、multiset

+ 允许**键值重复**，用的是`insert_equal()`而非`insert_unique()`

#### 5.6、multimap

+ 同5.5节

#### 5.7、hashtable

##### 5.7.1、hashtable概述

#### 5.8、hash_set

#### 5.9、hash_map

#### 5.10、hash_multiset

#### 5.11、hash_multimap

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

### 最后

#### 参考

+ https://github.com/SilverMaple/STLSourceCodeNote/tree/master