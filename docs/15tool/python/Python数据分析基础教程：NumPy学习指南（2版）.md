## Python数据分析基础教程：NumPy学习指南（第2版）

+ *最关键的是numpy怎么创建数组，自己不太明白呢*

### chap1、NumPy快速入门

+ 先要安装Python，然后再安装NumPy
+ 还有其它的，**SciPy，Matplotlib，IPython**

#### 1.10、编译源代码

```shell
# python-devel是Python开发包
git clone git://github.com/numpy/numpy.git numpy
python setup.py build
sudo python setup.py install --prefix=/usr/local
```



#### 1.11、数组对象

+ NumPy数组在数值运算方面的效率优于Python提供的list容器。使用NumPy可以在代码中省去很多循环语句

```python
def numpysum(n):
	a = numpy.arange(n) ** 2  # 创建包含0~n 的整数的NumPy数组。
	b = numpy.arange(n) ** 3
	c = a + b
return c
```



#### 1.13 IPython：一个交互式 shell 工具

#### 1.14、在线资源和帮助

+ IPython中的**pylab模式**，输入`help`命令，即`help arrange`或者，**加问号**，`arrange?`

### chap2、NumPy基础

#### 2.1、NumPy 数组对象

+ **ndarray**是一个多维数组对象，两部分组成，**实际的数据，描述这些数据的元数据**
  + 大部分数组，只能操作**元数据**，而不改变底层的实际数据。

```python
import numpy as np
a = np.arange(5)  # 这个简写的不成样子，直接用arange(5)是不能运行的
a.dtype
# dtype('int32')  #这是输出的
```



2.2、

```python
import numpy as np
n = np.array(np.ararnge(2),np.arange(2))
```

#### 2.4、一维数组的索引和切片

+ `a[3:7]`选择元素3~6
+ `a[:7:2]`，**下标0~7，步长是2**
+ `a[::-1]`，**翻转数组**

#### 2.7、数组的组合

+ 水平组合、垂直组合和深度组合等多种组合方式，我们将使用vstack、dstack、hstack、column_stack、row_stack以及concatenate函数

### chap6、深入学习NumPy模块

#### 6.1、线性代数

+ `numpy.linalg`，计算逆矩阵、求特征值、解线性方程组以及求解行列式等

#### 6.2、动手实践：计算逆矩阵