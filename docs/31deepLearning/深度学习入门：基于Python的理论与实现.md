## 深度学习入门：基于Python的理论与实现

### chap1、Python入门

#### 1.5、NumPy

##### 1.5.3、

##### 1.5.5、广播

+ 形状不同的数组之间也可以进行运算。

##### 1.5.6、访问元素

#### 1.6、Matplotlib

##### 1.6.3、显示图像

### chap2、感知机（perceptron）

+ 1957年，Frank Rosenblatt
+ 严格来说，本章中的感知机，应该叫**人工神经元**

#### 2.1、感知机是什么

+ **感知机**接收多个输入信号，输出一个信号。
  + **信号**，只有2种取值（0和1）
  + 图中的圆圈叫**神经元**
  + 只有总和超过某个界限值时，才会输出1，也叫**神经元被激活**
  + 每个输入信号，对应一个**权重**（weight）

#### 2.2、简单逻辑电路

##### 2.2.1、与门（and gate）

+ `and`，and运算

##### 2.2.2、与非门和或门

+ `NAND gate`，Not AND
+ **真值表**

#### 2.3、感知机的实现

##### 2.3.1、简单的实现

##### 2.3.2、导入权重和偏置

##### 2.3.3、使用权重和偏置的实现

#### 2.4、感知机的局限性

##### 2.4.1、异或门

+ *为啥无法实现，自己要看一下，lionel*

##### 2.4.2、线性和非线性

+ **感知机的局限性就在于它只能表示由一条直线分割的空间。**
+ **非线性空间**：由曲线分割的空间
+ **线性空间**：由直线分割而成的空间

#### 2.5、多层感知机

+ **感知机的绝妙之处在于它可以“叠加层”**（通过叠加层来表示异或门）

##### 2.5.1、已有门电路的组合

+ 组合前面做好的与门、与非门、或门进行配置。

##### 2.5.2、异或门的实现

+ 图2-13，用感知机表示异或门

#### 2.6、从与非门到计算机

+ **多层感知机**可以实现比之前见到的电路更复杂的电路。

#### 2.7、小结

+ 感知机是神经网络的基础

### chap3、神经网络

+ 感知机中，**设定权重的工作**（是由人工进行的），即确定合适的、能符合预期的输入与输出的权重。
+ 神经网络（能解决这个问题），**可以自动季从数据中学习到合适的权重参数**。

#### 3.1、从感知机到神经网络

##### 3.1.1、神经网络的例子

+ 输入层、中间层、输出层
  + 中间层，有时也叫**隐藏层**，（隐藏层的神经元肉眼看不到）
+ 几层的描述
  + 有的是把**实质上拥有权重的层数**

##### 3.1.2、复习感知机

##### 3.1.3、激活函数登场

+ **激活函数**：`h(x)`将输入信号的总和转换为输出信号
+ $a=b+w_1x_1+w_2x_2$，然后$y=h(a)$，
+ **神经元**和**节点**等价
+ **激活函数**是连接感知机和神经网络的桥梁
  + 朴素感知机，指单层网络，指的是激活函数使用了**阶跃函数**的模型
  + 多层感知机，使用$sigmoid()$等平滑的激活函数的多层网络

#### 3.2、激活函数

+ **阶跃函数**：激活函数以阈值为界，一旦输入超过阈值，就切换输出。

##### 3.2.1、sigmoid函数

+ $h(x)=1/(1+e^{-x})$，e是纳皮尔常数2.7182...

##### 3.2.2、阶跃函数的实现

##### 3.2.3、阶跃函数的图形

3.2.4、sigmoid函数的实现

##### 3.2.5、sigmoid函数和阶跃函数的比较

+ 平滑性上
  + sigmoid函数是一条平滑的曲线，输出随着输入发生连续性的变化
  + 阶跃函数以0为界，输出发生急剧性的变化

3.2.6、非线性函数

+ **神经网络的激活函数必须使用非线性函数**，因为用线性函数的话，加深神经网络的层数就没有意义

##### 3.2.7、ReLU函数（Rectified Linear Unit）

+ `h(x) {x>0时，h(x)=x;  x<=0时，h(x)=0;}`

#### 3.3、多维数组的运算

##### 3.3.3、神经网络的内积

#### 3.4、3层神经网络的实现

+ *lionel，重点是这个*

##### 3.4.1、符号确认

3.4.2、各层间信号传递的实现

3.4.3、代码实现小结

#### 3.5、输出层的设计

+ 机器学习问题大致分为**回归**和**分类**问题
  + 回归：根据某个输入预测一个（连续的）数值问题，*lionel，简单说就是输出一个值的问题*，恒等函数
  + 分类：数据属于哪一个类别的问题，softmax函数

##### 3.5.1 恒等函数和softmax函数

##### 3.5.2、实现softmax函数时的注意事项

##### 3.5.3、softmax函数的特征

##### 3.5.4、输出层的神经元数量

#### 3.6、手写数字识别

+ 使用学习到的参数，**先实现神经网络的“推理处理”**，这个推理处理也称为神经网络的**前向传播**（forward propagation）
+ 使用神经网解决问题，
  + **训练**，使用训练数据（学习数据）进行权重参数的学习
  + **推理**，使用学习到的参数，对输入数据进行分类

##### 3.6.1、MNIST数据集

##### 3.6.2、神经网络的推理处理

##### 3.6.3、批处理

3.7、小结

+ 本章介绍了神经网络的前向传播。

  + [日拱一卒：深度学习笔记5——神经网络的前向传播](https://www.bilibili.com/read/cv15303804/)，*放在下文了，这就比较好的解释了什么叫前向传播*

  > 在神经网络中，当我们已知每个节点对应的权重与偏置这些系数后，就可以通过输入X，不断向前推进，最终计算出输出Y，这就是神经网络的前向传播。

### chap4、神经网络的学习

+ 这里所说的“学习”是指从训练数据中自动获取最优权重参数的过程。

#### 4.1、从数据中学习

+ 所谓“从数据中学习”，是指可以由数据自动决定权重参数的值。
+ *lionel，备注这个没看懂*

4.1.1 数据驱动

+ 考虑通过有效利用数据来解决这个问题（识别手写“5”）
  + 有3种办法【图4-2】
    + 1、人想到的算法
    + 2、人想到的特征量、机器学习
    + 3、神经网络（深度学习）
  + 方法1、先从图像中提取特征量，再用机器学习技术学习这些特征量的模式。
    + **特征量**：指可以从输入数据（输入图像）中准确地提取本质数据（重要的数据）的转换器。
    + 图像的特征量通常表示为向量的形式。
    + 在计算机视觉领域，常用的特征量包括SIFT、SURF和HOG等。
    + 使用这些特征量将图像数据转换为向量，然后对转换后的向量使用机器学习中的SVM、KNN等分类器进行学习。
  + 方法2、在神经网络中，连图像中包含的重要特征量也都是由机器来学习的

4.1.2 训练数据和测试数据

+ 为了正确评价模型的**泛化能力**，就必须划分训练数据和测试数据。
  + *lionel，泛化能力是啥？*
+ 训练数据也可以称为**监督数据**。
+ 只对某个数据集过度拟合的状态称为过拟合（over fitting），*可以顺利地处理某个数据集，但无法处理其他数据集的情况*

#### 4.2、损失函数

+ 神经网络以某个指标为线索寻找**最优权重参数**。
  + 这个指标称为损失函数（loss function）。

##### 4.2.1、均方误差

#### 4.3、数值微分

##### 4.3.1、导数

+ **导数**，某个瞬间的变化量（瞬时速度）
  + 10分钟跑2km，前1分钟奔跑的距离，前1秒钟，前0.1秒
+ **舍入误差**（rounding error），因省略小数的精细部分的数值，而造成最终的计算结果上的误差。
+ **中心差分**（**前向差分**）

##### 4.3.2、数值微分的例子

##### 4.3.3、偏导数

#### 4.4、梯度

+ **由全部变量的偏导数汇总而成的向量**称为**梯度**（gradient）

##### 4.4.1、梯度法

+ 最优参数是指损失函数取最小值时的参数。
+ 寻找最小值的梯度法称为**梯度下降法**（gradient descent method）
+ 寻找最大值的梯度法称为**梯度上升法**（gradient ascent method）

##### 4.4.2、神经网络的梯度

+ 这里的梯度是指**损失函数关于权重参数的梯度**。

#### 4.5、学习算法的实现

+ 神经网络学习分为4步：
  + 1、mini-batch
    + 从训练数据中随机选出一部分数据，这部分数据称为mini-batch。
    + 我们的目标是减小mini-batch的损失函数的值。
  + 2、计算梯度
    + 为了减小mini-batch的损失函数的值，需要求出各个权重参数的梯度。
    + **梯度**表示损失函数的值减小最多的方向。
  + 3、更新参数
    + 将权重参数沿梯度方向进行微小更新。
  + 4、重复1、2、3

##### 4.5.1、2层神经网络的类

##### 4.5.2、mini-batch的实现

4.5.3、基于测试数据的评价

4.6、小结

### chap5、误差反向传播法

+ 数值微分虽然简单且易实现，**但计算上比较费时间**，所以引入这个
+ 正确理解误差反向传播法，有2种方法
  + 一种是基于数学式
  + 另一种是基于**计算图**（computational graph）

#### 5.1、计算图

+ **计算图**将计算过程用图形表示出来。

##### 5.1.1、用计算图求解

##### 5.1.2、局部计算

##### 5.1.3、为何用计算图解题

+ 2个优点：
  + 一个是**局部计算**
  + 另一个是**利用计算图，可以将中间的计算结果全部保存起来**
  + **使用计算图最大的原因是，可以通过反向传播高效计算导数**

#### 5.2、链式法则

##### 5.2.1、计算图的反向传播

+ 沿着与正方向相反的方向，乘上局部导数
  + **局部层数**是指向正向传播中$y=f(x)$的导数，也就是y关于x的导数

5.2.2、什么是链式法则

5.2.3、链式法则和计算图

#### 5.3、反向传播

##### 5.3.1、加法节点的反向传播

##### 5.3.2、乘法节点的反向传播

+ 求导
+ 会将上游的值乘以正向传播时的输入信号的“翻转值”后传递给下游。
  + **翻转值**是一种翻转关系

##### 5.3.3、苹果的例子

5.4、简单层的实现

##### 5.4.1、乘法层的实现

##### 5.4.2、加法层的实现

5.5、激活函数层的实现

##### 5.5.1、ReLU层

5.5.2、Sigmoid层

5.6、Affine/Softmax层的实现

5.6.1、Affine层

5.6.2、批版本的Affine层

##### 5.6.3、Softmax-with-Loss层

#### 5.7、误差反向传播法的实现

5.7.1、神经网络学习的全貌图

5.7.2、对应误差反向传播法的神经网络的实现

5.7.3、误差反向传播法的梯度确认

##### 5.7.4、使用误差反向传播法的学习

5.8、小结

### chap6、与学习相关的技巧

+ 神经网络的学习中的一些重要观点，主题涉及**寻找最优权重参数的最优化方法，权重参数的初始值，超参数的设定方法**
+ 为了应对**过拟合**，还将介绍**权值衰减，Dropout等正则化方法**
+ 最后介绍了一下**Batch Normalization**

#### 6.1、参数的更新

+ **神经网络的学习的目的**是找到**使损失函数的值尽可能小的参数**，这种寻找最优参数的问题，解决这个问题的过程称为**最优化（optimization）**
+ 前几章，为了找到最优参数，将**参数的梯度（导数）**作为了线索，沿梯度方向更新参数，这个过程称为**随机梯度下降法（stochastic gradient descent）SGD**

##### 6.1.1、探险家的故事

+ 至深之地，**地面的坡度**，朝着当前所在位置的坡度最大的方向前进，这就是**SGD的策略**

##### 6.1.2、SGD

+ *没懂这个*

##### 6.1.3、SGD的缺点

+ SGD低效的根本原因是，**梯度的方向并没有指向最小值的方向**。

##### 6.1.4、Momentum

+ **动量**的意思，让人感觉**小球在斜面上滚动**

##### 6.1.5、AdaGrad

+ **学习率衰减（learning rate decay）**

##### 6.1.6、Adm

##### 6.1.7、使用哪种更新方法呢

+ 代码在ch06/optimizer_compare_naive.py中

##### 6.1.8、基于MNIST数据集的更新方法的比较

#### 6.2、权重的初始值

##### 6.2.1、可以将权重初始值设为0吗

+ **权值衰减**就是一种**以减小权重参数的值为目的进行学习的方法**
+ 如果想减小权重的值，一开始就将初始值设为较小的值才是正途。
+ 不能设置为0，不能将权重初始值设成一样，**因为在误差反向传播法中，所有的权重值都会进行相同的更新**。

##### 6.2.2、隐藏层的激活值的分布

+ 这里的**激活值**是指**激活函数的输出数据**。
+ **梯度消失**（gradient vanishing）

##### 6.2.3、ReLU的权重初始值

##### 6.2.4、基于MNIST数据集的权重初始化

#### 6.3、Batch Normalization

##### 6.3.1、Batch Normalization的算法

+ 优点
  + 使学习快速进行（增大学习率）
  + 不那么依赖初始值（对于初始值不用那么神经质）
  + 抑制过拟合（降低Dropout等的必要性）
+ Batch Norm，**以进行学习时的mini-batch为单位，按mini-batch进行正规化**，具体而言，**就是进行使数据分布的均值为0，方差为1的正规化**。

##### 6.3.2、Batch Normalization的评估

#### 6.4、正则化

##### 6.4.1、过拟合

+ 发生过拟合的原因，主要有以下2个：
  + 模型拥有大量参数，表现力强
  + 训练数据少

##### 6.4.2、权值衰减（weight decay）

+ **通过在学习的过程中对大的权重进行惩罚**

##### 6.4.3、Dropout

+ **在学习过程中随机删除神经元的方法**

#### 6.5、超参数的验证

6.5.1、验证数据

6.5.2、超参数的最优化

6.5.3、超参数最优化的实现

#### 6.6、小结

### chap7、卷积神经网络

+ Convolutional Neural Network，用于图像识别、语音识别

#### 7.1、整体结构

+ 如何组装层以构建CNN
+ 神经网络中，相邻层的所有神经元之间都有连接，称为**全连接**（fully-connected）
+ 我们用Affine层实现了全连接层
  + 5层的话（前4层是Affine+ReLu，最后1层是Affine+Softmax）
  + CNN的话，就会增加池化层

#### 7.2、卷积层

+ 一些术语**填充，步幅**

##### 7.2.1、全连接层存在的问题

+ **数据的形状被“忽视”了**，输入时图像通常是**高、长、通道方向上3维形态**，向全连接层输入时，需要将3维数据拉平为1维数据。
+ **卷积层可以保持形状不变**，有时将卷积层的输入输出数据称为**特征图**（feature map）

##### 7.2.2、卷积运算

+ 相当于图像处理中的**滤波器运算**，有时也用**核**

##### 7.2.3、填充

+ 在进行卷积层处理之前，有时要**向输入数据的周围填入固定的数据（比如0等）**，这称为**填充（padding）**。

##### 7.2.4、步幅

+ 应用滤波器的位置间隔称为**步幅**（stride）

7.2.5、3维数据的卷积运算

7.2.6、结合方块思考

7.2.7、批处理

#### 7.3、池化层

+ **池化**是缩小高、长方向上的空间的运算
+ 池化层特征
  + 1、没有要学习的参数
  + 2、通道数不发生变化
  + 3、对微小的位置变化具有鲁棒性

7.4、卷积层和池化层的实现

7.4.1、4维数组

7.5、CNN的实现

7.6、CNN的可视化

7.7、具有代表性的CNN

7.8、小结

### chap8、深度学习

+ 深度学习是**加深了层的深度神经网络**
+ 基于之前介绍的网络，只需通过**叠加层**，就可以创建深度网络

#### 8.1、深度网络

8.1.1、向更深的网络出发

8.1.2、进一步提高识别精度

##### 8.1.3、加深层的动机

+ 加深层的好处
  + 一、减少网络的参数数量
  + 二、使学习更加高效

#### 8.2、深度学习的小历史

+ 2012年ILSVRC，基于深度学习的方法（AlexNet）以压倒性的优势胜出

8.2.1、ImageNet

8.2.2、VGG

8.2.3、GoogleNet

8.2.4、ResNet

8.3、深度学习的高速化

8.4、深度学习的应用案例

#### 8.5、深度学习的未来

8.5.1、

8.5.2、图像的生成

8.5.3、自动驾驶

+ 基于CNN的神经网络SegNet

8.5.4、Deep Q-Network（强化学习）