## 《Effective Modern C++》

### chap1、Deducing Types

#### 01、理解模板型别推导

+ 要点速记
  + 在模板
  + 对万能引用形参进行推导时，左值实参会进行特殊处理
  + 对按值进行
  + 在模板型别推导过程中，数组或函数型别的实参会退化成对应的指针，除非它们被用来初始化引用

##### 情况1：ParamType是个指针或引用，但不是个万能引用

##### 情况2：ParamType是个万能引用

##### 情况3：ParamType既非指针也非引用

##### 数组实参

##### 函数实参

#### 02、理解auto型别推导

+ 要点速记
  + 在一般情况下，但是auto型别推导会假定用大括号括起的初始化表达式代表一个std::initializer_list，但模板型别却不会
  + 在函数返回值或lambda式的形参中使用auto，意思是使用模板型别推导而非auto型别推导

#### 03、理解decltype

+ 要点速记
  + 绝大多数情况下，decltype会得出变量或表达式的型别而不作任何修改
  + 对于型别为T的左值表达式
  + C++14支持decltype(auto)，和auto一样，它会从其初始化表达式出发来推导型别，但是它的型别推导使用的是decltype的规则
+ C++11中，decltype的主要用途大概就在于声明那些返回值型别依赖于形参型别的函数模板。

#### 04、掌握查看型别推导结果的方法

+ 要点速记
  + 利用IDE编辑器、编译器错误消息和Boost.TypeIndex库常常能够查看到推导而得的型别
  + 有些工具产生的结果可能会无用，或者不准确。所以，理解C++型别推导规则是必要的。

##### IDE编辑器

##### 编译器诊断信息

##### 运行时输出

### chap2、auto

#### 05、优先选用auto，而非显式型别声明

+ 要点速记
  + auto变量必须初始化，
  + auto型别的变量都有着条款2和条款6中所描述的毛病

#### 06、当auto推导的型别不符合要求时，使用带显式型别的初始化物习惯用法

+ 要点速记
  + "隐形"的代理型别可以导致auto根据初始化表达式推导出”错误的“型别
  + 带显式型别的初始化物习惯用法强制auto推导出你想要的型别

### chap3、Moving to Modern C++

#### 07、Distinguish between（）and {} when creating objects

+ brace：**花括号，大括号**，{}       parentheses：**圆括号**，()

+ 要点速记
  + 大括号初始化可以应用的语境最为宽泛，可以防止隐式窄化型别转换，还对最令人苦恼之解析语法免疫
  + 在构造函数重载决议期间
  + 使用小括号还是大括号，会造成结果大相径庭的一个例子是：使用两个实参来创建一个`std::vector<数值型别>对象`
    + `vector<int> input(1,2);//这个我忘了，把1个元素初始化为2`与`vector<int> inputs{1,2}; //这表示2个元素`
  + 在模板内容进行对象创建时，到底应该使用小括号还是大括号会成为一个棘手问题

+ 对于内置类型

```cpp
int x(0);
int y=0;
int z{0};  
int z={0};  //等价于 int z{0};
```



+ 对于自定义类型

```cpp
Widget w1;//调用 默认构造函数
Widget w2=w1; //不是赋值，调用复制构造函数
w1=w2; //不是赋值，调用 复制赋值运算符
```

+ std::initializer_list

#### 08、Perfer nullptr to 0 and NULL

+ 要点速记
  + 相对于0或NULL，优先使用nullptr
  + 避免在整型和指针型别之间重载
+ *《深入理解C++11》介绍得更详细些*
+ 3个重载版本，**0的型别是int，而非指针**

```cpp
void f(int);
void f(bool);
void f(void*);

f(0);  //调用f(int)，而不是f(void*)
f(NULL); //可能通不过编译，可能调用f(int)，但决不会调用f(void *)
```

+ **nullptr，不具备整型型别，也不具备指针型别**

#### 09、Perfer alias declarations to typedefs

+ 要点速记
  + typedef不支持模板化，但别名声明支持
  + 别名模板可以让人免写"::type"后缀，并且在模板内，对于内嵌typedef的引用经常要求加上typename前缀

#### 10、Perfer scoped enums to unscoped enums

+ 要点速记
  + C++98网格的枚举型别时，现在称为不限范围的枚举型别
  + 限定作用域的枚举型别仅在枚举型别内可见。它们只能通过强制型别转换以转换至其他型别
  + 限定作用域的枚举型别和不限范围的枚举型别都支持底层型别指定。
  + 限定作用域的枚举型别总是可以进行前置声明，而不限范围的枚举型别却只有在指定了默认底层型别的前提下才可以进行前置声明
+ enum class Color**限制范围**，不加class就表示不限制范围

```cpp
enum Color {black,white,read}; //3者所在的作用域，和Color相同
auto white=false;//white被声明过了

enum class Color {black,white,read}; //3者所在的作用域，被限定在Color内
auto white=false;//可以，没有被声明
Color c = Clolor::white;//这个没问题
Color c = white;//这个就有问题
```

11、

12、

13、

14、

15、

16、保证const

+ 要点速记
  + 保证const成员函数的线程安全性，除非可以确信它们不会用在并发语境中
  + 运用std::atomic型别的变量会比运用互斥量提供更好的性能，但前者仅适用对单个变量或内存区域的操作

#### 17、理解特种成员函数的生成机制