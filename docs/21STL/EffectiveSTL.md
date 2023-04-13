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

#### 19、

+ find的相等是`operator==`，std::insert是**等价**，以`operator<`为基础

+ **等价关系是以“在已排序的区间中对象值的相对顺序”为基础的**。*不区分大小写？*

+ *没有完全看懂啊，lionel*
#### 20、

+ `set<string *> ssp;`是`set<string *, less<string*>> ssp;`，最精确的说，是`set<string *, less<string*>, allocator<string*>> ssp;`的缩写。
	+ **自己编写比较函数子类**
#### 21、
#### 22、
	
+ set和multiset时，Key的值是`const`的
#### 23、

+ 如果只是单纯想查找用**hash**，**关联容器是平衡二叉树**
#### 24、

+ map中的`operator[]`有问题啊
#### 25、

+ hash_set（标准库，没有啊）

### Part4、迭代器（26-29）

0、
+ 为啥需要4种迭代器？iterator、const_iterator、reverse_iterator、const_reverse_iterator，*是不是废弃掉啦？*

#### 26、
+ iterator优先使用
#### 27、
+ 把const_iterator转换成iterator，*为啥不要用const呢？*
#### 28、
#### 29、
+ `istream_iterator`使用`operator>>`函数来完成实际的读操作，而默认情况下`operator>>`函数会跳过空白字符。

### Part5、算法

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

### 备注

+ 前置知识《CPP Primer》5th的chap9顺序容器。
+ 2021-11-25花了50min整理形成了第1版。

