## 《深度探索C++对象模型》

### chap1、关于对象

+ 类和结构体的定义
+ 为什么**一个ADT或class hierarchy的数据封装**比**在C程序中程序性地使用全局数据**好？

#### 1.0、加上封装后的布局成本（Layout Costs for Adding Encapsulation）

+ 其实没有啥成本（**数据成员直接在class object中**【跟C一样】，**成员函数只是在声明中，不在class object中**）

+ **C++在布局以及存取时间上主要的额外负担是由`virtual`引起**，包括：
  + virtual function机制：**执行期绑定**
  + virtual base class：**多次出现在继承体系中的base class，有一个单一而被 共享的实体**
  + 以外，还有一些多重继承下的额外负担

#### 1.1、C++对象模式（The C++ Object Model）

+ C++中有两种类数据成员【class data members】：static和nonstatic
+ 三种类成员函数【class member functions】：static，nonstatic和virtual。
+ 代码中的`class Point{};`定义
+ **如何模型出各种data members和function members呢？**

##### 1.1.1、简单对象模型（A Simple Object Model）

+ 大小：**指针大小，乘以class中所声明的members数目**。  【lionel，这不像是本节的内容】
+ **并没有应用到实际产品上**，思路是，**members本身并不放在object之中，只有“指向member的指针”才放在object内**（每个声明的都会有个指针位置，根据图1.1表述，是这个意思）

##### 1.1.2、表格驱动对象模型

+ member抽取出来，放在data member table和member function table之中，**class object本身则内含指向这个两个表格的指针**

+ **没有真正使用**，但**member function table**这个理念用了

##### 1.1.3、C++对象模型

+ Nonstatic data members被配置于每一个class object之内，static data members则被放在所有class object之外。static和nonstatic function members也被放在所有class object之外。**virtual functions则以两个步骤支持之**：
  + 每一个class产生出一堆指向virtual functions的指针，放在表格之中，称为**vtbl**（virtual table）
  + 每一个class object都添加了一个指针，指向相关的virtual table。通常每个指针称为**vptr**

+ *lionel，没太看呢*

###### 1.1.3.1、加上继承（Adding Inheritance）

##### 1.1.4、对象模型如何影响程序（How the Object Model Effects Programs）

+ *本节内容没有看呢，lionel*

#### 1.2、关键词所带来的差异（A Keyword Distinction）

##### 1.2.1、关键词的困扰

##### 1.2.2、策略性正确的struct（The Politically Correct Struct）

+ *这2节内容，都没怎么看呢，lionel*

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


##### 1.3.1、指针的类型（The Type of a Pointer）

+ **转型（cast）其实是一种编译器指令**。大部分情况下它并不改变一个指针所含的真正地址，它只影响“被指出之内存的大小和其内容”的解释方式。

##### 1.3.2、加上多态之后（Adding Polymorphism）

+ C++也支持具体的ADT程序风格，称为object-based（OB），一个OB设计可能比一个对等的OD设计速度更快而且空间更紧凑。**OB设计比较没有弹性**。
  + 速度快：所有的函数引发操作都在编译时期解析完成，对象建构起来时不需要设置virtual机制
  + 空间紧凑：因为每一个class object不需要负担传统上为了支持virtual机制而需要的额外负荷

### chap02、构造函数语义学

#### 2.1、Default Constructor的建构操作

##### 2.1.1、“带有Default Constructor“的Member Class Object

##### 2.1.2、“带有Default Constructor“的Base Class

##### 2.1.3、“带有一个Virtual Function“的Class

##### 2.1.4、“带有一个Virtual Base class“的Class

##### 总结

#### 2.2、Copy Constructor的建构操作

##### 2.2.1、Default Memberwise Initialization

##### 2.2.2、Bitwise Copy Semantics（位逐次拷贝）

##### 2.2.3、不要Bitwise Copy Semantics!

##### 2.2.4、重新设定的指针Virtual Table

##### 2.2.5、处理Virtual Base Class Subobject

#### 2.3、程序转换语意学（Program Transformation Semantics）

##### 2.3.1、明确的初始化操作（）

##### 2.3.2、参数的初始化（）

##### 2.3.3、返回值的初始化（）

##### 2.3.4、在使用者层面做优化（）

##### 2.3.5、在编译器层面做优化（）

##### 2.3.6、Copy Constructor：要还是不要？

##### 2.3.7、摘要

#### 2.4、成员们的初始化队伍（Member Initialization List）

+ 必须使用Member initialization list：
  + 1、当初始化一个reference member时
  + 2、当初始化一个const member时
  + 3、当调用一个base class的constructor，而它拥有一组参数时
  + 4、当调用一个member class的construcotr，而它拥有一组参数时

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

### chap4、Function语意学

#### 4.1、Member的各种调用方式

+ 最开始C++只支持nonstatic member functions
+ virtual是20世纪80年代中期加进来的
+ static member functions是最后被引入，1987年

##### 4.1.1、Nonstatic Member Functions

+ **至少要和一般的member functions有相同的效率**
+ member function被内化为nonmember的形式，转步步骤：
  + 1、改写函数的signature（函数原型）以安插一个额外的参数到member function中，**用以提供一个存取管道**，使class object得以调用该函数，**该额外参数被称为this指针**
  + 2、将每一个“对nonstatic data member的存取操作”改为经由this指针来存取
  + 3、将member function重新写成一个外部函数，对函数名称进行“mangling”处理，使它在程序中成为独一无二的语汇

###### 4.1.1.1、名称的特殊处理（Name Mangling）

##### 4.1.2、Virtual Member Functions（）

+ 虚成员函数normalize`ptr->normalize()`被会内部转化为`(*ptr->vptr[1])(ptr)`;
  + vptr是个
  + vptr[1]中的1是virtual table slot的索引值，关联到normalize()函数
  + **第2个ptr表示this指针**（*lionel，这个自己不熟*）

##### 4.1.3、Static Member Functions（）

+ **主要特性是它没有this指针**

#### 4.2、Virtual Member Functions（虚拟成员函数）

+ 多态是**“以一个public base class的指针（或reference），寻址出一个derived class object”**的意思
+ 在实现上，可以在每一个多态的class object身上增加两个members:
  + 1、一个字符串或数字，表示class的类型
  + 2、一个指针，指向某表格，表格中带有程序的virtual functions的执行期地址
+ 问题是，如何找到表格中的地址，也分2步：
  + 1、为了找到表格    157（189/358）
  + 2、为了找到函数地址，每一个virtual function被指派一个表格索引值

##### 4.2.1、多重继承下的Virtual Functions

##### 4.2.2、虚拟继承下的Virtual Functions

#### 4.3、函数的效能

#### 4.4、指向Member Functionsr的指针（Pointer-to-Member Functions）

+ 取一个nonstatic data member的地址，**得到的是该member在class布局中的bytes位置（再加1）**，需要绑定到某个class object的地址上，才能够被存取

##### 4.4.1、支持“指向Virtual Member Functions”之指针

##### 4.4.2、在多重继承之下，指向Member Functions的指针

##### 4.4.3、“指向Member Functions之指针”的效率

#### 4.5、Inline Functions

+ 一般而言，处理一个inline函数，有两个阶段：
  + 1、分析函数定义，以决定函数的"intrinsic inline ability"（本质的inline能力）
  + 2、真正的inline函数扩展操作是在调用的那一点上，这会带来参数的求值操作（evaluation）以及临时性对象的管理

##### 4.5.1、形式参数（Formal Argument）

+ **每一个形式参数都会被对应的实际参数取代**

##### 4.5.2、局部变量（Local Variables）

### chap5、构造、解构、拷贝语意学

#### 5.0、

##### 5.0.1、纯虚拟函数的存在（Presence of a Pure Virtual Function）

5.0.2、虚拟规格的存在（）

5.0.3、虚拟规格中const的存在

##### 5.0.4、重新考虑class的声明

#### 5.1、无继承情况下的对象构造

5.1.1、抽象数据类型（）

5.1.2、为继承做准备

5.2、继承体系下的对象构造

##### 5.2.1、虚拟继承（Virtual Inheritance）

##### 5.2.2、初始化语意学（The Semantics of the vptr Initialization）

5.3、对象复制语意学（）

5.4、对象的功能（）

5.5、解构语意学（Semantics of Destruction）

### chap6、执行期语意学

6.1、对象的构造和解构

6.1.1、全局对象

6.1.2、局部静态对象

6.1.3、对象数据

6.1.4、Default Constructors和数组

6.2、new和delete运算符

6.2.1、针对数组的new语意

6.2.2、Placement Operator new的语意

6.3、临时性对象

6.3.1、临时性对象的迷思

### chap7、站在对象模型的类端

7.1、Template

7.2、异常处理（Exception Handing）

7.3、执行期类型识别（，RTTI）

7.4、效率有了，弹性呢？

7.4.1、动态共享函数库

7.4.2、共享内存

### 最后

#### 履历

+ 20230720开始看这本书，全天大概过了一遍，chap2确实不太好懂，看前言建议**chap1、3、4先看**，先过一遍，把脉络架起来，边思考边琢磨