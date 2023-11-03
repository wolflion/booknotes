## 《Effective Modern C++》

### chap1、Deducing Types

### chap2、auto

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

