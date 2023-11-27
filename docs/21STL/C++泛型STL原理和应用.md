## 《C++泛型STL原理和应用》

+ 书上[代码](https://gitee.com/fewolflion/BookNote/tree/master/01lioneloutput/60BookCode/C++%E6%B3%9B%E5%9E%8BSTL%E5%8E%9F%E7%90%86%E5%92%8C%E5%BA%94%E7%94%A8)
+ chap1、模板
  + 主要是概念类，自己搞清楚就行，理解一下**特化**，*auto与decltype，右值，转移语义，完美转发forward，要配合c++11再看下*
+ chap2、
  + 数据类型表、类型萃取机，不太熟悉，*简单的说，就是把所有类型都定义一遍，这样，类外就可以访问？lionel*
+ chap3、
  + **运算符重载是基础**，衍生了仿函数，lambda表达式，智能指针
+ chap4、模拟STL三大件
  + 容器、迭代器、算法
+ chap5、
  + 遗留：P145，P150，还有本章节中的代码没有输入，hash相关的要再想一想；multi_set在刷题中的用处
+ chap6、
  + P178

### chap1、C++泛型技术基础--模板

+ 20201017

#### 1.1 泛型和模板

##### 1.1.1 泛型的概念

+ 泛型（generic type）：就是一种通用类型。
+ `template <typenamet T> T max( T x, T y) {}`  //这里T是类型**占位符**。
+ 使用了类型占位符的代码称为**“模板“**，根据模板和具体类型生成的代码叫**程序实体**，产生程序实体的过程叫**模板的具现**。

##### 1.1.2 C++模板及其定义

+ **模板**：除了需要声明类型占位符之外，其它内容应与目标实体代码完全相同。 声明一个模板时，需在实体代码前面，加一条说明语句。*前面是template关键字。*
  + 函数模板，1-1.cpp
    + `template<typename T, typename R, typename S>  T max(R x, S y){}`，**早期关键字用class，现在用typename**，调用的话是 `long max<int, double>(x,y);`，*都不确定是不是这么写了*
  + 类模板，1-2.cpp
    + 在声明语句前加 `template<typename T>`  

##### 1.1.3 几点说明和小结

+ `template<typename T>` 早期也写成 `template<class T>`，但不支持这么写
+ C++11和14中，给关键字 **auto** 增加了新的语义，同时增加了 **decltype表达式** 
  + auto关键字是为自动推导数据类型所设 `auto i=100;`
  + 之前auto只对系统的内置数据推荐有效，现在利用*编译器的类型记忆能力*，可以记住自定义的类型。
  + 变量类型难以确定的问题主要出现在函数的返回值上，因此函数的返回值类型的位置常常会出现auto关键字，称为 **auto返回值占位**
  + `auto Multiply(T t, U u)->decltype(t*u) {return t*u;}`
+ 把auto看做数据类型的话，auto其实也可以算一种泛型，不过它无须关键字typename声明。
+ decltype表达式（用在声明和实现时），实现函数返回值类型的自动推导，`auto Multiply(T t V v)->decltype(t*v) {return t*v;}`
+ C++新标准引入了**变量模板**的概念。`pi<T>` 【作者没有介绍更多，可以翻看C++11】

#### 1.2 关于模板参数

+ 预编译期间进行传递并被编译的

#####  1.2.1 模板参数的种类

+ 类型参数
  + 以类模板作为函数实参的例子 `MyTest<Test1<int>>TT;`，**用关键字typename声明的参数**
+ 非类型参数，1-3.cpp
  + `template <typename T，/*类型参数*/ int b/*非类型参数*/> `，**非类型参数可以定义时赋值，int b = 10**
    + 是常量，不能被修改
    + 只能是int、枚举、指针、引用类型
+ 模板定义型参数，1-4.cpp
  + C++也允许以类模板的定义作为类模板参数
    + **需要的目的：** 除了强调这个参数的实参必须为类模板之外，还强调这个类模板所具有的参数个数。

#####  1.2.2 模板形参和实参的结合

+ 函数模板实参的隐式提供
  + `add<int>(45,46);` 直接变成 `add(45,46);`，**隐式实例化**，显式的话可以`add<double>(45,10.3);`返回double类型
+ 指针实参
  + 在C++中，指针被看作一种数据类型。
  + 见具体的代码； 【*作者没有描述，我还没太懂这部分*】
+ *函数模板实参的隐式提供*
+ 修饰字const和&的使用
  + 见具体的代码示例； 【*没见过具体的应用场景*】

#### 1.3 特化模板和模板具现规则

#####  1.3.1 特化(特列化)模板

+ **特化(特列化)模板**：为有特殊算法要求的数据类型另行编写模板
  
  + 通用模板：门票10块
  + 特化模板：学生和老人半价
  
+ 1、函数模板中的特化模板
  
  + 在实体代码前加以下声明 `tmplate<>`
  
  ```cpp
  tmplate<> //只是在原来实体代码前写一个模板声明
  char* mymax(char* t1, char* t2) {
      return (strcmp(t1,t2)<0)?t2:t1;
  }
  ```
  
  
  
  + 1-5.cpp，测试上述mymax函数模板并运行之
  
+ 2、类模板的特化和偏特化
  
  + **偏特化：**只特化参数中的某一个或某几个
  + 1-6.cpp，含有一个偏特化模板
  + 偏特化模板的写法 `template<typename T2> struct Test<int,T2>{}`
  + 全特化模板的写法 `template<> struct Test<int,float>{}`

#####  1.3.2 模板的具现

+ 因为有了以上泛型模块的共存，编译器就需要选择一个生成实体模块代码，就有了规则。
+ 具体的优先顺序
  + 特化模板（函数或类）
  + 偏特化模板（函数或类）
  + 普通模板（函数或类）
  + 系统

#### 1.4 右值引用和模板

+ **转移语义**是C11推出的新概念和新技术。

##### 1.4.1 右值引用

+ 临时对象具有了将资源转移到另一个对象，免去深拷贝的麻烦

+ **右值：** 只能出现在赋值运算符右边。*仅能代表数据；匿名，无固定地址的对象。*
+ **左值：** 既可以出现在赋值运算符左边，又可以出现在赋值运算符右边。*有名字，有固定地址的表达式*，**代表一块存储空间**
+ 右值引用，1-8.cpp
  + C11之前有两种表示方式 `T& 别名 = lvalue;//引用` 和 `const T& 别名 = lvalue;//常引用`
  + C11之后仅定义一种：`const T& 别名 = rvalue;`
    + *强行为右值命名一个变量名，目的就是为延长右值生命期。* 
    + **右值的非常量引用** `T&& 名称=rvalue;`

##### 1.4.2 右值引用的应用1--转移语义

+ **深拷贝**
+ **浅拷贝**

##### 1.4.3 右值引用的应用2--转移函数move()

+ 右值引用有好处后，左值也想利用，move的原型 `T&& move(T& val);`

##### 1.4.4 右值引用的应用3--参数完美转发模板()

+ **参数转发**
+ 1 完美转发问题的提出及解决思路
+ 2 模板参数类型推导规则--引用符折叠规则
+ 3 参数类型正确转发的保证--forward()函数模板
  + `static_cast()`：这个转换只对参数为右值时有用。
  + C++11将`static_cast()`封装成函数模板std::forward，于是`Func(forwad<T>(a));`

### chap2、C++泛型机制的基石--数据类型表

+ 20201017

#### 2.1 类模板的公有数据类型成员

+ 原属模板私有的数据类型向外公开，使其成为一种“全局”类型。

##### 2.1.1 类的数据类型成员

+ **类的数据类型成员** ：在一个类中使用`typedef`定义一个已知数据类型的别名。 同时也称为 嵌入式类型(nested type)。

+ 在C++中，这种在类模板中定义的数据类型也称为nested type，嵌入式，可以给相应访问属性

+ 例2-1 定义了数据类型成员的类模板示例代码

+ 前面要加`typename`关键字，为了与 类静态成员 写法相区分。

+ `typedef long double LDBL;`，**LDBL就是数据类型成员，简称类型成员**。

+ 类模板中定义的数据类型，也称为**nested type（嵌入式类型）**，既然是类中定义的，就有`public`这样的属性。

+ 加一个`typename`以示，与**引用类静态成员区别**

+ **C++规定，凡属于类型的名称前面还必须在类型名前面冠以关键字typename**

  ```cpp
  template <typename T1, typename T2>
  class MyTraits{
  public:
      typedef T1 my_Type1;
  };
  int main(){
      typename MyTraits<int,double>::my_Type1 a=100; //多一个typename，以示区别
      cout<<a<<endl;
      return 0;
  }
  ```

  

##### 2.1.2 再谈typedef

+ 别名的好处
  + 增加可读性；
  + 复杂名称起一个简单别名
  + 编写平台无关的代码；   

#### 2.2 内嵌式数据类型表及数据类型衍生

+ 数据类型衍生：就是`T &`和`T *`
+ 内嵌式数据类型：**放在类前面**

+ **内嵌式数据类型表**：一个类模板会把typedef定义的所有公有数据类型集中形成一个数据类型表，并放在类模板中靠前的位置。这个数据类型表存在于类模板内部。

+ 例2-2 编写程序，演示内嵌数据类型表以及数据类型的外部引用。

+ *不是太特别理解，lionel*

+ ```cpp
  template <typename T>
  class map{
  public:
      typedef T value_type;
      typedef T& reference;
      typedef T* pointer;
  };
  
  map<int>::value_type a=100;
  ```

+ 相当于是`map<int>::`**这种别名的方式**提供了value、reference、pointer的形式。

+ 这种内嵌式数据类型表具有自模板传入类型衍生出其他相关类型的能力，所以也叫**类型萃取机**或**类型榨取机**。

#### 2.3 数据类型表

##### 2.3.1 数据类型表的概念

+ **数据类型表**：一个类模板中，其全部成员都是公有数据类型，那么相对于嵌入式数据类型表，这个类模板就叫做独立数据类型表，简称**数据类型表**。数据类型表是一种**类模板**
+ 例2-3
+ 例2-4

##### 2.3.2 数据类型表的应用

+ *要理解代码才行*
+ 2-5.cpp

```cpp
void T function(typename T::value_type x, typename T::reference y, typename T::pointer z) {}
```




#### 2.4 特化数据类型表

+ 特化数据类型表作用
  + 实现同一业务逻辑不同接口的统一。 
+ 2-6.cpp，*没懂*

```cpp
class Test1{
public:
    char Compute(int x, int y{
        return x;}
};
                 
class Test2{
public:
    double Compute(int x, int y{
        return x;}
};
                   
template<typename Arg1, typename Arg2, typename Ret>
class Test{
public:
    Ret Compute(Arg1 x, Arg2 y{
        return x;}
};//需要提供3个实参

class Test1; //为使用特化模板而定义的类名
class Test2;                
//统一接口
template <typename T>
class TypeTbl{};

//特化1
template<>
class TypeTbl<Test1> {
public:
	typedef char ret_type;
	typedef int par1_type;
	typedef double par2_type;
};

//特化2
template<>
class TypeTbl<Test2> {
public:
	typedef double ret_type;
	typedef double par1_type;
	typedef double par2_type;
};

template<typename T>
class Test {
public:
	typename TypeTbl<T>::ret_type Compute(
		typename TypeTbl<T>::par1_type x,
		typename TypeTbl<T>::par1_type y
	) {
		return x;
	}
};
                
int main(){
    Test<Test1> t1;
    cout<<t1.Compute(62,88.5)<<endl;
    Test<Test1> t2;
    cout<<t2.Compute(2.26,66)<<endl;
    return 0;
}
```



#### 2.5 STL中的Traits表

+ **迭代器是类模板，并在模板中包含内嵌数据类型表**

+ **Traits**实质上是特化数据类型表在STL中的一个具体应用，也是数据类型表，但因为它构思巧妙，称Traits技巧。
  + 应用背景
  + 使用特化模板实现指针的数据类型表
  + 汇总同类类模板的内嵌数据类型表形成统一接口
+ 1、应用背景
  + 迭代器
+ 2、使用特化模板实现指针的数据类型表
  + 2-7.cpp
+ 3、汇总同类类模板的内嵌数据类型表形成统一接口
  + 2-8.cpp

### chap3、STL及其使用的其他C++技术

+ 20201017-20201018
+ STL以**数据类型表**为机制基础，使用**C++模板技术**编写的一套实用模板例程

####  3.1 初识STL

##### 3.1.1、STL是C++标准库中的模板类库

##### 3.1.2、STL应用程序示例

+ 例3-1：编写一个数组，里面存放10个随机产生的整型数，把其中的数据从大到小和从小到大排序，最后输出它们。
  + 对于这个例子，用到了STL的三大组件：*容器、迭代器、通用算法*。

####  3.2 STL常用的C++技术

##### 3.2.1 运算符重载

+ 运算符就是函数，**重载运算符**实质上就是**重写运算符函数**。在C++中，原生运算符函数的格式
  + `返回值类型  operator 运算符号(形参列表){函数体}`，**在运算符前面加上关键字operator**
+ **重载的运算符通常都只对某一类对象有效**，按照OO的设计思想，重载的运算符应该与其适用的数据绑在一起更为合适，有两种做法：
  + 将重载的运算符声明**为类的友元函数**  `class 类名{frined 返回类型 operator 运算符(形参表);};`
  + 将重载运算符直接声明于类中**作为的类的成员函数**

+ 例3-2：定义一个复数类，并以友元函数方式为其重载一个全局加法运算符+，使它可以实现复数的加法运算。
  + 友元函数
+ 例3-3：把例3-2中双目运算符+重载为类的成员函数。
  + **重载于类中运算符在参数的数目上具有如下区别：**
    + 双目运算符的参数列表仅有一个参数，**而且是该双目运算符的第二个参数**
    + 单目运算符因其参数隐含为本对象，故在参数列表中没有显式的参数
+ 例3-4：
  + `operator++()`，前置++，`++i;`
  + `operator++(int x);` 后置++，`i++;`
  + *lionel，这个怎么去记？*
+ 例3-5： 
  + 重载`[]`

##### 3.2.2、函数对象（仿函数）

+ `sort(intVector.begin(), intVector.end(),greater<int>());`
+ 函数指针只代表1个函数，函数对象可以携带其它数据成员以及函数成员
+ [例3-6 函数对象作为函数参数的示例程序](https://gitee.com/fewolflion/BookNote/blob/master/01lioneloutput/60BookCode/C++%E6%B3%9B%E5%9E%8BSTL%E5%8E%9F%E7%90%86%E5%92%8C%E5%BA%94%E7%94%A8/chap3STL%E5%8F%8A%E5%85%B6%E4%BD%BF%E7%94%A8%E7%9A%84%E5%85%B6%E4%BB%96C++%E6%8A%80%E6%9C%AF/E3-06.cpp)
+ *lionel，这节，没怎么看呢，以及后面的内容，都没怎么看*

##### 3.2.3 Lambda表达式

+ 无名/匿名仿函数
+ [例3-7 lambda表达式与具有相同功能的仿函数的比较](https://gitee.com/fewolflion/BookNote/blob/master/01lioneloutput/60BookCode/C++%E6%B3%9B%E5%9E%8BSTL%E5%8E%9F%E7%90%86%E5%92%8C%E5%BA%94%E7%94%A8/chap3STL%E5%8F%8A%E5%85%B6%E4%BD%BF%E7%94%A8%E7%9A%84%E5%85%B6%E4%BB%96C++%E6%8A%80%E6%9C%AF/E3-07.cpp)。**Lambda表达式**又叫 *无名仿函数（或匿名仿函数）*
  + `[](int x){cout << 2*x << endl;} // 相当于在声明function类的同时，又定义了定义该对象`
    + `[]`，仿函数中运算符`()`的重载函数名，也叫**lambda表达式前导符**
    + `[]`里面“捕获列表”，`[]`，不捕获，`[=]`，值传递捕获，`[&]`，引用捕获
    + `int x`中的int，是个**返回值类型**
    + void和auto关键字可以省略
    + **mutable关键字**
  + `[](int x){cout << 2*x << endl;}(10) //则是对象调用。`
+ [例3-8 多种lambda表达式调用示例程序](https://gitee.com/fewolflion/BookNote/blob/master/01lioneloutput/60BookCode/C++%E6%B3%9B%E5%9E%8BSTL%E5%8E%9F%E7%90%86%E5%92%8C%E5%BA%94%E7%94%A8/chap3STL%E5%8F%8A%E5%85%B6%E4%BD%BF%E7%94%A8%E7%9A%84%E5%85%B6%E4%BB%96C++%E6%8A%80%E6%9C%AF/E3-08.cpp)  
+ [例3-9 当用户输入一个15~20的数据时，输出2倍，否则输出本身](https://gitee.com/fewolflion/BookNote/blob/master/01lioneloutput/60BookCode/C++%E6%B3%9B%E5%9E%8BSTL%E5%8E%9F%E7%90%86%E5%92%8C%E5%BA%94%E7%94%A8/chap3STL%E5%8F%8A%E5%85%B6%E4%BD%BF%E7%94%A8%E7%9A%84%E5%85%B6%E4%BB%96C++%E6%8A%80%E6%9C%AF/E3-09.cpp)
+ **在lambda表达式中对捕获变量进行修改有两种办法**：
  + 使用引用传递
  + 使用mutable关键字
+ 例3-10 两个可以修改捕获变量的lambda表达式的示例程序。
  + 示例1：演示了引用捕获的程序代码
  + 示例2：演示了关键字mutable的应用
+ **lambda表达式定义的完整格式**：`[=]()mutable throw() -> int{}`
  + `[=]`，捕获变量列表
  + `()`，形参列表
  + `mutable`，值传递捕获变量可修改说明符
  + `throw()`，抛出异常
  + `int`，返回值类型
  + `{}`，函数体
  + `[](int a, int b) ->int{return a+b;}`，*lionel，是这么写嘛，如果只写个add函数的话*
+ 例3-11 把lambda表达式应用于STL通用算法的一个程序示例
+ *mutable与exception关键字，同时自己记住各种使用场景，lionel*

#### 3.3 智能指针

+ 把资源（内存）与指针对象的生命周期绑定起来，创建对象时立即为其分配资源，对象销毁时则立即释放资源

##### 3.3.1 智能指针的基本原理

+ 1、指针对象与资源的绑定
  + **RAII（Resource Acquisition is Initialization）**，是把裸指针（原始指针）和与它相关的操作封装成一个类，在创建对象时调用构造函数，对象销毁时调用析构函数。重载了`opertaor->`和`operator *`
  + [例3-12 编写一年智能指针示例程序](https://gitee.com/fewolflion/BookNote/blob/master/01lioneloutput/60BookCode/C++%E6%B3%9B%E5%9E%8BSTL%E5%8E%9F%E7%90%86%E5%92%8C%E5%BA%94%E7%94%A8/chap3STL%E5%8F%8A%E5%85%B6%E4%BD%BF%E7%94%A8%E7%9A%84%E5%85%B6%E4%BB%96C++%E6%8A%80%E6%9C%AF/E3-12.cpp)
+ 2、可共享资源的智能指针
  + 多个指针指向同一个资源（计数器）
  + 类中以友元类的方式指定了资源使用者的所属类SharePtr
  + 第一次创建时，用裸指针进行初始化，计数器置1，以后每创建一个副本，再加1
  + 创建时加1，销毁时减1，直到0为止。
  + [例3-13 编写一个演示程序](https://gitee.com/fewolflion/BookNote/blob/master/01lioneloutput/60BookCode/C++%E6%B3%9B%E5%9E%8BSTL%E5%8E%9F%E7%90%86%E5%92%8C%E5%BA%94%E7%94%A8/chap3STL%E5%8F%8A%E5%85%B6%E4%BD%BF%E7%94%A8%E7%9A%84%E5%85%B6%E4%BB%96C++%E6%8A%80%E6%9C%AF/E3-13.cpp)

##### 3.3.2 C++11支持的智能指针

+ **C++98里的`auto_ptr`废弃了**，不能放在容器中

+ 1、unique_ptr
  + **无法复制构造/复制赋值，可以移动构造/移动赋值**
  + 0
  + unique_ptr用在如下场合
    + 利用动态指针的RAII特性，保证程序出现异常时能确保动态资源的释放
    + 返回函数内创建的动态资源
    + 可放在STL容器中
    + 管理动态数组
    + 自定义资源释放操作
+ 2、shared_ptr
  + 2种方式创建shared_ptr
    + 通过make_shared辅助函数创建shared_ptr，`auto s_s = make_shared<string>("hello");`
    + 通过构造函数构建shared_ptr，
  + 使用
    + 在if语句中作为转移条件
    + 定制资源销毁操作deleter
+ 3、weak_ptr
  + weak_ptr配合shared_ptr工作
  + 没有重载operator->和operator*操作符，**不能直接通过weak_ptr使用对象**
  + **两个类互为对方指针的资源类型，出现了所谓的“循环”引用**，会造成计数器的计数失控，于是产生了**可以共享资源，但不参与计数的weak_ptr来解决这种循环引用**

```c++
#include <iostream>
#include <memory>
using namespace std;

struct B; //类前置声明
struct A{
    ~A(){
        cout<<"~A()"<<endl;
    }
    shared_ptr<B>b;
}

struct B{
    ~B(){
        cout<<"~B()"<<endl;
    }
    shared_ptr<A>a;  //循环引用，不可用
    //改成这样
    weak_ptr<A>a;//在先声明的B中用weak_ptr打破循环引用
}

int main(){
    shared_ptr<A> ap(new A);
    shared_ptr<B> bp(new B);
    ap->b = bp;
    bp->a = ap;
    return 0;
}
```



### chap4、模拟STL三大件

+ 20201019
+ 容器、算法、迭代器是三大件。

#### 4.1、容器

+ 核心是**数据存储管理**，类模板
+ 同样的数据结构，因配置的操作方法不同而形成了不同容器。

##### 4.1.1、向量vector的仿真MyVector

+ STL规定，向量是单端开口的容器，其核心的数据存储装置（原始容器）是一个数组，**开口位置在数组尾部**
+ 例4-1.cpp
  + *lionel，如何一步一步来的*

##### 4.1.2、列表list的仿真MyList

+ 以链表为数据存储核心的容器，**首尾双端开口容器**，双向链表
+ *lionel，链表的设计思路*
+ 例4-2.cpp

#### 4.2、迭代器

+ **中介装置**把容器与算法分离，这样的**智能指针**就叫**迭代器**
+ `[begin,end)`，end()，**尾元素的下一个位置**（不存在的元素），比较时用`==`或`!=`，没有<这种，*这行是从其它笔记上摘抄的*

##### 4.2.1、使用裸指针作为迭代器

+ **以存储空间连续的数组为其存储结构的容器**
+ [例4-3.cpp，程序中定义一个`MyVector<int>`的对象v1，使用裸指针作为对象v1的迭代器并遍历v1](https://gitee.com/fewolflion/BookNote/blob/master/01lioneloutput/60BookCode/C++%E6%B3%9B%E5%9E%8BSTL%E5%8E%9F%E7%90%86%E5%92%8C%E5%BA%94%E7%94%A8/chap04%E6%A8%A1%E6%8B%9FSTL%E4%B8%89%E5%A4%A7%E4%BB%B6/04_03.cpp)
  + 与4-1的区别
    + 在main中增加了`MyVector<int>::viter iter;`
    + `typedef Ty* viter;`，以及 *这两个的转化过程，要好限理解一下，lionel*，这个在4-1.pp里也有的啊
  + *lionel，这里面的裸指针，怎么理解？*

##### 4.2.2、迭代器是指针的类封装

+ **裸指针仅适用于连续空间**，如果存储空间不连续，就需要**加以改造（重载*，++，==，!=运算符）**，`[]`的重载，跟是不是裸指针没关系
+ 例4-4.cpp，为MyList容器设计迭代器MyList_iterator并编写程序进行测试

##### 4.2.3、迭代器的代码隔离作用

+ 使用者无需知道容器的内部结构是否为数组、线性表、树或图，只要像使用指针那样使用它即可
+ 重载*，++，==，!=运算符（操作list）
+ 例4-5.cpp，程序，使用上述查找函数find()分别在MyVector和MyList容器查找数据100

##### 4.2.4、STL迭代器的种类

+ 0
  + 虽然不同容器的迭代器面貌可以相似，但其内部结构及工作机制会因容器的不同而不同，并且迭代器的功能也有强弱之分。一般分为5种
  + **功能较强的迭代器必须包含有功能较弱迭代器的功能**
+ 1、输入迭代器（输入流，**有以下功能**）
  + 可复制，取出值，并自动指向下一个，判断是否最后一个，*，++，++(int)，!=，==
+ 2、输出迭代器
  + 除了同1外，还通过`operator*`对容器数据元素进行修改
+ 3、前向迭代器
  + 输入和输出迭代器的结合体，`operator*`既可以访问元素，也可以修改元素
+ 4、双向迭代器
  + 在前向迭代器的功能上，**允许向后移动**，支持`operator--`和`operator--(int)`
+ 5、随机存取迭代器
  + 双向支持随机

##### 4.2.5、迭代器的种类标记

+ 5种类型迭代器
  + 输入迭代器，`struct input_iterator_tag{};`
  + 输出，`struct output_iterator_tag{};`
  + 前向，`struct forward_iterator_tag:: public input_iterator_tag{};`
  + 双向`struct bidirectional_iterator_tag:: public forward_iterator_tag{};`
  + 随机，`struct random_access_iterator_tag:public bidirectional_iterator_tag {};`
+ 例4-6.cpp
+ 例4-7.cpp

##### 4.2.6、STL对迭代器的管理

+ 1、迭代器类模板的数据类型表
+ 2、迭代器的类型总表traits
+ 例4-8.cpp
+ *lionel，这部分不太熟悉，没怎么看*

#### 4.3、通用算法

+ 通用算法的几个含义
  + 对STL容器中数据进行操作的常用算法
  + 必须为所有数据类型通用，即它们是函数模板
  + 不仅独立于
  + 有对迭代器的功能的要求，但不绑定于特定迭代器
  + 通常只提供算法框架，但对算法的具体实现策略由用户决定

+ 例4-9.cpp
+ `sort()`默认从小到大排序，可以改变一下`sort(vect.begin(), vect.end(), greater<char>());`，*lionel，这种写法，我没怎么写过*
+ 稍微复杂一些算法的功能大致分为两部分：
  + 一部分提供算法框架
  + 另一部分提供功能运行策略接收函数。
+ 因此参数列表中列写的参数就分为两部分：
  + 一部分是与容器相关的迭代器
  + 另一部分是算法策略对象
+ 例4-10.cpp，使用例4-3中MyVector容器和上面设计的MySort()算法编写一个测试程序

### chap5、容器及其应用

+ 20201104
+ 负责数据的存储和管理，按照数据元素的组织方式分为
  + 序列容器：向量vector，列表list，双向队列deque
  + 关联容器：set，map

#### 自己从其它地方看的补充

+ deque（double-ended queue）
+ vector
  + **不能将引用**作为对象
  + 如果`{}`不能初始化，则转换构造函数初始化
  + *其实可能指定vetcor的大小，然后用下标`[]`操作的*
  + 删除的话，用`erase()`或者`pop_back()`，**要注意迭代器失效的问题**
+ stack
  + 用的是deque实现的，也可以是vector或list
  + s.emplace()是啥意思？
+ queue
  + 先进先出，默认deque，vector或list也可
+ priority_queue
  + 优先级最高的先出，默认deque，vector或list也可
  + .top()，只返回，不删除
  + **默认使用<排序，返回最大值>**，最小堆要自己指定一下
+ set
  + 无序的unorder_set，是*hash组织的？*
  + set的iterator是只读，const
  + 有序比较"<="，*这个自己不太会，笔记上代码也写得不清楚了*
+ map
  + map的iterator，解引用是pair<key,value>，输出是升序的
  + map支持`[]`操作，**multimap不支持**，同时**map的下标操作，只能用来插入，不能用来查询**
  + 有序比较"<="
  + 检查insert的返回值
  + 无序版本，非内置类型，自定义hash函数
+ multimap存在多结果情况
  + 先count，再遍历iterator
  + 使用low_bound和upper_bound
  + 使用lequal_range

####  5.1 向量vector

+ 占用连续内存，随机访问，下标和`at()`操作，**尾部开口的设计**，尾部操作方法为`push_back()`和`pop_back()`，重载了`operator=`，**尾部操作效率最高**（*也只有尾部能操作吧*）

##### 1、vector对象的定义及初始化

+ vector常用构造函数
  + `vector();`
  + `vector(vector&& _Right);`//使用向量初始化
  + `vector(size_type Count); //只有数量的话，初值为0？lionel`
  + `vector(size_type _Count, const Type& val);//设定大小并初始化值`
  + `template<class InputIterator> vector(InputIterator _First,InputIterator _Last); // 以迭代器_First和_Last之间的元素初始化该向量`
+ 例5-1 测试vector中的构造函数
  + `vector<int> v2(5,2);//长度为5，初始值为2的int型向量`
+ 表5-2，读向量容器元素的操作
  + `reference front();  const_reference front() const;` 返回第一个元素
  + `reference back();  const_reference back() const;` 返回最后一个元素
+ 表5-3，各种用于改变向量容器中数据的成员函数
  + `[]`
  + `at()`
  + `clear()`
  + `erase(position)`
  + `erase(begin,end)`
  + `insert(position,elem)`
  + `insert(position,n,elem)`
  + `push_back(elem)`
  + `pop_back()`
  + `resize(num)`
  + `resize(num,elem)`
+ 例5-2 编写一个程序，在程序中使用一个可以存放整型（int）数据的vector对象，并在其尾部逐次压入103、765、208、435数据，然后做如下工作：（1）分别使用函数at()以及下标法在显示器上显示向量内容（2）
+ 表5-4：计算向量容器大小的操作
  + `capacity()`，在当前存储空间下，容器vecCont可容纳元素的数目
  + `bool empty() const;`
  + `size()`，容器当前所存储元素的数目（向量长度）
  + `size_type max_size() const;`
+ 例5-3 编程程序，测试“表5-4：计算向量容器大小的操作”的成员函数，了解其功能
+ 新特性
  + 初始化`vector<int> v1{1,2,3};`，支持这种写法
  + for遍历，引用传递`for(auto &i :v1) i*=i;` ，平方操作后再返回
+ 例5-4 编写一个在vector上使用C++11新规则的程序。
+ [cplusplus-vector](http://www.cplusplus.com/reference/vector/)
+ [C++ vector用法](http://www.cnblogs.com/wang7/archive/2012/04/27/2474138.html)  *不过，这个开始看还行，现在来看，差不多都会了，算法部分接触得少。*

####  5.2、列表list

+ 核心是**双向链表**，**头和尾插入和删除**，`push_back()`还有`push_front()`，当然对有应的`pop_front()`和`pop_back()`，**不支持随机操作和`at()`操作，只能从到尾或从尾到头顺序地操作**
+ 表5-5：使用list构造函数
  + `list()`
  + `list(size_type n)`  //包含n个元素的列表
  + `list(const list& x)`
  + 可以用迭代器的下标
+ 表5-6：数据元素插入和删除
  + `assign()`
    + vector和list都支持这个用法，**对容器整体赋值**
  + `push_back()`
  + `remove_if()`
  + `remove()`
  + `unique()` ，删除相邻重复元素
+ 例5-5.cpp，在程序中观察push_front()、push_back()以及assign()函数的功能
+ 表5-7，可以返回list关键迭代器的函数
  + `begin()`
  + `end()`
  + `rbegin()`
  + `rend()`
+ 表5-8，可以提供list状态的函数
  + `empty()`
  + `max_size()`
  + `size()`
  + `resize()`
  + `reverse`，list自己有成员元素 反转的
  + `sort`
  + `merge`
  + `swap`
    + 成员函数版本是`c1.swap(c2);`
    + 非成员函数版本是`swap(c1,c2);`

#### 5.3、双向队列deque

+ **双向开口**，尾部和头部都可以操作的函数，`push_back()`还有`push_front()`，当然对有应的`pop_back()`和`pop_front()`，**类似于链表**【分多个段，每个段内连续】
+ 维护了一个**map的指针数组**，来维护段首地址，这样从map访问deque空间就是连续的，*lionel，具体的实现，还得看下代码*，P145
+ deque的特点
  + 支持随机访问（即支持下标`()`和`at()`），但性能没有vector好
  + 可以在内部进行插入和删除操作，但性能不及list
  + 两端能够快速插入和删除元素，vector只能尾部
  + 元素存取和迭代器操作会稍微慢一些，因为deque的内部结构会多一个间接过程
  + 使用内存比vector和list合理
+ 表5-9：deque的常用构造函数
  + `deque()`
  + `deque(size_type n)`
  + `deque(const deque& x)`
+ deque有以下两点与vector不同
  + deque不提供容量操作`capacity()`和`reverse()`
  + deque直接提供函数完成首尾元素的插入和删除
+ 需要注意的是
  + 除了`at()`函数，其它成员函数都不会检查索引或迭代器是否有效
  + 元素的插入和删除可能会导致内存重新分配
+ 例5-6.cpp，测试deque的成员函数，并理解其特点

#### 5.4、STL关联式容器

+ 关联式容器主要用于存储**具有固有顺序的已序容器**，这种容器能按照数据的固有顺序对数据进行管理。典型的是**映射**。带有键值的数据叫做关联式容器
  + **键值、实值**相等，就是set
+ 数据本身并无顺序，那么容器的各数据元素存储单元就必须具有固定的顺序，为**序列式容器**，如果数据本身有序，则容器没有必要一定有序
+ 原本无固定顺序的数据叫**可序数据**，具有固定顺序的数据叫**已序数据**

##### 1、关联式数据与STL二元组类模板pair

+ **关联数据**，多个互相有关联的数据组合成的一个数组组，pair没有指定键值、实值

+ utility.h中的`pair()`，**类模板**，称为**二元组**
  + 也可用`make_pair()`创建

```cpp
pair<string,int> pr2("heaven",7);
pair<string,int> *prp = new pair<string,int>("yards",0);
//也可用`make_pair()`创建
typedef struct pair<int,float> PAIR_IF;
PAIR_IF pair1 = make_pair(18,3.14f);
```



##### 2、STL对关联数据的组织与存储   ，P150

+ 二分查找树，*后面没有完全再看看*
+ 平衡二叉树
  + AVL树
  + 红黑树
+ *还是需要把DS再看一下，复习下tree的章节，lionel*

#### 5.5、map容器

+ 以红黑树形式，**pair形式出现的键-值对，叫做映射（map），set中的数据既是键值也是实值，叫做集合（set）**
+ 以**数据的键值**排序

##### 5.5.1、map容器的定义

+ *还能带分配器，是啥意思，是不是之前的纸质笔记，写得有问题，lionel*
+ 空间分配器，这个后面看到了再来看
+ 例5-7.cpp，使用map的默认构造函数创建一个map对象，然后使用insert()函数向其中插入11个pair数据
+ 例5-8.cpp
+ 例5-9.cpp，使用不同的构造函数常见map容器

##### 5.5.2、map的数据插入

+ 1、用insert函数插入pair数据
  + 例5-10.cpp
+ 2、insert()方法中指定数据类型的数据如下插入法
  + 例5-11.cpp
  + **当以重复键使用insert()方法插入数据时，只有第一次插入为有效操作，其后的所有都为无效操作，但编译器不报错**
  + 例5-12.cpp，使用insert()的方法返回值判断插入操作的有效性
+ 3、以下标方式使用键值插入数据
  + 例5-13.cpp，以下标形式向map对象插入数据
  + **下标方式可以重复的键值输入数据，后面的输入将覆盖前面的实值**

```cpp
map<int,string>mapStudent;
mapStudent[1]="student_one";
```

##### 5.5.3、map容器的其他常用成员方法

+ 1、size()方法
+ 2、count()和find()方法
  + 例5-14.cpp
+ 3、lower_bound()和upper_bound()方法
  + 返回键值空间的上界和下界，*有啥用？lionel*
  + 例5-15.cpp
+ 4、erase()方法
  + 有3种重载形式
  + 例5-16.cpp

##### 5.5.4、multimap容器

+ 例5-17.cpp，multimap容器应用示例程序
  + *应用中，这个不太会用啊*

#### 5.6、set容器

+ **set用来保存只有键值而没有实值这类数据的容器**，也是红黑树，只不过各节点的数据是单一值而不是pair。
+ 元素插入时会默认排序，**默认为`less<>`排序规则**
+ 例5-18.cpp，set容器应用示例程序

#### 5.7、hash表基础及hast容器

##### 5.7.1、hash表基础

+ 提出的背景：
  + 有{1,7,9,10,23}5个元素，如果**以存储数据单元的地址查找数据**就需要23个空间，但现在用了一个**函数关系**代替之前的**1比1**的关系

+ 散列表的链表法中有如下几个常用术语
  + 节点：链表中的节点，主要内容就是**用户数据**
  + 桶子（bucket）：数组元素，
  + 负载系数

##### 5.7.2、hash容器

+ C++11才纳入标准
+ 表5-10，C++11标准库的四种散列表
  + unordered_set
  + unordered_multiset
  + unordered_map
  + unordered_multimap
+ 例5-19.cpp，unordered_map的应用示例
  + `unordered_map<string, int, StrHash, StrCompare> StrMap;` //相当于后2个是指定的，lionel，要看一下unordered_map的原型，后面2个都是定义的类

### chap6、通用算法

+ 20201104
+ STL以函数模板形式向用户提供了大量通用算法
+ 通用算法是**迭代器技术与函数模板技术**相结合的产物，模板技术解决了算法的**类型通用**，迭代器解决了算法的**容器通用**

#### 6.1、通用算法的参数

+ STL通过两个措施保证了算法的通用性
  + 一是它把算法都设计成**函数模板**，从而依靠类型占位符保证了算法通用于各种参数类型
  + 二是它用可以屏蔽容器结构与数据差异的**迭代器**来指定算法的操作数，从而保证算法通用于各类容器
+ STL通用算法的3种参数
  + 算法的迭代器参数
  + 辅助参数
  + 谓词参数

##### 6.1.1、算法的迭代器参数

+ 两个迭代器指定一个区间
+ 1、单区域操作数的迭代器参数
  + `sort(Vect.begin(),Vect.end());`
+ 2、双区域操作数的迭代器参数
  + `result = find_first_of(Vect1.begin(), Vect1.end(),(Vect1.begin()+3),(Vect1.end()-2));`
  + swap_ranger()注意事项
+ 3、迭代器类型对通用算法的限制及容器特有方法
  + **迭代器**是裸指针的类封装，从而屏蔽了裸指针的一些基础功能而使之成为可以操作不同数据的安全指针。
  + 容器的种类繁多从而导致了迭代器的种类繁多
  + 将迭代器从强到弱规范出5个种类（等级）的迭代器类型
    + 随机迭代器（random access），**具有裸指针的全部功能**
    + 双向迭代器（bidirectional）
    + 前向迭代器（forward）
    + 输入迭代器（input）
    + 输出迭代器（output）
  + *本节，没太看懂全部啊，lionel*  P178

##### 6.1.2、辅助参数

+ **常量参数**，用来指定运算时所需要的系统、偏移量、极限值等。

+ 格式
  + `alg(__first,__last,__params);`，**一般find()就这样**，`result = find(Vect.begin(),Vect.end(),num_to_find);`

##### 6.1.3、谓词参数

+ P180

+ STL算法可以**函数、函数对象、lambda表达式**作为参数，目的是，**为了向算法传递用户自己的功能代码，从而便于实现通用算法的个性化和多样化**

+ 1、什么是谓词及谓词参数
  + **谓词**就是一个功能模块代码
  + *sort，重载预设的谓词，greater()*
+ 2、谓词的实质是回调函数
  + **调用**：用户程序使用系统程序
  + **回调**：系统程序使用用户程序
  + **回调函数**：用户提供的为系统程序所调用的程序段
  + 例6-1.cpp，在程序中创建一个向量对象并将其初始化，然后调用通用算法for_each，分别用lambda表达式、函数和函数对象作为谓词在计算机屏幕上显示向量内容
    + `for_each(Vect.begin(), Vect.end(), print)`，这个print是个函数，这里用函数名
    + `for_each(Vect.begin(), Vect.end(), printd());`这个printd是个类，重载了()，**成为仿函数，即 函数对象**
+ 3、STL对谓词的规范
  + 一元、二元谓词
    + `struct unary_function{}`
    + `struct binary_function{}`
  + 例6-2.cpp，按照unary_function格式定义谓词is_negative的程序示例
    + `struct is_negative : public unary_function<int,bool>{}`，用处在于继承了一元、二元谓词
    + `remove_if(Vect.begin(),Vect.end(),is_negative());`
  + 常用的STL预设二元谓词模板
    + `less<T>`
    + `less_equal<T>`
    + `equal<T>`
    + `not_equal<T>`
    + `greater_equal<T>`
    + `greater<T>`
    + `not2<B>`
    + *lionel，以上的用法，我就不太确定*
  + 上面的问题，就是**谓词，与谓词函数的区别**，目前啥区别，暂时不知道
    + *想到了，本身传的是函数名*

#### 6.2、算法时间复杂度

#### 6.3、常用通用算法

##### 6.3.1、查找和搜索算法

+ **不改变容器中数据**，非变异算法
+ 1、adjacent_find算法
  + 6-3.cpp
+ 2、binary_search算法
+ 3、count和count_if
  + **因为参数类型只是占位符，无法重载，后面加_if，表示需要谓词**
  + 例6-5.cpp
  + 例6-6.cpp，算法count_if()示例
+ 6-11.cpp

##### 6.3.2、变异算法

+ 6-12.cpp
+ 7、unique和unique_copy算法
+ 8、fill和fill_n（向指定序列中填充指定数据，只不过指定被操作序列的方式不同）
  + 6-24.cpp
+ 9、generate和generate_n算法
  + generate，使用谓词指定数据发生器产生的数据算法前两个参数容器指定的范围进行数据填充
  + generate_n，与上一样，**不过，它使用操作区域，首迭代器和长度来指示数据填充区的范围**
  + 6-25.cpp，
+ 10、transform算法，（使用参数中提供的谓词对指定序列进行变换，并将其结果复制到指定空间）
  + 6-26.cpp，

##### 6.3.3、排序算法

+ **带谓词，不带的区别**
+ 1、merge和inplace_merge算法
  + 6_27.cpp
  + 6_28.cpp，**带有谓词**
  + 6_29.cpp，inplace_merge，**内部归并算法**（对一个容器内部的两个方向相同的已序序列进行排序）
+ 2、
+ 3、
+ 4、
+ 5、
+ 6、reverse算法【容器中指定区域数据顺序进行反转】
  + 6-36.cpp，`reverse(Vect.begin(),Vect.end());`
+ 7、rotate算法【被操作的序列的前半部分与后半部分交换】
  + 6_37.cpp，`rotate(Vect.begin(),Vect.begin()+8,Vect.end());` //Vect.begin()+8，表示 分界迭代器

##### 6.3.4、算术算法与关系算法

+ 1、accumulate算法【有**附加参数**，**还可以带谓词**】
  + 6-38.cpp，`accumulate(fVect.begin(),fVect.end(),0.0f);//0.0是附加参数`
+ 2、partial_sum算法【指定序列求部分和**并将结果组成新的序列**】
+ 3、
+ 4、
+ 5、
+ 6、
+ 7、max和min算法
  + 6_44.cpp
  + *这两个也可以用谓词不，lionel*

##### 6.3.5、排列组合与集合算法

+ 1、next_premutation和prev_permutation算法
+ 2、set_union算法
  + 6_46.cpp
  + **集合算法**中还有，set_intersection，set_difference，set_symmetric_difference

### chap7、适配器模式在STL基础部件上的应用

+ 20201031

#### 7.1、适配器

+ 例子
  + 1、不提供基础功能，基础功能来自另一个对象，例如B的基础功能，全来自A
  + 2、通过转换基础功能对外接口的形式向用户提供服务

```cpp
class A {f1();f2();};
class B {A *a; g() {a->f1();a->f2();}};//模块B借助模块A，叫做B适配器
```

#### 7.2、STL容器适配器

+ stack、queue单向队列，priority_queue，优先权队列

##### 7.2.1、stack适配器

+ 使用默认deque
+ 使用vector或list
+ 7-1.cpp：使用STL提供的stack适配器并用默认的deque构成栈

```cpp
deque<int> mydeque(3, 100); //创建具有3个元素 初始值100的队列
stack<int> Mystack2(mydeque); //创建一个栈用mydeque对其初始化   【这是适配器的精华？】lionel，核心是这个，那原因呢？
```



+ 7-2.cpp：使用非默认的容器（list或vector）作为基础容器构造stack并测试它

```cpp
//使用list或vector作为栈的容器
stack<int, list<int>>a;  //这个用法，不太会呢？lionel，一般只放一个int或啥的
stack<int, vector<int>>b;
```



##### 7.2.2、queue适配器

+ **双向**，使用**双端开口**的list或deque作为核心，默认deque
+ 7-3.cpp：对STL提供的queue适配器进行测试
  + `queue<int> b;`**表示其使用的是默认的基本容器**
  + 但示例代码中用的是`queue<int,list<int>>a;，使用list作为底层结构的单向队列`

##### 7.2.3、priority_queue适配器

+ *本节，未读*，P266
+ 1、数组与完全二叉树
+ 2、堆与堆排序
+ 3、堆的创建
+ 4、堆排序
  + 7-4.cpp：编写一个堆排序程序
+ 5、priority_queue及其常用方法
  + 7-5.cpp：测试priority_queue的构造函数
  + 7-6.cpp：调用了优先权队列其他成员方法的程序示例

#### 7.3、迭代器适配器

+ 迭代器适配器，在**其类中封装了一个容器对象**，但被适配的对象不是容器，而是**附属容器的那个迭代器**，因此，适配的结果就不再是容器适配器，而是迭代器适配器
+ 迭代器适配器（3种）
  + 插入迭代器
  + 反向遍历迭代器
  + IO流迭代器

##### 7.3.1、插入迭代器

+ 通常**作为参数**向算法说明向哪个容器以及在该容器的哪个位置插入数据。比如`copy算法`

+ 反向插入，back_insert

+ 1、insert_iterator
  
  + 插入迭代器，**其实隐含了一个容器**
  
  ```cpp
  insert_iterator{
      _Container& __x, typename __Container::iterator __i):Container(&__x),iter(__i) ()
  }
  ```
  
  
  
  + 7-7.cpp：vector容器Vect，把3之后的数据插入到list容器Lst中的数据

```cpp
vector<int> Vect{ 1,2,3,4,5,6,7,8,9,10 };  //lionel,初始化的方式
//定义插入迭代器insert_it并使其指向被插入容器的待插入位置
insert_iterator<vector<int>>  insert_it(Vect, Vect.begin() + 3);
//将源数据对象中的数据插入迭代器指示的位置
copy(Lst.begin(), Lst.end(), insert_it);
```



+ 2、front_insert_iterator和back_insert_iterator
  + **默认的插入迭代器**，插入位置为目标的头部或尾部
  + **只有当容器提供push_front操作时，才能使用front_inserter**
  + 7-8.cpp：使用front_insert_iterator和back_insert_iterator迭代器实现数据的插入

##### 7.3.2、反向迭代器

+ reverse_iterator
+ 定义的格式，`vector<int>::reverse_iterator rpos(Vect.end());`
+ 7-9.cpp：观察正向和反向迭代器之间的关系
+ 7-10.cpp：使用了rbegin()和rend()操作的反向迭代器应用程序示例

##### 7.3.3、IO流迭代器

+ 1、输出流迭代器适配器ostream_iterator
  + 7-11.cpp
+ 2、输入流迭代器适配器istream_iterator
  + 7-12.cpp
  + 7-13.cpp，从标准设备中输入一些内容，然后用这些数据对一个向量容器进行初始化，然后显示初始化结果

+ 输入流，instream
+ 输出流，ostream
+ 输入缓冲，istreambuf_
+ 输出缓冲，ostreambuf_

#### 7.4、函数对象适配器

+ 函数对象的优点
  + 1、函数对象可以具有自己有数据，像一个独立的小程序，用起来比函数要方便灵活得多
  + 2、函数对象具有类型，所以把它作为通用算法的谓词参数时便于算法的承载

##### 7.4.1、函数对象的适配

+ **函数对象的适配器仍然是一个函数对象**

+ 7-14.cpp，在算法remove_if中验证一元函数对象greater_l的正确性
+ 7-15.cpp，在算法remove_if中验证第二参数绑定适配器binder2nd的正确性

##### 7.4.2、函数对象配接器

+ 例7-16.cpp，在算法remove_if中验证配接器bind2nd的功能
  + **函数对象配接器**，提供了一些能替用户完成定义和调用这些适配器的辅助函数
  + bind2nd本质就是调用了`binder2nd()`，要了解一下这个原型
  + [cpp-bind2nd](https://cplusplus.com/reference/functional/bind2nd/?kw=bind2nd)
+ 1、简单配接器
  + 表7-4，简单配接器
    + `bind1st(op,value)`
    + `bind2nd(op,value)`
    + `not1(op)`
    + `not2(op)`
  + 例7-17.cpp，在算法remove_if中验证配接器bind2nd的功能
    + `remove_if(Vect.begin(),Vect.end(),bind2nd(greater<int>(),100));`//移出向量中所有大于100的元素
  + 例7-18.cpp，在算法sort中验证配接器not2的功能
+ 2、对成员函数进行配接的函数配接器
  + 表7-5，对成员函数进行配接的函数配接器
    + `mem_fun_ref(op)`，调用op，op是某对象的一个const成员函数
    + `mem_fun(op)`
  + 例7-19，验证配接器mem_fun_ref(op)和mem_fun(op)的功能
    + `for_each(coll.begin(),coll.end(),mem_fun_ref(&testCls::print));`
+ 3、针对一般函数（非成员函数）而设计的函数配接器
  + 表7-6，针对一般函数（非成员函数）而设计的函数配接器
    + `ptr_fun(op)`，等同调用了函数op(param)
  + 例7-20，验证配接器mem_fun_ref(op)和mem_fun(op)的功能
+ 4、用户自定义函数对象配接器
  + 例7-21，定义一个用户函数对象并在transform算法中利用配接器将其作为谓词使用

### chap8、STL容器内存空间分配器

+ 20201031

#### 8.1、内存空间配置器及其设计基础

##### 8.1.1、什么是内存空间配置器

+ 内存的静态分配，动态分配

+ 封装了动态请求内存空间的代码，以函数的形式向容器提供了内存空间的请求与释放，不用new本身，而用`operator new()`和`spacement new()`

##### 8.1.2、内存空间配置器设计基础

+ 1、容器内存空间的结构及配置器设计思路
  + 连续的数组，vector
  + 非连续的链表，list
  + 多个连续空间组成的链表模式，deque
+ 2、内存空间配置器设计技术
  + 例8-1.cpp

#### 8.2、STL空间配置器接口

+ 处于容器与OS内存管理系统之间

##### 8.2.1、STL空间配置器接口及最简单的空间配置器

+ 1、STL空间配置器接口内容
  + **接口**：就一个程序部件提供给用户的那些函数
+ 2、
+ 3、内部不同数据类型数据的空间配置器

##### 8.2.2、典型STL容器空间的配置

+ 例8-2.cpp，使用向量容器对上述容器内存空间配置器进行测试，在测试中编写必要的测试代码以显示配置器的工作过程
+ 例8-3.cpp，修改8-2.cpp，请使用list容器来测试上述内存空间配置器

#### 8.3、内存池的概念及方法

+ **内存池**就是应用程序从系统那里批发来的内存空间。防止内存碎片、用内存块组成内存池（形成链表）

##### 8.3.1、内存池的规划

##### 8.3.2、内存池的设计

+ 1、MemoryBlock的设计
+ 2、MemoryPool的设计
  + 例8-5.cpp

### 附录

#### A、关于关键字explicit

+ 类的单参数构造函数能实现一个隐式转换，**加上explicit，就禁止转换**，`explicit Test(int x){a = x;}`，本来是支持`Test a=10;//现在就不行`，只能使用`Test a1(10);`

### 勘误

+ 264页，代码中有2处`cout << a.front() << " " << a.back() << endl;`，第2行的输出应该改成`cout << b.front() << " " << b.back() << endl;`