## 《Effective Python》

### chap1、用Pythoninc方式来思考（1-10）

#### 1、查询自己使用的Python版本

```python
import sys
sys.version
```



#### 2、遵循PEP8风格指南

+ Python Enhancement Proposal #8

#### 4、用支持插值的f-string取代C网格的格式字符串与str.format方法

#### 5、用辅助函数取代复杂的表达式

+ 要点
  + Python的语法很容易氢复杂的意思挤到同一行表达式里，这样写很能懂

#### 6、把数据结构直接拆分到多个变量里，不要专门通过下标访问

#### 7、尽量用enumerate取代range

+ **range适合用来迭代一系列整数**
+ enumerate能够**把任何一种迭代器（iterator）封装成惰性生成器（lazy generator)**

```python
it = enumerate(flavor_list)
print(next(it))
print(next(it))  #必须要通过next()，才能找到下一个

for i, flavor in enumerate(flavor_list):
    print(f'(i+1):{flavor})
```

#### 8、用zip函数同时遍历两个迭代器

```python
names =['Cecilia','Lise','Marie']
counts = [len(n) for n in names]  #这个输出就是每个names的长度吧，7,4,5
longest_name = None
max_count = 0
for name count in zip(names, counts):
    if count > max_count:
        longest_name = name
        max_count = count
```



+ 要点
  + zip会创建惰性生成器，让它每次只生成一个元组，所以无论输入的数据有多长，它都是一个个处理的

#### 9、不要在for与while循环后面写else块

+ 要点
  + 只有在整个循环没有因为break提前跳出的情况下，else块才会执行

#### 10、用赋值表达式减少重复代码

+ 要点
  + 赋值表达式通过**海象操作符**（`:=`）给变量赋值，并且让这个值成为这条表达式的结果，于是，我们可以利用这项特性来缩减代码。
  + 如果赋值表达式是大表达式里的一部分，就得用一对括号把它括起来

### chap2、列表与字典（11-18）

+ 列表list，把每项任务都当成列表中的一个元素，*相当于数组？*
+ 与列表list互补的是**字典dict**（key-value），**访问和赋值所花时间，是个常量**

#### 11、学会对序列做切片

+ `[3:5]`，不含end，只有3，4两个元素
+ *越界怎么办？*
+ *切片数在左侧，右侧，有啥区别？*

#### 12、不要在切片里同时指定起止下标与步进

+ `somelist[start:end:stride]`，stride翻译是条带，其实是步进

#### 13、通过带星号的unpacking操作来捕获多个元素，不要用切片

```python
#不知道在哪抄的
#带*号形成列表（可变长度参数、解包操作符、扩展操作符）
#带*号表达式
def my_func(a,bc):
    print(a,b,c)
my_list = [1,2,3]
my_func(*my_list)
```



#### 14、用sort方法的key参数来表示复杂的排序逻辑

```python
#自己的例子
topos = []
keyfunc = functools.cmp_to_key(lambda x,y: operator.eq(x,y))
topos.sort(key=keyfunc)
```

15、不要过分依赖给字典添加条目时所用的顺序

+ 会与创建时不一样

#### 16、用get处理键不在字典中的情况，不要使用in与KeyError

+ 字典的三种基本交互操作：**访问、赋值以及删除键值对**

17、用defaultdic处理内部状态中缺失的元素，而不要用setdefault

18、学会利用`__mising__`构造依赖键的默认值

### chap3、函数（19-26）

#### 19、不要把函数返回的多个数值拆分到三个以上的变量中

+ **unpacking机制**
+ 在返回多个值的时候，可以用带星号的表达式接收那些没有被普通变量捕获到的值。

#### 20、遇到意外状况时应该抛出异常，不要返回None

#### 21、了解如何在闭包里面使用外围作用域中的变量

+ 把辅助函数通过key参数传给列表的sort方法
  + 1、Python支持闭包（closure），*什么是闭包*
  + 2、函数在Python里是头等对象（first-class object）
  + 3、Python在判断两个序列（包括元组）的大小时，有自己的一套规则

```python
def sort_priority(values, group):
    def helper(x):
        if x in group:
            return (0,x)
        return (1,x)
    values.sort(key=helper)
```



#### 22、用数量可变的位置参数给函数设计清晰的参数列表

+ 要点
  + 用def定义函数时，可以通过`*args`的写法让函数接受数量可变的位置参数

#### 23、用关键字参数来表示可选的行为

```python
def remainder(number, divisor):
    return number % divisor
assert remainder(20,7) == 6

#调用函数时，在调用括号内可以把关键字的名称写在=左边，把参数值写在右边

#如果混用
remainder(number=20,7) #error，位置参数必须出现在关键字参数之前
remainder(20,number=7) #error，每个参数只能指定一次，不能既通过位置形式指定，又通过关键字形式指定
```



+ 关键字参数的灵活用法带来3个好处：
  + 1、用关键字参数调用函数可以让初次阅读代码的人更容易看懂
  + 2、它可以带有默认值，该值是在定义函数时指定的
  + 3、可以很灵活地扩充函数的参数，而不用担心会影响原有的函数调用代码

+ 要点
  + 函数的参数可以按位置指定，也可以用关键字的形式指定

#### 24、用None和docstring来描述默认值会变的参数

+ 要点
  + 如果关键字参数的默认值属于这种会发生变化的值，那就应该写成None，并且要在docstring里面描述函数此时的默认行为
  + 默认值为None的关键字参数，也可以添加类型注解

#### 25、用只能以关键字指定和只能按位置传入的参数来设计清晰的参数列表

+ 要点
  + Keyword-only argument是一种只能通过关键字指定而不能通过位置指定的函数。
  + Positional-only argument是这样一种参数，它不允许调用者通过关键字来指定，而是要求必须按位置传递。
  + 在参数列表中，位于`/`和`*`之间的参数，可以按位置指定，也可以用关键字来指定。

#### 26、用functools.wraps定义函数修饰器

+ *什么叫装饰器？*

### chap3、类与继承（22-28）

### chap6、内置模块（42-48）

### chap9、测试与调试（75-81）

#### 75、通过repr字符串输出调试信息

+ print函数、logging模块输出日志
+ **repr()**，是显示出其可打印形式，再用`eval()`给转换回去

```python
a = '\x07'
print(repr(a))
```

