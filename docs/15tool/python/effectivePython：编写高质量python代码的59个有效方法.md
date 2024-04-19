## 《Effective Python》

### chap1、用Pythoninc方式来思考

#### 7、用列表推导（list comprehension）来取代map和filter

+ 列表推导（根据一份列表来制作另一份），`a = [1,2,3,4,5,6,7,8,9,10]`

+ `alt = map(lambda x: x**2, filter(lambda x:x%2 == 0, a))`
+ 要点：
  + **列表推导，可以跳过输入列表中的某些元素**，如果改用map，就必须辅以filter才能实现

### chap2、函数（14-21）

### chap3、类与继承（22-28）

### chap6、内置模块（42-48）