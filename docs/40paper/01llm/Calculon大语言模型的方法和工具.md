## 《Calculon：大语言模型的方法和工具》

### Paper

+ https://dl.acm.org/doi/pdf/10.1145/3581784.3607102

#### ABSTRACT

This paper presents a parameterized analytical performance model of transformer-based Large Language Models (LLMs) for guiding high-level algorithm-architecture codesign studies.

+ 这篇论文展示了 基于transformer的LLMs 参数分析性能模型，为指导高级别算法架构codesign研究

This model de-rives from an extensive survey of performance optimizations that have been proposed for the training and inference of LLMs; the model’s parameters capture application characteristics, the hard-ware system, and the space of implementation strategies.

+ 这个模型性能优化的可执行调查，展示了LLMs的训练和推理目的；模型的参数捕捉应用字符，硬件系统，实现策略的空间。

With such a model, we can systematically explore a joint space of hardware and software configurations to identify optimal system designs un-der given constraints, like the total amount of system memory.

+  在这个模型中，我们系统和探索了硬件和软件的联合空间，去标识优化的系统设计在给定的限制中，像大量的系统内存。

We implemented this model and methodology in a Python-based open-source tool called Calculon.

+  我们实现了这个模型基于开源的python，叫Calculon

Using it, we identified novel system designs that look significantly different from current inference and training systems, showing quantitatively the estimated potential to achieve higher efficiency, lower cost, and better scalability.

+ 使用它，我们标识文本系统设计 与众不同的当前的推理和训练系统，展现质量，估计潜在达到高效，低成本和更好的规模

#### 1、INTRODUCTION

We consider the task of conducting high-level analyses for algorithm-architecture codesign of distributed clusters and transformer-based Large Language Models (LLMs) [3, 4, 36, 39, 40].

+  我们认为

By “high level,” we mean focusing on developing and using fast and coarse-grained analytical or semi-empirical models of the software and hardware, a stage of design that precedes detailed simulation or implemen-tation on actual hardware. 

The goal is to estimate the best-case
relative improvements that might come from significant changes
to the system or software, as well as combinations of system and

software configurations that might be unusual or otherwise costly
and difficult to implement. Since high level models are expected to
be much cheaper than detailed simulation, they should facilitate
rapid exploration of a potentially large parameter space during the
early phases of codesign.
In this work, our starting point is Megatron, a large family of
open-source LLM instances developed by NVIDIA [44]. The cost
of training such models is high: a version of Megatron having
one trillion (1T) parameters was recently trained over 84 days on
450 billion tokens using 3,072 NVIDIA A100 Graphics Processing
Unit (GPU) and executing more than 1,000 zettaFLOP (1 × 1021
floating-point operations) [29, 30]. This cost roughly equals seven
hundred years on a single GPU and over six million dollars (US)
assuming a single GPU at $1 per hour cloud-GPU rates. Incurring
such costs is commonplace; the PaLM-540B model recently trained
by Google used 2,572 zettaFLOP with similar numbers of Tensor
Processing Units (TPUs) and more than 8 million TPU hours [6].
Such high costs strongly motivate any combination of algorithmic,
software, or hardware redesign that can reduce them.
Consequently, there are proposed enhancements in algorithms
and software [29, 37, 38] and options for hardware acceleration [18,
31]. However, selecting a good configuration in practice has relied
primarily on heuristic reasoning [29]. While these proposals have
yielded impressive results, it has also been observed that distributed
training runs at a FLOP/s rate well below 50% of peak despite the
prevalence of matrix multiply (GEMM) operations [34].
The challenge is that the codesign landscape is quite large, mak-
ing it hard to reason about the impact of major changes to arbi-
trary combinations of hardware and software. For example, con-
sider that limited GPU memory capacity requires dividing a large
model among processors. Doing so can be achieved via model paral-
lelism, which combines two strategies known as tensor parallelism
and pipeline parallelism [29]. However, when using NVIDIA A100
GPUs, the size of the NVLink domain is 8, which can limit tensor
parallelism performance [32]. To compensate, one can increase the
degree of pipeline parallelism—but that may in turn produce other
inefficiencies such as reduced utilization due to pipeline bubbles
and needing to recompute intermediate features in light of memory
constraints [13, 29]. Alternatively, one might improve the computer
network to support larger tensor parallelism domains; companies
and researchers have indeed considered doing so [17, 32]. However,
if memory capacity is the root issue, then a more cost-effective

strategy is to increase capacity. Overall, this example shows that
codesign should carefully consider and delicately balance memory
capacity, memory bandwidth, processing throughput (i.e., FLOP/s),
network bandwidth, and network scalability, all of which interact
with choices made in software. Therefore, to reason about these
trade-offs we seek a principled analysis framework that can be
extended or adapted in a hardware and software landscape under-
going rapid and continual evolution.
We propose one such model-driven approach for high-level code-
sign of LLM training and inference systems. We first identify a
parameterized space of possible configurations that span major
system features and common algorithmic and software implemen-
tation strategies (Section 2). We then develop a unified analytical
model to estimate the end-to-end performance of LLM training as
a function of the configuration parameters. This model allows us to
pose, mathematically, a constrained optimization problem: find the
configuration that yields the best performance given fixed system
constraints such as memory capacity and system or network size.
We encode this performance model and model-optimizer in a
tool called Calculon. The parameter space includes the structure
and number of weights in the LLM model, the implementation
strategy, and a schematic description of the hardware system (Ta-
ble 1). Since Calculon’s model is analytical, it can calculate and
return a complete breakdown of projected training or inference
time quickly, typically in much less than a 1 ms per configuration.
It thus becomes possible to search an entire configuration space
having many millions of combinations in only a few minutes on a
standard desktop computer.
This paper includes several analyses we have conducted using
Calculon. The modeling formulae themselves are complex to write
out in full; therefore, to save space, we focus on the analyses as
“proof-of-concept,” with detailed formulas appearing in Calculon’s
open-source repository, where our formulas and assumptions ap-
pear in full, allowing replication of our results or modification of our
assumptions.1 The analyses showcase Calculon’s potential to facili-
tate high-level codesign, revealing several system and optimization
insights that may contradict conventional wisdom:
(1) None of the existing software-parallelism strategies is uniformly
the “best.” However, there is an optimal split-parallelism strat-
egy that balances system resources well, with the exact optimum
depending on system parameters, as we show.
(2) The speed of LLM training can be a sensitive function of system
size, with performance variability (ratio of highest to lowest
performer) exceeding 6×. These “efficiency cliffs” come from

difficulties mapping LLM structures to a given number of pro-
cessors when sizes do not “divide evenly.”
(3) Adding a second high-capacity tier of memory for tensor of-
floading reduces performance variability across various LLM
configurations and sizes, enabling efficient training of larger
models. Moreover, the bandwidth requirement for efficient of-
floading is within current technological capabilities.
While these findings are estimates, they suggest quantitatively what
performance improvements are possible, a critical first step for LLM
codesign. Our methodology is systematic and rigorous, enabling
future exploration via more detailed experiments and simulation.

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