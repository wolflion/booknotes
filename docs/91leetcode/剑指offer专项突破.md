## 《剑指offer专项突破版》

+ [滑动窗口详解](https://zhuanlan.zhihu.com/p/422908736)
  + https://leetcode.cn/circle/article/9gcJBk/

### chap1、整数

#### 基础知识

#### 题目

##### 1、整数除法

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

##### 6、排序数组中两个数字之和

### chap2、数组

#### 题目

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

10、和为 k 的子数组

11、0 和 1 个数相同的子数组

12、左右两边子数组的和相等

13、二维子矩阵的和

### chap3、字符串（14-20）

#### 基础知识

#### 题目

##### 14、字符串中的变位词

15、字符串中的所有变位词

16、不含重复字符的最长子字符串

17、含有所有字符的最短字符串

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



 20、回文子字符串的个数

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

#### 1、基础知识

#### 2、栈的应用

后缀表达式

 小行星碰撞

每日温度

直方图最大矩形面积

矩阵中最大的矩形

### chap7、队列

#### 1、基础知识

#### 2、队列的应用

滑动窗口的平均值

 最近请求次数

往完全二叉树添加节点

二叉树每层的最大值

 二叉树最底层最左边的值

二叉树的右侧视图

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

### chap10、前缀树

#### 1、基础知识

#### 2、前缀树的应用

 实现前缀树

替换单词

神奇的字典

最短的单词编码

单词之和

最大的异或

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

#### 1、基础知识

#### 2、集合的组合、排列

所有子集

含有 k 个元素的组合

允许重复选择元素的组合

 含有重复元素集合的组合

##### 87、复原IP

#### 使用回溯法解决其他类型的问题

### chap14、动态规划（88-104）

双序列问题

矩阵路径问题

背包问题

### chap15、图（105-119）

#### 1、基础知识

#### 2、图的搜索

拓扑排序

并查集



https://zxi.mytechroad.com/blog/leetcode-problem-categories/  题目列表