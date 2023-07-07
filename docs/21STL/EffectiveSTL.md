## 《Effective STL》

### Part1、容器

#### 1、慎重选择容器类型

+ 标准STL序列容器：vector、string、deque、list
+ 标准STL关联容器：set、multiset、map、multimap

#### 2、不要试图编写独立于容器类型的代码

#### 3、确保容器中的对象拷贝正确而高效

+ 存入容器的是**你所指定的对象的拷贝**。

#### 4、调用empty而不是检查size()是否为0

+ **empty对所有的标准容器都是常数时间操作**。

#### 5、区间成员函数优先于与之对应的单元素成员函数

+ **区间成员函数**（range member function）：它们像STL算法一样，使用两个迭代器来确定该成员操作所执行的区间。

#### 6、当心C++编译器最烦人的分析机制

#### 7、如果容器中包含了通过new操作创建的指针，切记在容器对象析构前将指针delete掉

#### 8、切勿创建包含auto_ptr的容器对象

+ **C++11已经废弃了这个关键字`auto_ptr`**。

#### 9、慎重选择删除元素的方法

+ 连续内存用`c.erase(remove(c.begin(),c.end(),1963),c.end());`
+ 关联容器用`c.erase(1963);`

#### 10、了解分配子（allocator）的约定和限制

#### 11、理解自定义分配子的合理用法

+ `allocator<T>`

#### 12、切勿对STL容器的线程安全性有不切实际的依赖

### Part2、vector和string

#### 13、vector和string优先于动态分配的数组

+ 使用了`new`的后遗症
  + 确保`delete`；
  + 确保用对了`delete`，`delete[]`形式
  + 确保只`delete`了一次；
+ `string`的实现使了**引用计数**技术，*这可能对于多线程带来性能问题*；`vector`的实现没有用。

#### 14、使用reverse来避免不必要的重新分配

+ 相对于`array`而言，是**可变增长**，realloc操作分为4部分：
  + 分配一个当前容量某个倍数的新内存。（一般是2）
  + 元素拷贝到新内存中
  + 析构旧内存的对象
  + 释放旧内存
+ 理解以下4个成员函数：
  + `size()`：容器里有多少个元素
  + `capacity()`：容器有多大
  + `resize(n)`：强迫到n的大小（不够的话，自己申请；多的话，就析构掉部分）
  + `reserve(n)`：强迫到n的大小（**n不会小于当前的大小**）
+ 体会一下这两段代码的区别：

```cpp
vector<int> v;
for(int i=0;i<=1000;++i)  v.push_back(i);  // 1000是2的10次方

// reserve
vector<int> v1;
v1.reserve(1000);
for(int i=0;i<=1000;++i)  v.push_back(i);
```

#### 15、注意string实现的多样性

+ *不同的STL库，对于string的具体实现有差异*。

#### 16、了解如何把vector和string数据传给旧的API

+ `char*`与`string`的互换：`s.c_str()`
+ `& *v.begin()`才与`&v[0]`等价。**`v.begin()`返回值是一个迭代器，不是指针**。
+ **练习题**：
  + 用C API中的元素初始化一个`vector`和`string`

#### 17、使用“swap技巧”除去多余的容量

+ 本例中的代码：

```cpp
string s;
// ... // 让s变大，然后再删除它的大部分字符
string(s).swap(s);  //对s做shrink-to-fit

string s1;
// ... // 使用s1
string().swap(s1);//清除s并把它的容量变为最小
```

+ C11新特性支持`shrink_to_fit()`：**将capacity()减少为与size()相同大小**。

#### 18、避免使用 `vector<bool>`

+ `vector<bool>`有两点不对
  + 1、它不是一个STL容器（并不支持`operator[]`）
  + 2、它不存储`bool`
+ **这个是尝试失败的试验品**，建议用`bitset`

### Part3、关联容器（19-25）

+ 有哪些特性：（自动排序，不允许有重复数据）
+ *有序列容器比，关联容器，一般是key,value关联起来组成的，个人看的一些理解，用来记忆一下，lionel，没找到官方的*

#### 19、理解相等（equality）和等价（equivalence）的区别

+ find的相等是`operator==`，std::insert是**等价**，以`operator<`为基础

+ **等价关系是以“在已排序的区间中对象值的相对顺序”为基础的**。*不区分大小写？*

+ *没有完全看懂啊，lionel*
#### 20、为包含指针的关联容器指定比较类型

+ `set<string *> ssp;`是`set<string *, less<string*>> ssp;`，最精确的说，是`set<string *, less<string*>, allocator<string*>> ssp;`的缩写。
	+ **自己编写比较函数子类**
#### 21、总是让比较函数在等值情况下返回false
#### 22、切勿直接修改set或multiset中的键

+ set和multiset时，Key的值是`const`的
#### 23、考虑用排序的vector替代关联容器

+ 如果只是单纯想查找用**hash**，**关联容器是平衡二叉树**
#### 24、当效率至关重要时，请在map::operator[]与map::insert之间谨慎做出选择

+ map中的`operator[]`有问题啊
#### 25、熟悉非标准的散列容器

+ hash_set（标准库，没有啊）

### Part4、迭代器（26-29）

0、
+ 为啥需要4种迭代器？iterator、const_iterator、reverse_iterator、const_reverse_iterator，*是不是废弃掉啦？*

#### 26、iterator优先于const_iterator、reverse_iterator及const_reverse_iterator
+ iterator优先使用
#### 27、使用distance和advance将容器的const_iterator转换成iterator
+ 把const_iterator转换成iterator，*为啥不要用const呢？*
#### 28、正确理解由reverse_iterator的base()成员函数所产生的iterator的用法
#### 29、对于逐个字符的输入请考虑使用istreambuf_iterator
+ `istream_iterator`使用`operator>>`函数来完成实际的读操作，而默认情况下`operator>>`函数会跳过空白字符。

### Part5、算法（30-37）

#### 30、确保目标区间足够大

+ **STL容器会自动扩充存储空间以容纳这些对象**，但并不一定总是能正确管理，*如果user触发得不对，lionel*
+ **插入型迭代器**

#### 31、了解各种与排序有关的选择

+ sort
+ qsort
+ partial_sort
+ nth_element
+ stable_sort
+ partiton
+ *各种不同排序有其适合的场合，要注意，lionel*

#### 32、如果确实需要删除元素，则需要在remove这一类算法之后调用erase

#### 33、对包含指针的容器使用remove这一类算法时要特别小心

#### 34、了解哪些算法要求使用排序的区间作为参数

#### 35、通过mismatch或lexicographical_compare实现简单的忽略大小写的字符串比较

#### 36、理解copy_if算法的正确实现

#### 37、使用accumulate或者for_each进行区间统计

### Part6、函数子、函数子类、函数及其他（38-42）

#### 38、遵循按值传递的原则来设计函数子类

#### 39、确保判别式是“纯函数”

#### 40、若一个类是函数子，则应使它可配接

#### 41、理解ptr_fun、mem_fun和mem_fun_ref的来由

+ *不太会啊*，43用到了mem_fun_ref

#### 42、确保`less<T>`与operator<具有相同的语义

### Part7、在程序中使用STL（43-50）

#### 43、算法调用优先于手写的循环

+ 自己手写for循环，与调用`for_each()`算法

+ 调用算法通常是更好的选择，它往往优先于任何一个手写循环，理由有3个：
  + 效率
  + 正确性
  + 可维护性
+ STL有70个算法名称，**知道（或查找到）每个算法的所做的事情**，*现在是不是更多啦，在哪找？cppreference，lionel*

#### 44、容器的成员函数优先于同名的算法

+ 原因有2个：【算法和成员函数虽然有同样的名称，但它们所做的事情往往不完全相同】
  + 一，成员函数往往速度更快
  + 二，成员函数通常与容器（特别是关联容器）结合得更加紧密

#### 45、正确区分count、find、binary_search、lower_bound、upper_bound和equal_range

#### 46、考虑使用函数对象而不是函数作为STL算法的参数

#### 47、避免产生“直写型”（write-only）的代码

#### 48、总是包含（#include）正确的头文件

#### 49、学会分析与STL相关的编译器诊断信息

#### 50、熟悉与STL相关的Web站点

### 参考书目

### 备注

+ 前置知识《CPP Primer》5th的chap9顺序容器。
+ 2021-11-25花了50min整理形成了第1版。

