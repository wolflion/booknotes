## 《剑指offer》

+ 全书共81道，计划从0725开始刷，leetcode上需要31天

+ https://leetcode.cn/study-plan/lcof/
+ 刷题分为3步:（这样才能确定下问题出在哪一步）
  + 1、读懂题目
  + 2、分析，推导，**逻辑上这题怎么解**
  + 3、把思路转化为代码
+ 20230725-发现自己的一个问题，**没有耐心一步步的往下推下去**，哪怕是简单的排序算法，**要先想着怎么去暴力**，也就是**要先做出来，然后再优化**，自己第一步都不好好整，这是个问题，*要逐步fixed掉这个问题*
  + **如何才算刷透《剑指offer》**-这个我还没有答案
  + 遇到瓶颈或疑问时，要问问**《剑指offer》怎么刷**
+ 重要的参考：
  + [《剑指offer》所有题目解法思路与出现频率汇总](https://blog.csdn.net/weixin_43954951/article/details/125821879)，*在这个里面找思路，便于自己记一下*

### 一、栈与队列

+ d1：9，30
+ d：59I、59II
+ 0531开始刷了，只做了2道开头，0725开始

#### 09、用2个栈实现队列

+ 我不太有思路
+ 思路：
  + 插入的时候，在stack1插即可
  + 删除的时候，得用到stack2，如果stack2为空，得把stack1入到stack2里，然后再再删，如果stack1也为空，就是异常

##### 代码

```cpp
class CQueue {
public:
    CQueue() {

    }
    
    void appendTail(int value) {

    }
    
    int deleteHead() {

    }
};

/**
 * Your CQueue object will be instantiated and called as such:
 * CQueue* obj = new CQueue();
 * obj->appendTail(value);
 * int param_2 = obj->deleteHead();
 */
```



#### 30、包含min函数的栈

+ 知道要有个最小值
+ 思路：
  + 辅助栈，存最小值（为啥需要m_min、m_data栈？）

##### Code

```cpp
class MinStack {
public:
    /** initialize your data structure here. */
    MinStack() {

    }
    
    void push(int x) {

    }
    
    void pop() {

    }
    
    int top() {

    }
    
    int min() {

    }
};

/**
 * Your MinStack object will be instantiated and called as such:
 * MinStack* obj = new MinStack();
 * obj->push(x);
 * obj->pop();
 * int param_3 = obj->top();
 * int param_4 = obj->min();
 */
```

#### 59、

### 二、链表

+ d2：6，24，35
+ 

#### 6、从尾到头打印链接

#### 24、反转链表

#### 35（M）、复杂链表的复制

### 三、字符串

+ d3：5，58II
+ d：20，67

#### 5、替换空格

#### 58II、左旋转字符串

#### 20（M）、

#### 67（M）、

### 四、查找

+ d4：3，53I，53II
+ d5：4，11，50

#### 3、数组中重复的数字

#### 53I、在排序数组中查找数字

#### 53II、0~n-1中缺失的数字

#### 4（M）、二维数组中的查找

#### 11、旋转数组的最小数字

#### 50、第一个只出现一次的字符

### 五、二叉树

+ d6：32I，32II，32III

#### 32I（M）、从上到下打印二叉树

32II

32III（M）、

### 六、搜索与回溯算法

+ d7：26，27，28
+ d14：12，13
+ d15：34，36，54

26（M）、树的子结构

27、二叉树的镜像

28、对称的二叉树

### 七、动态规划

+ d8：10I，10II，63
+ d9：42、47
+ d10：46，48

42、连续子数组的最大和

47、礼物的最大价值

46、把数字翻译成字符串

48、最长不含重复字符的字符串

### 八、双指针

+ d11：18，22
+ d12：25，52
+ d13：21，57，58I

#### 18、删除链表的节点

#### 22、链表中倒数第k个节点

25、

52、

21、

57、

58I、

### 最后

#### 自己心得

+ 如果写不出来，就干2件事
  + 1、用更抽象的语言去写，比如C++的string或一些算法，**自己有思路就行**
  + 2、如果思路都没有，就先**举例和分解**，先做对几种场景，或者用暴力的方法先解出后，再**想办法去优化**（因为分解后，就能慢慢找到具体优化哪一步了），**针对具体的地方优化**

#### ref（这些都不一定用得上）

+ [面试必刷-《剑指offer》刷题小结](https://zhuanlan.zhihu.com/p/56200260)

+ https://www.algomooc.com/ ， https://github.com/MisterBooo， 都是 吴师兄学算法
+ [《剑指offer》学习心得](https://doc.yonyoucloud.com/doc/wiki/project/for-offer/index.html)，人家的笔记，不过是用java写的，**胜在人家有ut**
+ https://github.com/Wang-Jun-Chao/coding-interviews  也是java的，解题可以去看他的csdn，blog
+ [剑指offer leetcode怎么刷](https://www.zhihu.com/question/271458173)  这是zhihu上的答案，自己有疑问时可以查查看