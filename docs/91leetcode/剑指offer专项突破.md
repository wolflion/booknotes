## 《剑指offer专项突破版》

+ [滑动窗口详解](https://zhuanlan.zhihu.com/p/422908736)
  + https://leetcode.cn/circle/article/9gcJBk/
+ chap8树，chap9堆，chap10前缀树、不熟悉（前2个还算看过，第3个根本不太了解）
+ https://github.com/doocs/leetcode/blob/main/lcof2/README.md
+ https://leetcode.cn/studyplan/coding-interviews-special/

### 0

#### 在其它地方刷过的题

+ 5、单词长度的最大乘积

### chap1、整数（1-5）

#### 1.1、基础知识

##### 1、整数除法

#### 1.2、二进制

##### 2、二进制加法

##### 3、前 n 个数字二进制中 1 的个数

##### 4、[只出现一次的数字](https://leetcode.cn/problems/WGki4K/?envType=study-plan&id=lcof-ii&plan=lcof&plan_progress=x95pysw2)

+ [137](https://leetcode-cn.com/problems/single-number-ii/)
+ 自己想到了hash，*但hash的用法，不太熟*
+ 参考链接：**位运算**的思路，
  + https://blog.csdn.net/qq_32523711/article/details/108041437
  + https://www.jb51.net/article/217687.htm

```cpp
//给你一个整数数组 nums ，除某个元素仅出现一次外，其余每个元素都恰出现三次。请你找出并返回那个只出现了一次的元素
//nums = [2,2,3,2]，返回3
int singleNumber(vector<int>& nums) {
    map<int,int> val;
    for(auto i:nums){
        val[i]++;
    }

    for(auto c: nums){
        if(val[c] == 1){
           return c;
        }
    }
    return 0;
}
```



##### 5、单词长度的最大乘积

+ *在其它地方刷过，但没用作者给的2个思路，用的是自己的*

+ 字符串数组words，保存的是"abcw","foo"，“bar”,"fxyz","abcdf"，因为"bar"和"foo"没有相同的字符，所以乘积是`3*3=9`，"abcw"和"fxyz"也没有相同的字符，乘积是`4*4=16`，所以最大值是16。
  + 自己思路是：两个for循环，找到没有相同的字符串
  + 作者的给的思路1：**用哈希表记录字符串中出现的字符**。
  + 作者的给的思路2：

### chap2、数组（6-13）

#### 2.1、基础知识

#### 2.2、双指针

+ 方向相反的双指针，用来求排序数组中的两个数字之和
+ 方向相同的双指针，用来求正数数组中子数组的和或乘积

##### 6、排序数组中的两个数字之和

+ 递增排序的数组和一个值k，找到两个和为k的数字并返回它们的下标？
  + `[1,2,4,6,10]`，k的值为8
  + 思路：`left=0,right=array.size()`，right>targe，right--; left+right<target，left++，直到它们相等

##### 7、[数组中和为 0 的三个数](https://leetcode.cn/problems/1fGaJU/?envType=study-plan&id=lcof-ii&plan=lcof&plan_progress=x95pysw2)，undo

+ [15]([leetcode.cn/problems/3sum/](https://leetcode.cn/problems/3sum/))
  + **快排+对撞指针**，2个我都不太熟悉（*为啥要排序*，之前觉得排序打乱输出位置了，其实最后并不会影响）
  + https://blog.csdn.net/weixin_44294809/article/details/106097652
  + https://blog.csdn.net/Innocence02/article/details/127427598
  + https://blog.csdn.net/weixin_45629285/article/details/117060550 【详细题解】
  + https://www.bilibili.com/video/BV1i3411R71x/?vd_source=66d6345790c729dfa0a136a5183762c9

```cpp
//给你一个整数数组 nums ，判断是否存在三元组 [nums[i], nums[j], nums[k]] 满足 i != j、i != k 且 j != k ，同时还满足 nums[i] + nums[j] + nums[k] == 0 。请你返回所有和为 0 且不重复的三元组。
//输入：nums = [-1,0,1,2,-1,-4]
//输出：[[-1,-1,2],[-1,0,1]]
//lionel，先尝试看看能不能看懂，或真正理解了
    vector<vector<int>> threeSum(vector<int>& nums) {
        vector<vector<int> > res;
        if(nums.size()<3)//边界条件
            return res;
        
        sort(nums.begin(),nums.end());//对数组排序
        
        for(int k=0;k<nums.size()-2;k++)
        {
            //当nums[k]>0时直接跳出,因为nums[j]>=nums[i]>=nums[k]>0,三数和必大于零
            if(nums[k]>0)
                break;
            
            //当k>0且nums[k]==nums[k-1]时跳过nums[k]：因为已经将nums[k-1]的所有组合加入到结果中，
            //本次搜索只会得到重复组合。
            if(k>0 && nums[k]==nums[k-1])
                continue;
            
            int i=k+1,j=nums.size()-1;//双指针
            while(i<j)
            {
                int sum=nums[k]+nums[i]+nums[j];
                vector<int> temp;
                if(sum==0)
                {
                    temp.push_back(nums[k]);
                    temp.push_back(nums[i]);
                    temp.push_back(nums[j]);
                    res.push_back(temp);//将满足条件的组合存入结果
                    while(i<j && nums[i]==nums[++i]) ;//跳过重复的nums[i]
                    while(i<j && nums[j]==nums[--j]) ;//跳过重复的nums[j]
                }
                else if(sum<0)
                {
                    while(i<j && nums[i]==nums[++i]) ;//跳过重复的nums[i]
                }
                else  //sum>0
                {
                    while(i<j && nums[j]==nums[--j]) ;//跳过重复的nums[j]
                }
            }  
        }
        
        return res;
    }
```



##### 8、[和大于等于 target 的最短子数组](https://leetcode.cn/problems/2VG8Kg/)

+ [209](https://leetcode-cn.com/problems/minimum-size-subarray-sum/)
  + 看到视频讲解的一道题

```cpp
int minSubArrayLen(int target, vector<int>& nums){
        //时间复杂度O(n)
        //空间复杂度O(1)
        int l=0,r=-1;//nums[l...r]为我们的滑动窗口
        int sum = 0;
        int res=nums.size()+1;

        while(l<nums.size()){
            if(r+1<nums.size()&&sum<target){
                r++;
                sum+=nums[r]; //右边界拓展
            } else{
                //sum-=nums[l++];//先减去，然后再++
                sum-=nums[l];
                l++;
            }

            if(sum>=target){
                res = min(res,r-l+1);
            }
        }
        if(res==nums.size()+1)
            return 0;
        return res;
    }
```





##### 9、乘积小于 K 的子数组

+ `[10,5,2,6]`，k的值是100，有8个子数组的所有数字的乘积小于100

  + p1和p2两个指针之间的数字组成一个子数组。

  + 先移动p2，如果不超过，再移过p2，直到超过100，再移动p1

  + ```cpp
    int numSubarrayProduceLessThanK(vector<int> nums, int k){
        long produce = 1;
        int left=0,right=0;
        int count=0;
        for(right=0;i<nums.size();right++){
            produce *= nums[right];
            while(left<right && produce >= k){
                produce /= nums[left++];  //lionel，这个实现有点6
            }
            count += right>=left?right-left+1:0;  //lionel，确实就算知道思路了，还是有很多种实现方法
        }
        return count;
    }
    ```

  + 

#### 2.3、累加数组数字求子数组之和

10、和为 k 的子数组

11、0 和 1 个数相同的子数组

12、左右两边子数组的和相等

13、二维子矩阵的和

#### 2.4、小结

### chap3、字符串（14-20）

#### 3.1、基础知识

+ C++的string类（*lionel，书中讲的是java*）

#### 3.2、双指针

+ 把字符串看成一个由字符组成的数组，那么也可以用两个指针来定位一个子字符串。
+ **使用哈希表来存储每个元素出现的次数**，解决问题时**同时使用 双指针 + 哈希表**（2个配合使用）

##### 14、字符串中的变位词

##### 15、字符串中的所有变位词

##### 16、不含重复字符的最长子字符串

##### 17、包含所有字符的最短字符串

+ s为"ADDBANCAD"，t为“ABC”，找出s中包含t的所有字符的最短子字符串。输出是**BANC**
  + 作者思路：
    + **哈希表统计字符串每个字符出现的次数**，先扫描t，统计每个字符的个数；再扫描s，每扫描一个字符，就检查哈希表中是否存在，不存在就忽略，存在hash表的值就减1。
  + *java中的HashMap类型对应C++的是哪种？*

#### 3.3、回文字符串

##### 18、[有效的回文](https://leetcode.cn/problems/XltzEq/)

+ [125]( https://leetcode-cn.com/problems/valid-palindrome/)
  + 知道**用指针了**，但啥时候**该转大小写，数字判断又迷糊了**
  + https://www.jb51.net/article/217081.htm
  + https://blog.csdn.net/qq_40416052/article/details/82504029

```cpp
//给定一个字符串 s ，验证 s 是否是 回文串 ，只考虑字母和数字字符，可以忽略字母的大小写。
//输入: s = "A man, a plan, a canal: Panama"
//输出: true
bool isPalindrome(string s) {
        int left = 0, right = s.size() - 1 ;
        while (left < right) {
            if (!isalnum(s[left])) ++left;
            else if (!isalnum(s[right])) --right;
            else if ((s[left] + 32 - 'a') %32 != (s[right] + 32 - 'a') % 32) return false;
            else {
                ++left; --right;
            }
        }
        return true;
    }
```



##### 19、[最多删除一个字符得到回文](https://leetcode.cn/problems/RQku0D/?envType=study-plan&id=lcof-ii&plan=lcof&plan_progress=x95pysw2)

+ [680](https://leetcode-cn.com/problems/valid-palindrome-ii/)
  + https://www.cnblogs.com/zzxcm/p/15864818.html *这个写得复杂了，我就没参考，但确实符合我第一次看到题的思路*
  + https://zxi.mytechroad.com/blog/string/leetcode-680-valid-palindrome-ii/  【这个答案有点儿意思】，我用了这个
  + https://github.com/keineahnung2345/leetcode-cpp-practices/blob/master/680.%20Valid%20Palindrome%20II.cpp *感觉都是这么写的*

```cpp
//给定一个非空字符串 s，请判断如果 最多 从字符串中删除一个字符能否得到一个回文字符串。
//输入: s = "aba"
//输出: true
class Solution {
public:
    bool validPalindrome(const string& s) {
        int l = -1;
        int r = s.length();
        while (++l < --r)
            if (s[l] != s[r])
                return isPalindrome(s, l+1, r) 
                    || isPalindrome(s, l, r-1);
        return true;
    }
private:
    bool isPalindrome(const string& s, int l, int r) {
        while (l < r)
            if (s[l++] != s[r--]) return false;
        return true;
    }
};
```



#####  20、回文子字符串的个数

+ 字符串"abc"有3个回文字符串（a,b,c），字符串"aaa"有6个回文串（a,a,a,aa,aa,aaa）。

#### 3.1、小结

+ **熟练掌握字符串常用操作**是解题的前提
+ **变位词与回文**是考查的重点
  + **变位词**，两个字符串包含的字符及每个字符出现的次数都相同，只是字符出现的顺序不同，**用哈希表统计每个字符出现的次数**
  + **回文**，从前往后还是从后往前读取其每一个字符，得到的内容都是一样的。**双指针**（两端往中间移，中间往两端移）两种思路

### chap4、链表（21-29）

#### 题目

21、删除链表的倒数第 n 个结点

##### 22、链表中环的入口节点

23、两个链表的第一个重合节点

##### 24、反转链表

25、链表中的两数相加

26、重排链表

27、回文链表

28、展平多级双向链表

29、排序的循环链表

### chap5、哈希（30-35）

#### 题目

插入、删除和随机访问都是 O(1) 的容器

最近最少使用缓存

##### 32、[有效的变位词](https://leetcode.cn/problems/dKk3P7/?envType=study-plan&id=lcof-ii&plan=lcof&plan_progress=x95pysw2)

+ [242](https://leetcode-cn.com/problems/valid-anagram/)
  + 虽然我解出来了，但官方的答案比我的优秀

```cpp
//给定两个字符串 s 和 t ，编写一个函数来判断它们是不是一组变位词（字母异位词）。
//输入: s = "anagram", t = "nagaram"
//输出: true
bool isAnagram(string s, string t) {
        if(s.size()!=t.size()) return false;   //1、为啥用size()而不用length()，区别在哪？
        if(s==t) return false;          //2、两个判断相等，其实可以直接判断，不用再逐个字符判断
        vector<int> vt1(26,0),vt2(26,0);  //3、用vector去存的，而并没有用所谓的hash里的一些map的用法，【26的取值，以及-'a'的操作，都值得学一下】
        for(int i=0;i<s.size();i++){
            vt1[s[i]-'a']++;
            vt2[t[i]-'a']++;
        }
        return vt1==vt2;
    }
```



变位词组

外星语言是否排序

最小时间差

### chap6、栈

#### 6.1、基础知识

#### 6.2、栈的应用

+ 单调栈（**保存在栈中的数据是排列的**）

##### 36、后缀表达式

##### 37、小行星碰撞

##### 38、每日温度

##### 39、直方图最大矩形面积

##### 40、矩阵中最大的矩形

#### 6.3、小结

### chap7、队列（41-46）

#### 7.1、基础知识

#### 7.2、队列的应用

##### 41、滑动窗口的平均值

##### 42、最近请求次数

##### 43、在完全二叉树中添加节点

##### 44、二叉树每层的最大值

##### 45、二叉树最底层最左边的值

##### 46、二叉树的右侧视图

#### 7.3、小结

### chap8、树

#### 1、基础知识

#### 2、二叉树的深度优先搜索

二叉树剪枝

序列化与反序列化二叉树

从根节点到叶节点的路径数字之和

向下的路径节点之和

节点之和最大的路径

#### 3、二叉搜索树

展平二叉搜索树

二叉搜索树中的中序后继

所有大于等于节点的值之和

 二叉搜索树迭代器

二叉搜索树中两个节点之和

值和下标之差都在给定的范围内

#### 4、TreeSet和TreeMap的应用

日程表

### chap9、堆

#### 1、基础知识

#### 2、堆的应用

数据流的第 K 大数值

出现频率最高的 k 个数字

和最小的 k 个数对

### chap10、前缀树（62-）

#### 10.1、基础知识

+ 前缀树（**字典树**），用一个树状结构存储一个字典中的所有单词。
  + can,cat,come,do,i,in,inn保存的样子（图10.1）
    + 第一层是c,第二层是a，**找共性，找不到，再另起一个**，比如同第一行就是d和i
+ **前缀树**是一个**多叉树**，一个节点可能有多个子节点。**前缀树的根节点不表示任何字符**。
+ 字符串在前缀树中的路径并不一定终止于叶节点。（一个单词是另一个单词的前缀）
+ 如果前缀树路径到达某个节点时它表示了一个完整的字符串，那么**字符串最后一个字符对应的节点有特殊的标识**。

##### 62、实现前缀树

+ 有3个功能
  + insert
  + search，查找字符串
  + startWith，查找字符串前缀

#### 10.2、前缀树的应用

##### 63、替换单词

##### 64、神奇的字典

##### 65、最短的单词编码

##### 66、单词之和

##### 67、最大的异或

#### 10.3、小结

+ 前缀树通常用来保存字符串，它的节点和字符串的字符对应，而路径与字符串对应。**为了标注某些节点和字符串的最后一个字符对应，前缀树节点中通常需要一个布尔类型的字段**。

+ 前缀树经常用来解决与字符串查找相关的问题。和哈希表相比，在前缀树中查找更灵活。既可以从哈希表中找出所有以某个前缀开头的所有单词，也可以找出修改了一个（或多个）字符的字符串。
+ 使用前缀树解决问题通常需要2步：
  + 1、创建前缀树
  + 2、在前缀树中查找

### chap11、二分查找

#### 1、基础知识

#### 2、在排序数组中二分查找

查找插入位置

山峰数组的顶部

排序数组中只出现一次的数字

按权重生成随机数

求平方根

狒狒吃香蕉

#### 3、在数值范围内二分查找

### chap12、排序

#### 1、基础知识

#### 2、计数排序

 合并区间

数组相对排序

#### 快速排序

数组中的第 k 大的数字

#### 归并排序

链表排序

合并排序链表

### chap13、回溯法（79-87）

#### 13.1、基础知识

+ 解决问题时的每一步都尝试所有可能的选项，最终找到所有可行的解决方案。非常适合**解决由多个步骤组成的问题，并且每个步骤都有多个选项**。
+ 求解`[1,2]`的所有子集
  + **子集的初始状态为空**
  + 分2步：是否在子集中添加1，是否在子集中添加2
+ 采用回溯法解决问题的过程实质上是在树形结构中从根节点开始进行深度优先遍历。
+ 如果在前往某个节点时对问题的解的状态进行了修改，那么在回溯到它的父节点时要记得清除对应的修改

#### 13.2、集合的组合、排列

+ 一个子集又可以称为一个组合

##### 79、所有子集

+ 输入不含重复数字的数据集合，找出它的所有子集。`[1,2]`的所有子集

  + **集合中包含n个元素，那么生成子集就可以分成n步**，每步从集合中取一个数字，有2个选项，加入或不加入

  + ```cpp
    //nums是集合，包含了所有元素；index是下标，当前的数字的下标；subset是当前子集；result是已经生成的子集
    void helper(vector<int> nums, int index, vector<int>subset, vector<vector<int>> result){
        if(index == nums.size()){
            result.push(subset);  //lionel，这个对不对 【加入的只是subset的拷贝】
        }else if(index<nums.size()){
            helper(nums,index+1,subset,resut);  //不加入，直接调用递归函数处理下一个（index+1）即可
        }
        
        subset.push(nums[index]); //把当前数字加入到当前子集
        helper(nums,index+1,subset,result);
        subset.remove();//递归完成后，把加入的再退出
    }
    ```

  + 

##### 80、含有 k 个元素的组合

##### 81、允许重复选择元素的组合

+ *lionel，这个要看一下，主要条件不再是index了*

#####  82、包含重复元素集合的组合

+ *lionel，没看*

#####  83、没有重复元素集合的全排列

+ 给定一个没有重复数字的集合，找出它的所有全排列。`[1,2,3]`有6个全排列
  + *lionel，没看*，用的是递归？

##### 84、包含重复元素集合的全排列

+ 给定一个包含重复数字的集合，找出它的所有全排列。`[1,1,2]`有3个全排列
  + *lionel，不熟悉*

#### 13.3、使用回溯法解决其他类型的问题

##### 85、生成匹配的括号

+ *lionel，题都没看*

##### 86、分割回文字符串

+ “google”就有3种分割办法是回文（goog,l,e）（g,o,o,g,l,e）（g,oo,g,l,e）
  + *lionel，没看*

##### 87、复原IP

+ 字符串“10203040”，可以恢复3个（10.20.30.40）（102.0.30.40）（10.203.0.40）

  + 字符串长度为n，就分成n步，每步有2个选项

    + 1，将当前字符拼到当前分段数字的末尾，拼接后在0-255之间
    + 2、将当前字符作为一个新的分段字符，但分段不能超过4，开始分段时前一个不能为空

  + ```cpp
    //s是源串，i是字符串的下标，segI是当前分段数字的下标（取值0-3），seg当前已经恢复的一个分段数字，ip就是恢复的IP
    void helper(string s, int i, int segI, string seg, string ip, vector<string> result){
        if(i==s.size() && sgeI==3 && isValidSeg(seg)){
            result.add(ip+seg);
        }else if(i<s.size() && segI <=3){
            char ch = s.at(i);
            if(isValidSeg(seg+ch)){
                helper(s,i+1,segI,seg+ch,ip,result);  //加入到原先的segI，同时下标加1
            }
            
            //2个if，也就是2种选项，都在else if()中，也正常，表明不符合条件add的，有2种选择
            if(seg.length()>0 && setI<3){
                helper(s,i+1,segI+1,""+ch,ip+seg+".",result);//index和segI都加1，但set和ip，都变了不同的值【后面的要想一下】
            }
        }
    }
    
    bool isValidSeg(string seg){
        return stoi(seg)<=255 && (seg.contains("0") || seg.at(0)!='0'); //lionel, seg.equals("0")是啥意思？
    }
    ```

  + 

#### 13.4、本章小结

### chap14、动态规划（88-104）

#### 14.1、基础知识

+ 求一个问题的最优解（求最大值或最小值），比如列举出整数集合`[2,3,5]`，元素之和等于8的组合有几个
+ 采用动态规划时总是用**递归的思路**分析问题，找出描述大问题的解和小问题的解之间递归关系的状态转移方程
+ **小问题之间存在重叠的部分**，是可以运用动态规划求解问题的另一个显著特色
+ 用代码实现动态规划算法时
  + 1、采用递归的代码按照从上往下的顺序求解，每求出一个小问题的解就缓存下来
  + 2、从下往上的顺序，从解决最小的问题开始，并把已经解决的小问题的解存储下来（大部分存储在一维或二维数组中），再把小问题的解组合起来

##### 88、爬楼梯的最小成本

+ 有4种解法（*lionel，要搞会*）

#### 14.2、单序列问题

+ 每一步在序列中增加一个元素，根据题目的特点找出该元素对应的最优解（或解的数目）和前面若干元素（通常是一个或两个）的最优解（或解的数目）的关系，并以此找出相应的状态转移方程

##### 89、房屋被盗

+ 在

  + ```cpp
    //维护一个dp，传入参数，这是一种做法
    void helper(vector<int> nums, int index, vector<int> dp){
        if(index == 0){
            dp[index] == nums[0];
        }
        else if (index == 1){
            dp[index]=max(nums[0],nums[1]);
        }else if(dp[index]<0){
            helper(nums,index-2,dp);
            helper(nums,index-1,dp);
            dp[index]=max(dp[index-1],dp[index-2]+nums[index]);
        }
    }
    
    //还有一种做法，不用helper()函数，直接用 for循环
    //先算出dp[0]
    dp[0]=nums[0];
    //再求出dp[1]
    if(nums.size()>1){
        dp[1]=max(nums[0],nums[1]);
    }
    
    for(int i=2;i<nums.size();i++){  //lionel，这个地方从2开始
        dp[i]=max(dp[i-1],dp[i-2]+nums[i]);
    }
    
    return dp[nums.length-1];
    ```

  + 

##### 90、环形房屋偷盗

91、粉刷房子

92、翻转字符

93、最长斐波那契数列

94、最少回文分割

#### 14.3、双序列问题

+ 输入是两个字符串或数组，那么状态转移方程通常有两个参数，`f(i,j)`，即第一个序列的（0,i）与第二个序列的（0,j）

95、最长公共子序列

96、字符串交织

97、子序列的数目

#### 14.4、矩阵路径问题

+ 一定要将`f(i,j)`的计算结果用一个二维数组缓存，以避免不必要的重复计算。

98、路径的数目

99、最小路径之和

100、三角形中最小路径之和

#### 14.5、背包问题

+ 有界背包，每种物品的数量有限
+ 无界背包，每种物品的数量无限（完全背包）

##### 101、分割等和子集

##### 102、加减的目标值

##### 103、最少的硬币数目

104、排列的数目

#### 14.6、小结

+ 找最优解
+ 推导状态转移方程，是**递归表达式**，直接用递归呢，存在大量重复计算，**将计算结果进行缓存**。
+ 递归的代码按照自上而下的顺序解决，迭代按照自下而上的顺序解决问题

### chap15、图（105-119）

#### 15.1、基础知识

#### 15.2、图的搜索

#### 15.3、拓扑排序

#### 15.4、并查集

##### 116、朋友圈

##### 119、最长连续序列

+ 题目：无序的整数组[10,5,9,2,4,3]，最长连续数值序列是[2,3,4,5]，因此输出4。

#### 15.5、小结



https://zxi.mytechroad.com/blog/leetcode-problem-categories/  题目列表