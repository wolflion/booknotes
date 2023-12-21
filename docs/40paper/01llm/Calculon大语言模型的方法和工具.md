## 《Calculon：大语言模型的方法和工具》

### Paper

+ https://dl.acm.org/doi/pdf/10.1145/3581784.3607102

#### ABSTRACT

#### 1、INTRODUCTION

#### 2、ANALYTICAL MODEL

##### 2.1、LLM Configuration

+ 采用Megatron，the framework of Megatron [44] for describing the structure of transformer-based [49] LLMs。
+ **同步小批量随机梯度下降**，**Adam优化器**，进行权重更新
+ 每个Transformer块具有相同的结构，由一个多头注意力块后跟一个多层感知器（MLP）块组成
  + hidden，隐藏大小
  + attn，the number of attention heads，注意力头数
  + the sequence size (seq)
  + training batch size (batch)
  + micro-batch size (m)
  + the number of transformer blocks (blocks)
  + *问题是，典型的LLM，transformer块结构是啥？*

##### 2.2、Hardware Configuration

+ 内存
+ 计算
+ 网络

##### 2.3、Execution Configuration

+ TP
+ PP
+ DP

2.4 Performance Calculation

2.5 Validation

### [Code](https://github.com/calculon-ai/calculon)

+ 整体思路是**先看整体架构，分成几个部分，各个部分是干啥的**，然后具体的几个部分再**通过单步调试的方式**去理解细节

#### 入口

+ bin/calculon

  + `__main__`
  + 引入了**argparse**库，*干啥的不清楚，lionel*
  + line44行是个啥？
    + 根据不同的version、runner、parameter去调用command_line中的**create_parser**

  ```python
    # Registers each command line interface.
    for cls in calculon.CommandLine.command_lines():
      cls.create_parser(sp)
  ```

  

  + *要单点看一下，lionel*

+ `__init__.py`怎么被加载的？lionel？

  + `from .runner import Runner`，表示从runner.py文件中导入Runner类
    + runner是模块的文件名
    + Runner是该模块中定义的类名

+ command_line.py中的定义的抽象类，都没实现，都是其它继承类实现的

  + **只实现了register**
  + 子类的最后一行都有`calculon.CommandLine.register(Version)`是啥意思？*啥时候执行呢，猜测其是注册，但何时注册呢*
  + **可能是个装饰器模式**
  + ` @staticmethod`是python中的装饰器，用于声明一个静态方法

#### 总体分几步

+ llm/runner.py中约定好了，但怎么调用的呢？
  + `class Runner(calculon.CommandLine):`，**表示类Runner，继承calculon.CommandLine类**
  + `calculon.CommandLine.register(Runner)`，每次都调用它们注册
    + 原型是`def register(cls):`，*其cls是啥意思，其实无所谓吗？lionel*

#### 第一步