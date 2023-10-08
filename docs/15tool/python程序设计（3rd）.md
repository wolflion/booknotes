## 《python程序设计》（3rd）

### chap2、编写简单程序

#### 2.3、程序要素

##### 2.3.1、名称

+ *标识符规则跟C一样*，**记住一些关键字**

##### 2.3.2、表达式

+ `“32”`是字符串，`'32'`是文本而不是数字，*lionel，要找个环境，测一下 单引 和双引的区别*

#### 2.4、输出语句

#### 2.5、赋值语句

+ **print语句中有end参数**，`print("The answer is", end="")  print(3+4)`，**允许多个print构建单行输出**

##### 2.5.1、简单赋值

+ `<variable> = <expr>`

##### 2.5.2、赋值输入

+ input()
+ eval()

##### 2.5.3、同时赋值

+ `score1, score2 = 86, 92`

#### 2.6、确定循环

+ 示例

```python
for i in range(10):
    x = 3.9 * x * (1-x)
    print(x)
```



+ `for i in [0,1,2,3,4,5,6,7,8,9]:`

#### 2.7、示例

### chap3、数字计算

3.1、数值数据类型

#### 3.2、类型转换和舍入

+ **转换为int，丢弃浮点值的小数部分**（被截断，而不是舍入）
+ **四舍五入，得用`round()`**

3.3、使用math库

3.4、累积结果：阶乘

3.5、计算机算术的局限性

### chap5、序列：字符串、列表和文件

#### 5.1、字符串数据类型

+ **字符串**是**字符序列**
  + **允许使用负索引，从字符串的右端索引**
  + **切片**，是个区间
  + **重复**，是`*`

```python
greet="Hello Bob"
greet[-1] #是 'b'
greet[0:3] #是'Hel'
3 * "spam" #是 'spamspamspam'
```



#### 5.2、简单字符串处理

#### 5.3、列表作为序列

+ **列表，可以是任意对象的序列**

```python
[1,2]+[3,4]  # 是[1,2,3,4]
[1,2] *3 #是[1,2,1,2,1,2]
grades=['A','B','C','D']
grads[2:4] #是['C','D']
```



#### 5.4、字符串表示和消息编码

##### 5.4.1、字符串表示

##### 5.4.2、编写编码器

#### 5.5、字符串方法

##### 5.5.1、编写解码器

5.6、列表也有方法

#### 5.9、文件处理

##### 5.9.1、多行字符串

##### 5.9.2、文件处理

### chap6、定义函数

#### 6.1、函数的功能

#### 6.2、函数的非正式讨论

+ *函数带有参数，如何处理，lionel，之前不太会啊*
+ *main也需要定义？是重定义吗？lionel*
  + [为什么Python没有main函数？](https://www.zhihu.com/question/587995854/answer/2928659729?utm_id=0)

```python
#happy.py
def happy():
    print("Happy Birthday to you!")
def sing(person): #冒号不能忘记，参数怎么用
    happy()
    happy()
    print("Happy birthday dear", person + ".")
    happy()
    
def main():
    sing("Fred")
    print()
    sing("Lucy")
    print()
    sing("Elmer")
    
main() # main也需要定义？是重定义吗？
    
```



#### 6.3、带有函数的终值程序

#### 6.4、函数和参数：令人兴奋的细节

+ 函数定义

```python
def <name> (<formal-parameters>):
    <body>
```



+ 在

#### 6.5、返回值的函数

+ triangle2.py
+ happy2.py

#### 6.6、修改参数的函数

+ addinterest3.py

#### 6.7、函数和程序结构

+ futval_graph4.py

### chap7、判断结构

#### 7.1、简单判断（if）

7.1.1、示例：温度警告

+ convert.py
+ convert2.py

7.1.2、形成简单条件

7.1.3、示例：条件程序执行

#### 7.2、两路判断（if-else）

+ quadratic.py
+ quadratic2.py
+ quadratic3.py

#### 7.3、多路判断（if-elif-else）

+ 格式

```python
if<condition1>:
    <case1 statements>
elif<condition2>:
    <case2 statements>
elif<condition3>:
    <case3 statements>
...
else:
    <default statements>
```



+ quadratic4.py

#### 7.4、异常处理

+ quadratic5.py
+ quadratic6.py

#### 7.5、设计研究：三者最大

7.5.1、策略1：比较每个值和所有其他值

##### 7.5.2、策略2：判断树

##### 7.5.3、策略3：顺序处理

```python
maxval = x1
if x2 > maxval:
    maxval= x2
if x3 > maxval:
    maxval = x3
#这样，判断x4的话，也只要接着后面写就行
```



##### 7.5.4、策略4：使用Python

```python
def main():
    x1,x2,x3=eval(input("Please enter three values: "))
    print("The largest value is", max(x1,x2,x3))
```



7.5.5、策略5：一些经验

#### 7.6、小结

### chap8、循环结构和布尔值

#### 8.1、for循环：快速回顾

+ average1.py，求平均值

#### 8.2、不定循环

+ for是个有限循环，**while循环是个 不定循环/条件循环**

```python
#语法
#while <condition>:
#    <body>
i = 0
while i <= 10:
    print(i)
    i = i + 1
```



#### 8.3、常见循环模式

##### 8.3.1、交互式循环

+ average2.py

##### 8.3.2、哨兵循环

+ 不断处理数据，直到达到一个特殊值，表明迭代结束。**特殊值称为“哨兵”**

+ average3.py
+ average4.py

##### 8.3.3、文件循环

+ average5.py
+ average6.py

##### 8.3.4、嵌套循环

+ average7.py

#### 8.4、布尔值计算

##### 8.4.1、布尔运算符

##### 8.4.2、布尔代数

+ DeMorgan第一定律

#### 8.5、其他常见结构

##### 8.5.1、后测试循环

##### 8.5.2、循环加一半

##### 8.5.3、布尔表达式作为判断

#### 8.6、示例：一个简单的事件循环

+ event_loop1.py-- keyboard-driven color changing window

### chap9、模拟与设计

#### 9.1、模拟短柄壁球

##### 9.1.1、一个模拟问题

##### 9.1.2、分析与规格说明

#### 9.2、伪随机数

#### 9.3、自顶向下的设计

##### 9.3.1、顶层设计

##### 9.3.2、关注点分离

##### 9.3.3、第二层设计

##### 9.3.4、

##### 9.3.5、

##### 9.3.6、

##### 9.3.7、

#### 9.4、自底向上的实现

##### 9.4.1、单元测试

##### 9.4.2、模拟结果

#### 9.5、其他设计技术

##### 9.5.1、原型与螺旋式开发

##### 9.5.2、设计的艺术

### chap10、定义类

### chap11、数据集合

+ 学习目标
  + 使用列表（数组）来表示相关数据的集合
  + 操作Python列表的函数和方法
  + Python字典存储无顺序集合

#### 11.1、示例问题：简单统计

#### 11.2、应用列表

+ 将整个值的集合放到一个对象中，`list (range(10))`，*不懂啥意思，lionel*

##### 11.2.1、列表和数组

+ **列表是有序的数据项序列**
+ **列表是动态的**，可以根据需要增长和缩小，**异质的**，单个列表中混合任意数据类型

##### 11.2.2、列表操作

+ 操作
  + `+`，连接
  + `*`，重复
  + `[]`，索引
  + `[:]`，切片，*lionel，不是很熟*
  + `for <var> in <seq>:`，迭代
  + `<expr> in <seq>`，成员检查（返回布尔值）
+ 方法
  + .append(x)，添加x到列表末尾
  + .sort()
  + .reverse()，
  + .index(x)，x第一次出现的索引
  + .insert(i,x)
  + .count(x)
  + .remove(x)
  + .pop(i)，删除列表中第i个元素并返回它的值
+ **列表的原则**
  + 列表是存储为单个对象的一系列数据项

##### 11.2.3、用列表进行统计

#### 11.3、记录的列表

#### 11.4、用列表和类设计

#### 11.5、案例分析：Python计算器

#### 11.6、案例研究：更好的炮弹动画

#### 11.7、无顺序集合

+ **Python为集合提供了几种内置的数据类型**，除列表外

##### 11.7.1、字典基础

+ Python的字典是**映射**
  + `passwd = ["quido":"superprogrammer"]`
  + `<dictionary>[<key>]`

##### 11.7.2、字典操作

+ `<key> in <dict>`
+ `<dict>.keys()`，返回键的序列
+ `<dict>.values()`
+ `<dict>.itmes()`，返回一个元组（key,value）的序列，表示键值对

##### 11.7.3、示例程序：词频

#### 11.8、小结

#### 11.9、练习

### chap12、面向对象设计

### chap13、算法设计与递归