## 《Effective C++》

+ Part06的知识，对于OO确实了解得不多，读了有点吃力（2023-08-15）

### Part1、自己习惯C++（1-4）

+ Accustoming Yourself to C++

#### 1、视C++为一个语言联邦，View C++ as a federation of languages

+ 结论：
  + C++高效编程守则视状况而变化，取决于你使用C++的哪一部分
+ 主要的次语言，4个方面
  + C
  + Object-Oriented C++
  + Template C++
  + STL
+ *不知道C++11，C++20的特性会不会再加大这个区分*

#### 2、尽量以const,enum,inline替换#define，Perfer consts, enums,and inlines to #define

+ 结论：
  + 对于单纯常量，最好以const对象或enums替换#defines
  + 对于形似函数的宏（macros），最好改用inline函数替换#defines
+ 0
  + `#define`时**宏名有可能并未记入符号表**。

#### 3、尽可能使用const，Use const whenever possible

+ 结论：
  + 将某些东西声明为const，可帮助编译器侦测出错误用法。const可被施加于任何作用域内的对象、函数参数、函数返回类型、成员函数本体。
  + 编译器强制实施bitwise constness，但你编写程序时应该使用“概念上的常量性”（conceptual constness）
  + 当const和non-const成员函数有着实质等价的实现时，令non-const版本调用const版本可避免代码重复
+ 0
  + const在*号左边，指针指向的值是常量 `const int * p;`
  + const在*号右边，指针自身常量。`int * const p;`
+ **const在`*`号左边，物是常量；const 在`*`号右边，指针是常量**
  + `const char* p = greeting;//const data`
  + `char* const p = greeting;//const pointer`
+ 迭代器中也有这样的区分
  + `::const_iterator` **迭代器所指的东西不可被改动**。
+ const成员函数
  + 是不是只能const成员函数，调用const对象？
  + const成员函数和非const成员函数，如何重载？
+ **显式加上`mutable`关键字**

##### 3.1、const成员函数

+ `void add(int x, int y) const`**const在末尾**

##### 3.2、const和non-const成员函数中避免重复

#### 4、确定对象被使用前已被初始化，Make sure that objects are initialized before they're used

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
+ 内置类型的初始化？
+ 自定义类型的初始化？
+ 内置类型的（local,static）的初始化不同。

### Part2、构造、析构、赋值（5-12）

+ Constructors、Destructors、and Assignment Operators

+ 几乎你写的每一个class都会有一或多个构造函数、一个析构函数、一个copy assignment操作符。

#### 5、了解C++默默编写并调用哪些函数，Know what functions C++ silently writes and calls

+ 结论
  + 编译器可以暗自为class创建default构造函数、copy构造函数、copy assignment操作符，以及析构函数。

#### 6、若不想使用编译器自动生成的函数，就该明确拒绝，Explicitly disallow the use of compiler-generated functions you do not want

+ 结论
  + **为了驳回编译器自动（暗自）提供的机能，可将相应的成员函数声明为private并且不予实现**。使用像Uncopyable这样的base class也是一种方法。

#### 7、为多态基类声明virtual析构函数，Declare destructions virtual in polymorphic base classes

+ 结论：
  + polymorphic（带多态性质的）base classes 应该声明一个virtual析构函数。如果class带有任何virtual函数，它就应该拥有一个virtual析构函数。
  + Classes的设计目的如果不是作为base classes使用，或不是为了具备多态性（polymorphically），就不该声明virtual析构函数。

#### 8、别让异常逃离析构函数，Prevent exceptions from leaving destructors

+ 结论：
  + 析构函数绝对不要吐出异常。如果一个被析构函数调用的函数可能抛出异常，析构函数应该捕捉任何异常，然后吞下它们（不传播）或结束程序。
  + 如果客户需要对某个操作函数运行期间抛出的异常做出反应，那么class应该提供一个普通函数（而非在析构函数中）执行该操作。

#### 9、绝不在构造和析构过程中调用virtual函数，Never call virtual functions during construction or destruction

+ 结论：
  + 在构造和析构期间不要调用virtual函数，因为这类调用从不下降至derived class（比起当前执行构造函数和析构函数的那层）。
+ **在derived class对象的base class构造期间，对象的类型是base class而不是derived class**。

#### 10、令operator=返回一个reference *this，Have assignment operators return a reference to *this

+ 结论：
  + 令赋值（assignment）操作符返回一个reference to *this。
+ *`this`*

#### 11、在operator=中处理“自我赋值”，Handle assignment to self in operator=

+ 结论：
  + 确保当对象自我赋值时`operator=`有良好行为。其中技术包括比较“来源对象”和“目标对象”的地址、精心周到的语句顺序、以及copy-and-swap。
  + 确定任何函数如果操作一个以上的对象，而其中多个对象是同一个对象时，其行为仍然正确。
+ `operator=`

#### 12、复制对象时勿忘其每一个成分，Copy all parts of an object

+ 结论：
  + Copying函数应该确保复制“对象内的所有成员变量”及“所有base class成分”。
  + 不要尝试以某个copying函数实现另一个copying函数。应该将共同机能放进第三个函数中，并由两个copying函数共同调用。
+ copy函数分为**copy构造函数、copy assignment操作符**

### Part3、资源管理（13-17）

#### 13、以对象管理资源，Use objects to manage resources

+ 结论：
  + 为防止资源泄漏，请使用RAII对象。它们在构造函数中获得资源并在析构函数中释放资源
  + 两个常被使用的RAII classes分别是tr1::shared_ptr和auto_ptr。【**后一个废弃了**】前者通常是较佳选择，因为其copy行为比较直观。若选择auto_ptr，复制动作会使它（被复制物）指向null

#### 14、在资源管理类中小心copying行为，Think carefully about copying behavior in resource-managing classes

+ 结论：
  + 复制RAII对象必须一并复制它所管理的资源，所以资源的copying行为决定RAII对象的copying行为。
  + 普遍而常见的RAII class copying行为是：抑制copying、施行引用计数法（reference counting）。不过其他行为也都可以被实现。

#### 15、在资源管理类中提供对原始资源的访问，Provide access to raw resources in resouce-manaing classes

+ 结论：
  + APIs往往要求访问原始资源（raw resources），所以每一个RAII class应该提供一个“取得其所管理之资源”的办法
  + 对原始资源的访问可能经由显式转换或隐式转换。一般而言显式转换比较安全，但隐式转换对客户比较方便。

#### 16、成对使用new和delete时要采用相同形式，Use the same form in corresponding uses of new and delete

+ 结论
  + 如果你在new表达式中使用[]，必须在相应的delete表达式中也使用[]。如果你在new表达式中不使用[]，一定不要在相应的delete表达式中使用[]
+ new和delete配对

#### 17、以独立语句将newed对象置入智能指针，Store newed objects in smart pointers in standalone statements

+ 结论
  + 以独立语句将newed对象存储于（置入）智能指针内。如果不这样做，一旦异常被抛出，有可能导致难以觉察的资源泄漏
+ newed对象
+ `std::tr1::shared_ptr<Widget> pw(new Widget); //在单独语句内以智能指针存储newed所得对象`

### Part4、设计与声明（18-25）

#### 18、让接口容易被正确使用，不易被误用，Make interfaces easy to use correctly and hard to use incorrectly

+ 结论：
  + 好的接口很容易被正确使用，不容易被误用。你应该在你的所有接口中努力达成这些性质
  + “促进正确使用”的办法包括接口的一致性，以及与内置类型的行为兼容
  + “阻止误用”的办法包括建立新类型、限制类型上的操作，束缚对象值，以及消除客户的资源管理责任
  + tr1::shared_ptr支持定制型删除器（custom deleter）。这可防范DLL问题，可被用来自动解除互斥锁（mutexs）等等
  
+ 首先**必须考虑客户可能做出什么样的错误**

  + `class Date {public: Date(int month, int day, int year)};`，可能会有2个错误：

    + 1、错误的次序传递参数，把月和日搞混了
    + 2、传递一个无效值，13月，40日

    ```cpp
    struct Day{};
    struct Month{};
    struct Year{};
    class Date{};
    //会不会是过度设计？
    ```

  + 用enums表现月份，**enums不具备我们希望拥有的安全性**

  + 如果以**函数替换对象，表达某个特定月份**，忘记了non-local static对象的初始化次序有可能出问题，*这个我没太懂，lionel*

+ 其次，**限制类型内什么事可做，什么事不能做**，常见的就是加上const
  
+ 除非有好的理由，否则应该尽量令你的types的行为与内置的types一致，避免无端与内置类型不兼容，**真正的理由是为了提供行为一致的接口**。
  
+ *里面提及的item13，我目前还不太懂啥意思*



+ *lionel，这一段，不是本item的啊*
  +  C++中的`explicit`关键字只能用于修饰只有一个参数的类构造函数, 它的作用是表明该构造函数是显示的, 而非隐式的, 跟它相对应的另一个关键字是implicit, 意思是隐藏的,类构造函数默认情况下即声明为implicit(隐式).
    + **explicit关键字的作用就是防止类构造函数的隐式自动转换.**

#### 19、设计class犹如设计type

+ 结论：
  + Class的设计就是type的设计。在定义一个新type之前，请确定你已经考虑过本条款覆盖的所有讨论主题。
+ 设计新type的checklist
  + 1、新type的对象应该如何被创建和销毁？
  + 2、对象的初始化和对象的赋值该有什么样的差别？
  + 3、新type的对象如果被passed by value，意味着什么？
  + 4、你的新type需要配合某个继承图系（inheritance graph）吗？
  + 5、你的新type需要什么样的转换？
  + 6、什么样的操作符和函数对此新type而言是合理的？
  + 7、什么样的标准函数应该驳回？
  + 8、谁该取用新type的成员？
  + 9、什么是新type的“未声明接口”（undeclared interface）
  + 10、你的新type有多么一般化？

#### 20、宁以pass-by-reference-to-const替换pass-by-value

+ 结论：
  + 尽量以pass-by-reference-to-const替换pass-by-value。前者通常比较高效，并可避免切割（slicing problem）问题
  + 以上规则并不适用于内置类型，以及STL的迭代器和函数对象。对它们而言，pass-by-value往往比较适当
+ 以by-value方式传递一个Student对象，总体成本是“六次构造函数和六次析构函数”

```cpp
class Person{};
class Student:public Person{};

bool validateStudent(Student s);//by value
Student plato; //柏拉图
bool platoIsOK = validateStudent(s); //函数调用
```

+ 以by reference方式传递参数也可以避免slicing（对象切割）问题

```cpp
class Window{string name() const;  virtual void display() const;};
class WindowWithScrollBars:public Window{ virtual void display() const;};
void printNameAndDisplay(Window w){  //不正确！参数可能被切割
    cout<<w.name;
    w.display();
}
```



+ 如果窥视C++编译器的底层，**references往往以指针实现出来，因为pass-by-reference通常意味真正传递的是指针**
  + **内置类型、STL的迭代器和函数对象，最好passed by value**，理论上**小型types都可以**【但现在小，不代表将来也小，所以谨慎使用】

#### 21、必须返回对象时，别妄想返回其reference，Don't try to return a reference when you must return an object

+ 结论：
  + 绝不要返回pointer或reference指向一个local stack对象，或返回reference指向一个heap-allocated对象，或返回pointer或reference指向一个local static对象而有可能同时需要多个这样的对象。item4已经为“在单线程环境中合理返回reference指向一个local static对象”提供了一份设计实例。
+ 0

```cpp
class Rational{
    public: Rational(int numerator=0, int denominator=1);
private: 
    int n,d; //n是分子（numerator），m是分母
    friend const Rational operator*(const Rational &lhs, const Rational &rhs);
}
```



+ 函数创建新对象的途径有二：**stack空间或heap空间**
  + local的就是stack空间创建的

```cpp
const Rational& operator*(const Rational &lhs, const Rational &rhs){
    Rational result(lhs.n * rhs.n, lhs.d * rhs.d); //警告，糟糕的代码
    return result;
}

const Rational& operator*(const Rational &lhs, const Rational &rhs){
    Rational *result = new Result(lhs.n * rhs.n, lhs.d * rhs.d); //警告，更糟的写法
    return *result;
}
```



+ **一个“必须返回新对象”的函数的正确写法：**就让那个函数返回一个新对象呗

```cpp
inline const Rational operator*(const Rational &lhs, const Rational &rhs){
    return Rational(lhs.n * rhs.n, lhs.d * rhs.d); //需要承受Operation*返回值的构造成本和析构成本
}
```



#### 22、将成员变量声明为private，Declare data members private

+ 结论：
  + 切记将成员变量声明为private。这可赋予客户访问数据的一致性、可细微划分访问控制、允诺约束条件获得保证，并提供class作者以充分的实现弹性。
  + protected并不比public更具封装性。
+ *lionel，第一个想说明啥*，如果写乱了，相当于为每个成员变量提供了getter和setter方法
+ **封装性**
  + 如果把原先成员变量的public取消了，**所有使用它的客户代码都会被破坏**
  + 如果把原先成员变量的protected取消了，**所有使用它的derived classees都会被破坏**

#### 23、宁以non-member,non-friend替换member函数，Perfer non-member non-friend functions to member functions

+ 结论：
  + 宁可拿non-member、non-friend函数替换member函数。这样做可以增加封装性、包裹弹性（packaging flexibility）和机能扩充性
+ 问题1：是member函数clearEverything 还是 non-member函数clearBrowser？ **clearBrowser好，让WebBrowser类有较大的封装性**

```cpp
class WebBrowser{
public:
    void clearCache();
    void clearHistory();
    void removeCookies();
    void clearEverything(); //调用上面3个
}

void clearBrowser(WebBrowser &wb){
    wb.clearCache();
    wb.clearHistory();
    wb.removeCookies();
}
```

+ 有2点需要注意的是：
  + 1、这个论述只适用于non-member non-friend函数
  + 2、只因在意封装性而让函数“成为class的non-member”，并不意味它“不可以是另一个class的member”
+ *lionel，这面2页就暂时没看了*

+ 
  + namespace和classes不同，前者可跨越多个源码文件而后者不能
  + 将所有便利函数放在多个头文件内但隶属同一个命名空间，意味客户可以轻松扩展这一组便利函数。

#### 24、若所有参数皆需类型转换，请为此采用non-member函数，Declare non-member functions when type conversions should apply to all parameters

+ 结论：
  + **本条款内含真理，但却不是全部的真理。**
  + 如果你需要为某个函数的所有参数（包括被this指针所指的那个隐喻参数）进行类型转换，那么这个函数必须是个non-member
+ 0

```cpp
class Rational{
public:
    Rational(int numerator = 0, int denominator = 1); //构造函数刻意不为explicit; 允许int-to-Rational隐式转换
    int numerator() const;
    int denominator() const; //分子(numerator)和分母（denominator）的访问函数（accessors）
    
    //后续加上，将operator*写成Rational成员函数的写法
    const Rational operator*(const Rational& rhs) const;
}
```



+ 
  + 令classes支持隐式类型转换通常是个糟糕的主意
  + 让`operator*`成为一个non-member函数
  + 无论何时如果你可以避免friend函数就该避免，因为就像真实世界一样，**朋友带来的麻烦往往多过其价值**。
+ 类型转换，采用non-member函数

#### 25、考虑写出一个不抛异常的swap函数，Consider support for a non-throwing swap

+ 结论：
  + 当std::swap对你的类型效率不高时，提供一个swap成员函数，并确定这个函数不抛出异常
  + 如果你提供一个member swap，也该提供一个non-member swap用来调用前者。对于classes（而非templates），也请特化std::swap
  + 调用swap时应针对std::swap使用using声明式，然后调用swap并且不带任何“命名空间资格修饰”
  + 为“用户定义类型”进行std templates全特化是好的，但千万不要尝试在std内加入某些对std而言全新的东西
+ "以指针指向一个对象，内含真正数据"那种类型。这种设计的常见表现形式是所谓**pimpl手法**（pointer to implementation）

```cpp
class WidgetImp{//针对Widget数据而设计的class;细节不重要
private:
    int a,b,c;
    vector<double>v; //可能有许多数据，意味复制时间很长
}

class Widget{  //这个class使用pimpl手法
public:
    Widget(const Widget& rhs);
    Widget& operator=(const Widget& rhs){  //复制Widget时，令它复制其WidgetImpl对象
        *pImpl = *(rhs.pImpl);
    }
private:
    WidgetImpl* pImpl;   //指针，所指对象内含Widget数据
}
```



+ *后面没怎么看呢，lionel*
+ 

### Part5、实现Implementation（26-31）

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

### Part6、继承与面向对象设计Inheritance and Object-Oriented Design（32-40）

+ 继承可以是单一继承或多重继承
  + 每一个继承连接（link）可以是public，protected或private，也可以是virtual或non-virtual
  + 成员函数的各个选项：virtual? non-virtual? pure virtual?
+ 缺省参数值与virtual函数有什么交互影响？
+ 继承如何影响C++的名称查找规则？设计选项有哪些？如果class的行为需要改变，virtual函数是最佳选择吗？

#### 32、确定你的public继承塑模出is-a关系，Make sure public inheritance models "is-a."

+ 结论：
  + public继承意味"is a"。适用于base classes身上的每一件事情一定也适用于derived classes身上，因为每一个derived class对象也都是一个base class对象。
+ **public inheritance（公开继承）意味“is-a”（是一种）的关系**。
  + 子类的对象，同时也是父类的对象（public继承的话）**即，父类更一般化** 【Liskov Substitution Principle】
  + [C++入门到精通：面向对象程序设计中的继承与派生！](https://zhuanlan.zhihu.com/p/344504759)，*lionel，是因为public继承后，父类有操作的各种权限？*
+ **public继承主张，能够施行于base class对象身上的每件事情（注意，是每件事情），也可以施行于derived class对象身上**。
+ 类之间的关系
  + is-a
  + has-a（有一个）
  + is-implemented-in-terms-of（根据某物实现出）

+ *private继承是啥？*
+ *本节的问题在于，我没有搞懂，为什么会是这么一种状况？*
#### 33、避免遮掩继承而来的名称，Avoid hiding inherited names

+ 结论：
  + derived classes内的名称会遮掩base classes内的名称。在public继承下从来没有人希望如此
  + 为了让被遮掩的名称再见天日，可以使用using声明式或**inline转交函数（forwarding function）**
+ **内层作用域会遮掩外围作用域的名称**（name-hiding rule），**遮掩名称**

```cpp
class Base01{
public:
    virtual void mf1() = 0;
    virtual void mf2();
    void mf3();
private:
    int x;
};

class Derived01:public Base01{
public:
    virtual void mf1();
    void mf4();
}

//03的情况
class Base03{
public:
    virtual void mf1() = 0;
    virtual void mf1(int);
    virtual void mf2();
    void mf3();
    void mf3(double);
private:
    int x;
};

class Derived03:public Base03{
public:
    using Base03::mf1;
    using Base03::mf3; //让Base03 class内名为mf1和mf3的所有东西在Derived作用域内都可见
    virtual void mf1();
    void mf3();
    void mf4();
}
```

+ *lionel，这个其实没有完全整理*

#### 34、区分接口继承和实现继承，Differentiate between inheritance of interface and inheritance of implementation

+ 结论：
  + 接口继承与实现继承不同。在public继承之下，derived classes总是继承base class的接口
  + pure virtual函数只具体指定接口继承
  + 简朴的（非纯）inpure virtual函数具体指定接口继承及缺省实现继承
  + non-virtual函数具体指定接口继承以及强制性实现继承



+ 声明一个pure virtual函数的目的是为了让derived classes只继承函数接口。
+ 声明简朴的（非纯）inpure virtual函数的目的，是让derived classes继承该函数的接口和缺省实现
#### 35、考虑virtual函数以外的其他选择

+ 结论：
  + virtual函数的替代方案包括NVI手法及Straategy设计模式的多种形式。NVI手法自身是一个特殊形式的Template Method设计模式
  + 将机能从成员函数移到class外部函数，带来的一个缺点是，非成员函数无法访问classr non-public成员
  + tr1::function对象的行为就像一般函数指针。这样的对象可接纳“与给定之目标签名式（target signature）兼容”的所有可调用物（callable entities）



+ 
+ Strategy模式
+ 摘要，有以下几个替换
	+ 使用non-virtual interface手法
	+ 将virtual函数替换为“函数指针成员变量”
	+ 以tr1::function成员变量替换virtual函数
	+ 将继承体系内的virtual函数替换为另一个继承体系内的virtual函数
#### 36、绝不重新定义继承而来的non-virtual函数，Never redefine an inherited non-virtual function

+ 结论
  + 绝对不要重新定义继承来的non-virtual

+ *写个代码来看下差异，lionel* 【重新定义继承来的non-virtual】
#### 37、绝不重新定义继承而来的缺省参数值，Never redefine a function's inherited default parameter value

+ 结论：
  + 绝对不要重新定义一个继承而来的缺省参数值，因为**缺省参数值都是静态绑定**，而virtual函数--你唯一应该覆写的东西--却是动态绑定

#### 38、通过复合塑模出has-a或“根据某物实现出”，Model "has-a"or" is-implemented-in-terms-of" through composition

+ 结论：
  + 复合（composition）的意义和public继承完全不同
  + 在应用域（application domain），复合意味has-a（有一个）。在实现域（implementation domain），复合意味 is-implemented-in-terms-of（根据某物实现出）

#### 39、明智而审慎地使用private继承，Use private inheritance judiciously

+ 结论：
  + Private继承意味is-implemented-in-terms of（根据实物实现出）。它通常比复合（composition）的级别低。但是当derived classs需要访问protected base class的成员，或需要重新定义继承而来的virtual函数时，这么设计是合理的。
  + 和复合（composition）不同，private继承可以造成empty base最优化。这对致力于“对象尺寸最小化”的程序库开发者而言，可能很重要

#### 40、明智而审慎地使用多重继承，Use multiple inheritance judiciously

+ 结论：
  + 多重继承比单一继承复杂。它可能导致新的歧义性，以及对virtual继承的需要
  + virtual继承会增加大小、速度、初始化（及赋值）复杂度等等成本。如果virtual base classes不带任何数据，将是最具实用价值的情况
  + 多重继承的确有正当用途。其中一个情节涉及“public继承某个Interface class”和“private继承某个协助实现的class”的两相组合

### Part7、模板与泛型编程（41-48）

#### 41、了解隐式接口和编译期多态

+ 结论
  + classes和templates都支持接口（interfaces）和多态（polymorphism）
  + 对于classes而言接口是显式的（explicit），以函数签名为中心。多态则是通过virtual函数发生于运行期
  + 对templates参数而言，接口是隐式的（implicit），奠基于有效表达式。多态则是通过template具现化和函数重载解析（function overloading resolution）发生于编译期

#### 42、了解typename的双重意义

+ 结论
  + 声明template参数时，前缀关键字class和typename可互换
  + 请使用关键字typename标识嵌套从属类型名称；但不得在base class lists（基类列）或member initialization list（成员初值列）内以它作为base class修饰符

#### 43、学习处理模板化基类内的名称

+ 结论
  + 可在derived class templates内通过"this->"指涉base class templates内的成员名称，或藉由一个明白写出的"base class资格修饰符"完成

#### 44、将与参数无关的代码抽象templates

+ 结论
  + Templates生成多个classes和多个函数，所以任何template代码都不该与某个造成膨胀的template参数产生相依关系
  + 因非类型模板参数（non-type template parameters）而造成的代码膨胀，往往可消除，做法是以函数参数或class成员变量替换template参数
  + 因类型参数（type parameters）而造成的代码膨胀，往往可降低，做法是让带有完全相同二进制表述（binary representations）的具现类型（instantiation types）共享实现码

#### 45、运用成员函数模板接受所有兼容类型

+ 结论
  + 请使用member function tmeplates（成员函数模板）生成“可接受所有兼容类型”的函数
  + 如果你声明member templates用于“泛化copy构造”或“泛化assignment操作”，你还是需要声明正常的copy构造函数和copy assignment操作符

#### 46、需要类型转换时请为模板定义非成员函数

+ 结论
  + 当我们编写一个class template，而它所提供之“与此template相关的”函数支持“所有参数之隐式类型转换”时，请将那些函数定义为“class template内部的friend函数”

#### 47、请使用traits classes表现类型信息

+ 结论
  + Traits classes使得“类型相关信息”在编译期可用。它们以templates和"templates特化"完成实现
  + 整合重载技术（overloading）后，traits classes有可能在编译期对类型执行if...else测试

#### 48、认识template元编程

+ Template metaprogramming（TMP，模板元编程）可将工作由运行期移往编译期，因而得以实现早期错误侦测和更高的执行效率
+ TMP可被用来生成“基于政策选择组合”（based on combinations of policy choices）的客户定制代码，也可用来避免生成对某些特殊类型并不适合的代码

### Part8、定制new和delete（49-52）

0、

+ ---定制new和delete--  【new失败了】

#### 49、了解new-handler的行为

+ `operator new`分配失败了，会**调用`new-handler`函数**，设计一个良好的new-handler函数
+ `new(std::nothrow) Widget`发生了两件事
	+ 1、nothrow版的operator new被调用，用以分配足够内存给Widget对象。
+ 使用nothrow new只能保证operator new不抛掷异常，不保证像“new (std::nothrow) Widget”这样的表达式绝不导致异常。

#### 50、了解new和delete的合理替换时机
+ 自己实现一个`operator new`，看看哪个开源比较强？
+ *什么场景下，需要自己实现一个operator new？*

#### 51、编写new和delete时需固守常规

#### 52、写了placement new也要写placement delete
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

#### 54、让自己熟悉包括TR1在内的标准程序库

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

#### 55、让自己熟悉boost

+ 结论：
  + Boost提供许多TR1组件实现品，以及其它许多程序库

## 收获

### 过程

+ 2022-11-24（那周开始）：要把Part1、2、3、9这几个部分都整理完，当天基本都过了一遍，至少懂了80%以上，*代码要再编编，调试一下*。
+ Part8部分，可以听下b站候捷内存管理的视频，`placement new`

https://github.com/wolflion/Coding-Reading/blob/master/99%E6%A0%B8%E5%BF%83%E8%AE%A1%E7%AE%97%E6%9C%BA%E4%B9%A6/EffectiveC%2B%2B/EffectiveC%2B%2B%EF%BC%88%E8%87%AA%E5%B7%B1%E6%95%B4%E7%90%86%EF%BC%89.md

+ 2023-08-13的那一周，把Part4的8个item看了一下书，花了2个cubi，读懂70%了吧，**后面复习，复习后，再来看看这部分**

### 参考

+ [Effective C++ 条款总结](https://www.cnblogs.com/deepllz/p/9171908.html)