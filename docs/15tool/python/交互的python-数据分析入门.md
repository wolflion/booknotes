+ https://shixiangwang.github.io/pybook/%E5%A4%A7%E7%BA%B2/

### chap6、NumPy

#### 6.1、NumPy简介与ndarry

##### 6.1.1、NumPy简介

```python
import numpy as np
np_array = np.arange(100) #表示np比list更快
py_array = list(range(100))
```

##### 6.1.2、创建ndarray

#### 6.2、ndarray数组操作

##### 6.2.1、数组运算

```python
arr = np.array([2,3,4],[4,5,6])
```



##### 6.2.2、索引与切片

##### 6.2.3、布尔型索引

##### 6.2.4、数组转置与轴转换

#### 6.3、ndarray数组函数与方法

##### 6.3.1、通用函数

##### 6.3.2、基本统计

### chap7、Matplotlib

+ 2024-04-09遇到一个问题

  + 本来之前`plt.show()`是可以的，我把各种方式封装成函数时，发现不行，*就搜了一下，搜到了这个，不是它解决的*，自己定义了一个main后，再调用函数，然后再加plt.show()后可以的

  + ```python
    if __name__ == '__main__':
        show_func() #自定义函数调用
        plt.show() #这时候plt的东西才打印出来
    ```

  + 问题：*不加个main那行，有啥区别？*

+ 自己抄人家代码，模改写例子里，也用到了`subplot()`，*不太明白啥意思*，7.3节讲到了

#### 7.1、Matplotlib入门

##### 7.1.2、命名约定

```python
import matplotlib as mpl  #mpl自己都没太用过
import matplotlib.pyplot as plt #.表示啥？
```

##### 7.1.3、如何展示图形

+ 绘制图形，`plt.plt()`用法，*自己不知道在哪看*

##### 7.1.4、保存图形

+ `fig.savefig("a.pdf")`，*fig对象是从哪来的？*
  + 看了下之前有`fig = plt.figure()  # 生成一个空白图形并将其赋给 fig 对象`

#### 7.2、基本图形绘制

##### 7.2.1、线图

##### 7.2.2、散点图

##### 7.2.3、条形图

##### 7.2.4、直方图

##### 7.2.5、饼图

##### 7.2.6、箱线图

#### 7.3、多图与自定义

+ 有时，需要在一个图形中绘制子图

##### 7.3.1、多图

```python
plt.subplot(2,2,3)  #2行2列，放在第3个，10以内可以plt.subplot(449)
plt.title()  #这个是自己看到的例子上会写
plt.text() #这是本书中的例子
```

+ subplots_adjust()调整子图间的空隙
  + `plt.subplots_adjust(hspace=0.4, wspace=0.4)  # 调整子图之间的高与宽   间隔`
+ **手绘坐标轴**

```python
ax1 = plt.axes()  # 标准坐标轴
ax2 = plt.axes([0.2, 0.65, 0.2, 0.2])# 子图距离左侧0.2，下方0.65，宽度0.2，高度0.2
```



+ 在

##### 7.3.2、设置风格

+ `plt.style.use('seaborn-white')`

##### 7.3.3、两种接口映射

+ *哪2种接口啊*，一种是Matlab样式接口，另一种是？

### chap8、Pandas简介

#### 8.1、Pandas 简介

+ Pandas 的名字来自**面板数据（Panel data）**和**数据分析（Data analysis）**的组合

#### 8.2、Pandas的数据结构

##### 8.2.1、Series

+ Series 形似字典，包含索引（也称数据标签）和数据两部分。
  + `scores = pd.Series([80, 90, 97])`只给了值，没给索引，**会自动创建**从0开始的索引
  + 通过`scores.index`来看到索引，`scores.value()`看到值
  + `scores = pd.Series([80, 90, 97], index=[u'语文',u'数学',u'外语'])`，**指定索引**

+ **Pandas 库是基于 NumPy 库构建的，所以 Series 实际上是通过一维的 NumPy 数组实现的。**

##### 8.2.2、DataFrame

+ **Series 对象只能有效地表示一维的数据**

```python
df = {'姓名': ['小明','小王','小张'], '语文':[80,85,90], '数学':[99,88,86] }
df = pd.DataFrame(df)
```

+ DataFrame 的 index、columns、values 属性

#### 8.3、Pandas对象基本操作

##### 8.3.1、查看数据

```python
s1 = pd.Series(np.random.rand(1000))
s1.head()  #默认5行

d1 = pd.DataFrame({'a':np.random.rand(1000), 'b':np.random.rand(1000)})
d1.head()
```



##### 8.3.2、转置

+ `d1.T`

##### 8.3.3、重索引

#### 8.4、基本统计分析

### chap10、数据导入

#### 10.1、csv文件

##### 10.1.3、使用Pandas库

+ `records = pd.read_csv('records.csv')`
+ `records.cloumns`和`records.index`

#### 10.2、csv变体

##### 10.2.2、使用Pandas导入

+ `pd.read_csv('records.tsv', sep='\t')`，**多了一个sep属性**

#### 10.3、Excel文件

##### 10.3.1、检查数据

##### 10.3.2、准备工作

##### 10.3.3、使用Pandas读写Excel