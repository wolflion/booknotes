## shell入门与进阶

+ 我的问题
  + 1、系统变量不熟
  + 2、循环，条件语句，用得少，语法不太熟悉

### 一、shell完整的示例

+ 演示了 Shell 的基本语法，包括变量、命令、用户输入、条件语句、循环语句和函数，*来自AI*

```shell
#!/bin/bash

# 定义变量
name="John Doe"
age=30

# 打印变量
echo $name
echo $age

# 执行命令
ls -l
pwd
date

# 获取用户输入
read -p "Enter your name: " input_name

# 打印用户输入
echo "Your name is: $input_name"

# 条件语句
if [ $age -gt 18 ]; then
  echo "You are an adult."
else
  echo "You are a child."
fi

# 循环语句
for i in {1..10}; do
  echo $i
done

# 函数
function my_function() {
  echo "This is a function."
}

# 调用函数
my_function

# 退出 Shell
exit
```



### 二、进阶的话，是不是怎么写函数？

+ 01-08（）
  + bash reference manual，GNU
  + posix shell tutorial
  + opengroup shell command language
  + `man bash`