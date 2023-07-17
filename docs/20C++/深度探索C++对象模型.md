## 《深度探索C++对象模型》

### chap1、关于对象

#### 0

+ 加上封装后的布局成本（Layout Costs for Adding Encapsulation）
  + 其实没有啥成本
+ **C++在布局以及存取时间上主要的额外负担是由`virtual`引起**，包括：
  + virtual function机制：**执行期绑定**
  + virtual base class：**多次出现在继承体系中的base class，有一个单一而被 共享的实体**

#### 1.1、C++对象模式（The C++ Object Model）

+ C++中有两种类数据成员【class data members】：static和nonstatic，三种类成员函数【class member functions】：static，nonstatic和virtual。
+ 简单对象模型（）
  + 大小：**指针大小，乘以class中所声明的members数目**。
+ 表格驱动对象模型
+ C++对象模型
+ 对象模型如何影响程序（How the Object Model Effects Programs）

#### 1.2、关键词所带来的差异（A Keyword Distinction）

+ 关键词的困扰
+ 策略性正确的struct（The Politically Correct Struct）

#### 1.3、对象的差异（An Object Distinction）

+ C++程序设计模型直接支持三种程序设计典范：
  + 程序模型（procedural model）
  + 抽象数据类型模型（abstract data type model，ADT）
  + 面向对象模型（object-oriented model）
+ C++以下列方法支持多态：
  + 经由一组隐含的转化操作
  + 经由virtual function机制
  + 经由`dynamic_cast`和`typeid`运算符
+ 需要多少内存才能表现一个class object？一般而言要有：
  + 其nonstatic data members的总和大小
  + 加上任何由于alignment（）的需求而填补（padding）上去的空间。
  + 加上为了支持virtual而由内部产生的任何额外负担（overhead）

+ 指针的类型（The Type of a Pointer）
  + **转型（cast）其实是一种编译器指令**。大部分情况下它并不改变一个指针所含的真正地址，它只影响“被指出之内存的大小和其内容”的解释方式。
+ 加上多态之后（Adding Polymorphism）
  + C++也支持具体的ADT程序风格，称为object-based（OB），一个OB设计可能比一个对等的OD设计速度更快而且空间更紧凑。**OB设计比较没有弹性**。
    + 速度快：所有的函数引发操作都在编译时期解析完成，对象建构起来时不需要设置virtual机制
    + 空间紧凑：因为每一个class object不需要负担传统上为了支持virtual机制而需要的额外负荷

### chap02构造函数语义学

#### 2.1、Default Constructor的建构操作

#### 2.2、Copy Constructor的建构操作

#### 2.3、程序转换语意学（Program Transformation Semantics）

#### 2.4、成员们的初始化队伍（Member Initialization List）

### chap03Data语意学

#### 0、

```c++
//求下列各个类的size()
class X {};
class Y : public virtual X {};
class Z : public virtual X {};
class A : public Y, public Z{};
```

+ Y和Z的大小，受到3个因素的影响：
  + 1、语言本身所造成的额外负担（overhead）
  + 2、编译器对于特殊情况所提供的优化处理
  + 3、Alignment的限制



#### 3.1、Data Member的绑定

#### 3.2、Data Member的布局

#### 3.3、Data Member的存取

##### Static Data Members

##### Nonstatic Data Members

#### 3.4、“继承”与Data Member

##### 只要继承不要多态

##### 加上多态

##### 多重继承

##### 虚拟继承

#### 3.5、对象成员的效率

#### 3.6、指向Data Members的指针

##### “指向Members的指针”的效率问题