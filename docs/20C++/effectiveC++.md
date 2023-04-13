## 《Effective C++》

### Part1、自己习惯C++（1-4）

#### 1、视C++为一个语言联邦

+ 结论：
  + C++高效编程守则视状况而变化，取决于你使用C++的哪一部分
+ 主要的次语言，4个方面
  + C
  + Object-Oriented C++
  + Template C++
  + STL

#### 2、尽量以const,enum,inline替换#define

+ 结论：
  + 对于单纯常量，最好以const对象或enums替换#defines
  + 对于形似函数的宏（macros），最好改用inline函数替换#defines
+ 0
  + `#define`时**宏名有可能并未记入符号表**。

#### 3、尽可能使用const

+ 结论：
  + 将某些东西声明为const，可帮助编译器侦测出错误用法。const可被施加于任何作用域内的对象、函数参数、函数返回类型、成员函数本体。
  + 编译器强制实施bitwise constness，但你编写程序时应该使用“概念上的常量性”（conceptual constness）
  + 当const和non-const成员函数有着实质等价的实现时，令non-const版本调用const版本可避免代码重复
+ 0
  + const在*号左边，指针指向的值是常量 `const int * p;`
  + const在*号右边，指针自身常量。`int * const p;`

##### 3.1、const成员函数

+ `void add(int x, int y) const`**const在末尾**

##### 3.2、const和non-const成员函数中避免重复

#### 4、确定对象被使用前已被初始化

+ 结论：
  + 为内置型对象进行手工初始化，因为C++不保证初始化它们
  + 构造函数最好使用成员初值列（member initalization list），而不要在构造函数本体内使用赋值操作（assignment）。初值列列出的成员变量，其排列次序应该和它们在class中的声明次序相同。
  + 为免除“跨编译单元之初始化次序”问题，请以local static对象替换non-local static对象。
+ C++有十分固定的“成员初始化次序”
  + base classes更早于其derived classes的初始化
  + class的成员变量总是以其声明次序被初始化
+ 不同编译单元内定义之non-local static对象
  + static对象：其寿命从被构造出来直到程序结束为止（global对象、定义于namespace作用域内对象、在class内、在函数内、以及在file作用域内被声明为static的对象）
  + local static对象：函数内的static对象（它们对函数而言是local）
  + non-local static对象：其它static对象
  + 编译单元（tranlation unit）是指产出单一目标文件（single object file）的那些源码。*基本上它是单一源码文件加所含入的头文件*
  + *lionel，多个编译单元，互相引用non-local static对象，C++没有规定它们的初始化次序*

### Part2、构造、析构、赋值（5-12）

#### 7、

+ *虚析构函数，声明的价值在哪？*

9、

+ **在derived class对象的base class构造期间，对象的类型是base class而不是derived class**。

10、

+ *`this`*

11、
+ `operator=`

12、

+ copy函数分为**copy构造函数、copy assignment操作符**

### Part3、资源管理（13-17）

#### 16、

+ new和delete配对

#### 17、

+ newed对象
+ `std::tr1::shared_ptr<Widget> pw(new Widget); //在单独语句内以智能指针存储newed所得对象`

### Part4、设计与声明（18-25）

#### 18、让接口容易被正确使用，不易被误用

+ 结论：
  + 好的接口很容易被正确使用，不容易被误用。你应该在你的所有接口中努力达成这些性质
  + “促进正确使用”的办法包括接口的一致性，以及与内置类型的行为兼容
  + “阻止误用”的办法包括建立新类型、限制类型上的操作，束缚对象值，以及消除客户的资源管理责任
  + tr1::shared_ptr支持定制型删除器（custom deleter）。这可防范DLL问题，可被用来自动解除互斥锁（mutexs）等等
+ 0
  + 必须考虑客户可能做出什么样的错误
  +  C++中的`explicit`关键字只能用于修饰只有一个参数的类构造函数, 它的作用是表明该构造函数是显示的, 而非隐式的, 跟它相对应的另一个关键字是implicit, 意思是隐藏的,类构造函数默认情况下即声明为implicit(隐式).
    + **explicit关键字的作用就是防止类构造函数的隐式自动转换.**

#### 19、设计class犹如设计type

+ 结论：
  + Class的设计就是type的设计。在定义一个新type之前，请确定你已经考虑过本条款覆盖的所有讨论主题。
+ 设计新type的checklist
  + 新type的对象应该如何被创建和销毁？

#### 20、宁以pass-by-reference-to-const替换pass-by-value

+ 结论：
  + 尽量以pass-by-reference-to-const替换pass-by-value。前者通常比较高效，并可避免切割（slicing problem）问题
  + 以上规则并不适用于内置类型，以及STL的迭代器和函数对象。对它们而言，pass-by-value往往比较适当
+ 0
  + 以by-value方式传递一个Student对象，总体成本是“六次构造函数和六次析构函数”
  + **references往往以指针实现出来，因为pass-by-reference通常意味真正传递的是指针**

#### 21、必须返回对象时，别妄想返回其reference

+ 结论：
  + 绝不要返回pointer或reference指向一个local stack对象，或返回reference指向一个heap-allocated对象，或返回pointer或reference指向一个local static对象而有可能同时需要多个这样的对象。item4已经为“在单线程环境中合理返回reference指向一个local static对象”提供了一份设计实例。

#### 22、成员变量声明为private

+ 结论：
  + 切记将成员变量声明为private。这可赋予客户访问数据的一致性、可细微划分访问控制、允诺约束条件获得保证，并提供class作者以充分的实现弹性。
  + protected并不比public更具封装性。

#### 23、宁以non-member,non-friend替换member函数

+ 结论：
  + 宁可拿non-member、non-friend函数替换member函数。这样做可以增加封装性、包裹弹性（packaging flexibility）和机能扩充性
+ 0
  + namespace和classes不同，前者可跨越多个源码文件而后者不能
  + 将所有便利函数放在多个头文件内但隶属同一个命名空间，意味客户可以轻松扩展这一组便利函数。

#### 24、若所有参数皆需类型转换，请为此采用non-member函数

+ 结论：
  + **本条款内含真理，但却不是全部的真理。**
  + 如果你需要为某个函数的所有参数（包括被this指针所指的那个隐喻参数）进行类型转换，那么这个函数必须是个non-member
+ 0
  + 令classes支持隐式类型转换通常是个糟糕的主意
  + 让`operator*`成为一个non-member函数
  + 无论何时如果你可以避免friend函数就该避免，因为就像真实世界一样，**朋友带来的麻烦往往多过其价值**。
+ 类型转换，采用non-member函数

#### 25、考虑写出一个不抛异常的swap函数

+ 结论：
  + 当std::swap对你的类型效率不高时，提供一个swap成员函数，并确定这个函数不抛出异常
  + 如果你提供一个member swap，也该提供一个non-member swap用来调用前者。对于classes（而非templates），也请特化std::swap
  + 调用swap时应针对std::swap使用using声明式，然后调用swap并且不带任何“命名空间资格修饰”
  + 为“用户定义类型”进行std templates全特化是好的，但千万不要尝试在std内加入某些对std而言全新的东西
+ 0
  + **pimpl手法**（pointer to implementation）

### Part5、实现（26-31）

#### 26、尽可能延后变量定义式的出现时间

+ 结论：
  + 尽可能延后变量定义式的出现。这样做可增加程序的清晰度并改善程序效率。
+ 0
  + *对于代码规范而言，就是需要用的时候，再定义*
  + *能整理规则了，但不一定能真正的融汇贯通？*
  + 本意是避免**构造（和析构）非必要对象，还可以避免无意义的default构造行为**。

#### 27、尽量少做转型动作

+ 结论：
  + 如果可以，尽量避免转型，特别是在注重效率的代码中避免dynamic_casts。如果有个设计需要转型动作，试着发展无需转型的替代设计
  + 如果转型是必要的，试着将它隐藏于某个函数背后。客户随后可以调用该函数，而不需将转型放进他们自己的代码内
  + 宁可使用C++-style（新式）转型，不要使用旧式转型。前者很容易辨识出来，而且也比较有着分门别类的职掌
+ 旧式转型
  + `(T) expression`
  + `T (expression)`
  + `class Widget {public: explicit Widget(int size); };` **唯一使用旧式转型的时机**
+ 新式转型
  + const_cast
  + dynamic_cast
  + reinterpret_cast
  + static_cast
+ 旧式转换、新式转换（*我没记，但要保证自己已经完全掌握了4种的区别*）
+ *写4个例子，来说明下 转型的使用说明*

#### 28、避免返回handles指向对象内部成分

+ 结论：
  + 避免返回**handles（包括references、指针、迭代器）**指向对象内部。遵守这个条款可增加封装性，帮助const成员函数的行为像个const，并将发生“虚吊号码”（dangling handles）的可能性降至最低
+ **handlers**是指（包括reference、指针、迭代器）
+ *不太懂，lionel*

#### 29、为“异常安全”而努力是值得的

+ 结论：
  + 异常安全函数（Exception-safe functions）即使发生异常也不会泄漏资源或允许任何数据结构败坏。这样的函数区分为三种可能的保证：基本型、强烈型、不抛异常型
  + “强烈保证”往往能够以copy-and-swap实现出来，但“强烈保证”并非对所有函数都可实现或具备现实意义
  + 函数提供的“异常安全保证”通常最高只等于等于其所调用之各个函数的“异常安全保证”中的最弱者
+ 当异常被抛出时，带有异常安全性的函数会：
  + 不泄漏任何资源
  + 不允许数据败坏
+ 异常安全函数（Exception-safe functions）提供以下三个保证之一：
  + 基本承诺：
  + 强烈保证：
  + 不抛掷（nothrow）保证：**承诺绝不抛出异常**
+ **copy and swap策略**：为你打算修改的对象（原件）做出一份副本，然后在那副本身上做一切必要修改。若有任何修改动作抛出异常，原对象仍保持未改变状态。待所有改变都成功后，再将修改过的那个副本和原对象在一个不抛出异常的操作中置换（swap）。
+ pimpl idiom 【这个黑话是啥意思？】
+ “异常安全”不太会

#### 30、透彻了解inlining的里里外外

+ 结论：
  + 将大多数inlining限制在小型、被频繁调用的函数身上。这可使日后的调试过程和二进制升级（binary upgradability）更容易，也可使潜在的代码膨胀问题最小化，使程序的速度提升机会最大化
  + 不要只因为function templates出现在头文件，就将它们声明为inline
+ 0
  + inline函数背后的整体观念是，**将“对此函数的每一个调用”都以函数本体替换之**
+ **inline函数无法随着程序库的升级而升级**
  + f是non-line函数，一旦它有任何修改，客户端只需重新连接就好，远比重新编译的负担少很多
+ **inline函数通常一定被置于头文件内**。（template通常也置于头文件内）
+ *inline函数无法随着程序库的升级而升级。lionel，啥意思？*
+ **大部分调试器面对inline函数都束手无策**（gtest就无法走入这个分支，印象中是这样）

#### 31、文件间的编译依存关系降至最低

+ 结论：
  + 支持“编译依存性最小化”的一般构想是：相依于声明式，不要相依于定义式。基于此构想的两个手段是Handle classes和Interface classes
  + 程序库头文件应该以“完全且仅有声明式”（full and declaration-only forms）的形式存在。这种做法不论是否涉及templates都适用
+ 0
  + null
+ 将Person分割为两个classes，一个只提供接口，另一个负责实现该接口（**implementation class取名为PersonImpl**）
+ **编译依存性最小化的本质：**现实中让头文件尽可能自我满足，万一做不到，则让它与其他文件内的声明式（而非定义式）相依。
  + 如果使用object references或object pointers可以完成任务，就不要使用objects。
  + 如果能够，尽量以class声明式替换class定义式
  + 为声明式和定义式提供不同头文件
+ 文件之间的依赖关系
+ **C++没有把“将接口从实现中分离”。class的定义式不只详细叙述了class接口，还包括十足的实现细目**。
+ 头文件中有任何一个被改变，那么每一个含入Person class的文件就得重新编译，任何使用Person classs的文件也必须重新编译。
+ **编译器必须在编译期间知道对象的大小**。
+ “声明的依存性”替换“定义的依存性”，**现实中让头文件尽可能自我满足，万一做不到，让它与其它文件内的声明式（而非定义式）相依**。
+ *Handle classes与Interface classes区别？*

### Part6、继承与面向对象设计（32-40）

#### 32、

+ Liskov Substitution Principle
+ public继承是"is a"，*private继承是啥？*
#### 33、
+ using声明式，**inline转交函数（forwarding function）**
#### 34、
+ 声明一个pure virtual函数的目的是为了让derived classes只继承函数接口。
+ 声明简朴的（非纯）inpure virtual函数的目的，是让derived classes继承该函数的接口和缺省实现
#### 35、
+ Template Method模式
+ Strategy模式
+ 摘要，有以下几个替换
	+ 使用non-virtual interface手法
	+ 将virtual函数替换为“函数指针成员变量”
	+ 以tr1::function成员变量替换virtual函数
	+ 将继承体系内的virtual函数替换为另一个继承体系内的virtual函数
#### 36、
+ *写个代码来看下差异，lionel* 【重新定义继承来的non-virtual】
#### 37、
+ **缺省参数是静态绑定**，virtual是动态绑定
#### 38、
+ **复合是has-a**
#### 39、
+ **private继承，编译器不会自动将一个derived class对象 转换为一个 base class对象**。
#### 40、
+ 多重继承会导致歧义

### Part7、模板与泛型编程（41-48）

### Part8、定制new和delete（49-52）

0、

+ ---定制new和delete--  【new失败了】

#### 49、

+ `operator new`分配失败了，会**调用`new-handler`函数**，设计一个良好的new-handler函数
+ `new(std::nothrow) Widget`发生了两件事
	+ 1、nothrow版的operator new被调用，用以分配足够内存给Widget对象。
+ 使用nothrow new只能保证operator new不抛掷异常，不保证像“new (std::nothrow) Widget”这样的表达式绝不导致异常。

#### 50、
+ 自己实现一个`operator new`，看看哪个开源比较强？
+ *什么场景下，需要自己实现一个operator new？*

#### 51、

#### 52、
+ `Widget *pw = new Widget;`有2个函数被调用：
	+ 用以分配内存的operator new
	+ Widget的default构造函数
	+ **问题，如果new成功了，构造函数失败了怎么办？无法归还内存了**
		+ 类内作用域
		+ 全员作用域

### Part9、杂项讨论（53-55）

#### 53、不要轻忽编译器的警告

+ 结论：
  + 

#### 54、熟悉TR1在内的标准程序库

+ 结论：
  + C++标准程序库的主要机能由STL、iostream、locals组成。并包含C99标准程序库
  + TR1添加了
+ C++98的标准库
+ TR1中的14个新组件
  + 智能指针：**非环形（acyclic）数据结构**
  + function
  + bind
  + 第1组
    + hash table：非序列容器
    + 正则表达式
    + Tuples
    + array
    + mem_fn：*这个没见过，lionel*
    + reference_wrapper：*这个没见过，lionel*
    + 随机数
    + 数学函数
    + C99兼容扩充
  + 第2组
    + type traits
    + result_of：*这个没见过*

#### 55、熟悉boost

+ 结论：
  + Boost提供许多TR1组件实现品，以及其它许多程序库

## 收获

### 过程

+ 2022-11-24（那周开始）：要把Part1、2、3、9这几个部分都整理完，当天基本都过了一遍，至少懂了80%以上，*代码要再编编，调试一下*。
+ Part8部分，可以听下b站候捷内存管理的视频，`placement new`

https://github.com/wolflion/Coding-Reading/blob/master/99%E6%A0%B8%E5%BF%83%E8%AE%A1%E7%AE%97%E6%9C%BA%E4%B9%A6/EffectiveC%2B%2B/EffectiveC%2B%2B%EF%BC%88%E8%87%AA%E5%B7%B1%E6%95%B4%E7%90%86%EF%BC%89.md



### 参考

+ [Effective C++ 条款总结](https://www.cnblogs.com/deepllz/p/9171908.html)